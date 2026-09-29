# Create Noise Latents {: #create-noise-latents }

**生成契合連線管線維度規格的隨機高斯噪訊潛在張量 (Random-noise Latent Tensor)——純文字生圖 (Text-to-Image) 與文字生視訊 (Text-to-Video) 工作流程的經典起點。**

類別目錄：`ModularDiffusion/Create`

## 核心速覽 {: #tldr }

- 在純文字生圖或文字生視訊工作流程中，請將此節點置於 **Pipeline Builder** 與 **Generate Media Latents** 之間。
- `width`（寬度）與 `height`（高度）以像素 (Pixel) 為單位填寫；節點內部會自動依據 VAE 縮放倍率轉換至潛在空間維度。
- `num_frames`（畫格數）僅在連線的管線屬於視訊模型時（如 LTX、LTX2、WAN）才會動態顯示，圖片管線下會自動隱藏。
- `seed`（隨機亂數種子）控制生成的可重現性——相同的種子 + 相同的管線 + 相同的尺寸 → 輸出完全一致的起始噪訊張量。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder → [Create Noise Latents] → Generate Media Latents → Decode Media Latent
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/create-noise-latents.png" alt="Create Noise Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 來自 Pipeline Builder。決定潛在張量維度形狀、VAE 下採樣倍率以及是否顯示 `num_frames` 參數。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_latent` | `LatentArtifact` | 解包且標準化 (~N(0,1)) 的高斯噪訊張量。接駁傳遞至 Generate Media Latents 節點。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `width` | int (像素) | `1024` | 像素空間的目標寬度。內部自動除以 VAE 下採樣因子。 |
| `height` | int (像素) | `1024` | 像素空間的目標高度。 |
| `num_frames` | int | `41` | 視訊總畫格數。**在純影像管線下自動隱藏**。 |
| `seed` | int | 隨機數 | 控制亂數種子以確保畫面可重現性。 |
| `num_inference_steps` | int | `20` | **僅在 SDXL 模型下顯示**——SDXL 用於計算起始噪訊縮放係數。其他管線會自動忽略此參數。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **解析度必須符合 VAE 整除限制**：大多數 VAE 架構要求幾何尺寸必須能被 8 或 16 整除。請選取標準尺寸（如 512, 768, 1024, 1280 等）以確保張量形狀合法。
- **跨不同模型的種子不可對比**：不同模型架構的潛在空間維度與 VAE 空間截然不同；相同的 Seed 僅在相同的管線架構下保證生成一致性。
- **圖生圖或二次擴散重繪請改用 [Encode Media Latent](encode_media_latent.md)**：將既有影像編碼為潛在張量能提供具備明確特徵的起始點；隨後可接駁至 Generate Media Latents 並開啟 `add_noise=True` 進行去噪。

## 相關延伸閱讀 {: #see-also }

- [Create Empty Latents](empty_latents.md) —— 全零填充的空白潛在張量，適用於遮罩合成或特定多階段技巧。
- [Encode Media Latent](encode_media_latent.md) —— 用於將實體影像/視訊編碼為潛在空間張量。
- [Generate Media Latents](generate_media_latents.md) —— 接收並消耗本節點輸出張量的核心去噪節點。
