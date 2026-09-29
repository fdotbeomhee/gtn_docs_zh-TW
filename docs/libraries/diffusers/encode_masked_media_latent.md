# Encode Masked Media Latent {: #encode-masked-media-latent }

**將來源影像與二值化遮罩聯合編碼為 `InpaintMaskArtifact`——這是觸發 Generate Media Latents 節點自動切換為局部重繪 (Inpainting) 模式的專用輸入規格。**

類別目錄：`ModularDiffusion/Encode\Decode`

## 核心速覽 {: #tldr }

- 專為**局部重繪 (Inpainting)** 任務打造。產生的產物將遮罩、遮罩影像潛在張量與重繪強度 (`strength`) 打包封裝在一起，免除下游節點分別接駁多條線路的繁瑣負擔。
- 將 `latents` 輸出端接駁至 Generate Media Latents 節點的 `input_latent` 輸入埠。Generate 節點會自動辨識該產物型別，並無縫切換至 Inpaint 專屬擴散管線類別。
- 僅相容具備註冊 Inpaint 管線實作的模型架構（例如 Flux Fill、SDXL Inpaint）。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Load Image ──┐
             ├─→ [Encode Masked Media Latent] → Generate Media Latents → Decode
Paint Mask ──┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/encode-masked-media.png" alt="Encode Masked Media Latent" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 必須支援局部重繪（底層驅動具備 `_inpaint_pipeline_class` 定義）。 |
| `image` | `ImageArtifact` / `ImageUrlArtifact` | 是 | 欲進行重繪的來源底圖。 |
| `mask` | `ImageArtifact` / `ImageUrlArtifact` | 是 | 二值化黑白遮罩——**白色像素 = 欲重繪消除區域**，黑色像素 = 原圖保留區域。 |
| `strength` | float (0.0–1.0) | 否 | 局部重繪去噪強度，預設值為 `1.0`。數值越低越保留原圖底層結構。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `latents` | `InpaintMaskArtifact` | 包含遮罩 + 來源/遮罩影像潛在張量 + 權重強度的結構化複合產物。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **管線架構必須相容 Inpaint**：若選取的基礎模型缺乏 Inpaint 驅動分支，節點將在校驗階段報錯。請在 Pipeline Builder 節點上選取專用重繪模型（如 Flux Fill 等）。
- **遮罩尺寸必須契合來源圖**：遮罩內部會被重採樣至潛在空間網格——若遮罩長寬比與原圖嚴重不符，將導致重繪區域產生形變拉伸。
- **重繪強度 `strength` 綁定於本節點**：請直接在本節點上調整強度參數，而非下游的 Generate Media Latents 節點。

## 相關延伸閱讀 {: #see-also }

- [Encode Media Latent](encode_media_latent.md) —— 無遮罩的標準編碼節點。
- [Generate Media Latents](generate_media_latents.md) —— 自動偵測並接收 Inpaint 複合張量的擴散去噪節點。
