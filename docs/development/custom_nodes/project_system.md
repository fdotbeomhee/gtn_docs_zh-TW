# 專案系統整合開發 (Working with the Project System) {: #working-with-the-project-system }

**專案系統 (Project System)** 是 Griptape Nodes 的集中式檔案管理框架，負責跨所有工作流程標準化處理檔案組織、命名與保存操作。它徹底摒棄了硬編碼的本機絕對路徑，為所有檔案 I/O 操作提供了高度一致且靈活可配的架構。本頁面面向節點開發者：闡述節點如何透過專案系統安全、優雅地保存產出檔案。

## 核心概念 {: #concepts }

專案系統背後的設計哲學——工作區 (Workspace)、專案範本、情境 (Situations)、巨集 (Macros)、目錄映射與環境變數——已詳盡收錄於 [專案系統指南](../../guides/projects/index.md)：

- [概述](../../guides/projects/index.md) —— 各核心組件如何協同運作
- [情境 (Situations)](../../guides/projects/situations.md) —— 具名存檔情境、檔名衝突處置原則以及預設情境清單
- [巨集 (Macros)](../../guides/projects/macros.md) —— 用於動態組裝檔案路徑的範本語法
- [目錄 (Directories)](../../guides/projects/directories.md) —— 巨集中引用的邏輯目錄至實體路徑映射
- [定製指南](../../guides/projects/customization.md) —— 使用者如何透過 `griptape-nodes-project.yml` 覆寫路徑與情境

**致節點作者的極簡摘要**：**情境 (Situation)** 為特定存檔行為命名（例如 `save_node_output`），其關聯的 **巨集 (Macro)** 範本（例如 `{outputs}/{node_name?:_}{file_name_base}{_index?:03}.{file_extension}`）在執行階段生成具體磁碟路徑，而使用者無需修改您的節點程式碼即可隨心自訂這一切。

## 在節點中使用專案系統 {: #using-the-project-system-in-nodes }

在自訂節點中處理專案檔案主要有兩種標準模式：

### 模式 1：ProjectFileParameter（節點輸出推薦首選） {: #pattern-1-projectfileparameter-recommended-for-node-outputs }

當您的節點產出實體檔案，且希望在 UI 上為使用者提供可配置的檔名輸入框時，請使用 `ProjectFileParameter`。

```python
from griptape_nodes.exe_types.param_components.project_file_parameter import ProjectFileParameter
from griptape_nodes.exe_types.core_types import Parameter, ParameterMode
from griptape.artifacts.video_url_artifact import VideoUrlArtifact


class MyVideoNode(ControlNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)

        # 宣告標準輸出參數
        self.add_parameter(
            Parameter(
                name="output_video",
                output_type="VideoUrlArtifact",
                tooltip="生成的影片",
                allowed_modes={ParameterMode.OUTPUT},
            )
        )

        # 新增專案檔案參數以供設定輸出檔名
        # `situation` 宣告該節點歸屬的存檔情境。預設為 "save_node_output"，顯式傳入以提升可讀性
        self._output_video_file = ProjectFileParameter(
            node=self,
            name="output_video_file",
            default_filename="output_video.mp4",
            situation="save_node_output",
        )
        self._output_video_file.add_parameter()

    def process(self) -> None:
        # ... 生成 video_bytes 位元組資料 ...

        # 調用 build_file() 獲取 ProjectFileDestination 目的地實例
        dest = self._output_video_file.build_file()
        saved = dest.write_bytes(video_bytes)

        # 使用保存完成的解析位置指派輸出參數
        self.parameter_output_values["output_video"] = VideoUrlArtifact(saved.location)
```

**關鍵原則：**

- `ProjectFileParameter` 會在節點 UI 上自動建立一個供使用者輸入檔名的參數控制項。
- 調用 `build_file()` 獲取 `ProjectFileDestination` 實例。
- 調用 `write_bytes()` 將二進位位元組寫入磁碟。
- 透過 `saved.location` 存取解析完成的最終檔案 URL 或路徑。

**情境是如何決定的：**

`situation` 引數是節點宣告其所屬情境的唯一入口。它是建構子引數而非節點上的 UI 參數：使用者在節點表面永遠不會看到或編輯情境名稱。它預設為 `save_node_output`，因此省略也是合法的，但顯式傳入能讓閱讀程式碼的人一眼洞察其意圖。

由此產生的 UI 參數（慣例命名為 `output_file`）僅容納基礎檔名，而非完整路徑。`build_file()` 會將該檔名拆解為 `file_name_base` 與 `file_extension`，隨後透過情境巨集展開，因此參數的值僅是最終路徑的一部分。使用者在宣告為 `save_node_output` 的節點鍵入 `render.png`，最終將被寫入 `outputs/MyNode_render.png`。

**❌ 最常見致命錯誤：未捕獲 `write_bytes()` 的回傳值**

```python
# 錯誤寫法——切勿這樣做：
dest = self._output_video_file.build_file()
dest.write_bytes(video_bytes)  # ❌ 未捕獲回傳值
artifact = VideoUrlArtifact(dest.location)  # 錯誤：使用的是 dest 而非保存完成的 saved 物件

# 這將引發崩潰報錯："Failed because missing required variables: file_extension, file_name_base"
```

**為何會報錯**：諸如 `{file_extension}` 與 `{file_name_base}` 等巨集變數是在 `write_bytes()` 實際執行存檔並回傳 saved 物件時才動態解析完成的，而不是在調用 `build_file()` 時。在寫入前直接存取 `dest.location` 會導致巨集變數解析失敗。

```python
# 正確寫法：
dest = self._output_video_file.build_file()
saved = dest.write_bytes(video_bytes)  # ✅ 捕獲回傳值
artifact = VideoUrlArtifact(saved.location)  # 使用已完全解析完成的 location
```

### 模式 2：直接使用 ProjectFileDestination（適用於公用函式） {: #pattern-2-projectfiledestination-directly-for-utility-functions }

在背景輔助工具函式中，或當不需要提供使用者介面手動配置檔名時，可直接調用 `ProjectFileDestination.from_situation()`：

```python
from griptape_nodes.files.project_file import ProjectFileDestination
from griptape.artifacts.video_url_artifact import VideoUrlArtifact


def frames_to_video_artifact(frames: list, fps: int = 30, video_format: str = "mp4") -> VideoUrlArtifact:
    """將影格清單轉換為 VideoUrlArtifact。"""
    # ... 將影格編碼為 video_bytes ...

    # 透過專案系統保存
    dest = ProjectFileDestination.from_situation(filename=f"video.{video_format}", situation="save_node_output")
    saved = dest.write_bytes(video_bytes)

    return VideoUrlArtifact(saved.location)
```

## 從 StaticFilesManager 遷移 {: #migration-from-staticfilesmanager }

### 舊式廢棄寫法 (Deprecated) {: #old-pattern-deprecated }

```python
from griptape_nodes.retained_mode.griptape_nodes import GriptapeNodes
import uuid


def old_save_video(video_bytes: bytes) -> VideoUrlArtifact:
    filename = f"{uuid.uuid4()}.mp4"
    url = GriptapeNodes.StaticFilesManager().save_static_file(video_bytes, filename)
    return VideoUrlArtifact(url)
```

### 現代推薦寫法 {: #new-pattern }

```python
from griptape_nodes.files.project_file import ProjectFileDestination


def new_save_video(video_bytes: bytes) -> VideoUrlArtifact:
    dest = ProjectFileDestination.from_situation(filename="video.mp4", situation="save_node_output")
    saved = dest.write_bytes(video_bytes)
    return VideoUrlArtifact(saved.location)
```

**遷移帶來的好處：**

- 不再需要手動生成無意義的 UUID
- 跨所有節點保持井然有序的目錄結構
- 使用者可透過專案範本隨心自訂路徑規則
- 自動妥善處理檔名衝突問題 (Collision handling)

## 常用情境與適用場景 {: #common-situations-and-when-to-use-them }

- **`save_node_output`**：節點生成核心產物時的主要情境（影像、影片、音訊等）。
- **`copy_external_file`**：自外部來源匯入或複製檔案時。
- **`download_url`**：自外部網路 URL 下載檔案時。
- **`save_preview`**：生成縮圖或快速預覽圖時。
- **`save_static_file`**：在多次運行間保持不變的靜態資產。

預設情境規格、巨集與衝突原則請參閱 [情境指南](../../guides/projects/situations.md#default-situations)。

## 最佳實踐準則 {: #best-practices }

1. **一律使用專案系統**儲存檔案——嚴禁寫死本機絕對路徑。
1. **挑選合適模式**：需要使用者於 UI 調整時用 `ProjectFileParameter`，內部工具函式用 `ProjectFileDestination`。
1. **選用具備語意的情境**：選擇最能如實描述當前操作的情境標籤。
1. **讓巨集全權負責命名**：切勿自行拼接 UUID 或時間戳記——交由情境的巨集與衝突原則自動處理。
1. **妥善維護暫存檔案**：中間運算請使用 Python 的 `tempfile`，僅將最終結果寫入專案系統中。
1. **及時清理暫存資料**：在資料成功複製至專案系統後，務必在 `finally` 區塊刪除中間暫存檔。
