# Configure ControlNet {: #configure-controlnet }

**配置單個 ControlNet 控制條目——包含控制模型、引導參考影像與條件約束權重，以供 ControlNet Pipeline 節點調用。**

類別目錄：`ModularDiffusion/Conditioning`

## 核心速覽 {: #tldr }

- 請務必先選定 `provider`（供應商）；下方其餘參數皆為**動態響應參數**，會隨供應商架構重新生成。
- 遵循「一個節點 = 一個 ControlNet」原則。若需疊加多個控制條件，可新增多個本節點，並將各自的 `control_net` 輸出端共同連線至 [ControlNet Pipeline](controlnet_pipeline.md) 節點的 `control_nets` 清單輸入埠。
- 支援供應商架構：**Flux、Qwen、Stable Diffusion、Z-Image** 等。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Load Image → [Configure ControlNet] → ControlNet Pipeline → Generate Media Latents
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/configure-controlnet.png" alt="Configure ControlNet" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `control_image` | `ImageArtifact` / `ImageUrlArtifact` | 是 | 控制訊號影像（Canny 邊緣、深度圖、人體姿態骨架等——取決於選用的 ControlNet 模型）。 |
| `controlnet_conditioning_scale` | float (0.0–1.0) | 否 | 此 ControlNet 的引導約束強度，預設值為 `1.0`。 |
| `controlnet_model` | HF repo picker | 是 | 欲使用的 ControlNet 模型權重存放庫；選項隨供應商即時更新。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `control_net` | `control_net` (dict) | 封裝好的結構化控制配置條目。接駁傳遞至 ControlNet Pipeline 節點。 |

## 供應商與模型特定行為 {: #provider-model-behavior }

模型下拉選單與附加專屬參數會隨選取的供應商動態變化。重點特化項目：

- **Flux**：提供 `control_mode`（控制模式：`canny` 邊緣、`tile` 分塊細節、`depth` 深度、`blur` 模糊、`pose` 姿態、`gray` 灰階、`low_quality` 低畫質修復）、`control_guidance_start`（控制介入起始步數比例）與 `control_guidance_end`（控制介入結束步數比例）。
- **Qwen / Stable Diffusion / Z-Image**：各自具備對應專屬的模型存放庫與條件微調旋鈕。

## 技巧與避坑指南 {: #tips-pitfalls }

- **供應商必須與基礎管線嚴格匹配**：若將 Flux ControlNet 接駁至 SDXL 管線中，將在下游 ControlNet Pipeline 節點的校驗階段被直接拒絕。
- **請預先在上游預處理控制影像**：此節點接收的是已提取特徵的控制訊號——請先在上游節點中完成邊緣檢測、深度估計或骨架識別，再將成品圖傳入本節點。
- **獨立控制強度權重**：每個 ControlNet 具備各自獨立的 `controlnet_conditioning_scale`，在多條件疊加時可精確權衡不同控制訊號的影響比重。

## 相關延伸閱讀 {: #see-also }

- [ControlNet Pipeline](controlnet_pipeline.md) —— 必備的下游管線封裝節點。
- 工作流程範本：`workflows/templates/ControlnetText2Image.py`。
