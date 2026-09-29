# Display EXR Channel {: #display-exr-channel }

**將 1 至 4 個獨立的 EXR 色彩通道組裝合併為 8-bit sRGB 或 RGBA PNG 影像，並支援在自訂背景底圖上執行 Alpha 混合疊加。**

類別目錄：`EXR`

## 核心速覽 {: #tldr }

- 每個通道插槽（R、G、B、A）皆接收 `EXRChannelArtifact` 產物且皆為選填項——通道可來自不同的 EXR 檔案或不同的內部部件 (Parts)。
- 未連線的 RGB 插槽將自動以數值 0 全黑填充；連線 A 插槽則產生帶有透明度的 RGBA 輸出，並支援與背景圖進行合成。
- 運算時**至少必須連線一個 RGB 通道插槽**，且所有已連線通道的像素解析度必須嚴格一致。
- 具備與 [Display EXR Part](display_exr_part.md) 相同的專業色彩管理能力：支援 `basic` 色調映射或透過 `OCIOColorParamsArtifact` 執行 `ocio` 映射。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Load EXR (提取通道) → [Display EXR Channel] → (影像輸出)
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/display-exr-channel.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `channel_r` | `EXRChannelArtifact` | 否 | 映射至紅色平面 (Red Plane) 的通道。 |
| `channel_g` | `EXRChannelArtifact` | 否 | 映射至綠色平面 (Green Plane) 的通道。 |
| `channel_b` | `EXRChannelArtifact` | 否 | 映射至藍色平面 (Blue Plane) 的通道。 |
| `channel_a` | `EXRChannelArtifact` | 否 | 作為 Alpha 透明度的通道。連線時輸出產物為 RGBA。 |
| `exposure` | `float` | 否 | 曝光補償（以 EV 曝光值為單位）。範圍 -10 至 +10，預設值為 `0.0`。 |
| `color_params` | `OCIOColorParamsArtifact` | 否 | 來自 [OCIO Color Parameters](../opencolorio/ocio_color_parameters.md)。當 `color_mode` 設為 `ocio` 時必填。 |
| `background` | `ImageArtifact \| ImageUrlArtifact \| str` | 否 | 用於執行 A-over-B 底圖合成的背景影像。若未連線 Alpha 通道則自動忽略。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `image` | `ImageUrlArtifact` | 供畫布檢視或後續處理引用的 8-bit sRGB 或 RGBA PNG 實體影像。 |
| `output_file` | `str` | 磁碟中保存之 PNG 檔案的本機路徑。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `color_mode` | choice (`basic` \| `ocio`) | 若安裝了 OpenColorIO 則預設為 `ocio`，否則為 `basic` | `basic` 採用內建色調映射；`ocio` 調用連線的 `OCIOColorParamsArtifact` 色彩轉換。 |
| `tone_mapping` | choice (`filmic` \| `linear`) | `filmic` | 當 `color_mode` 為 `basic` 時套用的本機色調映射。`filmic` 採用 Narkowicz 2015 擬真膠片曲線；`linear` 則截斷至 [0, 1]。在 `ocio` 模式下自動隱藏。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **至少連線一個 RGB 色彩通道**：若僅連接 Alpha 通道而未連線任何 R/G/B 插槽，節點將報錯拒絕執行。
- **所有通道解析度必須完全相同**：混用不同尺寸規格的通道會引發張量維度不對等錯誤——請預先在上游進行尺寸統一。
- **檢視 AOV 渲染通道的利器**：將單一 AOV 通道（如深度圖 `depth` 或法線向量 `normal.x`）連入 `channel_r`，即可在畫布上以紅黑灰階視覺化審查其數值；亦可將 `normal.x/y/z` 分別連入 R、G、B 組裝為完整法線貼圖。
- **背景合成必須依賴 Alpha 通道**：若未在 `channel_a` 連入有效遮罩通道，`background` 輸入端將被完全忽略。

## 相關延伸閱讀 {: #see-also }

- [Load EXR](load_exr.md) —— 提供各通道輸入端來源。
- [Display EXR Part](display_exr_part.md) —— 自動選取全通道渲染單一部件。
- [Save EXR](save_exr.md) —— 將重新組裝的通道資料寫回 EXR 檔案。
