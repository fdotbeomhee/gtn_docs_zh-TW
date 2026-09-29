# Generate Media Latents (Modular Diffusion Pipeline) {: #generate-media-latents-modular-diffusion-pipeline }

**執行核心擴散去噪迴圈 (Denoising Loop)。接收噪訊、編碼張量或部分去噪的潛在張量作為輸入，經過擴散演算後輸出完全或部分去噪的目標潛在張量。**

類別目錄：`ModularDiffusion/Processing`

## 核心速覽 {: #tldr }

- 模組化擴散生成的核心主力節點。連線 `pipeline` 與 `input_latent`；配置 Prompt 提示詞與步數即可啟動。
- 所有執行期參數（正負提示詞、引導權重、ControlNet 輸入等）皆為**動態響應參數**——會依據連線管線的模型架構自動調適呈現。
- 針對**多階段細化 (Multi-stage) 或二次擴散重繪 (Rediffusion)** 工作流程，可串聯多個 Generate 節點，並運用 `start_step` 與 `end_step` 切割時間步排程。
- 輸出成果為潛在空間張量——請接駁至 [Decode Media Latent](decode_media_latent.md) 解碼還原為最終影像或視訊。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder → Create Noise Latents → [Generate Media Latents] → Decode Media Latent
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/generate-media-latents.png" alt="Generate Media Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 來自 [Pipeline Builder](pipeline_builder.md) 或 [ControlNet Pipeline](controlnet_pipeline.md)。 |
| `input_latent` | `LatentArtifact` 或 `InpaintMaskArtifact` | 是 | 起始潛在張量。可使用噪訊張量（文字生圖/生視訊）、編碼張量（圖生圖/視訊生視訊），或前置 Generate 節點的中間輸出（多階段細化）。 |
| `controlnet_parameters` | `control_parameters` | 僅當 `pipeline` 為 ControlNet 管線時必填 | 來自 ControlNet Pipeline 節點的 `control_parameters` 輸出埠。 |
| `additional_parameters` | list[dict] | 否 | 轉發至底層管線調用的通用模型專屬 kwargs 關鍵字參數清單。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 去噪運算完成後的輸出潛在張量。 |
| `preview_image` | `ImageUrlArtifact` | 即時中間解碼預覽——僅在 **設定 → Modular Diffusion Library → Enable Image Preview Intermediates** 開啟時即時產生。 |
| `progress` | progress bar | 逐步生成進度指示器。 |
| `logs` | str | 記錄每一步去噪耗時與排程的時間戳記日誌。 |

## 參數說明 {: #parameters }

### 生成參數 *(動態參數——依模型架構自適應)* {: #generation-dynamic-provider-specific }

精確的參數清單取決於連線的模型。常見核心參數：

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `prompt` | str | 正向提示詞 (Positive Prompt)。 |
| `negative_prompt` | str | 反向提示詞 (Negative Prompt)——僅在支援無分類器引導 (CFG) 的模型管線下顯示。 |
| `guidance_scale` / `true_cfg_scale` | float | 無分類器引導強度 (CFG Scale)。不同模型的內部命名略有差異。 |
| `num_inference_steps` | int (預設值：`20`) | 擴散去噪排程的總步數。 |
| `seed` | int | 控制去噪過程的隨機亂數種子。 |

### 多階段 / 局部去噪控制 {: #multi-stage-partial-denoise }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `add_noise` | bool | `False` | 去噪前是否先向輸入張量重新注入額外高斯噪訊（適用於圖生圖或二次重繪流程）。 |
| `start_step` | int | `0` | 去噪排程的起始步數索引（以 0 為基準）。 |
| `end_step` | int | `-1` | 去噪排程的結束步數索引；`-1` 代表執行至最後一步。 |

## 供應商與模型特定行為 {: #provider-model-behavior }

- **ControlNet 條件控制**：當傳入的 `pipeline` 屬於 `ControlNetDiffusionPipelineArtifact` 時，介面會自動動態掛載 `controlnet_parameters` 輸入埠。
- **局部重繪 (Inpainting)**：當 `input_latent` 接收到 `InpaintMaskArtifact` 時（來自 [Encode Masked Media Latent](encode_masked_media_latent.md)），節點會自動切換為 Inpaint 管線類別，並採用該產物自帶的重繪強度 `strength`。

## 技巧與避坑指南 {: #tips-pitfalls }

- **切換管線時自動保留同名連線**：更換上游管線時，若新管線依然具備相同名稱的參數埠，既有連線會被自動保留，介面並會即時重新排版以反映新佈局。
- **即時預覽會降低推論效能**：預覽功能預設處於關閉狀態。若需開啟，請至 **設定 → Modular Diffusion Library → Enable Image Preview Intermediates**。
- **靈活運用 `start_step` 與 `end_step`**：您可以串聯兩個 Generate 節點（例如前置節點執行步數 `0–10` 確定構圖，後置節點執行 `10–20` 細化細節），實現靈活的多階段精細控制。
- **支援執行中途即時取消**：在介面點擊取消時，系統會設定管線的 `_interrupt` 標記，運算會在當前步驟結算後立即優雅終止。
- **解析度維度完全源自 `input_latent`**：Generate 節點本身沒有寬高設定——解析度一律由上游的 [Create Noise Latents](create_noise_latents.md) 或影像編碼節點全權決定。

## 相關延伸閱讀 {: #see-also }

- [Modular Diffusion Pipeline Builder](pipeline_builder.md)
- [Create Noise Latents](create_noise_latents.md) · [Encode Media Latent](encode_media_latent.md) · [Encode Masked Media Latent](encode_masked_media_latent.md) —— 常見的上游張量生成節點。
- [Decode Media Latent](decode_media_latent.md) —— 典型的下游視覺解碼節點。
- 工作流程範本：`workflows/templates/Text2Image.py`、`workflows/templates/MultistageText2Image.py`、`workflows/templates/Image2Image.py`。
