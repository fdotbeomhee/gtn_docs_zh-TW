# Media Generation Conditioning {: #media-generation-conditioning }

**將一張或多張條件參考圖片（或參考視訊）連同逐畫格時間軸定位與引導強度進行打包封裝，隨時可接駁至任何支援多媒體條件控制的擴散生成管線中。**

類別目錄：`ModularDiffusion/Conditioning`

## 核心速覽 {: #tldr }

- 連線管線輸入端後，節點會自動切換為針對該模型量身打造的預設佈局（例如 WAN 圖生視訊的首尾幀專用插槽）。
- 若未連線管線，節點會呈現高彈性的通用影像或視訊佈局，並提供 "Preset" 預設下拉選單；在 "Custom" 自訂模式下可透過滑桿自由指定圖片插槽數量。
- 將 `conditioning` 輸出端接駁至 Generate Media Latents 節點的 `media_conditions`（適用於 LTX / LTX2 / WAN）或 `reference_images`（適用於 Flux2 Klein）輸入埠。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder ──────────────────────────────────────────┐
                                                           ├─→ Generate Media Latents → Decode
Load Image (首幀圖片) ──┐                                  │
Load Image (尾幀圖片) ──┴─→ [Media Generation Conditioning] ──┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/media-gen-conditioning.png" alt="Media Generation Conditioning" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 否 | 連線後會立即將條件控制介面切換為該模型架構特化適配的佈局。 |
| `image_{i}` | `ImageUrlArtifact` | 否 | 插槽 `i` 的條件引導圖片。在影像模式下可見；插槽總數由使用者配置決定。 |
| `video` | `VideoUrlArtifact` | 否 | 條件引導視訊。僅在視訊模式下可見。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `conditioning` | `media_gen_conditioning` | 包含影像/視訊實體 + 各插槽畫格位置 + 引導強度的結構化負載。接駁至 Generate Media Latents 的對應輸入埠。 |

## 參數說明 {: #parameters }

### 模式與預設選擇 {: #mode-and-preset-selection }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `mode` | `image` \| `video` | `image` | 當連線的管線僅支援單一模式時會自動隱藏。切換此項會動態重置所有輸入插槽。 |
| `image_preset` | choice | `Custom` | 具名預設佈局下拉選單。選項與預設值取決於連線的管線。在視訊模式下自動隱藏。 |
| `num_images` | int 滑桿 | `0` | 圖片插槽數量。僅在 `image_preset = Custom` 時可見。預設範圍 0–8；可能受管線限制約束。 |

### 單張圖片插槽微調 *(影像模式下每個插槽各有一組)* {: #per-image-slot }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `image_{i}_frame_index` | int | `0` | 此圖片在輸出視訊時間軸上的畫格位置索引。在具名預設模式下（位置已鎖定）或不適用之管線（如 Flux2 Klein）會自動隱藏。 |
| `image_{i}_strength` | float (0.0–1.0) | `1.0` | 該張圖片的獨立條件引導權重。在不支援逐圖權重的管線下自動隱藏。 |

### 視訊模式專屬參數 {: #video-mode }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `frame_index` | int | `0` | 條件引導視訊在輸出畫格序列中開始插入的起始畫格索引。 |
| `video_strength` | float (0.0–1.0) | `1.0` | 條件視訊的引導約束權重。 |

## 供應商與模型特定行為 {: #provider-model-behavior }

### 未連線管線 (預設通用狀態) {: #no-pipeline-connected }

影像與視訊模式皆可自由切換。`Preset` 選項包含：**Custom**（預設）、First + Middle + Last（首/中/尾幀）、First + Last（首尾雙幀）、First frame（僅首幀）。在 Custom 模式下，`num_images`（0–8）、`frame_index` 與 `strength` 皆可完全手動調整。視訊模式下則呈現單一 `video` 輸入埠與相應的 `frame_index` 和 `video_strength`。

### WAN 圖生視訊 (Image-to-Video) {: #wan-image-to-video }

僅支援影像模式。`Preset` 選項：**First + Last**（預設首尾幀錨定）、First frame（首幀引導）。

### LTX / LTX2 {: #ltx-ltx2 }

與未連線狀態相仿，但在影像模式下，預設採用 **First + Middle + Last**（首/中/尾幀三點引導）。

### Flux2 Klein {: #flux2-klein }

僅支援影像模式。提供高度靈活的多圖參考插槽（1–8 張）。在 Generate Media Latents 節點上的 `reference_images` 輸入端僅在同時連接局部重繪遮罩 (Inpaint Mask) 時才會被啟用。

## 技巧與避坑指南 {: #tips-pitfalls }

- **優先連線管線**：連線後介面佈局會瞬間調適，自動為您掛載最契合該模型演算法的插槽設定。
- **具名預設會自動鎖定畫格位置**：當選用具名預設（如 "First + Last"）時，各插槽在視訊時間軸上的位置由演算法嚴格定義（例如插槽 0 釘選於畫格 0，插槽 1 釘選於最終尾幀），此時 `frame_index` 參數會自動隱藏以防誤改。若需在任意指定畫格插入參考圖，請切換回 **Custom** 預設。
- **`frame_index` 控制參考圖在*輸出視訊時間軸*上的位置**：數值 `0` 代表將參考圖作為生成視訊的第一幀起始畫面；填入最後一個畫格編號則將畫面錨定於結尾。

## 相關延伸閱讀 {: #see-also }

- [Generate Media Latents](generate_media_latents.md) · [Modular Diffusion Pipeline Builder](pipeline_builder.md)
