# Save Latent Tensor {: #save-latent-tensor }

**將 `LatentArtifact` 潛在張量作為 PyTorch `.pt` 檔案持久化寫入磁碟。常用於張量數值偵錯、跨流程傳遞或建立離線測試基準資料 (Test Fixtures)。**

類別目錄：`ModularDiffusion/IO`

## 核心速覽 {: #tldr }

- 儲存的是底層原始的潛在張量矩陣——絕非解碼後的視覺影像。若需重新引入工作流程，可使用 `torch.load(...)` 讀取 `.pt` 檔案並封裝為 `LatentArtifact`（或在自訂節點程式碼中直接調用）。
- 輸出埠（`saved_path`）會回報解析後的本機絕對路徑，方便您接駁至日誌記錄或中繼資料歸檔節點。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Media Latents → [Save Latent Tensor]
                       └→ Decode Media Latent → Save Image
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add docs/assets/nodes/save-latent-tensor.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `latent_tensor` | `LatentArtifact` | 是 | 欲儲存的目標潛在張量。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `saved_path` | str | 張量檔案成功寫入磁碟的絕對路徑。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `file_path` | str | `debug/latent.pt` | 輸出目標路徑。相對路徑以工作區目錄為基準解析；父層目錄不存在時會自動建立。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **自動分離並卸載至 CPU**：張量在寫入磁碟前會自動調用 `.detach().cpu()`——可確保在任何硬體裝置環境中安全重載。
- **純原始張量儲存**：檔案僅保存原始 Tensor 權重矩陣，不包含額外中繼資料。若後續解碼需要依賴特定的 `source_shape`，請自行額外記錄。
- **`.pt` 檔案體積考量**：一張 1024×1024 的 SDXL 潛在張量約 4 MB；而高畫質長時長 LTX 視訊潛在張量檔案可高達數百 MB。

## 相關延伸閱讀 {: #see-also }

- [Generate Media Latents](generate_media_latents.md) —— 典型產生待儲存張量的核心上游節點。
