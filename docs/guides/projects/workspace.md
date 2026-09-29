# 工作區 (Workspace) {: #workspace }

工作區是您所有工作任務的根目錄，亦是解析所有相對檔案路徑的基礎起點。

## 組態設定檔 (Config files) {: #config-files }

引擎的組態設定統一儲存於名為 `griptape_nodes_config.json` 的檔案中。以下三個實體存放位置具備明確意義：

| 檔案路徑 | 作用說明 |
| ----------------------------------------------------- | -------------------------------------------------------------------------------- |
| `~/.config/griptape_nodes/griptape_nodes_config.json` | 使用者組態 (User config) — 本機電腦的全域設定 |
| `<project_dir>/griptape_nodes_config.json` | 專案同級組態 (Project-adjacent config) — 隨專案設定檔一同分發的共用預設值 |
| `<workspace_dir>/griptape_nodes_config.json` | 工作區組態 (Workspace config) — 針對當前活躍專案的使用者個別覆寫項目 |

## 組態解析優先順序 {: #config-resolution-order }

各設定項依以下順序依序解析（後者覆蓋前者）：

1. 原生內建預設值 (Built-in defaults)
1. 使用者組態 (`~/.config/griptape_nodes/griptape_nodes_config.json`)
1. 專案同級組態 (`<project_dir>/griptape_nodes_config.json`)
1. 工作區組態 (`<workspace_dir>/griptape_nodes_config.json`)
1. 環境變數覆寫 (`GTN_CONFIG_*`)

當工作區目錄與專案目錄完全相同時（獨立自足型專案），第 3 層與第 4 層指向同一個檔案 — 該檔案只會被載入一次。

## 工作區路徑解析規則 {: #workspace-resolution }

當專案載入時，其工作區目錄由以下來源決定，**優先順序由高至低** — 最先提供有效值的來源勝出：

1. 專案自訂的 `workspace_dir` 欄位（詳見下文[專案宣告的工作區](#project-declared-workspace-workspace_dir)以及 [`workspace_dir` 欄位參考](projects.md#workspace-directory)）
1. 使用者組態中的 `project_workspaces` 映射條目（詳見下文[針對個別專案的工作區覆寫](#per-project-workspace-overrides)）
1. 環境變數 `GTN_CONFIG_WORKSPACE_DIRECTORY`
1. 專案同級組態中的 `workspace_directory` 鍵值
1. 沿著明確宣告的[父專案繼承鏈 (Parent-project chain)](projects.md#parent-projects)向上回溯，最近祖先專案所解析的工作區
1. 使用者組態中的全域 `workspace_directory`；若皆未配置，則自動退回至包含該專案檔案的實體資料夾（自動預設值）

當完全未涉及任何專案檔案時，工作區直接取自全域 `workspace_directory`（或引擎目前工作目錄下原生內建的 `GriptapeNodes/` 預設資料夾）。

## 針對個別專案的工作區覆寫 {: #per-project-workspace-overrides }

使用者個人組態中的 `project_workspaces` 設定項，可將專案檔案路徑映射至特定的本機工作區目錄覆寫項。當共用專案在每位團隊成員的電腦上需要解析至各自不同的本機工作區時，非常適用此機制：

```json
{
  "project_workspaces": {
    "//NAS/Projects/ProjectA/griptape-nodes-project.yml": "/Users/collin/ProjectA/",
    "//NAS/Projects/ProjectB/griptape-nodes-project.yml": "/Users/collin/ProjectB/"
  }
}
```

鍵名為專案檔案解析後的絕對路徑。當專案載入時，若其解析路徑與某個鍵名相符，則採用對應的值作為實體工作區目錄。

## 專案宣告的工作區 (`workspace_dir`) {: #project-declared-workspace-workspace_dir }

專案可在其專案設定檔中透過 `workspace_dir` 欄位直接宣告自身專屬的工作區目錄。這是**優先權最高**的工作區來源：它會覆寫每位使用者的 `project_workspaces` 映射、`GTN_CONFIG_WORKSPACE_DIRECTORY` 環境變數、專案同級組態、父專案繼承以及全域預設值。

當某個專案無論每位使用者的本機層級設定為何，皆應一律解析至固定的工作區時，請採用此設定。與 `project_workspaces`（每位使用者皆必須在自己的本機設定中配置）不同，`workspace_dir` 直接儲存於專案檔案內部隨行攜帶。

當您建立 **v1** 規格的專案時，建立介面會自動寫入 `workspace_dir: "./"`，因此預設情況下 v1 專案是獨立自足的 — 其工作區即為其專案資料夾本身。清空此欄位（或省略它）會讓專案恢復繼承父專案或全域預設工作區。參閱[綱要版本 (Schema versions)](projects.md#schema-versions)。

該設定值可為單一路徑，亦可為跨平臺映射物件：

```yaml
# 單一路徑：絕對路徑，或相對於本專案檔案所在目錄的相對路徑。
workspace_dir: "./workspace"

# 或是跨平臺映射；系統會採用當前平臺的鍵值，若無相符則退回 `default`。
workspace_dir:
  darwin: "/Volumes/fast/ProjectA"
  windows: "D:/ProjectA"
  default: "./workspace"
```

相對路徑會相對於包含該專案檔案的資料夾進行解析（與 `parent_project_path` 解析方式完全相同），因此使用 `workspace_dir: "./workspace"` 的專案具備高度可攜性：移動或複製專案資料夾，工作區亦會自動隨之遷移。目標目錄無需事先包含 `griptape_nodes_config.json` — 空資料夾亦屬合法目標。

完整綱要、跨平臺退回規則以及與父專案搭配的行為，請參閱 [`workspace_dir` 欄位參考手冊](projects.md#workspace-directory)。

## 實務應用場景解析 {: #example-scenarios }

### 場景 1：個人獨立開發者，無專案檔案 {: #scenario-1-solo-developer-no-project-file }

無專案檔案，亦無專案同級或工作區組態。工作區直接取自使用者個人全域組態或原生預設值。

```
~/.config/griptape_nodes/
  griptape_nodes_config.json    <- workspace_directory: ~/GriptapeNodes

~/GriptapeNodes/                <- 工作區
  griptape_nodes_config.json    <- 選填的工作區層級覆寫
```

### 場景 2：獨立自足的可攜式專案 {: #scenario-2-self-contained-portable-project }

專案檔案與設定檔存放於同一目錄中。工作區自動預設為該專案目錄。將整個資料夾複製至隨身碟或另一台電腦 — 完全無需修改任何設定即可運作。

```
/My_Indie_Short/
  griptape-nodes-project.yml
  griptape_nodes_config.json    <- 專案同級與工作區組態（同一個檔案）
  inputs/
  outputs/
```

### 場景 3：團隊共用專案，個別成員獨立工作區 {: #scenario-3-shared-project-per-user-workspaces }

專案檔案存放於共用網路磁碟機 (NAS) 上。每位團隊成員透過個人組態中的 `project_workspaces` 將其映射至各自的本機工作區。每位成員的本機工作區中皆可放置獨立的 `griptape_nodes_config.json` 進行個人偏好覆寫。

```
//NAS/Projects/ProjectA/
  griptape-nodes-project.yml
  griptape_nodes_config.json    <- 團隊共用預設值（例如慣用模型設定）
```

成員 Collin 的個人組態：

```json
{
  "project_workspaces": {
    "//NAS/Projects/ProjectA/griptape-nodes-project.yml": "/Users/collin/ProjectA/"
  }
}
```

成員 James 的個人組態：

```json
{
  "project_workspaces": {
    "//NAS/Projects/ProjectA/griptape-nodes-project.yml": "C:\\Projects\\ProjectA\\"
  }
}
```

每位使用者可在其本機工作區資料夾中放置 `griptape_nodes_config.json`，其實體個人偏好設定將優先於共用網路磁碟上的專案同級組態。

### 場景 4：工作室統一強制指定工作區 {: #scenario-4-studio-mandated-workspace }

專案同級組態直接指定共用工作區。凡未配置 `project_workspaces` 的創作者皆統一套用工作室預設值。位於共用磁碟上的工作區組態可承載進一步的共用設定。

```
//NAS/Projects/ProjectA/
  griptape-nodes-project.yml
  griptape_nodes_config.json    <- workspace_directory: //NAS/Workspaces/ProjectA/

//NAS/Workspaces/ProjectA/
  griptape_nodes_config.json    <- 共用工作區層級設定
```

算圖農場 (Render farm) 節點機器可透過 `GTN_CONFIG_WORKSPACE_DIRECTORY` 環境變數自訂覆寫工作區，完全無需修改任何共用檔案。

### 場景 5：使用者覆寫共用引擎組態 {: #scenario-5-user-overrides-shared-engine-config }

工作室在專案同級組態中宣告了 `log_level: "WARNING"`。某位開發者在本機需要除錯 `DEBUG` 等級。只需在其本機工作區組態中加入覆寫項目 — 工作區組態（第 4 層）會勝過專案同級組態（第 3 層）。

```
/Users/dev/ProjectA/
  griptape_nodes_config.json    <- {"log_level": "DEBUG"}
```

### 場景 6：多專案切換，不同本機工作區 {: #scenario-6-multiple-projects-different-workspaces }

`project_workspaces` 將各個共用專案分別映射至不同的本機工作區資料夾。切換當前活躍專案會同步切換實體工作區，並自動重新載入對應的工作區組態。

```json
{
  "project_workspaces": {
    "//NAS/ProjectA/griptape-nodes-project.yml": "/Users/dev/ProjectA/",
    "//NAS/ProjectB/griptape-nodes-project.yml": "/Users/dev/ProjectB/"
  }
}
```

### 場景 7：工作區自動探測 {: #scenario-7-workspace-auto-discovery }

Griptape Nodes 啟動時會在工作區目錄中搜尋 `griptape-nodes-project.yml`。若找到，專案將被自動載入。這與場景 2 相同 — 工作區與專案目錄一致，單一 `griptape_nodes_config.json` 同時扮演專案同級與工作區組態的角色。

### 場景 8：專案自行鎖定工作區 {: #scenario-8-project-pins-its-own-workspace }

若專案需要一律解析至固定工作區，可透過 `workspace_dir` 直接宣告，完全無需在每位使用者的本機設定中配置 `project_workspaces`。採用相對路徑可維持跨電腦的高度可攜性。

```yaml
# griptape-nodes-project.yml
name: "ProjectA"
workspace_dir: "./workspace"
```

```
/ProjectA/
  griptape-nodes-project.yml    <- workspace_dir: ./workspace
  workspace/                    <- 解析後的工作區（按需自動建立）
```

由於 `workspace_dir` 具備最高優先權，它會凌駕於任何 `project_workspaces` 映射或環境變數之上。無論將 `ProjectA/` 資料夾移動到何處，工作區皆能穩健解析至其內部的 `workspace/` 子資料夾。

## 相對路徑如何解析 {: #how-paths-resolve }

工作流程中的相對檔案路徑一律以**工作區目錄 (Workspace directory)** 為基礎進行解析。若工作區為 `/Users/you/workspace/` 且情境規則巨集解析為 `outputs/render_001.png`，最終生成的實體絕對路徑即為 `/Users/you/workspace/outputs/render_001.png`。

在專案設定檔中撰寫的相對路徑，則一律相對於包含該專案 YAML 的資料夾進行解析，這也是上述 `workspace_dir: ./workspace` 能順利定位工作區的原因。

節點程式庫是唯一的例外：它們預設安裝並解析於工作區相對路徑下的 `libraries` 資料夾，但專案可透過 [`libraries_dir`](projects.md#libraries-directory) 欄位將其重新導向（並跨專案樹共用），不受工作區影響。

**專案根基目錄 (Project base directory)**（即包含 `griptape-nodes-project.yml` 的資料夾）以內建變數 `{project_dir}` 形式提供，但它不作為相對路徑的解析基準。當路徑管理器嘗試將某個絕對路徑反向映射回巨集格式、且該路徑落於專案資料夾內但位於所有具名目錄之外時，它會作為最後的備援退回基準。

## 工作區與專案設定檔的關聯 {: #workspace-and-the-project-file }

當 Griptape Nodes 開機啟動時，會於工作區目錄中尋找 `griptape-nodes-project.yml`。若找到，該檔案內容會覆蓋合併於系統原生預設值之上，生成當前活躍的專案範本。若未找到，則直接套用系統預設值。

專案設定檔與繼承合併模型的詳細規範請參閱[專案設定檔與繼承 (Projects)](projects.md)。

## 核心設定摘要 {: #summary }

| 設定項目 | 作用說明 |
| -------------------------------------------- | --------------------------------------------------- |
| `workspace_dir` | 專案宣告的工作區；最高優先權來源 |
| `workspace_directory` | 所有工作任務的根目錄 |
| `project_workspaces` | 使用者個人組態中的個別專案工作區映射覆寫 |
| `griptape-nodes-project.yml` | 選填的專案範本設定檔 |
| `<project_dir>/griptape_nodes_config.json` | 隨專案分發的選填共用組態 |
| `<workspace_dir>/griptape_nodes_config.json` | 選填的使用者個別工作區組態 |
| `GTN_CONFIG_WORKSPACE_DIRECTORY` | 環境變數覆寫（具備極高優先權） |
