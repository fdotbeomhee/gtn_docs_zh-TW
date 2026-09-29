# 執行與生命週期 (Execution and Lifecycle) {: #execution-and-lifecycle }

本頁面詳細介紹了節點可覆寫的各項生命週期回呼函式，以便深度參與工作流程的調度運算；同時闡述了針對耗時長的非同步任務（例如整合外部 API 並進行狀態輪詢）的標準設計模式。

## 生命週期回呼函式 (Lifecycle Callbacks) {: #lifecycle-callbacks }

以下所有生命週期函式皆可在自訂節點中按需覆寫：

- **allow_incoming_connection**, **allow_outgoing_connection**：回傳布林值以執行自訂的連線合法性校驗。
- **after_incoming_connection**, **after_outgoing_connection**：在連線成功建立後觸發相應的業務邏輯。
- **after_incoming_connection_removed**, **after_outgoing_connection_removed**：在連線中斷斷開時執行清理工作。
- **before_value_set**：在數值實際指派給參數前攔截並回傳修改後的值。
- **after_value_set**：對參數數值的變更做出即時響應（例如更新其他參數的 UI 可見度）。
- **validate_before_workflow_run**, **validate_before_node_run**：在流程或節點執行前執行全面防禦校驗，回傳 `list[Exception] | None`。
- **on_griptape_event**：監聽並處理工作流程運行過程中的廣播事件。
- **initialize_spotlight**：配置節點的聚光燈 (Spotlight) 互動功能。
- **get_next_control_output**：回傳 `Parameter | None`，以動態決定控制流的分支走向。

### 常用輔助函式 {: #helper-methods }

- `hide_parameter_by_name()`, `show_parameter_by_name()`：動態控制參數的隱藏與顯示。
- `append_value_to_parameter()`：向支援追加的參數附加新元素。
- `publish_update_to_parameter()`：向前端廣播發布參數更新通知。
- `show_message_by_name()`, `hide_message_by_name()`, `get_message_by_name_or_element_id()`：管控行內訊息提示列 (Message Banner) 的呈現。

## 非同步 API 整合架構 {: #asynchronous-api-integration }

節點有兩種標準途徑來執行耗時的長任務，而絕不卡死引擎主迴圈：

1. **覆寫 `async def aprocess()`（強烈推薦）**。引擎在其事件迴圈中直接 `await` 該方法，因此真正的非同步 I/O（非同步 HTTP 用戶端、`await asyncio.sleep()` 輪詢）能與引擎的其他運算高效並行。
1. **覆寫 `process()` 並 `yield` 可調用物件 (`AsyncResult`)**。每個 yielded 出的可調用物件都會在**背景執行緒中同步執行**——引擎依然保持響應，但該任務本身的程式碼依然是阻塞性的循序執行。

若您正在開發全新的服務整合，請一律使用 `aprocess()` 與非同步 I/O。只有當您整合的 SDK 是純同步程式庫（如傳統 `requests`）且不想重構時，才考慮 `yield` 模式。

### 採用 `aprocess()` 的非同步處理（首選最佳實踐） {: #async-processing-with-aprocess-preferred }

```python
import asyncio

import httpx
from griptape_nodes.exe_types.node_types import ControlNode

POLLING_INTERVAL = 10  # 輪詢間隔（秒，請依 API 官方推薦設定）
MAX_POLLING_ATTEMPTS = 60  # 最大嘗試次數（例如上限 10 分鐘）


class MyAsyncNode(ControlNode):
    async def aprocess(self) -> None:
        """非同步處理外部請求。"""
        try:
            # 初始化安全預設值
            self._set_safe_defaults()

            # 驗證 API 金鑰
            api_key = self._validate_api_key()

            async with httpx.AsyncClient(timeout=60) as client:
                # 提交非同步任務
                task_id = await self._submit_task(client, api_key)

                # 輪詢直至完成
                result = await self._poll_for_completion(client, task_id, api_key)

            # 封裝輸出結果
            self.parameter_output_values["output"] = result

        except Exception as e:
            self._set_safe_defaults()
            self._log(f"處理失敗: {e}")
            raise RuntimeError(f"{self.name}: {e}") from e

    async def _submit_task(self, client: httpx.AsyncClient, api_key: str) -> str:
        response = await client.post(
            "https://api.example.com/v1/tasks",
            json=self._build_payload(),
            headers={"Authorization": f"Bearer {api_key}"},
        )
        response.raise_for_status()
        return response.json()["task_id"]

    async def _poll_for_completion(self, client: httpx.AsyncClient, task_id: str, api_key: str) -> str:
        for attempt in range(MAX_POLLING_ATTEMPTS):
            await asyncio.sleep(POLLING_INTERVAL)  # 在 aprocess 中切勿使用 time.sleep()

            response = await client.get(
                "https://api.example.com/v1/query/task",
                params={"task_id": task_id},
                headers={"Authorization": f"Bearer {api_key}"},
            )
            response.raise_for_status()
            status_data = response.json()

            if status_data["status"] == "Success":
                return status_data["result"]
            if status_data["status"] == "Fail":
                error_msg = status_data.get("error_message", "未知錯誤")
                raise RuntimeError(f"任務失敗: {error_msg}")
            # 針對 "Processing"、"Pending" 等中間狀態繼續輪詢

        raise RuntimeError(f"任務在 {MAX_POLLING_ATTEMPTS * POLLING_INTERVAL} 秒內未完成")
```

**關鍵原則：**

- 覆寫 `async def aprocess()` 而非 `process()`——引擎會直接 await 調用。
- 全程使用非同步 I/O：使用 `httpx.AsyncClient` 發送網路請求，使用 `await asyncio.sleep()` 實現輪詢等待。
- 切勿在 `aprocess()` 內部執行阻塞操作（如 `requests`、`time.sleep()`）——這會凍結引擎事件迴圈。若必須呼叫阻塞函式，請用 `await asyncio.to_thread(blocking_fn)` 包裝。
- 基底類別的預設 `aprocess()` 會自動封裝 `process()`，因此兩者只需覆寫其一。

### 在背景執行緒中處理阻塞任務 (`process()` + `yield`) {: #blocking-work-on-a-background-thread-process-yield }

針對使用同步第三方庫的整合，可覆寫 `process()` 並 yield 可調用物件。引擎會在背景執行緒中同步運行該可調用物件，並以其回傳值喚醒生成器：

```python
from griptape_nodes.exe_types.node_types import ControlNode, AsyncResult


class MyBlockingNode(ControlNode):
    def process(self) -> AsyncResult | None:
        """將阻塞任務 yield 至背景執行緒執行。"""
        yield lambda: self._process()

    def _process(self) -> None:
        """核心處理方法（在背景執行緒中同步運作）。"""
        try:
            self._set_safe_defaults()
            api_key = self._validate_api_key()
            task_id = self._submit_task(api_key)
            result = self._poll_for_completion(task_id, api_key)
            self.parameter_output_values["output"] = result
        except Exception as e:
            self._set_safe_defaults()
            self._log(f"處理失敗: {e}")
            raise RuntimeError(f"{self.name}: {str(e)}") from e
```

### 耗時任務的狀態輪詢架構 (Polling Pattern) {: #polling-pattern-for-long-running-tasks }

整合非同步任務型 API（例如影片生成、大型模型微調）時，請實作經典的三階段架構：

#### 步驟 1：任務提交 (Task Submission) {: #step-1-task-submission }

```python
def _submit_task(self, params: dict[str, Any], headers: dict[str, str]) -> dict[str, Any]:
    """提交任務並回傳包含 task_id 的回應字典。"""
    payload = self._build_payload(params)
    response = requests.post(self.API_BASE_URL, json=payload, headers=headers, timeout=DEFAULT_TIMEOUT)
    response.raise_for_status()
    return response.json()
```

#### 步驟 2：狀態輪詢 (Status Polling) {: #step-2-status-polling }

```python
def _poll_for_completion(self, task_id: str, headers: dict[str, str]) -> str | None:
    """輪詢 API 直至任務完成，回傳結果檔案標識碼。"""
    query_url = "https://api.example.com/v1/query/task"

    for attempt in range(MAX_POLLING_ATTEMPTS):
        time.sleep(POLLING_INTERVAL)

        response = requests.get(
            query_url,
            headers=headers,
            params={"task_id": task_id},  # 依據文檔使用查詢參數
            timeout=DEFAULT_TIMEOUT,
        )
        response.raise_for_status()

        status_data = response.json()
        status = status_data.get("status")

        if status == "Success":
            return status_data.get("file_id")
        elif status == "Fail":
            error_msg = status_data.get("error_message", "未知錯誤")
            raise RuntimeError(f"任務失敗: {error_msg}")

    raise RuntimeError(f"任務在逾時限額內未完成")
```

#### 步驟 3：結果提取 (Result Retrieval) {: #step-3-result-retrieval }

```python
def _retrieve_result(self, file_id: str, headers: dict[str, str]) -> str:
    """根據結果標識碼獲取最終下載 URL。"""
    retrieve_url = "https://api.example.com/v1/files/retrieve"
    response = requests.get(retrieve_url, headers=headers, params={"file_id": file_id}, timeout=DEFAULT_TIMEOUT)
    response.raise_for_status()
    return response.json().get("file", {}).get("download_url")
```

### 依據輸入端動態選取 API 端點 {: #dynamic-endpoint-selection-based-on-inputs }

當節點依據已連線的輸入端具備多種運行模式時（例如有影像連入時走「圖生影片」，無影像時走「文生影片」），請在運算方法中動態判定目標 URL，而非寫死單一端點：

```python
IMAGE2VIDEO_URL = "https://api.example.com/v1/videos/image2video"
TEXT2VIDEO_URL = "https://api.example.com/v1/videos/text2video"


def _process(self):
    image_data = self._get_image_data("start_frame")
    api_url = IMAGE2VIDEO_URL if image_data is not None else TEXT2VIDEO_URL

    payload = self._build_payload()
    if image_data:
        payload["image"] = image_data

    response = requests.post(api_url, headers=headers, json=payload, timeout=30)
```

### 本機 Localhost 網址轉換為 Base64 資料 {: #image-artifact-conversion-to-base64 }

**關鍵注意：處理本機 Localhost URL**

向外部公有雲 API 發送影像時，來自 Griptape 本機靜態儲存庫的 `ImageUrlArtifact` 為 `localhost` 網址，外部雲端服務絕對無法存取。請務必在節點中自動識別並將本機 URL 轉換為 Base64 Data URI：

```python
import base64


def _get_image_data(self, image_artifact: ImageArtifact | ImageUrlArtifact) -> str:
    """將影像產物轉換為公有 URL 或 Base64 Data URI。"""
    if isinstance(image_artifact, ImageUrlArtifact):
        url = image_artifact.value

        # 本機 Localhost URL 必須轉為 Base64 才能供外部 API 讀取
        if url.startswith(("http://localhost", "http://127.0.0.1", "https://localhost", "https://127.0.0.1")):
            self._log(f"正在將 Localhost 網址轉換為 Base64: {url[:100]}...")
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            image_bytes = response.content

            mime_type = response.headers.get("content-type", "image/jpeg")
            if not mime_type.startswith("image/"):
                mime_type = "image/jpeg"

            base64_data = base64.b64encode(image_bytes).decode("utf-8")
            return f"data:{mime_type};base64,{base64_data}"

        # 公有雲 URL 直接透傳
        return url

    if isinstance(image_artifact, ImageArtifact):
        if hasattr(image_artifact, "base64") and hasattr(image_artifact, "mime_type"):
            base64_data = image_artifact.base64
            mime_type = image_artifact.mime_type
            if base64_data.startswith("data:"):
                return base64_data
            return f"data:{mime_type};base64,{base64_data}"

    raise ValueError("不支援的影像產物格式")
```

### 已淘汰模型的平滑遷移與使用者提示 {: #deprecated-model-migration-and-user-notification }

當外部模型提供商淘汰舊端點時（例如 Preview 預覽模型被正式 GA 版本取代），節點應自動協助遷移舊工作流程，同時向使用者顯示提示條：

```python
from griptape_nodes.exe_types.core_types import Parameter, ParameterMessage
from griptape_nodes.traits.button import Button

DEPRECATED_MODELS: dict[str, str] = {
    "veo-3.1-generate-preview": "veo-3.1-generate-001",
    "veo-3.0-generate-001": "veo-3.1-generate-001",
}


def before_value_set(self, parameter: Parameter, value: Any) -> Any:
    """自動遷移已淘汰的模型並顯示通知橫幅。"""
    if parameter.name == "model" and value in DEPRECATED_MODELS:
        replacement = DEPRECATED_MODELS[value]
        message = self.get_message_by_name_or_element_id("model_deprecation_notice")
        if message is not None:
            message.value = (
                f"模型 '{value}' 已被官方淘汰。"
                f"系統已為您自動升級為 '{replacement}'。"
                "請儲存工作流程以固化此項變更。"
            )
            self.show_message_by_name("model_deprecation_notice")
        value = replacement

    return super().before_value_set(parameter, value)
```
