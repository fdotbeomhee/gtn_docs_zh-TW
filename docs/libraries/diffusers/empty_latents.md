# Create Empty Latents {: #create-empty-latents }

**生成契合連線管線維度形狀的全零填充 (Zero-filled) 空白潛在張量。**

類別目錄：`ModularDiffusion/Create`

## 核心速覽 {: #tldr }

- 維度尺寸推導邏輯與 [Create Noise Latents](create_noise_latents.md) 完全一致，但產出的張量數值全部為 0（無需亂數種子）。
- 常用作 [Latents Composite Mask](latents_composite_mask.md) 遮罩合成的**目標畫布 (Destination)**，或作為希望在下游完全精確控制噪訊注入的自訂起點。
- 對於常規的純文字生圖或文字生視訊任務，絕大多數情境請選用 [Create Noise Latents](create_noise_latents.md)。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder → [Create Empty Latents] → Latents Composite Mask → Generate Media Latents
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/create-empty-latents.png" alt="Create Empty Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 決定潛在空間張量形狀以及是否動態顯示 `num_frames` 參數。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 處於管線標準潛在空間維度中的全零填充空白張量。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `width` | int (像素) | `1024` | 像素空間的目標寬度。 |
| `height` | int (像素) | `1024` | 像素空間的目標高度。 |
| `num_frames` | int | `41` | 視訊總畫格數。**在純影像管線下自動隱藏**。 |

本節點無 `seed` 亂數種子參數——輸出產物在數學定義上完全確定。

## 技巧與避坑指南 {: #tips-pitfalls }

- **生成前必須注入噪訊**：純全零張量在執行擴散去噪前必須擁有噪訊基底——請在 [Generate Media Latents](generate_media_latents.md) 節點中開啟 `add_noise=True`，或預先透過張量數學運算/遮罩合成注入噪訊。若無噪訊，擴散模型將缺乏可去噪的特徵起點。

## 相關延伸閱讀 {: #see-also }

- [Create Noise Latents](create_noise_latents.md) —— 常規文字生圖/文字生視訊任務標準使用的起始噪訊張量。
- [Latents Composite Mask](latents_composite_mask.md) —— 接收本節點全零張量作為基底畫布的主要合成節點。
