# 嚴格模式參考手冊 (Strict Mode Reference) {: #strict-mode-reference }

> 有關何時以及如何將您的程式庫於 Worker 子程序中隔離運行的完整指引，請參閱 [透過 Worker 進行節點隔離 (Node Isolation with Workers)](node_isolation_with_workers.md)。本頁面為攔截隔離不相容問題的專屬規則清單手冊。
>
> 在本頁面中，"`aprocess`" 指的是框架圍繞您在節點上實作的 `process` 方法所構建的非同步包裝層。當規則提及「在 `aprocess` 期間」時，其意涵即為「在節點執行期間，包含在您編寫的 `process` 方法內部」。

嚴格模式 (Strict Mode) 是一項執行階段約定，明確界定了節點程式碼在編排主程序與 Worker 子程序邊界之間被允許進行的操作。當節點違反該約定時，框架會記錄具名的違規事件並將其路由至節點的結果承載物件中，以便作者在編輯器介面上直接看到具體的修復指引，而非面對靜默失效、死結崩潰或指向錯誤層級的堆疊追蹤資訊。

嚴格模式**始終預設開啟**。不存在任何組態旗標、環境變數或開關可將其關閉。嚴重性等級是依規則劃分的：正確性 (Correctness) 規則會直接導致運算執行失敗；易用性 (Ergonomics) 規則則輸出警告訊息。

## 違規資訊的呈現途徑 {: #how-it-surfaces }

違規事項會附加至發送的 `ResultPayload` 內的 `ResultDetails` 中。在前端編輯器中，節點的輸出面板會直觀展示該規則的識別碼、嚴重性級別與具體修復方案。在 Worker 端，正確性違規會將原先成功的 `ExecuteNodeResultSuccess` 強制升級為 `ExecuteNodeResultFailure`；而易用性違規則維持非致命狀態。

違規亦會透過 `griptape_nodes.strict_mode` 日誌記錄器輸出。將其日誌層級設為 `WARNING` 或更低即可在控制台中監控每筆違規記錄。

## 規則目錄清單 (Rule Catalog) {: #rule-catalog }

每項規則皆歸屬於 **正確性規則 (Correctness)**（在編排程序與 Worker 端皆會直接引發失敗）或 **易用性規則 (Ergonomics)**（在編排程序輸出警告，在 Worker 端除非規則豁免否則會升級為失敗）。

### 正確性規則（直接導致執行中斷失敗） {: #correctness-rules-fail-execution }

#### `reentrant-bus-in-init` {: #reentrant-bus-in-init }

**違規行為**：節點在其 `__init__` 建構子內部發送了事件匯流排請求。Worker 程式庫架構探測會在啟動時執行 `__init__` 以提取參數規格；在此處重入匯流排會引發 Worker 程序死結。

**修復指引 (Remediation)**：將該呼叫移至 `aprocess`（或移至建構後觸發的生命週期掛鉤）中執行。

### 易用性規則（警告與防禦） {: #ergonomics-rules-warnings }

#### `parameter-behaviors-dropped-in-schema` {: #parameter-behaviors-dropped-in-schema }

**違規行為**：`Parameter` 附加了無法被 Worker 架構序列化保留的 `converters`、`validators` 或 `traits`。編排程序的 Stub 類別無法重新執行這些行為，導致 UI 側的互動行為與 Worker 側的實際執行產生分歧。

**修復指引 (Remediation)**：在 `process` 內部重新執行轉換器或驗證器邏輯，以便 Worker 能將其確實套用於真實數值；或者接受此分歧僅作為編排程序專屬的 UI 前端裝飾。請注意，將邏輯移入 `process` 會失去編輯器行內的即時校驗回饋——使用者只有在節點實際運算時才會看到失敗。

#### `parameter-mutation-during-aprocess` {: #parameter-mutation-during-aprocess }

**違規行為**：節點在 `aprocess` 執行期間調用了 `add_parameter` 或 `remove_parameter_element`。在 Worker 端，這些修改僅作用於當前的暫態節點實例，且無法同步寫回編排程序。

在輸入還原期間自 `before_value_set` / `after_value_set` 所觸發的結構異動（標準的動態參數設計模式）**不會**觸發此規則。

**修復指引 (Remediation)**：透過 `GriptapeNodes.handle_request(...)` 發送 `AddParameterToNodeRequest` 或 `RemoveParameterFromNodeRequest` 請求，以便結構異動能正確傳播至編排程序的權威節點中。

#### `connection-hooks-inert-on-worker` {: #connection-hooks-inert-on-worker }

**違規行為**：Worker 託管的程式庫中的某個節點類別覆寫了一個或多個連線生命週期掛鉤（`allow_incoming_connection`、`before_incoming_connection`、`after_incoming_connection`、其對應的傳出版本或 `_removed` 斷開變體）。連線屬於編排程序全權管控的狀態，因此這些掛鉤是在編排程序上調用的——而 Worker 託管程式庫在編排程序端僅以不包含這些覆寫邏輯的動態 Stub 類別代表。作者編寫的覆寫程式碼將會被靜默忽略而永不執行。

此規則在程式庫載入階段由 Worker 架構探測針對每個節點類別觸發一次。由引擎內建基底類別或元件實作的掛鉤不會被標記；該規則精準鎖定由程式庫作者編寫的程式碼。

**修復指引 (Remediation)**：隔離程式庫不支援由拓撲連線動態驅動參數的設計模式。請將該程式庫改為在共享模式 (Shared Mode) 下運行，或移除這些無效的覆寫方法。

#### `value-hooks-execute-only-on-worker` {: #value-hooks-execute-only-on-worker }

**違規行為**：Worker 託管程式庫中的某個節點類別覆寫了 `before_value_set` / `after_value_set`。對於隔離程式庫而言，這些掛鉤僅在運算執行階段的輸入還原過程中，於 Worker 本機暫態節點上觸發：轉換傳入值依然可以運作，但當使用者在前端編輯器中修改數值時這些掛鉤完全不會觸發，且在掛鉤內對參數清單進行的任何異動皆會隨暫態節點一同被丟棄。

此規則在程式庫載入階段由 Worker 架構探測針對每個節點類別觸發一次。由引擎內建基底類別與元件實作的掛鉤不會被標記。

**修復指引 (Remediation)**：將數值掛鉤嚴格限制於純粹的數值型別轉換。若該節點需要編輯器即時響應（隨使用者修改數值動態增減其他參數），請將該程式庫置於共享模式 (Shared Mode) 下運行。
