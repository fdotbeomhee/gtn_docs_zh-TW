# 專案系統 (Project) {: #project }

專案系統控制著您在 Griptape Nodes 中工作時所有檔案的組織結構、命名規則以及儲存方式。每當節點儲存圖像、複製已上傳的檔案或下載外部網址檔案時，專案系統決定了該檔案存放於何處以及如何命名。

## 存在的必要性 {: #why-it-exists }

若缺乏專案系統，每個檔案儲存操作皆必須指定寫死的絕對路徑。專案系統將寫死的硬編碼路徑替換為具備命名定義的範本，稱為**巨集 (Macros)**；並將各類檔案儲存情境定義為具名的場景，稱為**情境 (Situations)**。這讓您能在單一位置統一自訂整個專案的檔案架構，所有執行檔案儲存的節點皆會自動遵循此規範。

## 架構組成與層級關係 {: #how-the-pieces-fit-together }

```
workspace/                         ← 工作區根目錄 (你的最頂層)
  griptape-nodes-project.yml       ← 選填：您的自訂專案設定
  my_workflow/                     ← 工作流程目錄
    inputs/                        ← 預設輸入目錄
    outputs/                       ← 預設輸出目錄
    temp/                          ← 預設暫存目錄
    .griptape-nodes-previews/      ← 預設預覽縮圖目錄
```

概念層級架構如下：

```
workspace (工作區)
  └── project template (專案範本)
        ├── situations    (各情境下檔案儲存位置與命名規則)
        ├── directories   (邏輯名稱 → 相對路徑映射)
        ├── variables     (專案擁有的變數，可用於節點與巨集中)
        └── environment   (用於巨集構建的自訂鍵值對)
```

- **Workspace (工作區)**：當前專案所有工作任務的根目錄，於應用程式設定中配置。
- **Project template (專案範本)**：主導本專案規範的組態設定檔。在啟動時載入 — 首先載入系統預設值，接著載入透過 `parent_project_path` 宣告的[父專案 (Parent projects)](projects.md#parent-projects)，最後套用本專案 `griptape-nodes-project.yml` 中的覆寫設定。專案範本負責掌控檔案存放位置與命名規範。
- **Situations (情境規則)**：針對檔案讀取或寫入所定義的具名情境。每個情境包含決定檔案路徑的巨集範本，以及決定同名檔案已存在時如何處理的覆寫原則 (Policy)。
- **Directories (目錄架構)**：邏輯名稱至實體路徑的映射。每個目錄具備一個簡短名稱（如 `outputs`），可在巨集中作為變數引用。該名稱映射至磁碟上的真實路徑 — 預設情況下 `outputs` 映射至同名子資料夾。目錄路徑可為相對路徑或絕對路徑，且本身亦可包含巨集變數或環境變數參照。
- **Variables (專案變數)**：由專案專屬維護的具名數值（字串或整數），支援在節點參數與巨集中進行 `{VAR}` 替換。可設定為可寫入權限，方便創作者在執行期進行動態修改，變更結果會自動寫回專案設定檔中保存。
- **Environment (環境變數)**：可在巨集中引用的自訂鍵值對集合。
- **Macros (巨集語法)**：在情境規則與目錄設定中用以動態生成具體實體檔案路徑的範本字串。

## 本章節導覽 {: #pages-in-this-section }

- [專案管理圖形介面 (Managing Projects - GUI)](gui_guide.md) — 在 GUI 中建立、檢視、編輯、切換與移除專案
- [工作區 (Workspace)](workspace.md) — 根工作環境以及相對路徑如何解析
- [專案設定檔與繼承 (Projects)](projects.md) — 專案檔案格式與繼承合併模型
- [版本鎖定 (Version Pinning)](version_pinning.md) — 將專案鎖定於特定引擎版本與指定程式庫版本
- [巨集語法 (Macros)](macros.md) — 動態產生檔案路徑的範本語法完整手冊
- [圖像序列 (Sequences)](sequences.md) — 連號圖像序列命名模式與缺格影格處理機制
- [目錄架構 (Directories)](directories.md) — 邏輯目錄名稱至實體路徑映射
- [情境規則 (Situations)](situations.md) — 各類具名檔案儲存情境與重名處理原則
- [環境與內建變數 (Environment & Builtin Variables)](environment.md) — 自訂變數與系統內建環境數值
- [專案變數 (Project Variables)](variables.md) — 具備型別、權限與執行期寫入特性的專案變數
- [副檔名目錄分流 (File Extension Directories)](file_extension_directories.md) — 依檔案類型自動分流歸檔至指定資料夾
- [實用自訂指引 (Customization Guide)](customization.md) — 常見工作流程自訂範例與實戰教學
