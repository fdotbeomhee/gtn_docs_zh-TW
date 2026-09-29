# 參數系統 (Parameters) {: #parameters }

參數定義了節點的輸入端、輸出端與自訂屬性。本頁面深入介紹 `Parameter` 的每個屬性欄位、特徵標籤 (Traits) 系統、參數輔助包裝類別、容器型別以及用於建立動態參數介面的進階模式。

關於參數型別至編輯器視覺部件的對應關係、支援的 `ui_options` 設定鍵與特徵標籤清單，請參閱 [參數 UI 參考指南](parameter_ui_reference.md)。

## 參數屬性欄位 (Parameter Attributes) {: #parameter-attributes }

`Parameter` 類別包含以下核心屬性：

- **name**：`str`，參數唯一識別名稱，不可包含空白字元。
- **tooltip**：`str` 或 `list[dict]`，在 UI 介面上懸浮顯示的提示說明文字。
- **default_value**：`Any`，參數未連線且未指定時採用的預設值。
- **type**：`str`，參數在引擎中的核心型別（例如 `"str"`、`"list[str]"` 或 `ParameterTypeBuiltin.STR.value`）。
- **input_types**：`list[str]`，允許連入此參數的型別清單。
- **output_type**：`str`，連線輸出至其他節點時的型別。
- **allowed_modes**：`set[ParameterMode]`，允許的模式集合，包含 `{INPUT, OUTPUT, PROPERTY}`。
- **ui_options**：`dict`，傳遞給前端以自訂 UI 呈現樣式的選項字典。
- **converters**：`list[Callable[[Any], Any]]`，數值寫入前的轉換器函式清單。
- **validators**：`list[Callable[[Parameter, Any], None]]`，數值校驗器函式清單。
- **hide/hide_label/hide_property**：常用 UI 便捷旗標（亦可透過 `ui_options` 設定；衝突時以 `ui_options` 為準）。
- **allow_input/allow_property/allow_output**：配置模式的便捷旗標（若已顯式宣告 `allowed_modes` 則會被忽略）。
- **settable**：`bool`（預設為 `True`）——對於純計算產生或唯讀的輸出參數，可設為 `False`。
- **serializable**：`bool`（預設為 `True`）——對於不可序列化的值（如驅動器 Driver 實例、檔案控制代碼等）請設為 `False`。當宣告在輸出端時，這會將數值保留在產生的處理程序內部，僅跨 Worker 處理程序傳遞輕量鍵值——詳見 [傳遞不可序列化值](passing_unserializable_values.md)。
- **user_defined**：`bool`（預設為 `False`）。
- **private**：`bool`（預設為 `False`）——對常規使用者編輯介面隱藏（供程式庫或引擎內部專用）。
- **exclude_from_metadata**：`bool`（預設為 `False`）——從明文出處中繼資料輸出（附屬 JSON 與嵌入式 PNG 區塊）中排除此參數的值。參數名稱仍會記錄在 `parameters_omitted` 中以供稽核。適用於保存密碼或使用者私密金鑰等敏感資訊的參數。
- **parent_container_name**：`str | None`——將此參數指派為 `ParameterContainer`（即 `ParameterList` 或 `ParameterDictionary`）的子項。用於清單型所有權結構。
- **parent_element_name**：`str | None`——將此參數巢狀掛載至 `ParameterGroup`（UI 分組視覺元件）下方。用於節點介面中的視覺折疊分組。

## 特徵標籤 (Traits) {: #traits }

透過 `add_trait()` 賦予參數額外的介面互動與約束能力：

- **Options**：`Options(choices=list[str] | list[tuple[str, Any]], show_search: bool = True, search_filter: str = "", allow_custom: bool = False)`
- **Slider**：`Slider(min_val: float, max_val: float, soft_limits: bool = False)`——`soft_limits=True` 會將範圍視為軟性限制：滑桿軌道維持在範圍內，但手動鍵入超出範圍的值時會被允許而非拋出例外。
- **Button**：`Button(label: str = "", variant=..., size=..., button_link=... | on_click=..., get_button_state=...)`
- **ColorPicker**：`ColorPicker(format="hex")`
- **FileSystemPicker**：`FileSystemPicker(...)`（檔案與目錄選取器 UI）

關於特徵標籤的完整清單、渲染部件及受控的 `ui_options` 鍵名，請參閱 [參數 UI 參考指南](parameter_ui_reference.md)。

**保存特徵標籤狀態**：特徵標籤透過 `to_state()` 與 `apply_state()` 選擇性加入狀態持久化機制。回傳符合建構子規格的文字、數值、布林值以及包含這些型別的列表或字典。不支援的值將被忽略並記錄警告。使用預設方法的特徵標籤不保存任何狀態。

**接收 UI 選項寫入**：實作 `state_from_ui_options()` 可將來自節點程式碼、編輯器或存檔的 `ui_options` 寫入操作映射至 `apply_state()` 所接受的相同狀態。預設行為會忽略寫入，這適合無可變狀態的純渲染鍵。若寫入本應改變特徵標籤的渲染結果但特徵標籤不接受該寫入，系統將記錄警告日誌。

## 參數輔助包裝類別 (`ParameterString`, `ParameterInt`, ...) {: #parameter-helper-constructs-parameterstring-parameterint }

Griptape Nodes 在 `griptape_nodes.exe_types.param_types.*` 命名空間下提供了一系列便捷的 Parameter 衍生輔助類別。
其設計初衷是讓常見的參數宣告**簡潔、一致且支援執行階段動態修改**（許多 UI 選項皆直接封裝為 Python 屬性）。

### 常用輔助型別速查表 {: #quick-reference-table }

| 輔助類別 | 強制指定的 `type` / `output_type` | 預設 `input_types` 行為 | 核心 UI 便捷引數 | 說明備註 |
| :--- | :--- | :--- | :--- | :--- |
| `ParameterString` | `"str"` / `"str"` | `accept_any=True` → `["any"]` 並自動轉為 `str` | `markdown`, `multiline`, `placeholder_text`, `is_full_width` | 建構子中的 `type`、`output_type` 與 `input_types` 引數會被忽略 |
| `ParameterBool` | `"bool"` / `"bool"` | `accept_any=True` → `["any"]` 並自動轉為 `bool` | `on_label`, `off_label` | 自動識別轉換常見字串，如 `"true"/"false"`、`"yes"/"no"` |
| `ParameterInt` | `"int"` / `"int"` | `accept_any=True` → `["any"]` 並自動轉為 `int` | `step`, `slider`, `min_val`, `max_val`, `validate_min_max` | 根據引數自動掛載約束特徵 (Clamp/MinMax/Slider) |
| `ParameterFloat` | `"float"` / `"float"` | `accept_any=True` → `["any"]` 並自動轉為 `float` | `step`, `slider`, `min_val`, `max_val`, `validate_min_max` | 根據引數自動掛載約束特徵 (Clamp/MinMax/Slider) |
| `ParameterDict` | `"dict"` / `"dict"` | `accept_any=True` → `["any"]` 並自動轉為 `dict` | (無) | 調用 `griptape_nodes.utils.dict_utils.to_dict()` 執行轉換 |
| `ParameterJson` | `"json"` / `"json"` | `accept_any=True` → `["any"]` 並自動轉為 JSON | `button`, `button_label`, `button_icon` | 使用 `json_repair.repair_json()` 達成容錯字串至 JSON 轉換 |
| `ParameterRange` | `"list"` / `"list"` | `accept_any=True` → `["any"]` 並自動轉為 `list` | `range_slider` + `min_val/max_val/step`，標籤設定 | 雙數值滑桿僅在數值恰好為包含兩個數值的清單時有效 |
| `ParameterImage` | `"ImageUrlArtifact"` / `"ImageUrlArtifact"` | `accept_any=True` → `["any"]` (不轉換) | `clickable_file_browser`, `webcam_capture_image`, `edit_mask`, `pulse_on_run` | 主要是 UI 便捷性；如需強迫型別轉換可自訂 converters |
| `ParameterAudio` | `"AudioUrlArtifact"` / `"AudioUrlArtifact"` | `accept_any=True` → `["any"]` (不轉換) | `clickable_file_browser`, `microphone_capture_audio`, `edit_audio`, `pulse_on_run` | 主要是 UI 便捷性；如需強迫型別轉換可自訂 converters |
| `ParameterVideo` | `"VideoUrlArtifact"` / `"VideoUrlArtifact"` | `accept_any=True` → `["any"]` (不轉換) | `clickable_file_browser`, `webcam_capture_video`, `edit_video`, `pulse_on_run` | 主要是 UI 便捷性；如需強迫型別轉換可自訂 converters |
| `Parameter3D` | `"ThreeDUrlArtifact"` / `"ThreeDUrlArtifact"` | `accept_any=True` → `["any"]` (不轉換) | `clickable_file_browser`, `expander`, `pulse_on_run` | 主要是 UI 便捷性；如需強迫型別轉換可自訂 converters |
| `ParameterButton` | `"button"` / `"str"` | `["str", "any"]` | `label`, `variant`, `size`, `icon`, `state`, `href` / `on_click` | **`label` 為顯示文字；`default_value` 為實際儲存值** |

### 輔助類別的共通行為 {: #shared-behavior-across-helpers }

- 所有輔助建構子皆完整繼承標準 `Parameter` 的建構引數（`allowed_modes` 或 `allow_input/allow_property/allow_output`、`hide/hide_label/hide_property`、`settable`、`serializable` 等）。
- 許多輔助類別預設啟用 `accept_any=True`。啟用時，輔助類別通常將 `input_types=["any"]` 並自動在前置佇列加入轉換函式（例如 `ParameterString` 將所有連入值轉為 `str`）。若需要嚴格的型別校驗，請將其關閉。
- 若同時提供明確的便捷引數（例如 `hide=True`）與 `ui_options` 中的同名鍵（例如 `ui_options={"hide": False}`），**一律以 `ui_options` 為準**，且 Griptape Nodes 會發出衝突警示。

### 重點輔助類別深入解析 {: #detailed-helper-notes }

#### `ParameterString` {: #parameterstring }

- 強制指定 `type="str"` 與 `output_type="str"`。
- `accept_any=True` 時將 `None` 轉為 `""`，其他數值則透過 `str(value)` 轉換。
- UI 便捷屬性：`markdown`、`multiline`、`placeholder_text`、`is_full_width`（皆支援執行階段讀寫）。

#### `ParameterBool` {: #parameterbool }

- 強制指定 `type="bool"` 與 `output_type="bool"`。
- `accept_any=True` 時將常見字串表示（如 `"true"`、`"yes"`、`"on"`、`"1"`）轉換為 `True`，將（`"false"`、`"no"`、`"off"`、`"0"`）轉換為 `False`。
- UI 便捷屬性：`on_label`、`off_label`（執行階段可變屬性）。

#### `ParameterInt` / `ParameterFloat`（基於 `ParameterNumber`） {: #parameterint-parameterfloat-via-parameternumber }

- 強制指定數值 `type` / `output_type`，並在 `accept_any=True` 時前置掛載型別轉換器。
- `step`：儲存於 `ui_options["step"]` 並進行步進校驗（值必須為當前步進的整數倍）。
- `slider`, `min_val`, `max_val`, `validate_min_max`：根據優先權自動掛載以下約束特徵之一：
    - 若 `slider=True` 則掛載 `Slider(min_val, max_val)`
    - 若 `validate_min_max=True` 則掛載 `MinMax(min_val, max_val)`
    - 若同時提供 `min_val` 與 `max_val` 則掛載 `Clamp(min_val, max_val)`
- `soft_limits`：僅在搭配 `slider=True` 時有效。將滑桿範圍設為軟限制——軌道維持在 `min_val`–`max_val` 區間，但允許使用者在輸入框手動鍵入超出此區間的數值。可透過 `soft_limits` 屬性在執行階段動態讀寫。

#### `ParameterJson` {: #parameterjson }

- 強制指定 `type="json"` 與 `output_type="json"`。
- `accept_any=True` 時使用 `json_repair.repair_json()` 嘗試修復並解析 JSON 字串（亦嘗試將非字串輸入序列化）。
- UI 便捷功能：可選的編輯器快開按鈕（`button`, `button_label`, `button_icon`）。

#### `ParameterDict` {: #parameterdict }

- 強制指定 `type="dict"` 與 `output_type="dict"`。
- `accept_any=True` 時利用 `to_dict(...)` 將各類輸入強制轉換為標準字典。

#### `ParameterRange` {: #parameterrange }

- 強制指定 `type="list"` 與 `output_type="list"`。
- `accept_any=True` 時將 `None` 轉為 `[]`，列表維持為列表，其他任何值包裝為 `[value]`。
- UI 便捷功能：`range_slider`（巢狀 `ui_options["range_slider"]` 物件），包含 `min_val/max_val/step` 與標籤能見度設定。
- 雙數值滑桿 UI 僅在參數值恰好為兩個數值的列表時生效。

#### `ParameterImage`（影像參數強烈推薦首選） {: #parameterimage-recommended-for-image-parameters }

**處理影像輸入/輸出時，請一律使用 `ParameterImage` 取代通用 `Parameter`。** 其優勢包括：

- 自動指派 `type="ImageUrlArtifact"` 與 `output_type="ImageUrlArtifact"`
- 內建檔案瀏覽器選取、視訊鏡頭擷取與遮罩繪製等完整 UI 支援
- 確保所有影像處理節點之間的介面與行為高度一致

這亦能有效控制參數傳輸體積：`ImageUrlArtifact` 僅保存短網址字串，而傳統的替代方案 `ImageArtifact` 會將整張影像的原始二進位位元組內嵌至 WebSocket 流量與存檔的工作流程中（除非顯式宣告 `serializable=False`）——詳見 [參數資料承載體積](error_handling.md#parameter-payload-size)。

**基本宣告範例：**

```python
from griptape_nodes.exe_types.param_types.parameter_image import ParameterImage

# 輸入影像參數
self.add_parameter(
    ParameterImage(
        name="input_image",
        tooltip="供處理的輸入影像",
        allow_output=False,  # 僅作輸入
    )
)

# 輸出影像參數
self.add_parameter(
    ParameterImage(
        name="output_image",
        tooltip="生成的輸出影像結果",
        allow_input=False,  # 僅作輸出
        allow_property=False,
    )
)
```

**支援的 UI 選項：**

- `clickable_file_browser`：啟用檔案瀏覽器選取影像
- `webcam_capture_image`：啟用視訊鏡頭即時拍照
- `edit_mask`：啟用遮罩繪製塗抹覆蓋層
- `pulse_on_run`：影像更新時在 UI 上呈現脈衝視覺回饋

**動態可見度切換範式：**

針對僅在特定模型下才需要呈現的參數：

```python
def __init__(self, **kwargs) -> None:
    super().__init__(**kwargs)

    # 新增影像參數（預設隱藏）
    self.add_parameter(
        ParameterImage(
            name="input_image",
            tooltip="圖生圖 (Image-to-Image) 專用的輸入影像",
            allow_output=False,
        )
    )

    # 依據預設模型初始化可見度
    self._initialize_parameter_visibility()


def _initialize_parameter_visibility(self) -> None:
    """依據當前選取的模型初始化參數可見度。"""
    model = self.get_parameter_value("model") or "default"
    if model in ["model-with-image-support", "another-model"]:
        self.show_parameter_by_name("input_image")
    else:
        self.hide_parameter_by_name("input_image")


def after_value_set(self, parameter: Parameter, value: Any) -> None:
    """當模型下拉選單切換時即時更新參數可見度。"""
    if parameter.name == "model":
        if value in ["model-with-image-support", "another-model"]:
            self.show_parameter_by_name("input_image")
        else:
            self.hide_parameter_by_name("input_image")
            self.set_parameter_value("input_image", None)  # 隱藏時清空殘留值

    return super().after_value_set(parameter, value)
```

**為何優先選用 `ParameterImage`：**

| 維度 | 通用 `Parameter` | `ParameterImage` |
| :--- | :--- | :--- |
| 型別安全 | 需手動配置 `type` 與 `input_types` | 自動綁定標準產物型別 |
| UI 特性 | 需自行撰寫大量 `ui_options` | 內建檔案瀏覽器、鏡頭擷取、遮罩塗繪 |
| 一致性 | 視個別開發者實作而異 | 全平台節點標準化統一行為 |
| 維護成本 | 充滿重複性樣板程式碼 | 程式碼乾淨凝練 |

**舊式寫法（強烈建議淘汰）：**

```python
# ❌ 請勿採用此寫法——請改用 ParameterImage
self.add_parameter(
    Parameter(
        name="input_image",
        input_types=["ImageArtifact", "ImageUrlArtifact", "str"],
        type="ImageArtifact",
        default_value=None,
        tooltip="輸入影像",
        allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
        ui_options={"display_name": "Input Image"},
    )
)
```

#### `ParameterAudio` / `ParameterVideo` / `Parameter3D` {: #parameteraudio-parametervideo-parameter3d }

- 強制指定對應的 `*UrlArtifact` 作為 `type` / `output_type`（例如 `AudioUrlArtifact`、`VideoUrlArtifact`、`ThreeDUrlArtifact`）。
- 這些輔助類別主要封裝了豐富的 UI 互動能力（檔案瀏覽/錄製擷取/編輯/展開器）。若需要字串至產物的型別轉換，可提供 `converters` 或在節點的 `before_value_set()` / `process()` 中進行處理。
- **音訊**：`ParameterAudio` 在傳輸負載體積上的考量與 `ParameterImage` 完全相同——`AudioArtifact` 會內嵌全部位元組資料，而 `AudioUrlArtifact` 僅傳遞短網址。
- **影片**：引擎並未設計原始位元組的 `VideoArtifact` 類別，因此不存在位元組內嵌風險。新節點中請一律宣告 `VideoUrlArtifact`——連線型別檢查要求型別名稱完全相符，因此 `"VideoArtifact"` 與 `"VideoUrlArtifact"` **不可**混用互換。
- **3D**：`ThreeDArtifact` 亦包含二進位位元組，請優先使用 `Parameter3D` 所強制規範的 `ThreeDUrlArtifact`。

#### `ParameterButton` {: #parameterbutton }

按鈕提供了點擊時觸發操作的互動式 UI 元件，例如更新參數、執行運算或在不同狀態間切換導覽。

**核心規格：**

- 強制指定 `type="button"` 與 `output_type="str"`。
- 預設為**純屬性** UI 元件 (`allow_property=True`, `allow_input=False`, `allow_output=False`)。
- 接收 `href="..."`（外部導航連結）或 `on_click=...`（自訂回呼函式）。
- `label` 為按鈕表面顯示的文字標籤。
- `icon` 支援指定視覺圖示。

**實作範式：** 按鈕必須包裹在 `ParameterButtonGroup` 容器中：

```python
from griptape_nodes.exe_types.core_types import ParameterButtonGroup
from griptape_nodes.exe_types.param_types.parameter_button import ParameterButton
from griptape_nodes.traits.button import Button, ButtonDetailsMessagePayload


class MyNode(DataNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        with ParameterButtonGroup(name="my_button_group") as button_group:
            ParameterButton(
                name="update_button",
                label="更新日期/時間",
                icon="calendar",
                on_click=self._handle_button_click,  # 直接傳入回呼
            )
        self.add_node_element(button_group)

        self.add_parameter(
            Parameter(
                name="display_value",
                tooltip="由按鈕更新的數值",
                type=ParameterTypeBuiltin.STR.value,
                allowed_modes={ParameterMode.PROPERTY},
                ui_options={
                    "display_name": "顯示數值",
                    "readonly": True,  # 禁止手動編輯
                },
                default_value="點擊按鈕以更新",
            )
        )

    def _handle_button_click(
        self,
        button: Button,
        button_payload: ButtonDetailsMessagePayload,
    ) -> None:
        """按鈕點擊處理常式。"""
        new_value = "更新於 " + datetime.now().strftime("%H:%M:%S")
        self.set_parameter_value("display_value", new_value)
```

## 容器型別 (Containers) {: #containers }

- **ParameterList**：管理多個子 `Parameter` 項目的容器參數（使用 `get_parameter_list_value()` 展平取值）。
- **ParameterDictionary**：管理有序鍵值對項目的容器參數（有別於純值輔助類別 `ParameterDict`）。
- **ParameterGroup**：純粹用於前端 UI 折疊分組的視覺容器。

**容器關鍵語意：**

- 容器參數在引擎內部以 `ParameterContainer` 物件表示。它們**永遠為真 (Always Truthy)**，即便內容為空（覆寫了 `__bool__()` 以防止過期快取造成邏輯漏洞）。
- `ParameterList` 支援多種前端顯示選項（如 `collapsed` 折疊、網格呈現、欄數設定）。
- `ParameterDictionary` 是有序的鍵值對集合（內部以列表維持順序）。

**`parent_container_name` 與 `parent_element_name` 的致命差異：**

| 屬性名稱 | 指向目標 | 設計目的 |
| :--- | :--- | :--- |
| `parent_container_name` | `ParameterContainer` (`ParameterList`, `ParameterDictionary`) | **所有權歸屬**。表示該參數是列表或字典容器的子項。引擎用於 `add_parameter()` 註冊、子項生命週期清理、數值匯總與儲存載入。 |
| `parent_element_name` | `ParameterGroup` | **UI 視覺分組**。將參數視覺上巢狀歸納於節點面板的折疊群組中。用於 UI 佈局、`_remove_existing_*()` 查找與介面還原。 |

**切勿混淆這兩者！** 若將 `parent_element_name` 誤寫為 `parent_container_name`（或反之）：

1. 參數會脫離分組，直接暴露於節點最頂層。
1. 在連續執行間無法被正確清理（殘留過期輸出）。
1. 工作流程在存檔與載入時會失敗——還原處理常式在以指定名稱查找容器或群組時，若型別不符，將在數值套用前直接靜默丟棄該參數。

**記憶口訣：**

- 放入 **`ParameterList`** 或 **`ParameterDictionary`**？ → 使用 `parent_container_name`
- 放入 **`ParameterGroup`** 進行面板視覺排版？ → 使用 `parent_element_name`

```python
# ✅ 正確：為了 UI 視覺分組掛載於 ParameterGroup 下
param = ParameterImage(
    name="cell_0_0",
    parent_element_name=self._grid_cells_group.name,  # ParameterGroup
    ...
)

# ❌ 錯誤：將 ParameterGroup 指定給 parent_container_name
param = ParameterImage(
    name="cell_0_0",
    parent_container_name=self._grid_cells_group.name,  # 錯誤：這是 ParameterGroup，並非 ParameterContainer
    ...
)
```

## ParameterList 設計範式 {: #parameterlist-pattern }

適用於接收多個同型別輸入的參數：

```python
self.add_parameter(
    ParameterList(
        name="tools",
        input_types=["Tool", "list[Tool]"],
        default_value=[],
        tooltip="連接個別工具或工具清單",
        allowed_modes={ParameterMode.INPUT},
    )
)

# 在 process 方法中提取值
tools = self.get_parameter_list_value("tools")  # 一律回傳平整列表
for tool in tools:
    # 逐一處理各個工具
```

**行為注意事項**：`get_parameter_list_value()` 會自動展平巢狀迭代物件並**自動濾除 Falsey 元素**（例如 `0`、`False`、`""`、空字典/列表）。若需完整保留這些值，請改用 `get_parameter_value()` 並自行手動處理。

## 常見參數設計範式 (Common Parameter Patterns) {: #common-parameter-patterns }

### 附帶佔位文字的搜尋輸入框 {: #search-input-with-placeholder }

```python
Parameter(
    name="search_query",
    input_types=["str"],
    type="str",
    tooltip="搜尋模型的關鍵字",
    allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
    ui_options={"placeholder_text": "例如：llama, bert, stable-diffusion"},
)
```

### 全寬清單輸出 (Full-Width List Output) {: #full-width-list-output }

```python
Parameter(
    name="results",
    output_type="list[dict]",
    type="list[dict]",
    tooltip="包含完整資訊的搜尋結果",
    allowed_modes={ParameterMode.OUTPUT},
    ui_options={"is_full_width": True},
)
```

### 多行文字輸入 (Multiline Text Input) {: #multiline-text-input }

```python
Parameter(
    name="prompt",
    input_types=["str"],
    type="str",
    tooltip="期望輸出的描述提示詞",
    allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
    ui_options={"multiline": True, "placeholder_text": "描述您欲產生的內容..."},
)
```

### 附帶檔案總管瀏覽器的檔案路徑 {: #file-path-with-browser }

若要讓使用者從磁碟挑選檔案或目錄，可在 `str` 參數附加 `FileSystemPicker` 特徵標籤。參數的值即為所選路徑：

```python
from griptape_nodes.traits.file_system_picker import FileSystemPicker

Parameter(
    name="config_path",
    type="str",
    tooltip="組態檔案的路徑",
    allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
    traits={
        FileSystemPicker(
            allow_files=True,
            allow_directories=False,
            file_extensions=[".json", ".yaml"],
        )
    },
)
```

`FileSystemPicker` 預設僅選取目錄，因此當欲選取檔案時請傳入 `allow_files=True`。它亦支援 `multiple`（多選）、`workspace_only`（僅限工作區）、檔案大小上限以及包含/排除模式——詳見 [參數 UI 參考手冊 (Parameter UI Reference)](parameter_ui_reference.md#traits)。

媒體參數的運作方式截然不同。`ParameterImage`、`ParameterAudio`、`ParameterVideo` 與 `Parameter3D` 各自在媒體檢視器中提供點擊瀏覽上傳功能（`clickable_file_browser`，預設開啟），且所選檔案會被上傳並以 `*UrlArtifact` 而非純路徑字串傳遞給節點。處理媒體時請優先使用這些輔助類別而非 `str` 路徑——參見上方的 [`ParameterImage`](#parameterimage-recommended-for-image-parameters)。

## 進階參數設計範式 (Advanced Parameter Patterns) {: #advanced-parameter-patterns }

### 動態參數可見度 (Dynamic Parameter Visibility) {: #dynamic-parameter-visibility }

使用 `after_value_set()` 生命週期回呼構建具備情境感知能力的前端 UI：

```python
def after_value_set(self, parameter: Parameter, value: Any) -> None:
    """根據所選模型動態更新參數的可見度。"""
    if parameter.name == "model":
        if value == "text-to-image":
            self.hide_parameter_by_name("input_image")
            self.show_parameter_by_name("prompt")
        elif value == "image-to-image":
            self.show_parameter_by_name("input_image")
            self.show_parameter_by_name("prompt")

    return super().after_value_set(parameter, value)
```

### 動態選項更新 (Dynamic Options Updates) {: #dynamic-options-updates }

在執行階段動態更新參數的下拉選項清單。這些選項會作為特徵標籤 (Trait) 狀態保存：

```python
from griptape_nodes.traits.options import Options


def _update_option_choices(self, param_name: str, choices: list, default_value: str):
    """動態更新 Options 特徵標籤的選項。"""
    param = self.get_parameter_by_name(param_name)
    if not param:
        return

    # 特徵標籤作為子元素儲存於 Parameter 上
    # （最常見的情況為更新 Options 特徵標籤）
    for trait in param.find_elements_by_type(Options):
        trait.choices = choices
        break
    self.set_parameter_value(param_name, default_value)
```

## 動態參數架構與 ParameterTransitionComponent {: #dynamic-parameter-schemas-with-parametertransitioncomponent }

某些進階節點包含一個下拉選單（例如模型切換器、模式選擇器、運算元挑選），每個選項都代表著截然不同的輸入與輸出參數組合。傳統作法（清空所有參數並從頭重建）會粗暴毀掉使用者已辛苦連好的所有連接線。`ParameterTransitionComponent` 透過精確計算當前參數介面與新選項所需介面的差異 (Diff) 完美解決此問題，針對每個參數名稱執行以下四種策略之一：

- **Preserve (保留)**——新舊兩側皆存在且簽章完全一致。分毫未動，連線與數值 100% 完好如初。
- **Replace (替換)**——兩側皆存在但簽章發生改變（型別或模式不同）。組件會先行捕獲當前的連入與連出關係，移除舊參數，建立帶有新簽章的新參數，隨後透過 `CreateConnectionRequest` 重新發送捕獲的連線請求。平台的連線處理常式會重新驗證型別相容性：新簽章下依然合法的連線會自動完美復原，不合法的連線則安全斷開。
- **Remove (移除)**——新選項不再需要該參數。透過 `RemoveParameterFromNodeRequest` 妥善移除並清理連線。
- **Add (新增)**——全新參數。透過呼叫方提供的工廠函式動態建立。

### 程式庫作者的實作方式 {: #what-the-node-author-writes }

```python
from griptape_nodes.exe_types.param_components.parameter_transition_component import (
    ParameterTransitionComponent,
    TransitionParameter,
)

# 在節點的 __init__ 中：
self._model_params = ParameterTransitionComponent(
    self,
    manages_parameter=lambda p: p.name in self._MODEL_PARAM_NAMES,
)

# 在模型下拉選單切換時的處理常式中：
desired = self._build_transition_parameters_for_model(selected_model)
self._model_params.transition_to(desired)
```

### 何時採用此組件 {: #when-to-reach-for-this }

- 單一節點的參數介面結構取決於**離散的使用者選項**（模型挑選器、運算模式切換器）。
- 使用者在正常的流程編排中會頻繁切換選項，且不希望每次切換都要重新連線。

### 何時不應使用此組件 {: #when-not-to-reach-for-this }

- 僅需根據數值**顯示或隱藏**某些既定參數——使用 `hide_parameter_by_name` / `show_parameter_by_name` 的 [動態可見度](#dynamic-parameter-visibility) 模式。
- 僅需**更新下拉選單的選項清單**——使用 [動態選項更新](#dynamic-options-updates) 模式。
- 參數結構完全固定，僅數值產生變化。

#### 與 ParameterGroup 搭配使用 {: #working-with-parametergroups }

該組件依名稱管理個別參數。若您的動態介面使用分組，請在 `add_request_factory` 中傳入 `parent_element_name`，以便新加入的參數能正確歸納至對應群組中；群組的建立與清理交由呼叫方處理。

### ParameterList 進階用法 {: #advanced-parameterlist-usage }

同時包含個別型別與清單型別以實現最高靈活性。此處的 `ImageArtifact` 僅用於相容舊版工作流程連線——新宣告的清單參數請一律使用 `ImageUrlArtifact`（參見 [參數傳輸大小限制](error_handling.md#parameter-payload-size)）：

```python
self.add_parameter(
    ParameterList(
        name="images",
        input_types=[
            "ImageUrlArtifact",
            "ImageArtifact",  # 僅供舊版連線相容
            "str",
            "list",
            "list[ImageUrlArtifact]",
            "list[ImageArtifact]",  # 僅供舊版連線相容
        ],
        default_value=[],
        tooltip="輸入影像（總計最多 10 張影像）",
        allowed_modes={ParameterMode.INPUT},
        ui_options={"expander": True, "display_name": "輸入影像"},
    )
)
```

### 控制參數在 UI 中的排列順序 {: #controlling-parameter-order-in-the-ui }

參數在 UI 中依其透過 `add_parameter()` 新增的順序呈現。這對使用者體驗至關重要——相關的參數應當在邏輯上分組排列。

**問題所在**：諸如 `BaseImageProcessor` 等基底類別會在自身的 `__init__` 中自動新增參數（如 `input_image`），這可能不是您所期望的順序。

**解決方案**：直接繼承 `SuccessFailureNode` 而非 `BaseImageProcessor`，藉此取得對參數排序的完全控制權：

```python
from griptape_nodes.exe_types.node_types import SuccessFailureNode
from griptape_nodes.exe_types.param_types.parameter_image import ParameterImage


class ColorMatch(SuccessFailureNode):
    """將參考影像的色彩風格轉移至目標影像。"""

    def __init__(self, name: str, metadata: dict[Any, Any] | None = None) -> None:
        super().__init__(name, metadata)

    # 參考影像排在第一位——色彩調色盤的來源
    self.add_parameter(
        ParameterImage(
            name="reference_image",
            tooltip="參考影像——欲轉移之色彩調色盤來源",
            ui_options={"clickable_file_browser": True, "expander": True},
        )
    )

    # 目標影像排在第二位——欲修改的影像
    self.add_parameter(
        ParameterImage(
            name="target_image",
            tooltip="目標影像——套用色彩風格轉移的影像",
            ui_options={"clickable_file_browser": True, "expander": True},
        )
    )

    # 依序新增其他參數...
```

### 雙影像處理節點標準範式 {: #two-image-processing-node-pattern }

對於需同時處理兩張影像的節點（混合、色彩比對、影像合成），請使用以下設計範式：

```python
from typing import Any, ClassVar
from PIL import Image

from griptape.artifacts import ImageUrlArtifact
from griptape_nodes.exe_types.core_types import Parameter
from griptape_nodes.exe_types.node_types import SuccessFailureNode
from griptape_nodes.exe_types.param_types.parameter_image import ParameterImage
from griptape_nodes_library.utils.image_utils import (
    dict_to_image_url_artifact,
    load_pil_from_url,
    save_pil_image_with_named_filename,
)
from griptape_nodes_library.utils.file_utils import generate_filename


class TwoImageProcessor(SuccessFailureNode):
    """處理兩張影像的節點基底範式。"""

    CATEGORY: ClassVar[str] = "image"

    def __init__(self, name: str, metadata: dict[Any, Any] | None = None) -> None:
        super().__init__(name, metadata)

        # 第一張影像輸入
        self.add_parameter(
            ParameterImage(
                name="image_a",
                tooltip="第一張輸入影像",
                ui_options={"clickable_file_browser": True, "expander": True},
            )
        )

        # 第二張影像輸入
        self.add_parameter(
            ParameterImage(
                name="image_b",
                tooltip="第二張輸入影像",
                ui_options={"clickable_file_browser": True, "expander": True},
            )
        )

        # 輸出影像
        self.add_parameter(
            ParameterImage(
                name="output_image",
                tooltip="處理後的成果",
                allowed_modes={ParameterMode.OUTPUT},
            )
        )

    def _get_image_artifact(self, param_name: str) -> ImageUrlArtifact | None:
        """將參數數值轉換為 ImageUrlArtifact。"""
        value = self.get_parameter_value(param_name)
        if value is None:
            return None
        if isinstance(value, dict):
            return dict_to_image_url_artifact(value)
        return value

    def _process_images(self) -> None:
        """處理兩張影像並設定輸出。"""
        image_a_artifact = self._get_image_artifact("image_a")
        image_b_artifact = self._get_image_artifact("image_b")

        if not image_a_artifact or not image_b_artifact:
            return

        # 載入為 PIL 影像
        pil_a = load_pil_from_url(image_a_artifact.value)
        pil_b = load_pil_from_url(image_b_artifact.value)

        # 執行影像運算處理（於子類別中覆寫）
        result_pil = self._do_processing(pil_a, pil_b)

        # 儲存運算成果
        filename = generate_filename(self.name, suffix="processed")
        output_artifact = save_pil_image_with_named_filename(result_pil, filename)
        self.parameter_output_values["output_image"] = output_artifact

    def _do_processing(self, image_a: Image.Image, image_b: Image.Image) -> Image.Image:
        """請覆寫此方法以實作具體的運算邏輯。"""
        raise NotImplementedError

    def after_value_set(self, parameter: Parameter, value: Any) -> None:
        """當兩張影像皆就緒時觸發即時預覽。"""
        if parameter.name in ("image_a", "image_b"):
            image_a = self.get_parameter_value("image_a")
            image_b = self.get_parameter_value("image_b")
            if image_a and image_b:
                self._process_images()

        return super().after_value_set(parameter, value)

    def process(self) -> None:
        """主要運算進入點。"""
        self._process_images()
```

**使用的核心工具函式**：

- `dict_to_image_url_artifact()`：將字典結構轉為產物物件
- `load_pil_from_url()`：從 URL 載入 PIL 影像（包含 localhost）
- `save_pil_image_with_named_filename()`：儲存 PIL 影像並回傳產物
- `generate_filename()`：根據節點名稱建立具備一致性的檔名

