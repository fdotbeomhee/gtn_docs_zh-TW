# Nuke Script {: #nuke-script }

**透過 `nuke -t` 在背景無介面 (Headless) 模式下執行既有的 `.nk` 腳本，將標註的 Read/Write 節點發布為 Griptape 畫布上的型別輸入/輸出連接埠。**

類別目錄：`Foundry Nuke`

## 核心速覽 {: #tldr }

- 將 `script_path` 指向 `.nk` 專案檔，並在 Nuke 內部利用 Griptape Annotator 面板標註其 I/O 節點，本節點便會動態擴充出對應這些標註的型別連接埠。
- 輸入端支援影像、影片或原始檔案路徑；輸出端可接收單幀影像、影像序列幀、影片或 3D 幾何模型。
- 點擊 **Open in Nuke** 按鈕可在 Nuke GUI 介面中載入該腳本並預先掛載 Annotator 面板；點擊 **Refresh UI** 則重新讀取標註並重新執行 Nuke 安裝路徑掃描。
- 標註資訊保存在 `.nk` 檔案旁的 `<script>.gt.json` 附屬檔中——當附屬檔看起來過期時，節點會發出警告提示。

## 典型工作流程架構 {: #typical-workflow-position }

```text
(影像/影片來源) → [Nuke Script] → (影像/影片/3D 消費端節點)
```

## 節點介面預覽 {: #node-preview }

<!-- TODO: add ../assets/nuke-script.png screenshot -->

## 快速上手 {: #quick-start }

1. 將 **Nuke Script** 節點拖曳至畫布中。
1. 將 **Script Path** 設定為您的 `.nk` 檔案路徑。
1. 點擊 **Open in Nuke**——此操作將啟動 Nuke，自動掛載 Griptape Annotator 面板，並預先套用目前的任何旋鈕覆寫值 (Knob Overrides)。
1. 在 Nuke 中開啟 **Panels > Griptape Annotator**。在 **Annotate** 分頁中，將 Read 節點標註為輸入埠，將 Write 節點標註為輸出埠，並可選擇性發布其他節點的旋鈕參數，隨後點擊 **Save Annotations**。
1. 回到 Griptape 介面，點擊 **Refresh UI**（或重新選取腳本路徑）。節點將根據標註動態生成相應的型別輸入/輸出連接埠。
1. 連接各輸入端，設定 **Frame Start** / **Frame End** 影格範圍，隨後執行節點。

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `script_path` | `str` | 是 | `.nk` 檔案的本機路徑。 |
| `<標註的 Read 節點>` | `str \| ImageArtifact \| ImageUrlArtifact \| VideoUrlArtifact \| BlobArtifact` | 否 | 每個標註為輸入埠的 Read 節點各對應一個連接埠。基於 URL 的產物會在渲染執行前自動下載為本機暫存檔案。 |

## 輸出端說明 {: #outputs }

標註為輸出埠的 Write 節點會在節點執行完畢後呈現在 **Outputs** 群組中。產物型別取決於當初的標註規格：

| 標註型別 | Griptape 輸出產物 |
| :--- | :--- |
| `ImageArtifact` / `ImageUrlArtifact` | `ImageUrlArtifact`（單一渲染影格） |
| `VideoUrlArtifact` | `VideoUrlArtifact`（渲染完成之影片檔，例如 `.mp4`） |
| `ImageSequenceArtifact` | 由 `ImageUrlArtifact` 組成的 `ListArtifact`——每個渲染影格對應一項 |
| `ThreeDUrlArtifact` / `GLTFUrlArtifact` | `ThreeDUrlArtifact`（`.obj` 或 `.glb` 檔案） |

## 參數說明 {: #parameters }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `nuke_installation` | choice | 自動偵測 | **Nuke Version** 下拉選單，由安裝路徑自動掃描機制填入（詳見 [程式庫概觀](index.md#configuring-nuke-installations)）。 |
| `frame_start` | `int` | 腳本預設值 | 欲渲染的第一個影格（起始幀）。 |
| `frame_end` | `int` | 腳本預設值 | 欲渲染的最後一個影格（結束幀）。 |
| `<發布的旋鈕參數>` | 視型別而定 | 腳本內建數值 | 透過 Annotator 面板的 **Expose** 分頁升級發布的參數，依其節點名稱分組呈現。欄位留空則沿用腳本原始數值；填寫則在渲染前進行強制覆寫。 |

### 進階設定 *(預設處於摺疊狀態)* {: #advanced }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `nuke_executable` | `str` | `""` | Nuke 二進位執行檔的本機絕對路徑；僅針對此單一節點覆寫引擎全域設定。 |
| `baked_script_path` | `str` | `""` | 烘焙導出之 `.nk` 複本的儲存目的地路徑。 |
| `save_baked_copy` | button | — | 將當前所有參數值永久烘焙寫入各旋鈕中並輸出為獨立腳本複本——便於封存歸檔或手動檢視。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **編輯腳本後務必重新儲存標註**：若在標註完成後修改並儲存了 `.nk` 檔，節點會發出 `<script>.gt.json` 附屬檔可能已過期的警告——請在 Nuke 中重新開啟並重新儲存標註以完成同步。
- **只有標註後才會出現連接埠**：剛指定腳本路徑的節點完全沒有任何 I/O 連接埠；請務必先透過 Annotator 面板完成標註，再點擊 **Refresh UI**。
- **執行前務必配置授權**：請在 Griptape 憑據管理面板中設定 `foundry_LICENSE`；無介面渲染若無法存取授權將直接中斷失敗。
- **請使用版本下拉選單，切勿依賴 `PATH` 機制**：透過 `nuke_installation`（或節點級 `nuke_executable` 覆寫）明確鎖定 Nuke 版本，以確保跨工作站渲染結果的一致性與可重現性。

## 相關延伸閱讀 {: #see-also }

- [Nuke Start Flow](nuke_start_flow.md) · [Nuke End Flow](nuke_end_flow.md) —— 包裝工作流程以供發布為 Nuke Gizmo 工具組。
