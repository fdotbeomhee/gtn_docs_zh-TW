# 最佳實踐與錯誤處理 (Best Practices and Error Handling) {: #best-practices-and-error-handling }

構建生產級高品質自訂節點的通用最佳實踐指南：涵蓋機密管理、依賴匯入、程式碼規範、錯誤處置、邊界校驗與除錯記錄。

## 開發最佳實踐 {: #best-practices }

### 核心設計原則 {: #core-principles }

- **具備充分說明的名稱與懸浮提示 (Tooltips)**
- **透過驗證器 (Validators) 構建強健的防禦性錯誤處理**
- **單一職責原則 (Single Responsibility per Node)**
- **透過請求機制安全存取 API 金鑰與敏感憑據**
- **所有模組依賴一律在檔案頂層 (Module Level) 完成匯入**
- **冪等性的運算方法 (`process()` 設計)**

### 機密憑據管理 (Secrets Management) {: #secrets-management }

使用 `GetSecretValueRequest` 讀取敏感金鑰：

```python
from griptape_nodes.retained_mode.events.secrets_events import (
    GetSecretValueRequest,
    GetSecretValueResultSuccess,
)
from griptape_nodes.retained_mode.griptape_nodes import GriptapeNodes


class MyNode(DataNode):
    SERVICE_NAME = "MyService"
    API_KEY_NAME = "MY_SERVICE_API_KEY"

    def _validate_api_key(self) -> str:
        result = GriptapeNodes.handle_request(GetSecretValueRequest(key=self.API_KEY_NAME))
        if not isinstance(result, GetSecretValueResultSuccess) or not result.value:
            raise ValueError(f"缺少必要金鑰: {self.API_KEY_NAME}")
        return result.value
```

**關鍵原則：**

- 在模組最頂部完成所有匯入，嚴禁在函式內部臨時匯入。
- 一律透過事件請求讀取，切勿使用 `GriptapeNodes.SecretsManager()`。當節點在獨立 Worker 處理程序中執行時，**直接存取 SecretsManager 會被強制拒絕**；而輔助函式可能同時在 Worker（執行運算時）與編排程序（執行校驗時）被調用——因此管理器語法會在一端成功而在另一端報錯。採用 `handle_request` 則在兩端皆百分之百正確運作。
- 將 `API_KEY_NAME` 定義為類別常數以維持整潔一致。
- 實際調用外部服務前，務必防禦性校驗金鑰是否存在。

### 參數資料承載體積 (Parameter Payload Size) {: #parameter-payload-size }

參數的數值可能會被完整序列化並內嵌至以下兩個地方：

- **儲存的工作流程檔案**：工作流程序列化器會將獨特的參數值以字面量形式直接寫入存檔的 `.py` 檔案中，且完全無體積上限——該值包含的任何 Python 狀態都會被完整寫入。
- **WebSocket 即時事件**：在請求/回應事件中傳遞的參數值會被即時序列化並廣播發送給每一個連線的用戶端（編輯器 UI、MCP 伺服器等）。

這兩條傳輸途徑都不會在發送前預先檢查數值體積，因此若包含超大數值，會直接導致存檔肥大並塞爆 WebSocket 網路流量。只要節點底層 API 允許，請**以參照方式（檔案路徑或 URL 網址）**保存大型二進位資料（影像、音訊、影片、3D 資產、模型權重），絕不要內聯原始位元組。

宣告 `Parameter(serializable=False)`（參閱 [參數屬性](parameters.md#parameter-attributes)）**僅能解決第一條途徑**。它能防止數值被寫入硬碟上的 `.py` 流程檔（適合驅動器、檔案控制代碼與大型暫存緩衝區），但對 WebSocket 無效：該值依然會被廣播給前端。由於 WebSocket 傳輸無法針對單一參數退出，因此**主動縮減數值體積是您唯一的控制槓桿**。在輸出端上，此宣告還會將數值保留在生產該值的處理程序中，僅將引用鍵跨 Worker 邊界傳遞——詳見 [傳遞不可序列化值](passing_unserializable_values.md)。

!!! warning "嚴格控制參數資料體積"

    `griptape.artifacts.BlobArtifact` 儲存原始位元組，而 `ImageArtifact` 與 `AudioArtifact` 皆是其衍生子類別——因此節點若將這些型別作為參數，會將整份二進位位元組發送至所有連線用戶端，且會寫入存檔中（除非宣告 `serializable=False`）。請一律改用 `ImageUrlArtifact` 與 `AudioUrlArtifact`（`ParameterImage` 與 `ParameterAudio` 輔助類別會強制規範它們——參見 [參數系統](parameters.md#parameterimage-recommended-for-image-parameters)），它們無論背後檔案多麼龐大，永遠只保存簡短的 URL 字串。

    二進位位元組不僅限於 `BlobArtifact` 系列——`ThreeDArtifact` 亦包含二進位資料。請依據參數實際承載的真實記憶體體積來評估，而非單憑型別名稱看似無害就放鬆警惕。

### 模組匯入最佳實踐 {: #import-best-practices }

**一律在檔案頂層匯入依賴，切勿在函式內部延遲匯入：**

❌ **錯誤做法**——條件式/延遲匯入：

```python
def _get_image_data(self, image_artifact):
    try:
        from PIL import Image  # 切勿在此處匯入
        from io import BytesIO
        img = Image.open(BytesIO(image_bytes))
```

✅ **推薦做法**——頂層統一匯入：

```python
# 位於檔案頂部
from PIL import Image
from io import BytesIO


def _get_image_data(self, image_artifact):
    img = Image.open(BytesIO(image_bytes))
```

**為何堅持頂層匯入：**

- 依賴關係一目了然
- 避免在執行過程中反覆重複匯入造成額外開銷
- 符合 Python 官方 PEP 8 規範
- 啟動階段即可及早暴露缺失的依賴套件
- 獲得最優秀的 IDE 靜態分析與智慧自動補全支援

### 第三方程式庫的型別檢查技巧 {: #type-checking-for-third-party-libraries }

匯入某些第三方函式庫時可能遇到靜態型別檢查錯誤。請依據實際情境精準附加 `type: ignore` 註解：

#### 情境 1：程式庫已安裝但缺乏 Type Stubs 型別定義檔 {: #scenario-1-library-installed-but-missing-type-stubs }

針對已安裝但未隨附型別宣告的庫（如 `sklearn`, `ultralytics`, `supervision`）：

```python
# ✅ 程式庫存在但無型別 Stub
from sklearn.cluster import KMeans  # type: ignore[import-untyped]
from ultralytics import YOLO  # type: ignore[import-untyped]
from supervision import Detections  # type: ignore[import-untyped]
```

#### 情境 2：程式庫未安裝於 CI 型別檢查環境中 {: #scenario-2-library-not-installed-in-ci-type-checking-environment }

針對屬於執行階段可選依賴或未包含於 CI 容器的特化處理庫（如 `color-matcher`）：

```python
# ✅ 程式庫未安裝於型別檢查環境
from color_matcher import ColorMatcher  # type: ignore[reportMissingImports]
from color_matcher.normalizations import norm_img_to_uint8  # type: ignore[reportMissingImports]
```

| 錯誤型別 | 抑制註解 | 適用時機 |
| :--- | :--- | :--- |
| `import-untyped` | `# type: ignore[import-untyped]` | 套件已安裝但缺乏型別定義 |
| `reportMissingImports` | `# type: ignore[reportMissingImports]` | 套件根本未安裝於檢查環境 |

### 函式引數管理與 Dataclass 封裝 {: #function-parameter-management }

透過 `dataclass` 將過多的函式參數（建議超過 5 個即進行封裝）整合為結構化物件：

❌ **不良設計**——散落過多零碎引數：

```python
def process_bbox(self, x: int, y: int, width: int, height: int,
                 dilation_percent: float, img_width: int, img_height: int):
    # 處理邊界框...
```

✅ **優良設計**——採用 Dataclass 聚合：

```python
from dataclasses import dataclass

@dataclass
class BoundingBox:
    x: int
    y: int
    width: int
    height: int
    dilation_percent: float
    img_width: int
    img_height: int

def process_bbox(self, bbox: BoundingBox):
    # 透過 bbox.x, bbox.y 等屬性進行清晰處理
```

### 程式碼品質與 Git 提交規範 {: #code-quality }

- 清除所有行尾空白字元（包含純空白行）。
- 維持一致的縮排（一律使用空格，嚴禁 Tab）。
- 盡可能將單行寬度限制在 120 個字元內。
- 僅在真正需要將目錄視為 Python 套件時才建立 `__init__.py`。

#### Pre-commit 本機強制驗證 (Required) {: #pre-commit-checks-required }

在向 `griptape-nodes` 提交 Commit 前，請務必執行格式化與靜態檢查，並修正所有錯誤：

```bash
make format
make check/lint
make check/types
```

#### 節點文件與導航配置 {: #node-docs-navigation }

在向核心程式庫新增節點時，必須同步提供參考文件：

- 建立文件頁面：`docs/nodes/<category>/<node>.md`
- 在 `mkdocs.yml` 的 `nav -> Nodes Reference -> <Category>` 底下完成導航掛載。

## 生產級錯誤處理策略 {: #production-error-handling }

### 執行前全面驗證 (Comprehensive Validation) {: #comprehensive-validation }

覆寫 `validate_before_node_run()` 進行複雜的邊界條件檢查：

```python
def validate_before_node_run(self) -> list[Exception] | None:
    """在節點運算前全面驗證各項參數。"""
    exceptions = []

    model = self.get_parameter_value("model")
    if model == "advanced":
        images = self.get_parameter_list_value("images") or []
        if len(images) > MAX_IMAGES:
            exceptions.append(ValueError(f"{self.name}: 影像上限為 {MAX_IMAGES} 張，目前傳入 {len(images)} 張"))

    return exceptions if exceptions else None
```

### 連線依賴校驗模式 {: #connection-validation-patterns }

針對具備複雜拓撲依賴的進階節點：

```python
def _validate_iterative_connections(self) -> list[Exception]:
    """驗證所有必要的拓撲連線是否皆已正確建立。"""
    errors = []
    node_type = self._get_base_node_type_name()

    if not _outgoing_connection_exists(self.name, self.exec_out.name):
        errors.append(
            Exception(
                f"{self.name}: 缺少來自 'On Each Item' 的必要控制連線。"
                f"必要操作：請將 {node_type} Start 連接至迴圈內部的計算節點。"
                "起始節點必須連線至其他節點以驅動迴圈主體執行。"
            )
        )

    return errors
```

### 安全預設值保障 (Safe Defaults Pattern) {: #safe-defaults-pattern }

在拋出例外中斷前，請一律為輸出端填入安全預設值，避免前端介面或下游節點存取未初始化變數：

```python
def _set_safe_defaults(self) -> None:
    """為所有輸出端指派安全預設值。"""
    self.parameter_output_values["result"] = None
    self.parameter_output_values["status"] = "error"
    self.parameter_output_values["count"] = 0


def process(self) -> None:
    try:
        result = process_data()
        self.parameter_output_values["result"] = result
    except Exception as e:
        self._set_safe_defaults()
        raise RuntimeError(f"運算處理中斷失敗: {str(e)}") from e
```

## 除錯記錄最佳實踐 (Logging Best Practices) {: #logging-best-practices }

### 異常抑制安全記錄 {: #safe-logging-pattern }

避免記錄日誌本身的異常導致核心業務崩潰：

```python
from contextlib import suppress
import logging

logger = logging.getLogger(__name__)


def _log(self, message: str) -> None:
    """透過異常抑制實現安全記錄。"""
    with suppress(Exception):
        logger.info(message)
```

### 敏感資訊淨化 (Request Sanitization) {: #request-sanitization }

在輸出日誌前，淨化或截斷敏感金鑰與超長 Base64 字串：

```python
from copy import deepcopy
import json

PROMPT_TRUNCATE_LENGTH = 100


def _log_request(self, payload: dict[str, Any]) -> None:
    """記錄已淨化敏感資訊的請求內容。"""
    with suppress(Exception):
        sanitized_payload = deepcopy(payload)

        # 截斷超長提示詞
        prompt = sanitized_payload.get("prompt", "")
        if len(prompt) > PROMPT_TRUNCATE_LENGTH:
            sanitized_payload["prompt"] = prompt[:PROMPT_TRUNCATE_LENGTH] + "..."

        # 遮蔽 Base64 影像資料
        if "image" in sanitized_payload:
            image_data = sanitized_payload["image"]
            if isinstance(image_data, str) and image_data.startswith("data:image/"):
                parts = image_data.split(",", 1)
                header = parts[0] if parts else "data:image/"
                b64_len = len(parts[1]) if len(parts) > 1 else 0
                sanitized_payload["image"] = f"{header},<base64 資料長度={b64_len}>"

        self._log(f"發送請求: {json.dumps(sanitized_payload, indent=2, ensure_ascii=False)}")
```
