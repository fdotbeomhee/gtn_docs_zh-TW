# Multiply Latents {: #multiply-latents }

**對兩個潛在空間張量執行逐元素相乘 (Elementwise Product) 運算。**

類別目錄：`ModularDiffusion/Transform`

## 核心速覽 {: #tldr }

- 數學邏輯：`output = left_latent * right_latent`（逐元素相乘）。
- 若目的為依據遮罩合成兩個不同的潛在張量，強烈建議改用 [Latents Composite Mask](latents_composite_mask.md) 節點——它是專為遮罩合成量身打造的專用節點，且會自動為您處理遮罩在像素空間與潛在空間之間的重採樣對齊。

## 典型工作流程架構 {: #typical-workflow-position }

```text
潛在張量 A ─┐
            ├─→ [Multiply Latents] → Add Latents / Generate
潛在張量 B ─┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/multiply-latents.png" alt="Multiply Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `left_latent` | `LatentArtifact` | 是 | 左側被乘數潛在張量。 |
| `right_latent` | `LatentArtifact` | 是 | 右側乘數潛在張量，形狀維度必須與 `left_latent` 完全相同。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 相乘後的全新潛在張量 (`left * right`)。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **純逐元素乘法，非矩陣乘法**：張量 `a` 中的每個位置純粹與張量 `b` 對應位置的數值相乘。
- **最適用於權重縮放**：將潛在張量乘以接近常數的遮罩或係數張量是最可預期的用法；將兩個語意完全不同的任意潛在張量逐元素相乘容易導致數值發散並產生不可預測的視覺崩壞。

## 相關延伸閱讀 {: #see-also }

- [Add Latents](add_latents.md) · [Subtract Latents](subtract_latents.md) · [Latents Composite Mask](latents_composite_mask.md) —— 同屬 Transform 類別的代數與合成節點。
