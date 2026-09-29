# OpenColorIO 程式庫 {: #opencolorio-library }

基於 [OpenColorIO](https://opencolorio.org/) (OCIO) 構建的影視工業級專業色彩管理節點——OCIO 是全球電影、視覺特效 (VFX) 與動畫製作管線中廣泛採用的色彩管理標準規範。

- **原始碼存放庫**：[griptape-ai/griptape-nodes-library-opencolorio](https://github.com/griptape-ai/griptape-nodes-library-opencolorio)
- **環境需求**：合法的 OCIO 設定檔（透過 `$OCIO` 環境變數或明確指定的檔案路徑提供）
- **節點分類目錄**：節點挑選器中的色彩空間 (`Colorspace`) 目錄

## 安裝指南 {: #installation }

在編輯器中，依序開啟 **管理 (Manage) → 程式庫管理 (Library Management)**，點擊 **新增程式庫 (Add Library)** 並貼入以下網址：

```text
https://github.com/griptape-ai/griptape-nodes-library-opencolorio
```

或透過 CLI 命令列執行安裝：

```bash
gtn libraries download https://github.com/griptape-ai/griptape-nodes-library-opencolorio
```

有關通用的安裝、更新與除錯指引，請參閱 [程式庫指南](../../guides/libraries.md)。

## 前置條件：`$OCIO` 環境變數配置 {: #prerequisites-the-ocio-environment-variable }

本程式庫優先透過 `$OCIO` 環境變數來尋找並載入色彩設定檔。請在啟動 Griptape Nodes 引擎前完成設定：

```bash
export OCIO=/path/to/your/config.ocio
```

多數專業電影工作室工作站與算圖農場環境皆已預先配置好 `$OCIO`。若您正在使用 Griptape [專案系統](../../guides/projects/index.md)，可在各專案的 `project.yml` 中進行專案級隔離設定：

```yaml
environment:
  OCIO: "{project_dir}/config/aces.ocio"
```

每次節點執行時皆會即時重新讀取 `$OCIO` 變數數值，因此切換專案時能自動套用新配置，完全無需重新載入工作流程。

本程式庫亦將 `OCIO` 註冊為知名系統環境變數，您亦可直接在 Griptape Nodes 的設定介面 (Settings UI) 中檢視或修改它，無需切換出應用程式。

## 節點索引參考 {: #nodes }

| 節點名稱 | 功能說明 |
| :--- | :--- |
| [Load OCIO Config](load_ocio_config.md) | 從 `$OCIO` 環境變數或指定磁碟路徑載入設定檔；輸出供下游引用的 `OCIOConfigArtifact` |
| [OCIO Color Parameters](ocio_color_parameters.md) | 將來源色彩空間 (Source Colorspace)、顯示裝置 (Display) 與檢視視圖 (View) 打包為可複用的 `OCIOColorParamsArtifact`；下拉選單自動從連線的 OCIO 配置中提取 |

## 快速上手 {: #quick-start }

1. 將 **Load OCIO Config** 節點新增至畫布中。
    - 若系統中已設定 `$OCIO`，節點會自動辨識並呈現該檔案路徑。
1. 將 `config` 輸出埠連線至 **OCIO Color Parameters** 節點，挑選來源色彩空間、目標顯示裝置與檢視視圖。
1. 將 `color_params` 輸出端接駁至任何接收 `OCIOColorParamsArtifact` 的下游節點——例如來自 [OpenEXR 程式庫](../openexr/index.md) 的 [Display EXR Part](../openexr/display_exr_part.md) 與 [Display EXR Channel](../openexr/display_exr_channel.md) 節點。

```text
Load OCIO Config → OCIO Color Parameters → (下游色彩變換 / 視覺化檢視節點)
```

## 與其他程式庫的深度整合 {: #used-by-other-libraries }

除了提供挑選設定節點之外，本程式庫還對外公開了色彩空間轉換中樞服務，可供其他擴充程式庫調用。[OpenEXR 程式庫](../openexr/index.md) 能自動偵測本服務：當安裝了 OpenColorIO 程式庫時，EXR 影像檢視節點會自動將色彩色彩處理模式預設切換為 OCIO 專業管理模式 (`color_mode: ocio`)，取代傳統的本機色調映射。

## 支援與反饋 {: #support }

若在運作中發現程式錯誤或有全新功能建議，歡迎隨時向社群回報：
[提交 GitHub Issue](https://github.com/griptape-ai/griptape-nodes-library-opencolorio/issues)。
