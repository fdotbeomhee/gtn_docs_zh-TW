# Load EXR {: #load-exr }

**載入 OpenEXR 檔案並將其內部完整階層結構——包含各部件 (Parts)、色彩通道 (Channels) 與檔頭中繼資料屬性——發布為具備型別的輸出端，預設完全不耗時讀取像素陣列。**

類別目錄：`EXR`

## 核心速覽 {: #tldr }

- 預設採用純檔頭讀取模式（由 `openexr.header_only` 設定管控），因此即便面對數 GB 的超大電影級多部件算圖檔案，亦能在數毫秒內瞬間掃描完成。
- 自動動態生成 **Parts** 與 **Channels** 輸出群組：每個部件發布一個 `EXRPartArtifact`，每個原始色彩通道發布一個 `EXRChannelArtifact`。
- 可將部件輸出埠接駁至 [Display EXR Part](display_exr_part.md)，或將個別通道接駁至 [Display EXR Channel](display_exr_channel.md) 及 [Save EXR](save_exr.md)。
- 內建 **Open in external viewer** 按鈕，可一鍵在配置的專業 HDR 播放器中開啟該檔案（請參閱 [程式庫設定](index.md#library-settings)）。

## 典型工作流程架構 {: #typical-workflow-position }

```text
[Load EXR] → Display EXR Part → (影像輸出)
           ↘ Display EXR Channel / Save EXR
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/load-exr.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `file_path` | `str` | 是 | 電腦本機 `.exr` 檔案的實體路徑。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `image_width` / `image_height` | `int` | 源自資料視窗 (Data Window) 的幾何解析度寬高。 |
| `part_count` / `channel_count` | `int` | 檔案包含的部件總數與通道總數。 |
| `compression` | `str` | 壓縮格式名稱（例如 `ZIP_COMPRESSION`、`DWAB_COMPRESSION`）。 |
| `storage_type` | `str` | 儲存規格：`scanlineimage`、`tiledimage`、`deepscanline` 或 `deeptiled`。 |
| `pixel_aspect_ratio` | `float` | 像素寬高比。 |
| `data_window` / `display_window` | `str` | 幾何視窗座標格式 `"xmin,ymin - xmax,ymax"`。 |
| `time_code` | `str` | SMPTE 時間碼 `HH:MM:SS:FF`（若無則留空）。 |
| `software` | `str` | 生成該檔案的軟體名稱（例如 Maya、Nuke、Blender 等）。 |
| `owner` | `str` | 數位資產所有者資訊。 |
| `chromaticities` | `str` | 包含紅/綠/藍/白場色度座標的 JSON 字串。 |
| `custom_attributes` | `str` | 檔案中所有非標準自訂中繼資料屬性的 JSON 字串。 |
| `parts` | `list[EXRPartArtifact]` | 檔案中所有部件的結構化描述描述符清單。 |

掃描完成後，介面會動態生成以下群組：

- **Parts**：每個部件對應一個 `EXRPartArtifact` 輸出埠，內部包含該部件專屬的各通道（單一部件檔案時此群組自動隱藏，通道直接列於 Channels 群組中）。
- **Channels**：每個原始通道對應一個 `EXRChannelArtifact`，標註有通道名、像素資料型別與採樣率（僅於單一部件檔案時直接呈現）。

## 技巧與避坑指南 {: #tips-pitfalls }

- **純檔頭模式下的像素型別估算**：在純檔頭讀取模式下，部分極特殊通道的像素型別可能為近似預估值。若需要 100% 精準的逐通道像素格式，可將 `openexr.header_only` 設定改為 `false`；代價為掃描時會將實體像素資料讀入記憶體。
- **多部件檔案隱藏扁平 Channels 群組**：對於包含 Beauty、Depth、Cryptomatte 等多層結構的多部件算圖檔，各個通道會歸整至各部件群組內部，避免介面混亂。
- **下游節點採惰性延遲載入 (Lazy Loading)**：檢視與儲存節點會依據描述符產物在運算時才讀取實際像素，因此純檔頭快速掃描完全不限制下游操作。

## 相關延伸閱讀 {: #see-also }

- [Display EXR Part](display_exr_part.md) · [Display EXR Channel](display_exr_channel.md) —— 將部件或通道渲染為可視覺化檢視的影像。
- [Save EXR](save_exr.md) —— 將通道資料重新寫回 EXR 檔案。
