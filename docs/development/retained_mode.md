# Griptape Nodes 保留模式腳本撰寫 {: #griptape-nodes-retained-mode-scripting }

「保留模式 (Retained Mode)」為與 Griptape Nodes 系統互動提供了一套功能完備的 Python 腳本介面。這讓使用者能透過簡潔優雅的 Python API 以程式化方式建立、修改與管理節點、參數、連線關係以及整個工作流程。

> **備註：** RetainedMode 的底層實際匯入指令為：
>
> ```python
> from griptape_nodes.retained_mode import RetainedMode as cmd
> ```
>
> 不過為了操作便利，在圖形介面 (GUI) 的腳本編輯器中，系統已自動為您預先完成了此項匯入，因此您可以直接隨意調用 `cmd.`。

## 腳本操作與執行方式 {: #working-with-scripts }

開發與執行腳本主要有兩種途徑：

1. **使用腳本編輯器 (Script Editor)**——直接在 Griptape Nodes 內建的腳本編輯器中編寫程式碼並即時執行，以動態修改或精確掌控您的工作流程。
1. **匯入外部腳本 (Importing External Scripts)**——將具備高重用性的腳本模組妥善存放於外部檔案中，在腳本編輯器中透過標準 Python 匯入機制載入，並調用其中的模組功能。

內建的腳本編輯器是進行所有程式化腳本操作的主要入口點。

腳本在自動化繁瑣重複任務時極具威力：包含連同屬性與連線完整複製節點、匯出節點設定值並批次匯入至其他流程、透過預設的節點與拓撲結構以程式化方式從零構建整套流程，或在迴圈中跨多個節點執行大量批次操作。您更可將保留模式與各類經典 Python 程式庫（如 pandas、numpy、requests 等）強強聯手，大幅拓展腳本的自動化能力疆界。

本頁面其餘部分為完整的 API 參考指南：包含流程管理、節點操作、參數維護、拓撲連線以及流程執行控制。

## 流程操作 (Flow Operations) {: #flow-operations }

### create_flow {: #create-flow }

在 Griptape 系統中建立一個全新流程。

```python
cmd.create_flow(flow_name=None, parent_flow_name=None)
```

#### 引數說明 {: #arguments }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | ⚪ |
| `parent_flow_name` | string | ⚪ |

#### 回傳值 {: #return-value }

攜帶流程建立狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description }

以指定的名稱建立新流程。若提供了 `parent_flow_name`，新流程將作為指定父流程的子流程建立。

---

### delete_flow {: #delete-flow }

```python
cmd.delete_flow(flow_name)
```

刪除既有的流程。

#### 引數說明 {: #arguments_1 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_1 }

攜帶流程刪除狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_1 }

從系統中永久移除指定的流程。

---

### get_flows {: #get-flows }

```python
cmd.get_flows(parent_flow_name=None)
```

列出父流程底下的所有流程。

#### 引數說明 {: #arguments_2 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `parent_flow_name` | string | ⚪ |

#### 回傳值 {: #return-value_2 }

包含流程清單的 `ResultPayload` 物件。

#### 功能說明 {: #description_2 }

回傳指定父流程內包含的所有子流程。若未提供 `parent_flow_name`，則列出所有的頂層流程。

---

### get_nodes_in_flow {: #get-nodes-in-flow }

```python
cmd.get_nodes_in_flow(flow_name, node_types=None)
```

列出流程內包含的所有節點，並支援依據 Python 類別名稱進行篩選。

#### 引數說明 {: #arguments_3 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |
| `node_types` | list[string] \| None | 🔴 |

#### 回傳值 {: #return-value_3 }

包含節點名稱清單的 `ResultPayload` 物件。

#### 功能說明 {: #description_3 }

回傳指定流程中存在的所有節點。當傳入 `node_types` 時，僅回傳 Python 類別名稱符合該清單項目的節點（例如 `["StartFlow", "Agent"]`）。省略 `node_types` 或傳入 `None` 則回傳所有節點。

---

### run_flow {: #run-flow }

```python
cmd.run_flow(flow_name)
```

執行指定的流程。

#### 引數說明 {: #arguments_4 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_4 }

攜帶流程執行狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_4 }

啟動並執行指定的流程。

---

### reset_flow {: #reset-flow }

```python
cmd.reset_flow(flow_name)
```

將流程重設為初始狀態。

#### 引數說明 {: #arguments_5 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_5 }

攜帶流程重設狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_5 }

清除並取消流程中所有節點的解析狀態 (Unresolve)，使流程完全回歸至初始就緒狀態。

---

### get_flow_state {: #get-flow-state }

```python
cmd.get_flow_state(flow_name)
```

回傳流程當前的執行狀態。

#### 引數說明 {: #arguments_6 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_6 }

包含流程狀態資訊的 `ResultPayload` 物件。

#### 功能說明 {: #description_6 }

讀取指定流程當下的運行與執行狀態。

---

### cancel_flow {: #cancel-flow }

```python
cmd.cancel_flow(flow_name)
```

取消流程的執行運算。

#### 引數說明 {: #arguments_7 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_7 }

攜帶流程取消狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_7 }

即刻中止並停止指定流程的執行。

---

### single_step {: #single-step }

```python
cmd.single_step(flow_name)
```

在流程中以單一步驟執行單一節點。

#### 引數說明 {: #arguments_8 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_8 }

攜帶單步執行狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_8 }

在指定流程中僅執行單一個運算節點。

---

### single_execution_step {: #single-execution-step }

```python
cmd.single_execution_step(flow_name)
```

在流程中執行單一執行步驟。

#### 引數說明 {: #arguments_9 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_9 }

攜帶執行步驟狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_9 }

在指定流程中推進執行單一執行步驟。

---

### continue_flow {: #continue-flow }

```python
cmd.continue_flow(flow_name)
```

繼續執行處於暫停狀態的流程。

#### 引數說明 {: #arguments_10 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `flow_name` | string | 🟢 |

#### 回傳值 {: #return-value_10 }

攜帶流程繼續執行狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_10 }

恢復並繼續運作先前處於暫停狀態的流程。

## 節點操作 (Node Operations) {: #node-operations }

### create_node {: #create-node }

```python
cmd.create_node(node_type, specific_library_name=None, node_name=None, parent_flow_name=None, metadata=None)
```

建立一個全新節點。

#### 引數說明 {: #arguments_11 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_type` | string | 🟢 |
| `specific_library_name` | string | ⚪ |
| `node_name` | string | ⚪ |
| `parent_flow_name` | string | ⚪ |
| `metadata` | dict | ⚪ |

#### 回傳值 {: #return-value_11 }

節點名稱或攜帶節點建立狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_11 }

建立指定型別的新節點。選填參數允許進一步明確指定程式庫來源、自訂節點名稱、所屬父流程以及附加中繼資料。

---

### delete_node {: #delete-node }

```python
cmd.delete_node(node_name)
```

刪除既有的節點。

#### 引數說明 {: #arguments_12 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |

#### 回傳值 {: #return-value_12 }

攜帶節點刪除狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_12 }

從系統中徹底移除指定的節點。

---

### run_node {: #run-node }

```python
cmd.run_node(node_name)
```

獨立執行單一節點。

#### 引數說明 {: #arguments_13 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |

#### 回傳值 {: #return-value_13 }

攜帶節點執行狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_13 }

解析並直接觸發執行指定的單一節點。

---

### get_resolution_state_for_node {: #get-resolution-state-for-node }

```python
cmd.get_resolution_state_for_node(node_name)
```

回傳節點當前的解析狀態。

#### 引數說明 {: #arguments_14 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |

#### 回傳值 {: #return-value_14 }

包含節點解析狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_14 }

讀取指定節點當前的解析就緒或完成狀態。

---

### get_metadata_for_node {: #get-metadata-for-node }

```python
cmd.get_metadata_for_node(node_name)
```

讀取節點的中繼資料。

#### 引數說明 {: #arguments_15 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |

#### 回傳值 {: #return-value_15 }

包含節點中繼資料的 `ResultPayload` 物件。

#### 功能說明 {: #description_15 }

取得與指定節點關聯的所有中繼資料字典。

---

### set_metadata_for_node {: #set-metadata-for-node }

```python
cmd.set_metadata_for_node(node_name, metadata)
```

設定節點的中繼資料。

#### 引數說明 {: #arguments_16 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |
| `metadata` | dict | 🟢 |

#### 回傳值 {: #return-value_16 }

攜帶中繼資料更新狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_16 }

為指定的節點覆寫或指派中繼資料。

---

### exists {: #exists }

```python
cmd.exists(node)
```

檢查節點是否存在。

#### 引數說明 {: #arguments_17 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node` | string | 🟢 |

#### 回傳值 {: #return-value_17 }

布林值，表示該節點是否存在。

#### 功能說明 {: #description_17 }

若系統中存在指定的節點則回傳 `True`，否則回傳 `False`。

---

### ls {: #ls }

```python
cmd.ls(**kwargs)
```

列出系統中的物件。

#### 引數說明 {: #arguments_18 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `**kwargs` | dict | ⚪ |

#### 回傳值 {: #return-value_18 }

符合篩選條件的物件名稱清單。

#### 功能說明 {: #description_18 }

列出系統中的各類物件，並可根據傳入的關鍵字參數進行自訂條件篩選。

## 參數操作 (Parameter Operations) {: #parameter-operations }

### list_params {: #list-params }

```python
cmd.list_params(node)
```

列出節點上的所有參數。

#### 引數說明 {: #arguments_19 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node` | string | 🟢 |

#### 回傳值 {: #return-value_19 }

參數名稱字串清單。

#### 功能說明 {: #description_19 }

回傳指定節點上定義的所有參數清單。

---

### add_param {: #add-param }

```python
cmd.add_param(
    node_name,
    parameter_name,
    default_value,
    tooltip,
    type=None,
    input_types=None,
    output_type=None,
    edit=False,
    tooltip_as_input=None,
    tooltip_as_property=None,
    tooltip_as_output=None,
    ui_options=None,
    mode_allowed_input=True,
    mode_allowed_property=True,
    mode_allowed_output=True,
    **kwargs,
)
```

為節點新增參數。

#### 引數說明 {: #arguments_20 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |
| `parameter_name` | string | 🟢 |
| `default_value` | any | 🟢 |
| `tooltip` | string or list | 🟢 |
| `type` | string | ⚪ |
| `input_types` | list of strings | ⚪ |
| `output_type` | string | ⚪ |
| `edit` | boolean | ⚪ |
| `tooltip_as_input` | string or list | ⚪ |
| `tooltip_as_property` | string or list | ⚪ |
| `tooltip_as_output` | string or list | ⚪ |
| `ui_options` | ParameterUIOptions | ⚪ |
| `mode_allowed_input` | boolean | ⚪ |
| `mode_allowed_property` | boolean | ⚪ |
| `mode_allowed_output` | boolean | ⚪ |

#### 回傳值 {: #return-value_20 }

攜帶參數新增狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_20 }

根據所提供的各項配置為指定節點新增參數。若設定 `edit=True`，則改為修改已存在的同名參數。

---

### del_param {: #del-param }

```python
cmd.del_param(node_name, parameter_name)
```

從節點中移除參數。

#### 引數說明 {: #arguments_21 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |
| `parameter_name` | string | 🟢 |

#### 回傳值 {: #return-value_21 }

攜帶參數移除狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_21 }

從指定節點中刪除所選取的參數。

---

### param_info {: #param-info }

```python
cmd.param_info(node, param) 或 cmd.param_info("node.param")
```

讀取參數的詳細規格資訊。

#### 引數說明 {: #arguments_22 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node` | string | 🟢 |
| `param` | string | 🟢 |

#### 回傳值 {: #return-value_22 }

包含參數規格細節的 `ResultPayload` 物件。

#### 功能說明 {: #description_22 }

回傳指定參數的完整詳細資訊。可傳入分立的節點名與參數名，亦可傳入統一的 `"node.param"` 字串。

---

### get_value {: #get-value }

```python
cmd.get_value(node, param) 或 cmd.get_value("node.param")
```

讀取參數的當前數值。

#### 引數說明 {: #arguments_23 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node` | string | 🟢 |
| `param` | string | 🟢 |

#### 回傳值 {: #return-value_23 }

參數的實際數值，或回傳失敗結果。

#### 功能說明 {: #description_23 }

回傳指定參數的當前儲存值。支援容器型別的索引存取語法（例如 `"node.param[0]"`）。

---

### set_value {: #set-value }

```python
cmd.set_value(node, param, value) 或 cmd.set_value("node.param", value)
```

設定參數的數值。

#### 引數說明 {: #arguments_24 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node` | string | 🟢 |
| `param` | string | 🟢 |
| `value` | any | 🟢 |

#### 回傳值 {: #return-value_24 }

攜帶數值更新狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_24 }

設定指定參數的值。支援容器型別的索引存取語法（例如 `"node.param[0]"`）。

## 連線操作 (Connection Operations) {: #connection-operations }

### connect {: #connect }

```python
cmd.connect(source, destination)
```

在兩個參數之間建立拓撲連線。

#### 引數說明 {: #arguments_25 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `source` | string | 🟢 |
| `destination` | string | 🟢 |

#### 回傳值 {: #return-value_25 }

攜帶連線建立狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_25 }

從來源參數 (`source`) 連線至目的地參數 (`destination`)。兩者皆應採用 `"node.param"` 格式指定。

---

### exec_chain {: #exec-chain }

```python
cmd.exec_chain(*node_names)
```

在一系列節點之間建立執行控制連線鏈。

#### 引數說明 {: #arguments_26 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `*node_names` | string(s) | 🟢 |

#### 回傳值 {: #return-value_26 }

包含每個連線嘗試結果的字典物件。

#### 功能說明 {: #description_26 }

依序在傳入的節點串列間建立 `exec_out -> exec_in` 連線，將它們有效串聯以進行循序執行。

---

### delete_connection {: #delete-connection }

```python
cmd.delete_connection(source_node_name, source_param_name, target_node_name, target_param_name)
```

刪除參數之間的連線。

#### 引數說明 {: #arguments_27 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `source_node_name` | string | 🟢 |
| `source_param_name` | string | 🟢 |
| `target_node_name` | string | 🟢 |
| `target_param_name` | string | 🟢 |

#### 回傳值 {: #return-value_27 }

攜帶連線刪除狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_27 }

解除指定來源參數與目標參數之間的實體連線。

---

### get_connections_for_node {: #get-connections-for-node }

```python
cmd.get_connections_for_node(node_name)
```

列出節點的所有連線。

#### 引數說明 {: #arguments_28 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `node_name` | string | 🟢 |

#### 回傳值 {: #return-value_28 }

包含連線資訊的 `ResultPayload` 物件。

#### 功能說明 {: #description_28 }

回傳所有涉及該節點的輸入連線與輸出連線。

## 程式庫操作 (Library Operations) {: #library-operations }

### get_available_libraries {: #get-available-libraries }

```python
cmd.get_available_libraries()
```

列出所有可用的節點程式庫。

#### 引數說明 {: #arguments_29 }

無。

#### 回傳值 {: #return-value_29 }

包含程式庫名稱清單的 `ResultPayload` 物件。

#### 功能說明 {: #description_29 }

回傳系統中所有已成功註冊的節點程式庫。

---

### get_node_types_in_library {: #get-node-types-in-library }

```python
cmd.get_node_types_in_library(library_name)
```

列出特定程式庫中的所有節點型別。

#### 引數說明 {: #arguments_30 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `library_name` | string | 🟢 |

#### 回傳值 {: #return-value_30 }

包含節點型別名稱清單的 `ResultPayload` 物件。

#### 功能說明 {: #description_30 }

回傳指定程式庫所提供的所有可用節點型別。

---

### get_node_metadata_from_library {: #get-node-metadata-from-library }

```python
cmd.get_node_metadata_from_library(library_name, node_type_name)
```

讀取節點型別的中繼資料。

#### 引數說明 {: #arguments_31 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `library_name` | string | 🟢 |
| `node_type_name` | string | 🟢 |

#### 回傳值 {: #return-value_31 }

包含節點型別中繼資料的 `ResultPayload` 物件。

#### 功能說明 {: #description_31 }

回傳指定程式庫中該節點型別的結構定義中繼資料。

## 設定操作 (Config Operations) {: #config-operations }

### get_config_value {: #get-config-value }

```python
cmd.get_config_value(category_and_key)
```

讀取設定鍵的數值。

#### 引數說明 {: #arguments_32 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `category_and_key` | string | 🟢 |

#### 回傳值 {: #return-value_32 }

包含設定數值的 `ResultPayload` 物件。

#### 功能說明 {: #description_32 }

回傳指定設定鍵（例如 `"category.key"`）的設定值。

---

### set_config_value {: #set-config-value }

```python
cmd.set_config_value(category_and_key, value)
```

寫入設定鍵的數值。

#### 引數說明 {: #arguments_33 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `category_and_key` | string | 🟢 |
| `value` | any | 🟢 |

#### 回傳值 {: #return-value_33 }

攜帶設定更新狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_33 }

為指定的設定鍵指派新數值。

---

### get_config_category {: #get-config-category }

```python
cmd.get_config_category(category=None)
```

讀取某個分類下的所有設定值。

#### 引數說明 {: #arguments_34 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `category` | string | ⚪ |

#### 回傳值 {: #return-value_34 }

包含該分類下所有設定項目的 `ResultPayload` 物件。

#### 功能說明 {: #description_34 }

回傳指定分類下的所有設定參數。若未提供 `category`，則回傳全域所有分類的完整設定。

---

### set_config_category {: #set-config-category }

```python
cmd.set_config_category(category=None, contents={})
```

批次設定某個分類下的所有項目。

#### 引數說明 {: #arguments_35 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `category` | string | ⚪ |
| `contents` | dict | ⚪ |

#### 回傳值 {: #return-value_35 }

攜帶分類設定更新狀態的 `ResultPayload` 物件。

#### 功能說明 {: #description_35 }

以傳入的字典內容更新指定分類下的所有設定值。

## 公用公用程式 (Utility Operations) {: #utility-operations }

### run_arbitrary_python {: #run-arbitrary-python }

```python
cmd.run_arbitrary_python(python_str)
```

執行任意 Python 程式碼字串。

#### 引數說明 {: #arguments_36 }

| 名稱 | 引數型別 | 是否必填 |
| :--- | :---: | :---: |
| `python_str` | string | 🟢 |

#### 回傳值 {: #return-value_36 }

包含執行狀態與結果的 `ResultPayload` 物件。

#### 功能說明 {: #description_36 }

在 Griptape 執行環境中動態執行傳入的 Python 程式碼字串。

## 範例示範 {: #examples }

### 建立流程與節點 {: #creating-a-flow-and-nodes }

```python
# 建立新流程
flow = cmd.create_flow(flow_name="MyFlow")

# 在流程中建立兩個節點
node1 = cmd.create_node(node_type="CreateText", node_name="MyText", parent_flow_name="MyFlow")
node2 = cmd.create_node(node_type="RunAgent", node_name="MyAgent", parent_flow_name="MyFlow")
```

### 設定參數值並建立連線 {: #setting-parameter-values-and-creating-connections }

```python
# 設定參數數值
cmd.set_value("MyText.text", "這是一段供摘要處理的示範文字。")

# 連接兩個節點
cmd.connect("MyText.text", "MyAgent.prompt")
```

### 執行流程 {: #running-a-flow }

```python
# 執行流程
cmd.run_flow("MyFlow")

# 讀取執行結果
summary = cmd.get_value("MyAgent.output")
print(summary)  # 印出產生的摘要內容！
```

### 管理與自訂參數 {: #working-with-parameters }

```python
# 為節點動態新增參數
cmd.add_param(
    node_name="MyAgent",
    parameter_name="sunglasses",
    default_value=100,
    tooltip="太陽眼鏡的帥氣程度",
    type=["int"],
)

# 修改新參數的值
cmd.set_value("MyAgent.sunglasses", 50)

# 更新既有參數的規格
cmd.add_param(node_name="MyAgent", parameter_name="sunglasses", tooltip="減半後的太陽眼鏡帥氣程度", edit=1)
```

### 清單查詢與探索 {: #listing-and-querying }

```python
# 列出流程內的所有節點
nodes = cmd.get_nodes_in_flow("MyFlow")
print(nodes)  # ["MyText", "MyAgent"]

# 列出節點上的所有參數
params = cmd.list_params("MyAgent")
print(params)  # ["agent", "prompt_driver", "tools", "rulesets", "prompt", "prompt_context", "output"]

# 檢查節點是否存在
if cmd.exists("MyText"):
    print("MyText 節點確實存在")  # True
```
