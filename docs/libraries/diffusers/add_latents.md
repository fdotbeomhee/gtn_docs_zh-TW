# Add Latents {: #add-latents }

**對兩個潛在空間張量執行逐元素加法 (Elementwise Sum) 運算。**

類別目錄：`ModularDiffusion/Transform`

## 核心速覽 {: #tldr }

- 數學邏輯：`output = left_latent + right_latent`（逐元素相加）。
- 兩個輸入端張量必須擁有完全一致的形狀維度與潛在空間規格（即源自相同模型架構的管線）。
- 常見應用場景：融合混合兩個已去噪的潛在張量、精準注入受控噪訊、在多階段生成間添加殘差 (Residuals)。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Media Latents (分支 A) ─┐
                                 ├─→ [Add Latents] → Generate / Decode
Generate Media Latents (分支 B) ─┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/add-latents.png" alt="Add Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `left_latent` | `LatentArtifact` | 是 | 左側被加數潛在張量。 |
| `right_latent` | `LatentArtifact` | 是 | 右側加數潛在張量，形狀維度必須與 `left_latent` 完全相同。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 相加後的全新潛在張量 (`left + right`)。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **兩者形狀維度必須完全一致**：若解析度或畫格數不對等，請先透過上採樣或幾何適配節點調整其中一個張量。
- **嚴格維持在相同的模型家族內運算**：每個擴散模型採用的 VAE 與潛在空間流形各不相同——跨模型混用張量（例如將 Flux 與 SDXL 的張量相加）將產生完全混亂失真的無意義數值。

## 相關延伸閱讀 {: #see-also }

- [Subtract Latents](subtract_latents.md) · [Multiply Latents](multiply_latents.md) · [Latents Composite Mask](latents_composite_mask.md) —— 同屬 Transform 類別的其他潛在空間數學與合成節點。
