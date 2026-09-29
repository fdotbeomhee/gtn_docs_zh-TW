# Display EXR Part {: #display-exr-part }

**將 EXR 單一部件 (Part) 渲染為 8-bit sRGB（或 RGBA）PNG 影像，支援曝光補償與色調映射，或直接整合 OCIO 色彩管理管線。**

類別目錄：`EXR`

## 核心速覽 {: #tldr }

- 接收來自 [Load EXR](load_exr.md) 節點的 `EXRPartArtifact`，自動挑選 RGB 色彩通道，並生成可在畫布中即時預覽的 `ImageUrlArtifact` 影像。
- 支援兩種色彩處理模式：`basic`（本機曝光補償 + 膠片感/線性色調映射）與 `ocio`（連線 `OCIOColorParamsArtifact` 驅動 OCIO Display-View 色彩轉換）。
- 當部件內部包含 Alpha 通道（`A`）時，節點會自動將透明度無損包含在輸出的 PNG 影像中。
- 節點執行後，可透過 **Open in external viewer** 按鈕一鍵在外部 HDR 播放器中開啟原始 EXR（參見 [程式庫設定](index.md#library-settings)）。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Load EXR → [Display EXR Part] → (影像輸出)
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/display-exr-part.png screenshot -->

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `part` | `EXRPartArtifact` | 是 | 欲渲染的 EXR 部件。預設視為場景線性 (Scene-linear) HDR 資料；不進行硬性色域裁切。 |
| `exposure` | `float` | 否 | 在套用色調映射或 OCIO 轉換前所施加的曝光補償（以 EV 曝光值為單位）。範圍 -10 至 +10，預設值為 `0.0`。 |
| `color_params` | `OCIOColorParamsArtifact` | 否 | 來自 [OCIO Color Parameters](../opencolorio/ocio_color_parameters.md)。當 `color_mode` 設為 `ocio` 時必填。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `image` | `ImageUrlArtifact` | 供畫布檢視或後續處理引用的 8-bit sRGB 或 RGBA PNG 實體影像。 |
| `output_file` | `str` | 磁碟中保存之 PNG 檔案的本機路徑。 |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `color_mode` | choice (`basic` \| `ocio`) | 若安裝了 OpenColorIO 則預設為 `ocio`，否則為 `basic` | `basic` 採用內建色調映射；`ocio` 調用連線的 `OCIOColorParamsArtifact` 色彩轉換。 |
| `tone_mapping` | choice (`filmic` \| `linear`) | `filmic` | 當 `color_mode` 為 `basic` 時套用的本機色調映射。`filmic` 採用 Narkowicz 2015 擬真膠片曲線；`linear` 則單純將數值截斷至 [0, 1]。在 `ocio` 模式下自動隱藏。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **OCIO 模式下必須連線 `color_params`**：若缺乏該參數，節點將明確拋出錯誤中斷執行——若未配置 OCIO 設定檔，請將 `color_mode` 切換回 `basic`。
- **預設輸入為場景線性資料**：節點不擅自進行色域變換；若您的 EXR 內部存放的是 Log 對數或顯示參照資料，請務必啟用 OCIO 模式並選定正確的來源色彩空間。
- **OCIO 異常嚴格回報**：若設定檔損毀或 View 選項不合法，節點會直接報錯，絕不會靜默降級至基礎色調映射，確保色彩無瑕疵。

## 相關延伸閱讀 {: #see-also }

- [Load EXR](load_exr.md) —— 提供本節點所需的 `part` 輸入端。
- [Display EXR Channel](display_exr_channel.md) —— 自訂提取並組合個別通道進行檢視。
- [OCIO Color Parameters](../opencolorio/ocio_color_parameters.md) —— 提供本節點所需的 `color_params` 輸入端。
