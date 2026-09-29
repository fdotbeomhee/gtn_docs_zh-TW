# OCIO Color Parameters {: #ocio-color-parameters }

**將來源色彩空間 (Source Colorspace)、顯示裝置 (Display) 與檢視視圖 (View) 整合打包為單一可重複複用的 `OCIOColorParamsArtifact` 產物，供下游所有支援色彩管理的節點調用。**

類別目錄：`Colorspace`

## 核心速覽 {: #tldr }

- 將來自 [Load OCIO Config](load_ocio_config.md) 節點的 `config` 產物連線至此節點，介面上的三個下拉選單將會自動從該設定檔中提取可用項目。
- **來源色彩空間 (Source Colorspace)** 下拉清單優先展示角色別名 (Role Aliases，例如 `scene_linear`、`compositing_log`)，確保管線相容性最穩固的名稱排在最頂部。
- 輸出型別為 `OCIOColorParamsArtifact`——單一節點可同時驅動多個下游消費端，實現色彩設定「單點集中配置」。
- 每次執行時皆會對比最新設定檔即時重新校驗；若選項不再匹配會發出介面內嵌警告，但不會阻斷工作流程中斷執行。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Load OCIO Config → [OCIO Color Parameters] → (下游色彩變換 / 視覺化檢視節點)
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/ocio-color-parameters.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `config` | `OCIOConfigArtifact` | 否 | 來自 [Load OCIO Config](load_ocio_config.md)。負責驅動下拉選單自動填寫與合法性校驗。若未連線，選單將顯示佔位文字並輸出空字串。 |
| `source_colorspace` | `str` (choice) | 否 | 來自連線設定檔的來源色彩空間名稱。優先列出 Role 角色別名，隨後展示所有色彩空間全名。 |
| `display` | `str` (choice) | 否 | 來自連線設定檔的目標顯示裝置名稱。 |
| `view` | `str` (choice) | 否 | 選定顯示裝置下的檢視轉換視圖名稱。當切換 `display` 時選項會自動連動更新。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `color_params` | `OCIOColorParamsArtifact` | 封裝了 `source_colorspace`、`display` 與 `view` 的結構化產物。 |
| `was_successful` | `bool` | 參數封裝是否成功輸出的布林標誌。 |
| `result_details` | `str` | 校驗與輸出過程的詳細狀態資訊（摺疊於狀態群組中）。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **優先使用角色別名 (Roles) 作為來源色彩空間**：如 `scene_linear` 等標準角色名稱在設定檔版本迭代中具備極高穩定性；而底層色彩空間具體全名可能在下次設定檔改版時被更名或移除。
- **警告訊息不會阻斷流程執行**：若先前選定的數值在最新設定檔中已不存在，節點仍會原樣輸出產物並提示內嵌警告——在正式輸出渲染前請務必檢查此警告。
- **多下游節點共享單一配置**：將本節點的 `color_params` 接駁至所有下游變換與檢視節點，避免在多個節點上重複配置選單。
- **無連線狀態方便原型佈局**：在設定檔尚未就緒前，您依然可以預先搭建流程拓撲；未連線時節點輸出為安全空字串。

## 相關延伸閱讀 {: #see-also }

- [Load OCIO Config](load_ocio_config.md) —— 提供本節點所需的 `config` 輸入端。
