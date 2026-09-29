# Subtract Latents {: #subtract-latents }

**對兩個潛在空間張量執行逐元素減法 (Elementwise Difference) 運算。**

類別目錄：`ModularDiffusion/Transform`

## 核心速覽 {: #tldr }

- 數學邏輯：`output = left_latent - right_latent`（逐元素相減）。
- 兩個輸入端張量必須擁有完全一致的形狀維度與潛在空間規格（即源自相同模型架構的管線）。
- 常見應用場景：分離殘留噪訊（去噪後張量 − 原始帶噪張量）、潛在空間消融分析 (Ablation Experiments) 與特徵向量差值運算。

## 典型工作流程架構 {: #typical-workflow-position }

```text
潛在張量 A ─┐
            ├─→ [Subtract Latents] → 潛在張量數學運算 / Generate
潛在張量 B ─┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/subtract-latents.png" alt="Subtract Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `left_latent` | `LatentArtifact` | 是 | 左側被減數潛在張量。 |
| `right_latent` | `LatentArtifact` | 是 | 右側減數潛在張量，形狀維度必須與 `left_latent` 完全相同。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 相減後的全新潛在張量 (`left − right`)。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **運算順序極其關鍵**：減法不具備交換律；`a − b` 與 `b − a` 結果完全相反。
- **兩者形狀維度必須完全一致**：運算前請確認兩者維度規格嚴格對齊。

## 相關延伸閱讀 {: #see-also }

- [Add Latents](add_latents.md) · [Multiply Latents](multiply_latents.md) —— 同屬 Transform 類別的潛在空間代數運算節點。
