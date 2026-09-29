# Latent Upsampler {: #latent-upsampler }

**在完全不脫離潛在空間的前提下，對潛在張量進行空間解析度放大——通常置於兩個 Generate 生成階段之間，用於實現極速且精緻的高解析度細化渲染。**

類別目錄：`ModularDiffusion/Processing`

## 核心速覽 {: #tldr }

- 直接在**潛在空間 (Latent Space)** 中完成超解析度上採樣，完全免除「先解碼為像素影像 → 傳統演算法放大 → 再次編碼回潛在張量」的耗時高損耗往返轉換。
- 透過 `provider` 下拉選單挑選上採樣器模型家族。
- 典型用法為置於兩個 Generate Media Latents 節點之間：低解析度快速生成基礎結構 → 潛在放大 → 高解析度細節細化去噪。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Media Latents (低解析度) → [Latent Upsampler] → Generate Media Latents (細化重繪) → Decode
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/latent-upsampler.png" alt="Latent Upsampler" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `input_latent` | `LatentArtifact` | 是 | 欲放大的潛在張量。必須處於管線標準潛在空間維度中（任何 Generate 或 Encode 節點的輸出皆可相容）。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 空間幾何尺寸完成放大的全新潛在張量。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `provider` | choice | 上採樣器模型家族（例如 `LTX2`）。切換此項會自動重置下方的模型選取器。 |
| `upsampler_model` | HF repo picker | 用於放大的神經網路權重 Hugging Face 存放庫 ID。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **潛在空間放大模型具備高度模型相依性**：LTX2 專用放大器無法處理 SDXL 或 Flux 的潛在張量。請務必確保上採樣器家族與上游生成管線嚴格匹配。
- **放大後強烈建議銜接二次細化生成**：上採樣器純粹負責提高張量維度解析度，並不負責去噪修潤——通常應在放大後接續一個步數較短（低去噪強度）的 Generate Media Latents 節點以補充豐富微觀細節。

## 相關延伸閱讀 {: #see-also }

- [Generate Media Latents](generate_media_latents.md) —— 分別置於本節點上游（初生成）與下游（細節細化）的核心去噪節點。
- [Decode Media Latent](decode_media_latent.md) —— 工作流程末端將放大後的張量解碼為實體畫面的最終節點。
