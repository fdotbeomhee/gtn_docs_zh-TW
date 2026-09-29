---
name: griptape-nodes-workflows
description: 透過操作引擎的 MCP 伺服器來構建、執行與檢查 Griptape Nodes 工作流程。當使用者要求構建節點工作流程、執行既有流程、設定參數值、在節點間建立連線或讀取執行產出時使用。觸發情境包含「構建工作流程」、「新增節點至流程」、「連接這些節點」、「執行流程」、「節點 X 輸出了什麼」。
---

# Griptape Nodes 工作流程構建指南 (Workflow Construction Guide) {: #griptape-nodes-workflow-construction-guide }

本技能涵蓋了面向引擎 MCP 伺服器的完整冷啟動週期（構建 → 連線 → 執行 → 讀取），以及從實際工作流程運行中總結出的核心範式與常見陷阱。

## 心智模型 (Mental Model) {: #mental-model }

- **工作流程 (Workflow)**：頂層命名空間。同一時間**僅允許一個**處於活動中狀態。透過 `ClearAllObjectStateRequest` 進行全域重設。
- **流程 (Flow)**：工作流程內部的畫布。一個工作流程具備精確的一個頂層「畫布」流程。支援子流程，但通常極少需要。
- **節點 (Node)**：具備參數（輸入、輸出、屬性）的最小運算工作單元。
- **連線 (Connection)**：兩個參數之間的拓撲邊。共有兩種類型：
    - **資料流 (Data flow)**：一個節點的型別參數 → 另一個節點的型別參數。引擎會自動自資料依賴關係推導執行順序，因此通常僅需資料連線即可。
    - **控制流 (Control flow)**：`exec_out` → `exec_in`。僅當兩個節點必須按特定次序執行但彼此不共享資料時（如副作用步驟、條件分支）才需要。預設可直接略過。
- **當前情境堆疊 (Current Context)**：包含 (workflow → flow → node)。多數請求在省略名稱時預設指向「當前活動」實例。

## MCP 伺服器所暴露的真實介面 {: #what-the-mcp-server-actually-exposes }

每個 MCP 工具與 `SUPPORTED_REQUEST_EVENTS` (`src/griptape_nodes/servers/mcp.py`) 中註冊的 `RequestPayload` 類別呈 1:1 對應。工具名稱會自動冠上伺服器前綴，因此 `CreateNodeRequest` 對應的工具名稱為 `griptape_nodes_CreateNodeRequest`。核心特徵：

- **個別請求不存在複數形式。** 不存在 `CreateNodesRequest` 或 `CreateConnectionsRequest`。建立 N 個節點需發送 N 次 `CreateNodeRequest` 調用——或將其封裝至單一 `EventRequestBatch` 批次呼叫中（詳見下文）。
- **`CreateNodeRequest` 不接受初始參數值。** 設定參數一律是在節點建立後發送獨立的 `SetParameterValueRequest`。建立時不存在 `parameter_values` / `inputs` 捷徑。
- **`EventRequestBatch` 是唯一的批次散發原語。** 它是專門的合成工具（無對應的單一 `RequestPayload` 類別），可在單一傳輸訊框中傳遞有序的內部請求清單。

## 優先調查工作區環境 (Survey the Workspace First) {: #survey-the-workspace-first }

在構建任何流程前，請務必先查明當前引擎實際載入了哪些程式庫。已註冊的程式庫與其暴露的節點型別清單，是後續所有步驟的基礎真值來源，提早花費少量網路往返進行調查，能徹底避免猜測不存在的節點名稱所引發的錯誤。

### 在磁碟上尋找工作區目錄 {: #find-the-workspace-directory-on-disk }

MCP 介面並未直接暴露 `GetConfigValueRequest`，因此請直接讀取使用者的設定檔來解析工作區路徑。在 macOS / Linux 上其位置為：

```
~/.config/griptape_nodes/griptape_nodes_config.json
```

核心設定鍵：

- `workspace_directory`：工作區絕對根目錄（或帶有 `~` 前綴）。沙盒程式庫即位於此目錄下。
- `app_events.on_app_initialization_complete.libraries_to_register`：本機註冊的 `griptape_nodes_library.json` 路徑清單。

沙盒子目錄鍵 (`sandbox_library_directory`) 選填且預設為 `sandbox_library`。因此沙盒的絕對路徑為 `<workspace_directory>/<sandbox_library_directory>`。

### 透過 MCP 調查已註冊的程式庫 {: #survey-the-registered-libraries-via-mcp }

```
A. griptape_nodes_ListRegisteredLibrariesRequest()
   → 目前載入的程式庫名稱清單（如 "Griptape Nodes Library"、"Sandbox Library"）。

B. griptape_nodes_ListNodeTypesInLibraryRequest(library="<name>")
   → 獲取各程式庫暴露的精確節點型別名稱字串。

C. (選用) griptape_nodes_ListCategoriesInLibraryRequest(library="<name>")
   → 探索程式庫內的分類清單。
```

## EventRequestBatch：將建置階段壓縮為單次往返 {: #eventrequestbatch-collapse-a-build-phase-into-one-round-trip }

`EventRequestBatch`（MCP 工具名稱：`griptape_nodes_EventRequestBatch`）可將多個有序的內部請求封裝於單一傳輸訊框中。當建置拓撲的規格已敲定時即可採用此工具——典型範式為：N 個 `CreateNodeRequest` + M 個 `SetParameterValueRequest` + K 個 `CreateConnectionRequest` + 1 個 `AutoLayoutFlowRequest`，全數於一次往返內完成。

資料結構範例：

```json
{
  "requests": [
    {"request_type": "CreateNodeRequest",
     "request": {"node_type": "TextInput", "node_name": "TextInput_1"}},
    {"request_type": "SetParameterValueRequest",
     "request": {"node_name": "TextInput_1", "parameter_name": "text", "value": "..."}}
  ],
  "timeout_ms": 60000
}
```

運作行為特徵：

- **循序調度執行。** 引擎依提交順序依序 `await` 各個內部請求，因此在同一批次中先建立節點緊接著設定該節點參數是百分之百安全的。
- **預檢校驗。** 在傳輸前會針對對應的 `RequestPayload` 類別進行建構校驗。非法的 `request_type` 或參數錯誤會直接在上游拒絕整個批次。
- **單一槽位錯誤隔離。** 批次發送後，某個槽位的失敗**不會**中斷其他兄弟請求的執行。失敗槽位回傳 `{"ok": false, "details": "..."}`。請逐一檢查結果陣列。
- **禁止巢狀嵌套。** 批次內部不得再包含批次。

### 關鍵技巧：預先為後續槽位引用的節點明確命名 {: #critical-idiom-pre-name-nodes-you-reference-later-in-the-same-batch }

在單一批次內部，您無法在提交後續槽位前動態讀取先前槽位自動產生的節點名稱。因此最佳策略為：

- 在每個 `CreateNodeRequest` 上傳入明確的 `node_name`，並在隨後的 `SetParameterValueRequest` 與 `CreateConnectionRequest` 中如實複用該名稱。

## 經典冷啟動構建食譜 (Canonical Cold-Start Recipe) {: #canonical-cold-start-recipe }

以標準的 3 節點線性管線 (`TextInput → Agent → DisplayText`) 為例：

```
1. griptape_nodes_EnsureWorkflowAndFlowRequest()
   → 回傳 workflow_name、flow_name。具備冪等性。

2. griptape_nodes_DescribeNodeTypeRequest(node_type="TextInput")
3. griptape_nodes_DescribeNodeTypeRequest(node_type="Agent")
4. griptape_nodes_DescribeNodeTypeRequest(node_type="DisplayText")
   → 獲取精確的參數名稱、型別與模式。

5. griptape_nodes_CreateNodeRequest(node_type="TextInput")
6. griptape_nodes_CreateNodeRequest(node_type="Agent")
7. griptape_nodes_CreateNodeRequest(node_type="DisplayText")
   → 建立各個節點，讀取回傳的 node_name。

8. griptape_nodes_SetParameterValueRequest(
       node_name="TextInput_1", parameter_name="text", value="...")

9. griptape_nodes_CreateConnectionRequest(
       source_node_name="TextInput_1",   source_parameter_name="text",
       target_node_name="Agent_1",       target_parameter_name="prompt")
10. griptape_nodes_CreateConnectionRequest(
       source_node_name="Agent_1",       source_parameter_name="output",
       target_node_name="DisplayText_1", target_parameter_name="text")

11. griptape_nodes_AutoLayoutFlowRequest()
    → 多節點構建後必備步驟。自動進行拓撲排序並分配畫布座標，避免節點重疊於 (0, 0)。

12. griptape_nodes_StartFlowRequest(wait_for_completion=True, completion_timeout_ms=60000)
    → 阻塞直至流程運算完成。

13. griptape_nodes_GetParameterValueRequest(node_name="DisplayText_1", parameter_name="text")
    → 讀取終端節點的輸出結果。
```

### 批次優化版（僅需 4 次往返） {: #batched-variant-4-round-trips }

```
1. EnsureWorkflowAndFlowRequest                        (1 次調用)
2. EventRequestBatch([                                  (1 次調用，循序查詢)
     DescribeNodeTypeRequest("TextInput"),
     DescribeNodeTypeRequest("Agent"),
     DescribeNodeTypeRequest("DisplayText"),
   ])
3. EventRequestBatch([                                  (1 次調用，循序建置)
     CreateNodeRequest(node_type="TextInput",   node_name="TextInput_1"),
     CreateNodeRequest(node_type="Agent",       node_name="Agent_1"),
     CreateNodeRequest(node_type="DisplayText", node_name="DisplayText_1"),
     SetParameterValueRequest(node_name="TextInput_1", parameter_name="text", value="..."),
     CreateConnectionRequest(source_node_name="TextInput_1",   source_parameter_name="text",
                             target_node_name="Agent_1",       target_parameter_name="prompt"),
     CreateConnectionRequest(source_node_name="Agent_1",       source_parameter_name="output",
                             target_node_name="DisplayText_1", target_parameter_name="text"),
     AutoLayoutFlowRequest(),
   ])
4. StartFlowRequest(wait_for_completion=True, completion_timeout_ms=60000)
   + GetParameterValueRequest("DisplayText_1", "text")  (2 次調用)
```

## 核心設計範式 (Key Idioms) {: #key-idioms }

- **挑選節點前先調查工作區**。透過設定檔與清單 API 確認節點型別真實存在及其所屬程式庫。
- **建立連線前先查詢結構 (Discover before wiring)**。務必先針對目標節點型別調用 `DescribeNodeType`。
- **已知規格後採用 `EventRequestBatch` 批次提交**。
- **優先連接資料流，避免無謂的控制流**。引擎全自動依資料依賴性推導次序。
- **多節點建立後務必調用 AutoLayout**。
- **在 `StartFlowRequest` 上使用 `wait_for_completion=True`**。
- **確實讀取回應，切勿猜測命名**。

## 常用工具速查表 (Tool Cheat Sheet) {: #tool-cheat-sheet }

| 達成目標 | MCP 工具名稱 |
| :--- | :--- |
| 自空白冷啟動建立 workflow 與 flow | `EnsureWorkflowAndFlowRequest` |
| 單次往返批次執行 N 個請求 | `EventRequestBatch` |
| 探索探索程式庫與節點型別 | `ListRegisteredLibrariesRequest`, `ListNodeTypesInLibraryRequest` |
| 檢查節點型別的參數架構 | `DescribeNodeTypeRequest` |
| 建立節點實例 | `CreateNodeRequest` |
| 連接參數拓撲邊 | `CreateConnectionRequest` |
| 建置後自動排版畫布 | `AutoLayoutFlowRequest` |
| 設定參數數值 | `SetParameterValueRequest` |
| 讀取參數輸出值 | `GetParameterValueRequest` |
| 同步執行流程直至完成 | `StartFlowRequest(wait_for_completion=True, completion_timeout_ms=...)` |
| 自磁碟 Python 原始碼註冊沙盒節點 | `RegisterSandboxNodeFromSourceRequest` |
| 全域徹底重設清空 | `ClearAllObjectStateRequest(i_know_what_im_doing=True)` |
