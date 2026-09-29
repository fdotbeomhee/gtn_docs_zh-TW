# 節點開發快速上手指南 {: #getting-started-with-node-development }

!!! tip "致 AI 助理與程式碼生成智慧體 (AI Coding Agents)"

    本指南提供專為 AI 程式碼輔助工具後處理的 Markdown 格式。本站點公開了完整的機器可讀介面；索引請參閱 [智慧體使用指南](../../for_agents.md)。

    - **快速上手** (本頁面)：[Markdown](https://docs.griptapenodes.com/en/stable/development/custom_nodes/getting_started/index.md)
    - **概述**：[Markdown](https://docs.griptapenodes.com/en/stable/development/custom_nodes/index.md)
    - **範例程式碼**：[檢視 Python 範例](https://raw.githubusercontent.com/griptape-ai/griptape-nodes/main/docs/development/custom_nodes/example_control_node.py)

    **調用語法：** 將您的 AI 助理指向這些 URL 並下達如下提示詞：
    `"請閱讀此節點開發指南：[URL]，並協助我建構自訂節點"`

本頁面專為**剛接觸 Griptape Nodes 生態系統**、希望胸有成竹打造自訂節點的開發者量身打造。

它是通往本章節其餘深度、詳盡技術文件的一扇新手友善「大門」——完整參考導覽請參閱 [開發概述](index.md)。

## 動筆編寫程式碼前的核心心智模型 {: #what-youll-build-mentally-before-you-write-code }

從宏觀架構來看：

- **節點 (Node)** 是一個定義了**參數 (Parameters)**（輸入/輸出/屬性）與 `process()` 計算方法的 Python 類別。
- **工作流程 (Workflow / Flow)** 是一個由參數拓撲連線相連的節點有向圖 (Graph)。
- 參數同時扮演著雙重角色：
    - **UI 視覺化元件**（使用者在介面中所看到、編輯與手動連線的部件），以及
    - **強型別檢查連線點**（嚴格限定什麼型別能與什麼型別相接）。

### 挑選合適的基礎節點型別 {: #choose-the-right-base-node-type }

- **`DataNode`**：當您的節點純粹處理數據轉換且無需分支控制執行時選用。
- **`ControlNode`**：當您的節點需要顯式的時序控制（`exec_in` / `exec_out` 控制連線）時選用。
- **`SuccessFailureNode`**：當您希望為成功與失敗狀態提供獨立的控制流輸出分支時選用。
- **迭代迴圈節點**：引擎內建的迴圈基底建立在 `BaseIterativeStartNode` / `BaseIterativeEndNode` 之上。

若您尚不確定，建議先從 `DataNode` 起步，只有在真正需要時再升級至 `ControlNode` 或迴圈節點。

## 快速上手（推薦路徑） {: #quick-start-recommended-path }

若您從未建構過 Griptape 節點，這是通往可用節點最迅速的捷徑：

- 從 [程式庫範本存放庫](https://github.com/griptape-ai/griptape-nodes-library-template/) 起步（詳見 [開發概述](index.md#start-from-the-template-repository)）。
- 優先打造單一 `DataNode`（暫不涉及控制流程）。
- 針對常見資料型別，優先採用內建的 `Parameter*` 輔助包裝類別。
- 透過 `validate_before_node_run()` 進行嚴格的執行前輸入驗證。
- 需要存取憑據金鑰時，使用 `GetSecretValueRequest`，切勿直接調用 `GriptapeNodes.SecretsManager()`。

## 您的第一個節點（極簡範例） {: #your-first-node-minimal-example }

這是您可以構建的最精簡但功能完整的節點：讀取字串、將其轉為大寫並輸出字串。

```python
from griptape_nodes.exe_types.core_types import Parameter, ParameterMode
from griptape_nodes.exe_types.node_types import DataNode


class UppercaseText(DataNode):
    def __init__(self, **kwargs) -> None:
        # 務必調用父類別建構子，以便引擎初始化節點內部狀態並註冊節點上下文
        super().__init__(**kwargs)

        # add_parameter(...) 向節點註冊一個 Parameter。
        # 參數定義了：
        # - 使用者可於介面配置的值 (PROPERTY 模式)
        # - 可接收其他節點連入的端點 (INPUT 模式)
        # - 可連線輸出至其他節點的端點 (OUTPUT 模式)
        self.add_parameter(
            Parameter(
                name="text",
                # 參數的 "type" 是其在引擎中的主要資料型別。
                # 它決定了 UI 預設呈現方式與連線型別檢查。
                type="str",
                # input_types 管控哪些型別可以連入此參數。
                # 若需要彈性連線，可允許多個連入型別。
                input_types=["str"],
                # default_value 是當未連接任何節點且使用者未在 UI 中設定值時的預設值。
                default_value="Hello Griptape Nodes",
                allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
                tooltip="輸入文字",
            )
        )

        self.add_parameter(
            Parameter(
                name="uppercased",
                type="str",
                output_type="str",
                allowed_modes={ParameterMode.OUTPUT},
                tooltip="大寫轉換後的輸出文字",
            )
        )

    def process(self) -> None:
        # 當節點在流程中執行時會調用 process()。
        # 透過 get_parameter_value(...) 讀取輸入，並透過 parameter_output_values 寫入輸出。
        text = self.get_parameter_value("text") or ""
        self.parameter_output_values["uppercased"] = text.upper()
```

### 隨建隨測 (Test as you go) {: #test-as-you-go }

- 在新增或編輯節點後，將其放入簡單的測試流程中執行並驗證：
    - 參數介面呈現是否正確（輸入端、屬性列、輸出端）
    - 輸出值是否如預期即時更新於 UI 中
    - 驗證錯誤訊息是否具備清晰的行動指引

## 參數系統：實戰指南 {: #parameters-the-practical-model }

每個參數皆可在三種「模式 (Modes)」下運作：

- **Input (輸入)**：接收來自其他節點的輸出連線
- **Output (輸出)**：提供連線給其他節點的輸入端
- **Property (屬性)**：在節點 UI 面板中供使用者直接填寫或調整的數值

### 善用 `Parameter*` 常用輔助型別 {: #use-parameter-helpers-for-common-cases }

核心引擎在 `griptape_nodes.exe_types.param_types.*` 底下提供了高度封裝的參數輔助建構子，例如：

- `ParameterString`、`ParameterInt`、`ParameterFloat`、`ParameterBool`
- `ParameterJson`、`ParameterDict`、`ParameterRange`
- `ParameterImage`、`ParameterAudio`、`ParameterVideo`、`Parameter3D`
- `ParameterButton`

這些輔助類別極為實用，原因在於它們：

- 預先固化了預期的 `type` / `output_type` 與常見的 `ui_options`
- 大多支援 `accept_any=True` 以達成安全的型別強制轉換
- 將許多常用的 UI 選項直接封裝為 Python 屬性，方便於執行階段動態更新

若需速查，請參閱參數參考指南中的 [參數輔助建構子](parameters.md#parameter-helper-constructs-parameterstring-parameterint)。

### 容器型別：`ParameterList` 與 `ParameterDictionary` {: #containers-parameterlist-and-parameterdictionary }

- **`ParameterList`**：當您希望在節點 UI 中管理「多個同型別項目」時使用。
    - 讀取方式：`get_parameter_list_value()` 會自動展平 (Flatten) 巢狀可迭代物件。
    - 注意：目前實作會自動過濾掉 Falsey 項目（例如 `0`、`False`）。若需完整保留這些數值，請使用 `get_parameter_value()` 並自行手動展平。
- **`ParameterDictionary`**：當您需要在 UI 中提供有序鍵值對 (Key/Value) 列表時使用。

### 特徵標籤 (Traits)：UI 行為與數值驗證 {: #traits-ui-behaviors-and-validation }

特徵標籤 (Traits) 附加在參數上，用於賦予參數豐富的 UI 表現與互動行為。

常見的核心特徵標籤包括：

- `Options(...)`：下拉選單（選項清單儲存於 `ui_options` 中以確保序列化穩定）
- `Slider(min_val, max_val)`：數值滑桿 UI 介面 + 範圍約束驗證
- `FileSystemPicker(...)`：檔案/目錄選取器（支援副檔名篩選與工作區範圍限制）

範例：建立一個帶有滑桿的浮點數參數：

```python
from griptape_nodes.exe_types.core_types import Parameter, ParameterMode
from griptape_nodes.traits.slider import Slider

self.add_parameter(
    Parameter(
        name="temperature",
        type="float",
        default_value=0.7,
        tooltip="採樣溫度（數值越高越隨機）",
        # 若希望同時允許手動輸入與節點連線，可同時宣告 INPUT 與 PROPERTY
        allowed_modes={ParameterMode.INPUT, ParameterMode.PROPERTY},
        # 程式碼庫中的通用模式：透過 `traits` 引數內嵌附加特徵
        traits={Slider(min_val=0.0, max_val=2.0)},
    )
)
```

## 驗證、錯誤處理與使用者體驗 {: #validation-error-handling-and-user-experience }

對於新手開發者，推薦遵循以下預設原則：

- 使用 `validate_before_node_run()` 進行執行前的參數邊界檢查。
- 儘早拋出明確且具備指引性的錯誤訊息（告訴使用者應連接何種端點或配置何種數值）。
- 若節點運算可能失敗但您希望整個流程維持運轉，請繼承 `SuccessFailureNode` 並顯式分流失敗路徑。

文件中可供參考的範例：

- [Start Flow](../../nodes/execution/start_flow.md) 與 [End Flow](../../nodes/execution/end_flow.md) 展示了控制流概念與狀態回報機制。

## 開發避坑指南 {: #common-gotchas }

- **`get_parameter_list_value()` 會過濾 Falsey 元素**：若您的清單可能合法包含 `0` 或 `False`，請改用 `get_parameter_value()` 並自行手動處理。
- **`ui_options` 優先權衝突**：若同時傳入 `hide=...` 與 `ui_options={"hide": ...}`，將一律以 `ui_options` 的設定為準。
- **機密安全**：絕不要在程式碼中寫死 API 金鑰。請使用 `GriptapeNodes.handle_request(GetSecretValueRequest(key=...))`。

## 機密金鑰與環境設定 {: #secrets-and-configuration }

當自訂節點需要存取 API 金鑰或其他機密資訊時：

- 在程式庫設定檔 (`griptape_nodes_library.json`) 中登記機密鍵名。
- 透過 `GriptapeNodes.handle_request(GetSecretValueRequest(key="NAME"))` 讀取金鑰。當節點在 Worker 獨立處理程序中執行時，直接調用管理器會被拒絕；而透過請求機制則由掌管機密的引擎核心跨處理程序安全回應。

## 真實參考範例存放處 {: #where-to-look-for-real-examples }

- **標準庫節點原始碼**：`libraries/griptape_nodes_library/griptape_nodes_library/`
- **引擎核心內部實作**（進階）：`src/griptape_nodes/`

## 後續延伸閱讀 {: #next-steps }

- 閱讀更深層的技術參考指南：[參數系統](parameters.md)、[執行與生命週期](execution_and_lifecycle.md) 以及本章節的其餘頁面。
- 瀏覽標準庫中的實作節點，直接參考符合您應用場景的最佳模式。
