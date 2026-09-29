# Save EXR {: #save-exr }

**將 8-bit 一般影像或 EXR 通道產物儲存為單一部件 (Single-part) EXR 檔案，並在通道模式下完美保留高精確度浮點數資料。**

類別目錄：`EXR`

## 核心速覽 {: #tldr }

- 支援兩種寫入模式：**模式 A (Mode A)** 接收 8-bit 的 `ImageArtifact`/`ImageUrlArtifact`，自動將 [0, 255] 整數正規化為 [0.0, 1.0] 浮點數；**模式 B (Mode B)** 接收至多四個 `EXRChannelArtifact` 通道插槽，並 100% 完整保留原始浮點數精確度。
- 模式 B 具備絕對優先權：只要有任何一個通道插槽連線，`image_in` 輸入端將被完全忽略。
- 輸出端包含剛寫入部件的 `EXRPartArtifact` 描述符，方便您直接接駁至 [Display EXR Part](display_exr_part.md) 進行即時驗證。
- 預設摺疊的 **Metadata（中繼資料）** 群組提供豐富的檔頭欄位自訂（資產所有者、時間碼、註記、自訂 JSON 屬性等）。

## 典型工作流程架構 {: #typical-workflow-position }

```text
(影像節點) ─────────→ [Save EXR] → Display EXR Part / Load EXR
Load EXR (通道提取) ─↗
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/save-exr.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `image_in` | `ImageArtifact \| ImageUrlArtifact` | 否 | 8-bit 影像來源（模式 A）。只要任何通道插槽已連線，此埠將被自動忽略。 |
| `channel_r` | `EXRChannelArtifact` | 否 | 寫入為紅色 R 的通道（模式 B）。 |
| `channel_g` | `EXRChannelArtifact` | 否 | 寫入為綠色 G 的通道（模式 B）。 |
| `channel_b` | `EXRChannelArtifact` | 否 | 寫入為藍色 B 的通道（模式 B）。 |
| `channel_a` | `EXRChannelArtifact` | 否 | 寫入為透明度 A 的通道（模式 B）。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_file` | `str` | 磁碟中成功寫入之 `.exr` 檔案的本機路徑。 |
| `output_part` | `EXRPartArtifact` | 剛寫入部件的結構描述符——可直接接駁至 [Display EXR Part](display_exr_part.md) 進行預覽。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `compression` | choice (`ZIP \| ZIPS \| PIZ \| DWAA \| NONE`) | `ZIP` | 寫入檔案所套用的壓縮編解碼器演算法。 |
| `pixel_type` | choice (`HALF \| FLOAT`) | `HALF` | `HALF` 為 16-bit 半精度浮點數（影視工業標準）；`FLOAT` 為 32-bit 單精度全浮點數。 |

### 中繼資料欄位 *(預設處於摺疊狀態)* {: #metadata }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `part_name` | `str` | `""` | 此 EXR 部件的自訂名稱。 |
| `pixel_aspect_ratio` | `float` | `1.0` | 像素寬高比。 |
| `owner` | `str` | `""` | 數位資產所有者/創作者。 |
| `comments` | `str` | `""` | 自由文字註記。 |
| `capture_date` | `str` | `""` | 拍攝或渲染日期（例如 `2025-01-01T12:00:00`）。 |
| `software` | `str` | `""` | 生成該檔案的軟體名稱。 |
| `time_code` | `str` | `""` | 影音剪輯時間碼 (`HH:MM:SS:FF`)。 |
| `custom_attributes` | `str` | `""` | 非標準的自訂檔頭中繼資料屬性（格式為 JSON 物件）。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **模式 B 具備排他性優先權**：若您打算儲存 `image_in` 影像，但通道插槽上殘留有連線，影像輸入端將被完全忽略——請中斷通道連線以切換回影像模式。
- **8-bit 來源不會自動變成真正的 HDR**：模式 A 僅將常規 8-bit 數值映射至 [0.0, 1.0] 區間；無法憑空還原被高光截斷的動態範圍。若需真正的 HDR，請使用模式 B 傳入浮點數通道。
- **針對大型 Beauty 算圖推薦選用 `DWAA`**：DWAA 為感知失真壓縮，能大幅壓縮檔案體積；而對於深度圖 (Depth)、法線貼圖 (Normals) 等關鍵資料通道，請務必維持 `ZIP` 或 `PIZ` 無損壓縮。
- **`FLOAT` 檔案體積為 `HALF` 的兩倍**：僅在極度需要高精度階調（例如深度圖或世界座標 Position Pass）時才選用 32-bit `FLOAT`。

## 相關延伸閱讀 {: #see-also }

- [Load EXR](load_exr.md) —— 提供通道輸入來源或重新讀取已寫入的 EXR。
- [Display EXR Part](display_exr_part.md) —— 透過 `output_part` 即刻檢視剛儲存的算圖部件。
