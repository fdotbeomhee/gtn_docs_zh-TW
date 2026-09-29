# Decode HDR Latents {: #decode-hdr-latents }

**解碼 HDR 高動態範圍潛在張量、套用色調映射 (Tone Mapping)，並支援選用匯出未經壓縮的原始線性 OpenEXR 畫格序列。**

類別目錄：`ModularDiffusion/Encode\Decode`

## 核心速覽 {: #tldr }

- 作為 [Decode Media Latent](decode_media_latent.md) 的高階擴充節點：具備完全相同的輸入端與動態影像/視訊輸出，並額外針對 HDR 視訊管線（例如 LTX 2.3 HDR）增加了色調映射與 EXR 序列圖匯出能力。
- 主輸出端一律傳回經過色調映射的標準 SDR 影像或 MP4 視訊——設定 `exr_output_folder` 可在套用色調映射之前，將原始無損的線性 HDR 畫格同步匯出為 OpenEXR 檔案序列。
- 對於標準（非 HDR）管線，本節點行為與 Decode Media Latent 完全一致；此時不會套用色調映射。
- HDR 輸出專門針對傳回線性 `np.ndarray` 畫格陣列的管線驅動（如 LTX 2.3 HDR）；其餘模型自動走常規標準解碼路徑。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Generate Media Latents → [Decode HDR Latents] → Save Image / Save Video
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/decode-hdr-latents.png" alt="Decode HDR Latents" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 必須與生成該潛在張量的管線完全匹配。需選用具備 HDR 能力的模型（如 LTX 2.3 HDR）方可獲取線性高動態畫格。 |
| `latent_tensor` | `LatentArtifact` | 是 | 欲進行解碼的目標潛在張量。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `output_image` | `ImageArtifact` | 色調映射後的 SDR 實體影像。於影像管線下呈現。 |
| `output_video` | `VideoUrlArtifact` | 色調映射後的 SDR MP4 實體視訊。於視訊管線下呈現。 |
| `logs` | str | 逐畫格 EXR 寫入磁碟日誌。僅在設定 `exr_output_folder` 時記錄。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `tone_mapping` | choice (`clip` \| `reinhard` \| `aces_filmic` \| `cv2_reinhard` \| `cv2_mantiuk`) | `aces_filmic` | 在將線性 HDR 畫格編碼為 SDR 輸出前所套用的色調映射算子。對非 HDR 管線自動忽略。 |
| `fps` | int (1–120) | `25` | 視訊輸出畫面更新率。僅在視訊管線下呈現。 |
| `exr_output_folder` | path | — | 儲存原始線性 HDR EXR 序列圖的目標資料夾或 `.exr` 檔案路徑。留空則略過 EXR 序列匯出。 |
| `save_exr_as_half_float` | bool | `True` | 是否以 float16 (Half Float) 格式儲存 EXR 畫格——檔案體積縮小 2.5 倍且畫質損耗微乎其微。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **`aces_filmic` 比 `clip` 具備更好的高光壓制能力**：相較於粗暴截斷的 `clip`，ACES 膠片色調映射能自然平滑過渡極亮高光細節。僅在極度追求色度飽和度精確或與原數值進行數學比對時才選用 `clip`。
- **EXR 輸出命名支援三種路徑模式：**
    - `output/frames/`（帶結尾斜線的資料夾路徑）→ 檔案將依序命名為 `output/frames/frame_0000.exr`、`frame_0001.exr` 等。
    - `output/frames`（無結尾斜線，不帶副檔名的資料夾路徑）→ 同上，預設前綴為 `frame`。
    - `output/frames/shot.exr`（以 `.exr` 結尾的具體檔名）→ 檔案將依序命名為 `output/frames/shot_0000.exr`、`shot_0001.exr` 等。
- **解碼 VRAM 最佳化**：高動態視訊畫格解碼極消耗記憶體。請在上游 Pipeline Builder 節點中開啟 `vae_slicing` 以避免顯示卡記憶體溢位。

## 相關延伸閱讀 {: #see-also }

- [Decode Media Latent](decode_media_latent.md) —— 基礎解碼節點；適用於常規非 HDR 管線。
- [Generate Media Latents](generate_media_latents.md) —— 典型產生待解碼張量的核心上游節點。
