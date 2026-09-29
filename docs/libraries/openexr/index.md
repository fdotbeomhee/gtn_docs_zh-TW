# OpenEXR 程式庫 {: #openexr-library }

為視覺特效 (VFX) 與高動態範圍 (HDR) 管線量身打造的專業 [OpenEXR](https://openexr.com/) 支援程式庫。載入 EXR 檔案並解析其內部結構、將單一部件 (Parts) 或手動組合的色彩通道渲染為經過色調映射的預覽影像，並支援將影像或通道矩陣資料重新寫入為高品質 EXR 檔案。

- **原始碼存放庫**：[griptape-ai/griptape-nodes-library-openexr](https://github.com/griptape-ai/griptape-nodes-library-openexr)
- **環境需求**：無強制依賴項（可選配 [OpenColorIO 程式庫](../opencolorio/index.md) 以實現專業色彩管理檢視）
- **節點分類目錄**：節點挑選器中的 `EXR` 目錄

## 安裝指南 {: #installation }

在編輯器中，依序開啟 **管理 (Manage) → 程式庫管理 (Library Management)**，點擊 **新增程式庫 (Add Library)** 並貼入以下網址：

```text
https://github.com/griptape-ai/griptape-nodes-library-openexr
```

或透過 CLI 命令列執行安裝：

```bash
gtn libraries download https://github.com/griptape-ai/griptape-nodes-library-openexr
```

有關通用的安裝、更新與除錯指引，請參閱 [程式庫指南](../../guides/libraries.md)。

## 節點索引參考 {: #nodes }

| 節點名稱 | 功能說明 |
| :--- | :--- |
| [Load EXR](load_exr.md) | 載入 EXR 檔案並將其完整結構（部件 Parts、通道 Channels、檔頭屬性 Header）發布為型別輸出端——預設僅讀取檔頭，因此即便面對超大多部件算圖檔亦能秒級解析 |
| [Display EXR Part](display_exr_part.md) | 將 EXR 單一部件渲染為 8-bit sRGB/RGBA PNG 圖片，支援曝光補償與色調映射（或 OCIO 色彩管理） |
| [Display EXR Channel](display_exr_channel.md) | 將 1 至 4 個獨立 EXR 通道重新組合成 8-bit 檢視影像，支援在背景底色上進行 Alpha 通道合成 |
| [Save EXR](save_exr.md) | 將 8-bit 影像或 EXR 通道產物寫入為單一部件 EXR 檔案，具備壓縮演算法、像素資料型別與檔頭中繼資料控制能力 |

## 快速上手 {: #quick-start }

```text
Load EXR → Display EXR Part → (影像輸出)
         ↘ Display EXR Channel (從任意部件或檔案組合 R/G/B/A 通道)
```

1. 新增 **Load EXR** 節點並將 `file_path` 指向本機 `.exr` 檔案。節點會秒級掃描檔頭並動態生成 **Parts** 與 **Channels** 參數群組。
1. 將部件輸出埠連線至 **Display EXR Part**（或將個別通道連線至 **Display EXR Channel**），即可獲得經過色調映射的 PNG 影像，供畫布預覽或串接至其他影像處理節點。
1. 使用 **Save EXR** 節點將生成的影像或通道資料重新保存為標準 EXR 檔案。

## OpenColorIO 深度整合 {: #opencolorio-integration }

影像檢視節點支援兩種色彩轉換模式：

- **`basic`**：本機曝光補償 + 色調映射（`filmic` 膠片感或 `linear` 線性映射）。
- **`ocio`**：連線來自 [OpenColorIO 程式庫](../opencolorio/index.md) 的 `OCIOColorParamsArtifact` 產物，由 OCIO 驅動專業的 Display-View 色彩轉換。

當系統安裝了 OpenColorIO 程式庫時，檢視節點會自動將色彩模式預設切換為 `ocio`；否則預設使用 `basic` 模式。OCIO 依賴項為選配項目——缺乏時完全不影響基礎功能運作。

## 程式庫全域設定 {: #library-settings }

設定項位於引擎全域配置的 `openexr` 目錄下：

| 設定項目 | 預設值 | 功能說明 |
| :--- | :--- | :--- |
| `openexr.header_only` | `true` | 當設為 `true` 時，**Load EXR** 僅讀取檔頭中繼資料（速度極快）。若設為 `false`，則會將像素陣列載入記憶體以讀取精確的逐通道像素型別。 |
| `openexr.viewer_executable` | `""` | 外部專業 HDR 播放器執行檔的本機絕對路徑（例如 `/usr/bin/djv` 或 Nuke 執行檔）。留空時將直接調用作業系統預設檔案關聯開啟。 |
| `openexr.viewer_args` | `""` | 傳遞給外部播放器的額外命令列參數（置於檔案路徑之前），例如 `--hdr --linear`。支援類 Shell 引號解析規則。 |

每個節點介面皆內建 **Open in external viewer（在外部播放器中開啟）** 按鈕，點擊可立即在配置的外部播放器中檢視來源 EXR 檔案。該調用為非同步背景獨立處理——點擊絕不會凍結編輯器畫布。

## 支援與反饋 {: #support }

若在運作中發現程式錯誤或有全新功能建議，歡迎隨時向社群回報：
[提交 GitHub Issue](https://github.com/griptape-ai/griptape-nodes-library-openexr/issues)。
