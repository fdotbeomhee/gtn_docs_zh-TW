# 設計模式與實用範例 (Patterns and Examples) {: #patterns-and-examples }

進階架構模式與開發參考資料合集：涵蓋外部 API 整合策略、生產級節點的 UI/UX 設計典範、高彈性的產物處理技巧，以及依賴匯入、工具函式與核心型別的速查表格。

## 進階技術主題 {: #advanced-topics }

### REST API vs SDK 整合取捨 {: #rest-api-vs-sdk-integration }

**面臨挑戰**：Python 官方 SDK 在支援新功能時往往落後於底層 REST API。REST API 開發者文件中已公佈的關鍵參數，在 SDK 程式庫中可能尚未暴露。

**何時應直接調用 REST API**：

- SDK 缺失已在官方文件公佈的 API 核心特性（例如 Google Gemini 的 `image_config` 參數）
- 需要第一時間存取雲端服務新發布的 API 參數
- 現有 SDK 存在已知缺陷或環境限制
- 希望保持更輕量的相依套件依賴

**REST API 呼叫實作範式**：

```python
import base64
import requests
from google.oauth2 import service_account
from google.auth.transport.requests import Request

# 身分驗證處理
credentials = service_account.Credentials.from_service_account_file(
    service_account_file, scopes=["https://www.googleapis.com/auth/cloud-platform"]
)


def _get_access_token(self, credentials) -> str:
    """從憑證獲取最新的存取權杖。"""
    if not credentials.valid:
        credentials.refresh(Request())
    return credentials.token


# 構建完全匹配 REST API 規格的 JSON 承載資料
payload = {
    "contents": {
        "role": "USER",
        "parts": [
            {"text": prompt},
            {"inline_data": {"mime_type": "image/jpeg", "data": base64.b64encode(image_bytes).decode("utf-8")}},
        ],
    },
    "generation_config": {
        "temperature": 1.0,
        "topP": 0.95,
        "candidateCount": 1,
        "response_modalities": ["TEXT", "IMAGE"],
        "image_config": {  # SDK 尚未支援的新功能！
            "aspect_ratio": "16:9"
        },
    },
}

# 發送帶有身分驗證標頭的請求
access_token = self._get_access_token(credentials)
headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}

api_endpoint = f"https://{location}-aiplatform.googleapis.com/v1/projects/{project_id}/locations/{location}/publishers/google/models/{model}:generateContent"

response = requests.post(api_endpoint, headers=headers, json=payload, timeout=120)
response.raise_for_status()
response_data = response.json()

# 解析 JSON 回應（相容處理 camelCase 與 snake_case）
candidates = response_data.get("candidates", [])
for cand in candidates:
    parts_list = cand.get("content", {}).get("parts", [])
    for part in parts_list:
        if "inlineData" in part or "inline_data" in part:
            inline_data = part.get("inlineData") or part.get("inline_data", {})
            mime = inline_data.get("mimeType") or inline_data.get("mime_type")
            data_b64 = inline_data.get("data", "")
            data_bytes = base64.b64decode(data_b64)
```

**關鍵注意事項**：

1. **依賴精簡**：僅引入 `google-auth`，取代龐大的完整 SDK (`google-cloud-aiplatform`, `google-genai`)
1. **區域可用性 (Regional Availability)**：部分特定模型僅於專屬區域運作（如 `us-central1`），不支援 `global`
1. **模型命名規範**：留意預覽版模型與正式 GA 版本模型在 `-preview` 後綴上的差異
1. **授權範圍 (Scopes)**：Vertex AI 調用請統一使用 `https://www.googleapis.com/auth/cloud-platform`
1. **回應欄位相容**：同時相容解析駝峰式 (camelCase) 與蛇形命名 (snake_case)
1. **Base64 編碼**：REST API 傳輸二進位二進位資料一律要求 Base64 字串
1. **錯誤處理**：解析 JSON 回應中的錯誤物件以獲取詳細的遠端報錯資訊

**架構權衡評估**：

- ✅ 即時掌握官方 API 的全部最新特性
- ✅ 極致輕量的外部依賴體積
- ✅ 對底層 HTTP 傳輸具備百分之百的掌控權
- ❌ 需額外編寫更多的底層解析程式碼
- ❌ 必須手動維護 Token 更新與授權流程
- ❌ 需自行主動追蹤上游 API 的版本變更

### 複雜型別動態協商系統 {: #complex-type-management-systems }

適用於需要在多個連接埠之間動態協商資料型別的進階節點（如條件分支節點）：

```python
class IfElse(BaseNode):
    def __init__(self, name: str, metadata: dict[Any, Any] | None = None) -> None:
        super().__init__(name, metadata)

        # 用於動態型別協商的狀態追蹤器
        self._possibility_space: list[str] = []  # 輸出目標端可接受的型別集合
        self._locked_type: str | None = None  # 由當前輸入連線鎖定的具體型別
        self._connected_inputs: set[str] = set()  # 追蹤已連線的輸入端
        self._output_connected: bool = False  # 追蹤輸出端連線狀態

    def _update_parameter_types(self) -> None:
        """依據當前拓撲狀態全面更新各參數的合法型別。"""
        if self._locked_type:
            # 已鎖定至具體型別——強制所有分支採用該型別
            self.output_if_true.input_types = [self._locked_type]
            self.output_if_false.input_types = [self._locked_type]
            self.output.output_type = self._locked_type
        elif self._possibility_space:
            # 在協商後的可能性空間內保持動態彈性
            self.output_if_true.input_types = self._possibility_space.copy()
            self.output_if_false.input_types = self._possibility_space.copy()
            self.output.output_type = ParameterTypeBuiltin.ALL.value
        else:
            # 預設狀態——相容接受任意型別
            self.output_if_true.input_types = ["any"]
            self.output_if_false.input_types = ["any"]
            self.output.output_type = ParameterTypeBuiltin.ALL.value
```

**最佳實踐**：在多輸入、多輸出的資料路由節點中，採用動態型別協商機制以兼顧拓撲自由度與型別安全性。

### 代理節點架構 (Agentic Nodes) {: #agentic-nodes }

繼承 `ControlNode` 實現 Griptape Agent 執行個體狀態的管線式傳遞：

```python
from griptape.structures import Agent
from griptape_nodes.exe_types.node_types import ControlNode


class MyAgentNode(ControlNode):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.add_parameter(Parameter(name="agent_in", input_types=["Agent"], type="Agent"))
        self.add_parameter(Parameter(name="agent_out", output_type="Agent"))

    def process(self) -> None:
        agent_state = self.get_parameter_value("agent_in")
        agent = Agent.from_dict(agent_state) if agent_state else Agent()
        # 執行 Agent 運算與工具調用...
        self.parameter_output_values["agent_out"] = agent.to_dict()
```

### 節點家族的抽象基底類別 (Abstract Base Classes) {: #abstract-base-classes-for-node-families }

針對具備相似行為特徵的一組相關節點，建立抽象基底類別以複用通用邏輯：

```python
from abc import abstractmethod
from typing import Any

from griptape_nodes.exe_types.base_iterative_nodes import BaseIterativeStartNode


class BaseCustomIterativeStartNode(BaseIterativeStartNode):
    """自訂反覆運算起始節點家族的通用抽象基底類別。"""

    @abstractmethod
    def _get_compatible_end_classes(self) -> set[type]:
        """回傳此起始節點允許連線的合法結束節點 (End Node) 類別集合。"""

    @abstractmethod
    def _get_parameter_group_name(self) -> str:
        """回傳承載反覆運算資料的參數分組名稱。"""

    @abstractmethod
    def _get_exec_out_display_name(self) -> str:
        """回傳控制流輸出端 (exec_out) 的 UI 顯示名稱。"""

    @abstractmethod
    def _get_exec_out_tooltip(self) -> str:
        """回傳控制流輸出端的提示資訊 (Tooltip)。"""

    @abstractmethod
    def _get_iteration_items(self) -> list[Any]:
        """獲取待遍歷處理的項目清單。"""

    @abstractmethod
    def is_loop_finished(self) -> bool:
        """判定迴圈是否已完成所有反覆運算。"""
```

**最佳實踐**：透過抽象基底類別在整個節點家族間共享架構邏輯，同時嚴格規範子類別必須實作的關鍵方法。

### 記憶體快取模式 (Caching) {: #caching }

使用 `ClassVar` 類別層級變數實現模型跨運算的共享快取：

```python
from typing import ClassVar, Any


class CachedModelNode(DataNode):
    _cache: ClassVar[dict[str, Any]] = {}

    def get_model(self, model_id: str) -> Any:
        if model_id not in self._cache:
            self._cache[model_id] = load_model(model_id)
        return self._cache[model_id]
```

### 模型中樞整合 (以 HuggingFace 為例) {: #hub-integration-eg-huggingface }

```python
# 偵測是否為受門禁管控的模型 (Gated Model)
is_gated = getattr(model, "gated", False)
model_dict["gated"] = is_gated

# 針對門禁模型即時更新狀態列警示
if getattr(model_info, "gated", False):
    self.publish_update_to_parameter("status", "🔒 門禁受限模型 - 可能需要上游授權審核")
```

### 使用 ParameterMessage 呈現外部連結與動態狀態更新 {: #parametermessage-for-external-links-and-status-updates }

```python
from griptape_nodes.exe_types.core_types import ParameterMessage

# 外部說明文件連結範例
ParameterMessage(
    name="model_card_link",
    title="模型卡 (Model Card)",
    variant="info",
    value="檢視模型官方架構與授權說明",
    button_link=f"https://huggingface.co/{model_id}",
    button_text="在 HuggingFace 上開啟",
)


# 動態即時進度狀態列範例
class MyIterativeNode(BaseIterativeStartNode):
    def __init__(self, name: str, metadata: dict[Any, Any] | None = None) -> None:
        super().__init__(name, metadata)

        # 用於即時反映狀態的訊息列參數
        self.status_message = ParameterMessage(
            name="status_message",
            variant="info",
            value="",
        )
        self.add_node_element(self.status_message)

    def _update_status_message(self, status_type: str = "normal") -> None:
        """依據當前運算進度更新提示訊息。"""
        if self._total_iterations == 0:
            status = "已完成 0 (共 0 項)"
        elif status_type == "break":
            status = f"於第 {self._current_iteration_count} 項中止 (共 {self._total_iterations} 項) - 中斷"
        elif self.is_loop_finished():
            status = f"已全數完成 {self._total_iterations} (共 {self._total_iterations} 項)"
        else:
            status = f"正在處理第 {self._current_iteration_count} 項 (共 {self._total_iterations} 項)"

        self.status_message.value = status
```

**最佳實踐**：善用 `ParameterMessage` 展示靜態外部超連結、動態進度回饋與模型淘汰警示橫幅。淘汰提示與自動平滑遷移的完整模式詳見 [已淘汰模型的平滑遷移與使用者提示](execution_and_lifecycle.md#deprecated-model-migration-and-user-notification)。

## 現代 UI/UX 設計典範 {: #modern-ui-ux-patterns }

### UI Options 樣式微調選項 {: #ui-options }

官方支援的 `ui_options` 鍵值、各型別所渲染的預設部件，以及管轄它們的特徵標籤皆已權威收錄於 [參數 UI 參考指南](parameter_ui_reference.md)。未記載於該頁面的設定鍵屬編輯器內部實作細節，未來可能在不另行通知的情況下調整。

#### 隱藏參數的最佳實踐 {: #hidden-parameters-best-practice }

宣告 `"hide": True` 可在 UI 畫布上隱藏高階/進階參數：

```python
num_images_param = Parameter(
    name="num_images",
    input_types=["int"],
    type="int",
    default_value=1,
    tooltip="欲生成的影像張數 (1-9)",
    allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
    ui_options={
        "display_name": "Number of Images",
        "hide": True,
    },
)
num_images_param.add_trait(Slider(min_val=1, max_val=9))
self.add_parameter(num_images_param)
```

少數舊節點中存在歷史遺留的 `"hidden": True` 鍵；新程式碼請一律採用標準的 `"hide": True`。

**隱藏參數的常見使用情境：**

- 專家級進階配置項
- 內部流程控制訊號
- 除錯與診斷開關
- 可選的實驗性擴充屬性
- 僅供外部腳本程式化指派的參數

### 成功/失敗節點架構 (SuccessFailureNode) {: #success-failure-node-pattern }

針對容易因外部因素發生中斷的 I/O 運算，繼承 `SuccessFailureNode` 以標準化報告狀態：

```python
from griptape_nodes.exe_types.node_types import SuccessFailureNode


class LoadImage(SuccessFailureNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        # 透過內建輔助方法註冊狀態參數
        self._create_status_parameters(
            result_details_tooltip="影像載入運算結果的詳細回報",
            result_details_placeholder="載入嘗試的詳細資訊將呈現於此。",
        )

    def process(self) -> None:
        # 開始運算前重設執行狀態
        self._clear_execution_status()

        # 清空輸出端以防殘留上一輪的陳舊快取資料
        self.parameter_output_values["image"] = None

        try:
            # 核心運算業務邏輯
            result = load_image()
            self.parameter_output_values["image"] = result

            # 成功狀態回報
            success_details = f"已成功自 {source} 載入影像"
            self._set_status_results(was_successful=True, result_details=f"SUCCESS: {success_details}")

        except Exception as e:
            error_details = f"載入影像失敗: {e}"
            self._set_status_results(was_successful=False, result_details=f"FAILURE: {error_details}")
            self._handle_failure_exception(e)
```

**最佳實踐**：凡涉及網路請求、外部檔案存取等具備失敗可能的操作，一律採用 `SuccessFailureNode` 以向創作者清晰報告錯誤詳情。

### 參數視覺可見度初始化 {: #parameter-initialization }

在節點建構時依據預設值初始化參數的可見性：

```python
def _initialize_parameter_visibility(self) -> None:
    """依據預設參數值初始化 UI 可見性狀態。"""
    default_model = self.get_parameter_value("model") or "default"
    if default_model == "text-only":
        self.hide_parameter_by_name("image_input")
    else:
        self.show_parameter_by_name("image_input")
```

### 產物與路徑連動綁定架構 (Artifact Path Tethering) {: #artifact-path-tethering-pattern }

在處理實體檔案的節點中，使用綁定模式維持路徑字串與產物物件之間的即時同步：

```python
from griptape_nodes_library.utils.artifact_path_tethering import (
    ArtifactPathTethering,
    ArtifactTetheringConfig,
)


class LoadImage(SuccessFailureNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        # 配置連動轉換規則
        self._tethering_config = ArtifactTetheringConfig(
            dict_to_artifact_func=dict_to_image_url_artifact,
            extract_url_func=self._extract_url_from_image_value,
            supported_extensions=self.SUPPORTED_EXTENSIONS,
            default_extension="png",
            url_content_type_prefix="image/",
        )

        # 建立產物參數
        self.image_parameter = Parameter(
            name="image",
            input_types=["ImageUrlArtifact", "ImageArtifact", "str"],
            type="ImageUrlArtifact",
            output_type="ImageUrlArtifact",
            ui_options={"clickable_file_browser": True},
        )

        # 使用連動工具建立路徑參數
        self.path_parameter = ArtifactPathTethering.create_path_parameter(
            name="path",
            config=self._tethering_config,
            display_name="File Path or URL",
        )

        # 註冊連動管理器以保持雙向同步
        self._tethering = ArtifactPathTethering(
            node=self,
            artifact_parameter=self.image_parameter,
            path_parameter=self.path_parameter,
            config=self._tethering_config,
        )

    def after_value_set(self, parameter: Parameter, value: Any) -> None:
        # 委派連動同步邏輯
        self._tethering.on_after_value_set(parameter, value)
        return super().after_value_set(parameter, value)
```

## 雙模式 UI 設計架構 (簡易模式 + 自訂模式) {: #two-mode-ui-pattern-simple-custom }

**適用情境**：既希望為新手提供零門檻的極簡上手體驗，又想為進階專業使用者提供精細的控制參數。

**經典實例**：音樂與影片生成 API 通常具備「自然語言簡易描述」模式與「專業分軌細部控制」模式。

### 實作架構範例 {: #implementation-pattern }

```python
class GenerativeNode(DataNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        # 模式切換開關
        mode_param = Parameter(
            name="custom_mode",
            input_types=["bool"],
            type="bool",
            default_value=False,
            tooltip="自訂模式：完全手動精細控制。簡易模式：由提示詞全自動生成。",
            allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
            ui_options={"display_name": "Custom Mode"},
        )
        self.add_parameter(mode_param)

        # 核心提示詞（語意隨模式切換而動態變化）
        prompt_param = Parameter(
            name="prompt",
            input_types=["str"],
            type="str",
            default_value="",
            tooltip=[
                {"type": "text", "text": "自訂模式：精確歌詞或詳細分鏡腳本"},
                {"type": "text", "text": "簡易模式：宏觀風格與意境描述"},
            ],
            allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
            ui_options={"multiline": True, "display_name": "Prompt"},
        )
        self.add_parameter(prompt_param)

        # 進階參數（僅在自訂模式下顯現）
        style_param = Parameter(
            name="style",
            input_types=["str"],
            type="str",
            default_value="",
            tooltip="音樂曲風/視覺風格（僅限自訂模式）",
            allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
            ui_options={"hide": True},  # 預設折疊隱藏
        )
        self.add_parameter(style_param)

        title_param = Parameter(
            name="title",
            input_types=["str"],
            type="str",
            default_value="",
            tooltip="作品標題（僅限自訂模式）",
            allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
            ui_options={"hide": True},  # 預設折疊隱藏
        )
        self.add_parameter(title_param)

        # 初始化視覺呈現狀態
        self._initialize_parameter_visibility()

    def _initialize_parameter_visibility(self) -> None:
        """依據當前選取模式初始化各參數的隱藏/顯示狀態。"""
        custom_mode = self.get_parameter_value("custom_mode") or False
        if custom_mode:
            self.show_parameter_by_name("style")
            self.show_parameter_by_name("title")
        else:
            self.hide_parameter_by_name("style")
            self.hide_parameter_by_name("title")

    def after_value_set(self, parameter: Parameter, value: Any) -> None:
        """動態響應模式切換以更新 UI 佈局。"""
        if parameter.name == "custom_mode":
            if value:
                self.show_parameter_by_name("style")
                self.show_parameter_by_name("title")
            else:
                self.hide_parameter_by_name("style")
                self.hide_parameter_by_name("title")

        return super().after_value_set(parameter, value)

    def validate_before_node_run(self) -> list[Exception] | None:
        """依據所處模式執行差異化的邊界條件驗證。"""
        exceptions = []
        custom_mode = self.get_parameter_value("custom_mode")

        if custom_mode:
            # 自訂模式下風格與標題為必填項
            style = self.get_parameter_value("style") or ""
            title = self.get_parameter_value("title") or ""

            if not style.strip():
                exceptions.append(ValueError(f"{self.name}: 自訂模式下必須指定 Style 風格"))
            if not title.strip():
                exceptions.append(ValueError(f"{self.name}: 自訂模式下必須填寫 Title 標題"))
        else:
            # 簡易模式下僅需提示詞
            prompt = self.get_parameter_value("prompt") or ""
            if not prompt.strip():
                exceptions.append(ValueError(f"{self.name}: 簡易模式下提示詞不能為空"))

        return exceptions if exceptions else None
```

## 音樂與音訊生成 API 整合模式 {: #music-audio-generation-api-patterns }

### 依模型劃分的字元配額限制 {: #character-limits-by-model }

許多生成式模型具備專屬的字元上限。建議將限制宣告為類別常數：

```python
class MusicGenerationNode(DataNode):
    # 各模型在自訂模式下的提示詞長度上限
    PROMPT_LIMITS_CUSTOM = {
        "V3_5": 3000,
        "V4": 3000,
        "V4_5": 5000,
        "V5": 5000,
    }
    PROMPT_LIMIT_SIMPLE = 500

    # 各模型的風格字元上限
    STYLE_LIMITS = {
        "V3_5": 200,
        "V4": 200,
        "V4_5": 1000,
        "V5": 1000,
    }

    TITLE_LIMIT = 80

    def validate_before_node_run(self) -> list[Exception] | None:
        """依據選定模型查驗字元配額。"""
        exceptions = []
        model = self.get_parameter_value("model")
        custom_mode = self.get_parameter_value("custom_mode")

        if custom_mode:
            prompt = self.get_parameter_value("prompt") or ""
            prompt_limit = self.PROMPT_LIMITS_CUSTOM.get(model, 3000)
            if len(prompt) > prompt_limit:
                exceptions.append(
                    ValueError(
                        f"{self.name}: 提示詞超過 {model} 允許的 {prompt_limit} 字元上限 "
                        f"(目前長度: {len(prompt)} 字元)"
                    )
                )

            style = self.get_parameter_value("style") or ""
            style_limit = self.STYLE_LIMITS.get(model, 200)
            if len(style) > style_limit:
                exceptions.append(
                    ValueError(
                        f"{self.name}: 風格描述超過 {model} 允許的 {style_limit} 字元上限 "
                        f"(目前長度: {len(style)} 字元)"
                    )
                )

        return exceptions if exceptions else None
```

### 多變體雙音軌輸出模式 (Dual Track Output) {: #dual-track-output-pattern }

適用於一次請求同時生成多個變體曲目的 API：

```python
# 輸出多個音訊網址清單
music_urls_param = Parameter(
    name="music_urls",
    output_type="list[str]",
    type="list[str]",
    tooltip="生成的音軌下載網址清單（包含 2 個不同變體）",
    allowed_modes={ParameterMode.OUTPUT},
    settable=False,
    ui_options={"is_full_width": True, "display_name": "Music URLs"},
)
self.add_parameter(music_urls_param)


def process(self) -> None:
    # ... 調用生成 API ...
    urls = self._extract_music_urls(response_data)
    self.parameter_output_values["music_urls"] = urls

    # 組裝結構化回報文本
    result_lines = [
        f"✓ 已成功生成 {len(urls)} 首曲目變體",
        "",
        "下載網址清單:",
    ]
    for i, url in enumerate(urls, 1):
        result_lines.append(f"{i}. {url}")

    self.parameter_output_values["result_details"] = "\n".join(result_lines)
```

## 靈活的產物處理模式 (Flexible Artifact Processing) {: #flexible-artifact-processing }

下方列出的模式旨在相容處理可能自舊版工作流程或上游節點傳入的 `ImageArtifact`——屬於執行階段的輸入容錯處理，而非建議宣告全新參數的範本。宣告全新參數時請一律使用 `ImageUrlArtifact`（參見 [參數資料承載體積](error_handling.md#parameter-payload-size)）。

### 產物物件的鴨子型別判定 (Duck Typing) {: #duck-typing-for-artifacts }

優雅相容多種產物資料結構：

```python
def _extract_image_value(self, image_input: Any) -> str | None:
    """自各類影像輸入物件中提取字串路徑或 Data URI。"""
    if isinstance(image_input, str):
        return image_input

    try:
        # ImageUrlArtifact: .value 存放 URL 字串
        if hasattr(image_input, "value"):
            value = getattr(image_input, "value", None)
            if isinstance(value, str):
                return value

        # ImageArtifact: .base64 存放原始位元組或 Data URI
        if hasattr(image_input, "base64"):
            b64 = getattr(image_input, "base64", None)
            if isinstance(b64, str) and b64:
                return b64
    except Exception as e:
        self._log(f"提取影像數值失敗: {e}")

    return None
```

### 外部 API 影像格式動態轉碼 {: #image-format-conversion-for-external-apis }

**面臨挑戰**：外部公有雲 API 通常具備嚴格的格式約束（通常僅接受 JPEG、PNG、WebP），但相機拍攝或使用者匯入的圖檔可能為 MPO (多圖物件 3D 連拍格式) 或 TIFF 等特殊格式。

**解決方案**：自動識別不支援的格式並在記憶體中即時轉碼為標準 JPEG：

```python
from io import BytesIO
from PIL import Image


def _get_image_data(self, image_artifact: ImageArtifact | ImageUrlArtifact) -> str:
    """將影像轉換為符合公有 API 規格的相容格式。"""
    # ... 獲取 image_bytes 位元組資料 ...

    try:
        img = Image.open(BytesIO(image_bytes))

        # 將不支援的格式 (MPO, TIFF, BMP 等) 轉碼為 JPEG
        if img.format not in ["JPEG", "PNG", "WEBP"]:
            self._log(f"正在將 {img.format} 格式轉碼為 JPEG 以符合 API 相容性")
            # 轉換為標準 RGB 色彩空間（MPO 常使用非 RGB 模式）
            if img.mode not in ["RGB", "L"]:
                img = img.convert("RGB")
            # 儲存為記憶體 JPEG 位元組
            output = BytesIO()
            img.save(output, format="JPEG", quality=95)
            image_bytes = output.getvalue()
            mime_type = "image/jpeg"
        else:
            format_to_mime = {"JPEG": "image/jpeg", "PNG": "image/png", "WEBP": "image/webp"}
            mime_type = format_to_mime.get(img.format, "image/jpeg")
    except Exception as e:
        self._log(f"無法自動偵測影像格式: {e}")
        mime_type = "image/jpeg"

    # 編碼為 Base64 Data URI
    base64_data = base64.b64encode(image_bytes).decode("utf-8")
    return f"data:{mime_type};base64,{base64_data}"
```

### 拓撲連線查詢工具常式 {: #utility-function-patterns }

透過請求機制而非直接呼叫管理器來安全查驗連線狀態（確保在 Worker 隔離環境中亦能正確解析）：

```python
def _connections_for(node_name: str) -> ListConnectionsForNodeResultSuccess | None:
    from griptape_nodes.retained_mode.events.connection_events import (
        ListConnectionsForNodeRequest,
        ListConnectionsForNodeResultSuccess,
    )
    from griptape_nodes.retained_mode.griptape_nodes import GriptapeNodes

    result = GriptapeNodes.handle_request(ListConnectionsForNodeRequest(node_name=node_name))
    if isinstance(result, ListConnectionsForNodeResultSuccess):
        return result
    return None


def _outgoing_connection_exists(source_node: str, source_param: str) -> bool:
    """查驗指定來源節點/參數是否已建立對外連線。"""
    connections = _connections_for(source_node)
    if connections is None:
        return False

    return any(c.source_parameter_name == source_param for c in connections.outgoing_connections)


def _incoming_connection_exists(target_node: str, target_param: str) -> bool:
    """查驗指定目標節點/參數是否已接收對內連線。"""
    connections = _connections_for(target_node)
    if connections is None:
        return False

    return any(c.target_parameter_name == target_param for c in connections.incoming_connections)
```

## 附錄與速查表 (Appendix) {: #appendix }

### 常用匯入速查 (Imports) {: #imports }

```python
# 核心參數與型別匯入
from griptape_nodes.exe_types.core_types import (
    Parameter,
    ParameterList,
    ParameterMode,
    ParameterTypeBuiltin,
    ParameterGroup,
    ParameterMessage,
    ControlParameterInput,
    ControlParameterOutput,
)
from griptape_nodes.exe_types.node_types import DataNode, ControlNode, BaseNode, SuccessFailureNode, StartNode, EndNode
from griptape_nodes.exe_types.base_iterative_nodes import BaseIterativeStartNode, BaseIterativeEndNode
from griptape_nodes.traits.options import Options
from griptape_nodes.traits.slider import Slider
from griptape_nodes.traits.color_picker import ColorPicker
from griptape_nodes.traits.file_system_picker import FileSystemPicker

# 核心產物型別
from griptape.artifacts import ImageArtifact, ImageUrlArtifact, TextArtifact

# 公用工具模組
from griptape_nodes_library.utils.artifact_path_tethering import ArtifactPathTethering, ArtifactTetheringConfig
from griptape_nodes_library.utils.image_utils import (
    dict_to_image_url_artifact,
    load_pil_from_url,
    save_pil_image_with_named_filename,
)
from griptape_nodes_library.utils.file_utils import generate_filename
```

### 影像公用函式速查 (`image_utils`) {: #utility-function-reference }

| 函式名稱 | 用途說明 | 回傳型別 |
| :--- | :--- | :--- |
| `dict_to_image_url_artifact(d)` | 將字典反序列化為 ImageUrlArtifact | `ImageUrlArtifact` |
| `load_pil_from_url(url)` | 自 URL 載入為 PIL Image（自動相容處理 localhost 網址） | `PIL.Image.Image` |
| `save_pil_image_with_named_filename(img, filename)` | 透過專案系統保存 PIL 影像 | `ImageUrlArtifact` |

### 核心列舉型別速查 (Enumerations) {: #enumerations }

- **NodeResolutionState**（解析狀態）：`UNRESOLVED`、`RESOLVING`、`RESOLVED`
- **ParameterMode**（參數模式）：`INPUT`、`OUTPUT`、`PROPERTY`
- **ParameterTypeBuiltin**（內建型別）：`STR("str")`、`BOOL("bool")`、`INT("int")`、`FLOAT("float")`、`ANY("any")`、`NONE("none")`、`CONTROL_TYPE("parametercontroltype")`、`ALL("all")`
