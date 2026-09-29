# Latents Composite Mask {: #latents-composite-mask }

**依據指定的像素偏移座標，將來源潛在張量貼合至目標底圖潛在張量上，並支援透過可選的影像遮罩進行平滑羽化混合。**

類別目錄：`ModularDiffusion/Transform`

## 核心速覽 {: #tldr }

- 依照座標 `(x_offset, y_offset)` 將來源潛在張量注入目標底層張量中。遮罩中的**白色像素 → 取來源張量**；**黑色像素 → 保留目標底圖張量**。
- 偏移量以**像素空間 (Pixel-space)** 為單位設定；節點內部會自動除以 VAE 縮放因子 (÷ 8) 轉換為潛在空間座標。
- 開啟 `resize_source` 可在合成前，自動將來源張量雙線性縮放至與目標畫布完全一致的空間尺寸。
- 遮罩影像會自動重採樣以匹配來源潛在張量解析度。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Latents (目標底圖) ──┐
Generate Latents (來源前景) ──┼─→ [Latents Composite Mask] → Generate Media Latents
Paint Mask (繪製遮罩) ────────┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/latents-composite-mask.png" alt="Latents Composite Mask" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `destination_latent` | `LatentArtifact` | 是 | 作為背景底圖的基準潛在張量。 |
| `source_latent` | `LatentArtifact` | 是 | 欲貼合覆蓋的來源前景潛在張量。 |
| `mask_image` | `ImageArtifact` / `ImageUrlArtifact` | 否 | 混合權重遮罩。若留空不連線，來源張量將在定位區域內直接整塊覆蓋底圖。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 完成來源內容合成後的目標潛在張量。 |

## 參數說明 {: #parameters }

### 定位排版 (Placement) {: #placement }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `x_offset` | int (像素，0–2000) | `0` | 水平像素偏移量（內部自動除以 8）。 |
| `y_offset` | int (像素，0–2000) | `0` | 垂直像素偏移量（內部自動除以 8）。 |
| `resize_source` | bool | `False` | 合成前是否使用雙線性插值將來源張量尺寸縮放對齊至目標畫布。 |

### 遮罩進階選項 *(預設處於摺疊狀態)* {: #mask-options }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `channel` | choice | `alpha` | 選取 `mask_image` 的哪個色彩通道作為混合權重（支援 alpha、red、green、blue、luminance）。 |
| `invert_mask` | bool | `False` | 反轉遮罩：黑白對調，反轉來源與目標的替換區域。 |
| `grow_shrink` | float (-100..100) | `0` | 遮罩邊緣膨脹 (+) 或侵蝕 (-)。 |
| `blur_mask` | float (0..100) | `0` | 遮罩邊界的高斯模糊羽化半徑。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **僅對幾何重疊區域執行合成運算**：若 `x_offset` + 來源寬度超出了目標畫布邊界，超出部分將被自動截斷捨棄；若來源與目標完全無交集重疊，則直接原樣傳回目標張量。
- **偏移量使用直觀的像素單位**：例如 1024 像素的目標畫布對應 128 個潛在空間單元。參數請直接依照一般螢幕像素填寫，節點內部會精準轉換。
- **全面支援 4D 影像張量與 5D 視訊張量**：對於視訊潛在張量，遮罩會自動在整個時間維度 (Temporal Dimension) 上進行廣播對齊。
- **輸出規格繼承規則**：輸出張量嚴格繼承目標底圖的 `shape` 形狀維度。中繼資料將進行合併；當鍵名衝突時以目標張量為準。

## 相關延伸閱讀 {: #see-also }

- [Create Empty Latents](empty_latents.md) —— 經典的空白全零畫布來源。
- [Add Latents](add_latents.md) —— 用於無遮罩的純代數元素疊加。
