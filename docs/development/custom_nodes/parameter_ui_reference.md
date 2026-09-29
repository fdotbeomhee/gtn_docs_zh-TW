# 參數 UI 參考指南 (Parameter UI Reference) {: #parameter-ui-reference }

本頁面詳細介紹了您在 Python 中於 `Parameter` 上宣告的設定如何映射至編輯器介面所渲染的視覺部件。共有三項要素共同決定了參數的 UI 呈現：

1. **參數的 `type`** 決定選用何種基礎部件：`str` 呈現單行文字框、`bool` 呈現開關切換鈕、`ImageUrlArtifact` 呈現專用影像檢視器，以此類推。
1. **`ui_options`** 字典對該部件進行樣式微調：將其完全隱藏、拉伸至全寬、將文字框改為多行編輯區域、附加即時視訊鏡頭擷取按鈕等。
1. **特徵標籤 (Traits)** 將 UI 樣式與行為邏輯綑綁封裝：例如 `Slider` 渲染數值滑桿*同時*強制驗證範圍邊界，`Options` 渲染下拉選單*同時*約束數值必須為選項之一。在底層，特徵標籤會自動將相應的設定鍵寫入 `ui_options` 中——使用特徵標籤是配置這些設定鍵的官方推薦正規途徑。

若有現成的輔助工具，請優先採用 [參數輔助包裝類別](parameters.md#parameter-helper-constructs-parameterstring-parameterint)（`ParameterString`、`ParameterImage` 等）與特徵標籤；僅在需要本頁面列出的樣式微調時才直接設定原始 `ui_options` 鍵值。

!!! warning "未收錄於文件的設定鍵屬於內部保留私有 API"

    編輯器內部讀取的 `ui_options` 鍵值遠多於本頁面所列出的項目。任何未在此處記載（或非由特徵標籤自動釋出）的設定鍵皆屬編輯器內部實作細節，未來可能在不另行通知的情況下修改或移除。

## 部件選取判定順序 {: #how-the-widget-is-chosen }

針對每個參數，編輯器依循以下優先順序挑選適當的視覺部件：

1. 若 `ui_options` 攜帶 `widget` 與 `library`（由 [`Widget` 特徵標籤](#traits) 指派），編輯器會自該程式庫載入自訂前端部件。
1. 否則，依據參數的 `type` 從下方表格挑選內建部件。宣告為 `list[...]` 的型別（無論何種元素型別）皆選用清單部件。
1. 若某個型別不存在任何映射規則，則完全不呈現行內部件——該參數在 UI 上僅顯示文字標籤與拓撲連線連接埠，但無法在畫布介面直接編輯。

## 型別至部件對應表 (Type-to-Widget Mapping) {: #type-to-widget-mapping }

| `type` | 視覺部件 (Widget) |
| :--- | :--- |
| `str` | 單行文字輸入框（透過下述 `ui_options` 與特徵標籤可變更為多行文字框、Markdown 編輯器或選取器） |
| `int`, `float` | 數值輸入框（可透過 `Slider` 特徵標籤變更為滑桿；透過 `step` 調整步進刻度） |
| `bool` | 開關切換鈕 (Toggle switch) |
| `json`, `JsonArtifact` | JSON 檢視器 / 結構編輯器 |
| `python`, `yaml` | 語法高亮程式碼編輯器 |
| `html`, `xml` | HTML/XML 專用程式碼編輯器 |
| `dict` | 鍵值對 (Key-Value) 編輯器；亦支援渲染為影像/影片對比滑桿（詳見 [dict 選項](#dict)） |
| `list`, `list[...]` | 清單編輯器，為列表中的每個元素渲染對應的子部件 |
| `button` | 可點擊按鈕（透過 `Button` 特徵標籤自訂） |
| `Status` | 狀態 / 訊息提示區塊 |
| `UrlArtifact` | 網址 URL 連結展示 |
| `ImageUrlArtifact` / `ImageArtifact`, `VideoUrlArtifact` / `VideoArtifact`, `AudioUrlArtifact` / `AudioArtifact`, `ThreeDUrlArtifact` / `ThreeDArtifact`, `SplatUrlArtifact` / `SplatArtifact` | 專用多媒體檢視器與編輯器——功能說明請參閱 [媒體檢視器與編輯器指南](../../guides/editor/media_editors.md) |
| 其他任何型別 | 無行內部件；僅呈現名稱標籤與拓撲連線連接埠 |

舊有的 `GLTFArtifact` / `GLTFUrlArtifact` 為了向下相容性仍會被渲染為 3D 檢視器；在新開發的節點中請一律使用 `ThreeD` 系列型別。

## 通用 `ui_options` 選項（適用所有型別） {: #common-ui-options-any-type }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `hide` | 完全隱藏該參數（包含標籤、行內部件與連線端點）。 |
| `hide_label` | 隱藏參數名稱標籤，但保留行內部件。 |
| `hide_property` | 隱藏行內部件，但保留名稱標籤與連線端點。 |
| `display_name` | 顯示於 UI 上的自訂標籤文字（當欲與 Python 參數名不同時使用）。 |
| `is_full_width` | 將部件水平拉伸至佔滿節點面板的全寬。 |
| `parameter_render_location` | 參數相對於同層相鄰參數的渲染垂直位置：`"top"`（頂部）、`"in-order"`（預設依照宣告順序）或 `"bottom"`（底部）。 |

## 依型別劃分的 `ui_options` {: #per-type-ui-options }

### `str` 字串型別 {: #str }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `multiline` | 渲染為多行文字文字區域 (Textarea) 取代單行文字框。 |
| `markdown` | 以 Markdown 格式即時預覽與編輯文字內容。 |
| `placeholder_text` | 欄位為空時呈現的淡色提示佔位字串（程式碼編輯器亦適用）。 |

### `int` / `float` 數值型別 {: #int-float }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `step` | 數值微調按鈕 (Stepper) 的步進增量；引擎亦會依此校驗數值是否合規。 |
| `progress_bar` | 將數值渲染為進度條而非可編輯輸入框（適用於節點回報的進度值，如 0–100）。 |

若需帶有範圍邊界的滑桿，請使用 `Slider` 特徵標籤，切勿手動撰寫 `ui_options["slider"]`——特徵標籤會在底層自動驗證數值範圍。
當範圍僅作為滑桿軌道的比例參考時，可傳入 `soft_limits=True`，這樣使用者手動鍵入超出軌道的值依然能被合法接受。

### 影像型別 (Image types) {: #image-types }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `clickable_file_browser` | 點擊空白參數區域可開啟檔案瀏覽器；拖放或選取的圖檔會自動上傳。 |
| `expander` | 允許檢視器面板展開或折疊。 |
| `crop` / `crop_image` | 顯示裁切按鈕，點擊開啟內建影像裁切編輯器。 |
| `edit_mask` | 顯示遮罩按鈕，點擊開啟 Paint Mask 塗鴉遮罩編輯器。 |
| `edit_excalidraw` | 顯示編輯按鈕，點擊啟動 Image Bash 白板工具。 |
| `webcam_capture_image` | 將縮圖替換為即時視訊鏡頭畫面與拍照按鈕。 |
| `aspect_ratio` | 固定預覽顯示區域的長寬寬高比（例如 `"16:9"`）。 |
| `object_fit` | 影像填滿畫框的方式（CSS object-fit 規範值，如 `"contain"` / `"cover"`）。 |
| `hide_details` | 隱藏縮圖下方的名稱、解析度維度與檔案大小資訊行。 |
| `pulse_on_run` | 節點運算執行時在檢視器上產生發光脈衝動畫（音訊亦適用）。 |

### 音訊型別 (Audio types) {: #audio-types }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `clickable_file_browser` | 支援點擊選取本機檔案並自動上傳，行為與影像完全一致。 |
| `microphone_capture_audio` | 顯示錄音按鈕，支援直接透過硬體麥克風擷取即時語音。 |
| `pulse_on_run` | 節點運算執行時在播放器上呈現動態脈衝效果。 |

### `dict` 字典型別 {: #dict }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `compare` | 將字典（`{"input_image_1": ..., "input_image_2": ...}`）渲染為左右對比滑桿。請搭配 `CompareImagesTrait` 校驗字典結構。 |
| `video_compare` | 將字典渲染為並排左右對照的影片播放器。 |

### `json` 型別 {: #json }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `modal` | 改在彈出強制回應對話框 (Modal) 中開啟 JSON 編輯器，而非行內展開。 |

### `list` 清單型別 {: #list }

| 設定鍵 | 視覺效果與行為 |
| :--- | :--- |
| `collapsed` | 預設以折疊狀態呈現清單內容。 |
| `columns` | 在網格 (Grid) 佈局中以指定的欄數排版清單子項目。 |

`ParameterList` 輔助建構子會自動為您處理常見的清單選項——詳見 [ParameterList 設計範式](parameters.md#parameterlist-pattern)。

## 特徵標籤 (Traits) 規格表 {: #traits }

特徵標籤位於 `griptape_nodes.traits` 模組中，透過 `add_trait()` 或參數建構時的 `traits={...}` 附加至參數上。下表詳述了各特徵標籤所渲染的部件及其管控的 `ui_options` 鍵。請一律透過特徵標籤進行配置，特徵標籤的優先權高於同名儲存鍵。來自節點程式碼、編輯器或存檔的任何 `ui_options` 寫入操作皆會經由 [`state_from_ui_options`](parameters.md#traits) 路由處理。

| 特徵標籤 | 常見適用型別 | 功能描述 | 其所管控的 `ui_options` 設定鍵 |
| :--- | :--- | :--- | :--- |
| `Options` | `str`, 任意型別 | 約束於固定候選項的下拉選單，支援搜尋篩選。設為 `allow_custom=True` 則轉為自動補全輸入框。 | `simple_dropdown`, `show_search`, `search_filter`, `allow_custom` |
| `MultiOptions` | `list` | 支援複選的多選下拉清單。 | `multi_options` |
| `Slider` | `int`, `float` | 介於 `min_val` 與 `max_val` 之間的滑桿。超出範圍的值無法通過校驗，除非宣告 `soft_limits=True`。 | `slider` |
| `Clamp` | `int`, `float`, 序列 | 賦值時自動將數值鉗位鎖定在指定區間內。無獨立前端 UI。 | — |
| `Button` | `button` | 配置按鈕的文字標籤、樣式變體、尺寸及點擊回呼行為。 | `button_label`, `variant`, `size`, `state`, `full_width` |
| `ColorPicker` | `str` | 點擊開啟調色盤的色塊；支援驗證色彩格式（如 `"hex"` 等）。 | `color_picker` |
| `FileSystemPicker` | `str` | 瀏覽按鈕，點擊開啟具備過濾篩選能力的本機檔案/目錄選取對話框。 | `fileSystemPicker` |
| `NumbersSelector` | `dict` | 包含最小值/最大值/步進值的數值範圍選擇器。 | `numbers_selector` |
| `CompareImagesTrait` | `dict` | 驗證影像對比滑桿所使用的雙影像字典結構（搭配 `ui_options["compare"]`）。 | — |
| `Widget` | 任意型別 | 將內建預設部件替換為由外部節點程式庫提供的自訂視覺部件。 | `widget`, `library` |

建構子簽章與使用範例請參閱 [參數參考指南中的特徵標籤章節](parameters.md#traits)。

### 當選項僅作為輸入提示建議時 (Typeahead) {: #when-the-choices-are-only-hints }

預設情況下，`Options` 將 `choices` 視為所有合法值的唯一閉集：參數呈現為標準下拉選單，不在清單中的值會被強制拉回第一個選項，且直接在程式碼賦值時會拋出校驗失敗。

當該清單僅為便民提示而非唯一合法集合時，請傳入 `allow_custom=True`。此時參數會渲染為文字框，並在使用者鍵入時彈出符合項目的自動完成建議，同時完整儲存使用者手動鍵入的任何內容：

```python
Parameter(
    name="model",
    type="str",
    tooltip="挑選建議的模型或自行輸入模型識別碼",
    traits={Options(choices=SUGGESTED_MODELS, allow_custom=True)},
    default_value=SUGGESTED_MODELS[0],
)
```

該旗標會自動卸載轉換器與校驗器，因此在執行階段動態替換 `choices` 清單時，絕不會有使節點已保存的值失效的風險。

當清單無法預知所有合法值時，請採用此模式：例如模型提供商在節點發布後新推出的模型 ID，或使用者自行微調 (Fine-tune) 的私有模型名稱。若未列出的值在執行階段必定引發報錯，則應維持預設下拉選單行為——在前端介面及早拒絕非法輸入，遠比讓節點在流程執行中途拋出崩潰更為優雅。
