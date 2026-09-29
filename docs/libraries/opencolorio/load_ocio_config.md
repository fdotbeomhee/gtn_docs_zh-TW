# Load OCIO Config {: #load-ocio-config }

**從 `$OCIO` 系統環境變數或明確指定的覆寫路徑中載入 OpenColorIO 設定檔，並輸出供下游節點引用的 `OCIOConfigArtifact` 產物。**

類別目錄：`Colorspace`

## 核心速覽 {: #tldr }

- 若作業系統環境中已設定 `$OCIO`，本節點會自動辨識並載入——只需將節點拖曳入畫布即可直接執行。
- 每次節點運算時皆會即時重新讀取 `$OCIO` 變數，因此能完美尊重 Griptape 專案級環境變數設定（`project.yml` 中的 `environment:` 區塊）。
- 輸出型別為 `OCIOConfigArtifact`——請接駁至 [OCIO Color Parameters](ocio_color_parameters.md) 或任何接收 `OCIO Config` 輸入埠的節點。
- 預設摺疊的 **Advanced（進階）** 參數群組允許您指定明確的 `.ocio` 檔案路徑以覆寫 `$OCIO`，非常適合用於單元測試。

## 典型工作流程架構 {: #typical-workflow-position }

```text
[Load OCIO Config] → OCIO Color Parameters → (下游色彩變換 / 視覺化檢視節點)
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/load-ocio-config.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `context_vars` | `dict` | 否 | OCIO 上下文環境變數（例如 `{"SHOT": "sh010", "SEQ": "sq020"}`）。將封裝於輸出產物中向下傳遞。 |
| `file_path` | `str` | 否 | 指向本機 `.ocio` 設定檔的實體路徑。僅在啟用 **Override OCIO Config** 時生效。內建檔案挑選器。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `config` | `OCIOConfigArtifact` | 封裝了已解析的設定檔路徑與上下文環境變數。 |
| `was_successful` | `bool` | 設定檔是否成功解析載入的布林標誌。 |
| `result_details` | `str` | 載入結果的詳細狀態日誌（摺疊於狀態群組中）。 |

## 參數說明 {: #parameters }

### 進階選項 *(預設處於摺疊狀態)* {: #advanced }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `override_ocio_config` | bool | `False` | 啟用時，將強制改用下方指定的 `file_path` 實體路徑，取代 `$OCIO` 環境變數。適合在不改動系統全域變數的前提下測試特定配置。 |
| `file_path` | file path | `""` | 當開啟覆寫開關時顯示的檔案路徑輸入框。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **注意介面內嵌訊息**：節點面板會主動提示偵測到的 `$OCIO` 路徑資訊；當缺少 `$OCIO` 時會呈現警告；當 `$OCIO` 在多次執行間發生異動時亦會發出提醒（重新執行節點即可同步刷新下游連線數據）。
- **覆寫模式會完全取代 `$OCIO`**：當勾選 **Override OCIO Config** 時，系統環境變數將被全盤忽略；取消勾選即可還原為環境變數優先模式。
- **無 `$OCIO` 且無覆寫路徑時節點將執行失敗**：兩者必須擇一提供；推薦在專案的 `project.yml` 中透過 `environment:` 定義以維持乾淨的專案隔離。

## 相關延伸閱讀 {: #see-also }

- [OCIO Color Parameters](ocio_color_parameters.md) —— 本節點 `config` 產物的最主要下游接收端。
