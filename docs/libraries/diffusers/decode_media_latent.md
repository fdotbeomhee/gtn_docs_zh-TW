# Decode Media Latent {: #decode-media-latent }

**調用擴散管線的 VAE 解碼器 (VAE Decoder) 將潛在空間張量解碼還原為實體影像或視訊畫面——通常作為生成工作流程的終點節點。**

類別目錄：`ModularDiffusion/Encode\Decode`

## 核心速覽 {: #tldr }

- 輸出端具備**動態自我調適能力**：連線影像管線時輸出 `output_image`，連線視訊管線時（LTX、LTX2、WAN）則自動切換為 `output_video`（並顯示 `fps` 參數）。連線 `pipeline` 時會自動無縫切換。
- 幾乎一律作為擴散流程的最後一個處理步驟。通常接駁至 Save Image 或 Save Video 節點將成果寫入本機磁碟。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Media Latents → [Decode Media Latent] → Save Image / Save Video
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/decode-media-latent.png" alt="Decode Media Latent" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 必須與生成該潛在張量的擴散管線架構完全匹配。 |
| `latent_tensor` | `LatentArtifact` | 是 | 欲進行視覺解碼的目標潛在張量。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_image` | `ImageArtifact` | 純影像管線下的輸出實體圖片產物。 |
| `output_video` | `VideoUrlArtifact` | 視訊管線下的輸出實體視訊檔案產物。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `fps` | int (1–120) | `25` | 視訊輸出畫面更新率 (FPS)。**僅在視訊管線下動態顯示**。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **必須使用與生成張量相同的管線進行解碼**：每條管線皆綁定與其神經網路一同聯合訓練的專屬 VAE 解碼權重——使用錯誤或不匹配的 VAE 解碼張量將導致畫面嚴重色彩失真或徹底損毀。
- **大尺寸潛在張量解碼極其消耗 VRAM**：高解析度圖片或超長畫格視訊在解碼階段需要大量顯示卡記憶體。若遭遇 VRAM 溢位，請在上游 Pipeline Builder 節點中開啟 `vae_slicing` 以透過分塊解碼大幅壓低記憶體尖峰峰值。

## 相關延伸閱讀 {: #see-also }

- [Encode Media Latent](encode_media_latent.md) —— 影像編碼至潛在空間的逆向操作節點。
- [Generate Media Latents](generate_media_latents.md) —— 典型產生待解碼張量的核心上游節點。
