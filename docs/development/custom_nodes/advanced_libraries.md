# 進階程式庫開發 (Advanced Libraries) {: #advanced-libraries }

大多數程式庫只需透過其 `griptape_nodes_library.json` 資訊清單即可完整描述：引擎讀取清單、匯入指定的節點模組並註冊這些類別。而**進階程式庫 (Advanced Library)** 則是一個選用的 Python 類別，允許您的程式庫在其自身生命週期的五個關鍵節點執行自訂程式碼：節點載入前、節點載入後、程式庫註銷前、引擎收集請求處理常式時，以及引擎收集調度後掛鉤時。

本頁面為該類別的開發者參考指南。有關資訊清單本身（元數據、分類、宣告、依賴管理）的說明，請參閱 [程式庫編寫與發布 (Authoring Libraries)](authoring_libraries.md)。

## 我是否需要編寫進階程式庫？ {: #should-i-write-one }

**您無需編寫進階程式庫，若**您的程式庫僅由清單中列出的固定節點類別檔案組成。這是最常見的情況，除了節點程式碼之外無需額外的 Python 邏輯。

**您應當編寫進階程式庫，若**您需要：

- **在程式庫載入與卸載時獲取或釋放全行程範圍的系統資源**：例如 GPU 運算環境、原生 SDK 的 Python 綁定、背景執行緒、資料庫連線集區。
- **動態註冊未在資訊清單中靜態列出的節點型別**（例如基於外部資料驅動或動態生成的節點）。參閱 [不透過清單宣告動態註冊節點型別](#registering-node-types-without-listing-them-in-the-manifest)。
- **提供專屬於您程式庫的自訂請求型別服務**，使其他程式庫與節點可向其發送呼叫。參閱 [`get_request_handlers`](#get-request-handlers)。
- **在引擎處理完其既有請求後執行響應操作**，例如在每次儲存工作流程時自動觸發專屬稽核步驟。參閱 [`get_post_dispatch_hooks`](#get-post-dispatch-hooks)。
- **為引擎請求型別註冊競爭提供者 (Competing Provider)**，例如自訂工作流程發布器。參閱 [發布指南 (Publishing)](../../guides/publishing.md)。

## 配置與掛載 {: #wiring-one-up }

在資訊清單中透過 `advanced_library_path` 指向對應的 Python 檔案（相對於資訊清單路徑）：

```json
{
  "name": "My Library",
  "advanced_library_path": "advanced_library.py",
  "nodes": []
}
```

接著在該 Python 檔案中繼承 `AdvancedNodeLibrary` 並僅覆寫您所需的回呼函式。每個回呼皆具備預設的空操作 (No-op) 實作：

```python
from griptape_nodes.node_library.advanced_node_library import AdvancedNodeLibrary


class MyLibrary(AdvancedNodeLibrary):
    def after_library_nodes_loaded(self, library_data, library) -> None:
        print(f"已成功載入 {len(library.get_registered_nodes())} 個節點")
```

引擎在尋找並建構您的類別時依循以下三項關鍵原則。若違反任一原則將導致整個程式庫載入失敗：

- **類別必須直接定義於該檔案中。** 引擎會掃描模組中 `__module__` 與剛匯入之模組名稱相符的 `AdvancedNodeLibrary` 子類別。*匯入* 至該檔案的他處子類別將被跳過。若希望核心實作置於套件內部，請在清單指定的檔案中完成子類化。
- **以首個匹配項為準。** 引擎依照模組順序選取第一個合格的子類別並立即停止搜尋。請確保該檔案中僅精確定義一個子類別以避免歧義。
- **`__init__` 不得包含必要引數。** 引擎在實例化您的類別時不會傳入任何引數。請在 `__init__` 或各生命週期掛鉤中自行衍生所需的狀態。

若該模組匯入失敗、未包含合格子類別或無法被實例化，該程式庫將被標記為 `UNUSABLE`，註冊隨之失敗，且編輯器會呈現夾帶底層錯誤的 `AdvancedLibraryLoadFailureProblem` 提示。

## 載入生命週期序列 {: #the-load-sequence }

深入理解各個掛鉤所處的精確時機，是確保回呼順利運作而非靜默失效的關鍵。註冊程式庫時，引擎會依序執行以下步驟：

| 步驟 | 引擎執行操作 |
| :--- | :--- |
| 1 | 解析並驗證 `griptape_nodes_library.json` 清單 |
| 2 | 將程式庫目錄及其虛擬環境 site-packages 加入 `sys.path` |
| 3 | 匯入您的進階程式庫模組並實例化自訂類別 |
| 4 | 在 `LibraryRegistry` 中註冊該 `Library` 實例 |
| 5 | 保存清單中宣告的所有程式庫設定項目 |
| 6 | **調用 `before_library_nodes_loaded`** |
| 7 | 遍歷 `library_data.nodes` 並逐一註冊各節點型別 |
| 8 | 註冊清單中聲明的視覺部件 (Widgets) |
| 9 | **調用 `after_library_nodes_loaded`** |
| 10 | **調用 `get_request_handlers`** 並註冊其回傳的處理常式 |
| 11 | **調用 `get_post_dispatch_hooks`** 並註冊其回傳的掛鉤 |
| 12 | 計算程式庫健全度 (Fitness) 並標記狀態為 `LOADED` |

兩項至關重要的設計推論：

- 您的類別實例在步驟 3 即已生成，而程式庫在步驟 4 才正式註冊，因此 `__init__` 中**無法**在 `LibraryRegistry` 內查找到自身。
- 步驟 7 讀取 `library_data.nodes` 是在步驟 6 執行**之後**發生，這正是實現動態生成與註冊節點的核心基礎。

卸載註銷 (Unregistering) 則依循逆向順序執行：

| 步驟 | 引擎執行操作 |
| :--- | :--- |
| 1 | **調用 `before_library_unregistered`** |
| 2 | 移除該程式庫的應用程式事件監聽器、調度前與調度後掛鉤 |
| 3 | 移除該程式庫所註冊的所有請求處理常式 |
| 4 | 註銷該程式庫的視覺部件 |
| 5 | 從 `LibraryRegistry` 中移除該程式庫 |

## 各項回呼函式詳解 {: #the-hooks }

### `before_library_nodes_loaded` {: #before-library-nodes-loaded }

```python
def before_library_nodes_loaded(self, library_data: LibrarySchema, library: Library) -> None: ...
```

在程式庫已完成註冊、但任何節點型別尚未載入之前觸發。可用於準備節點匯入所依賴的前置環境，或直接向 `library_data.nodes` 動態附加節點定義。

`library_data` 為當前 `Library` 持有的真實動態 `LibrarySchema` 實例而非複本。在此處對其進行的修改將直接影響引擎在步驟 7 中載入的項目及其後續狀態報告。

若此回呼拋出例外，引擎會記錄一個 `BeforeLibraryCallbackProblem` 並**繼續執行後續載入流程**。程式庫狀態將被標記為 `FLAWED` 而非直接崩潰失敗——這意味著節點本身可能仍可使用，但您的前置初始化作業並未成功執行。因此切勿盲目假設其已初始化成功。

### `after_library_nodes_loaded` {: #after-library-nodes-loaded }

```python
def after_library_nodes_loaded(self, library_data: LibrarySchema, library: Library) -> None: ...
```

在清單中宣告的所有節點型別皆已完成註冊後觸發。此時調用 `library.get_registered_nodes()` 可獲取完整的已註冊清單，適合執行需要審視完整程式庫就緒狀態的後續作業。

這也是透過 `LibraryManager.on_register_event_handler()` 註冊競爭提供者事件處理常式的地方，例如宣告程式庫作為自訂工作流程發布器。

此處的錯誤處理方式與 before 掛鉤一致：記錄 `AfterLibraryCallbackProblem` 並繼續完成載入流程。

### `before_library_unregistered` {: #before-library-unregistered }

```python
def before_library_unregistered(self, library_data: LibrarySchema, library: Library) -> None: ...
```

在引擎卸載任何組件之前執行，因此在其執行期間，您註冊的所有監聽器與處理常式依然處於有效運作狀態。可用於釋放在載入階段獲取的各類資源：原生系統綁定、GPU 運算環境、背景執行緒或連線集區。

此處拋出的錯誤會被**記錄於日誌並直接吞噬**。卸載流程仍將強制繼續，確保失敗的清理作業不會卡死引擎核心。然而這也意味著未成功釋放的系統資源將發生隱蔽的靜默洩漏。

### `get_request_handlers` {: #get-request-handlers }

```python
def get_request_handlers(self) -> list[tuple[type[RequestPayload], Callable]]: ...
```

回傳 `(request_type, handler)` 二元組清單，引擎會代為統一註冊，並在程式庫卸載時自動予以註銷。同步函式與非同步協程皆可作為處理常式。這是程式庫對外暴露服務的核心機制，其自身的節點、其他第三方程式庫以及外部用戶端皆可對其進行調用。

實作該機制包含三個組成步驟。完整情境範例請參閱 [下方實用範例](#example-a-library-that-serves-a-request-type)。

**1. 定義資料承載物件 (Payloads)。** 定義一個請求型別與至少一個回傳結果型別，每個皆為透過 `PayloadRegistry` 註冊的 dataclass，以便能夠透過名稱在 WebSocket 與 MCP 介面邊界進行雙向解析：

```python
@dataclass
@PayloadRegistry.register
class ConvertColorspaceRequest(RequestPayload):
    color: tuple[float, float, float]
    source: str
    target: str


@dataclass
@PayloadRegistry.register
class ConvertColorspaceResultSuccess(WorkflowNotAlteredMixin, ResultPayloadSuccess):
    color: tuple[float, float, float]


@dataclass
@PayloadRegistry.register
class ConvertColorspaceResultFailure(WorkflowNotAlteredMixin, ResultPayloadFailure):
    pass
```

建議將它們定義於獨立的專屬模組中，供進階程式庫與各節點共同匯入。由於在兩者載入時程式庫根目錄已被加入 `sys.path`，因此可直接透過 `import colorspace_events` 解析，兩端將獲得完全相同的模組物件與承載類別。請為該模組指定具備辨識度的獨特名稱：所有程式庫目錄皆共處於同一個 `sys.path` 中，若命名為通用的 `events.py` 將極易與其他程式庫發生匯入衝突。

**2. 在回呼掛鉤中回傳配對清單。**

```python
def get_request_handlers(self) -> list[tuple[type[RequestPayload], Callable]]:
    return [(ConvertColorspaceRequest, self._handle_convert_colorspace)]
```

型別註解請直接使用無泛型引數的 `Callable`。基底類別宣告為 `Callable[[RequestPayload], ResultPayload]`，若處理常式標註為具體的請求型別將由於參數型別的逆變性 (Contravariance) 而引發型別檢查報錯。保持處理常式自身具體型別的清晰精確，遠比機械式迎合基底方法簽章更有價值。

**3. 調度請求。** 編排程序 (Orchestrator) 中的任何調用方皆可透過標準事件匯流排發送該請求並接收計算結果：

```python
result = GriptapeNodes.handle_request(ConvertColorspaceRequest(color=(1.0, 0.0, 0.0), source="rgb", target="hsv"))
if result.failed():
    msg = f"在節點 '{self.name}' 中嘗試轉換色彩失敗，成因：{result.result_details}"
    raise RuntimeError(msg)

success = cast("ConvertColorspaceResultSuccess", result)
```

調用方的兩項強制準則：

- **務必在 `process` 階段調度請求，切勿在 `__init__` 中發送。** 在節點建構子中發送請求會違反 `reentrant-bus-in-init` 嚴格模式規則，並可能與等待引擎啟動的處理常式發生致命死結。詳見 [嚴格模式參考 (Strict Mode Reference)](strict_mode.md)。
- **一律妥善處理失敗情況。** 提供該服務的程式庫可能未安裝、載入失敗或已被使用者卸載。在這些情況下該請求根本不存在處理常式，引擎將回傳通用的底層失敗結果，而非您程式庫自訂的失敗型別。在縮窄型別至成功結果前，請務必先透過 `result.failed()` 進行防禦判斷。

機制的邊界限制：

- **您的程式庫必須全權擁有該請求型別。** 該 `RequestPayload` 子類別必須定義於您自訂的套件內部。
- **全引擎範圍內，每個請求型別僅允許註冊一個處理常式。** 重複註冊已有處理常式的型別將拋出異常，並暴露為 `RequestHandlerRegistrationProblem`。若屬於多個程式庫提供競爭實作並由調用方依名稱挑選的情境，請改在 `after_library_nodes_loaded` 中使用 `LibraryManager.on_register_event_handler()`。
- **僅限編排程序 (Orchestrator)。** 在獨立 Worker 子程序中隔離運行的程式庫會將處理常式註冊於該 Worker 內部，編排程序無法跨行程存取它們，請求將因 "No manager found" 而失敗。引擎在載入時會標記 `RequestHandlersWorkerIncompatibleProblem` 警示。參見 [透過 Worker 進行節點隔離 (Node Isolation with Workers)](node_isolation_with_workers.md)。

外部程式碼可透過 `library.get_registered_request_handler_types()` 探索已載入程式庫暴露的型別，並搭配 `dataclasses.fields()` 與 `typing.get_type_hints()` 進行反射檢查。

### `get_post_dispatch_hooks` {: #get-post-dispatch-hooks }

```python
def get_post_dispatch_hooks(self) -> list[tuple[type[RequestPayload], Callable]]: ...
```

回傳 `(request_type, callback)` 二元組清單，由引擎負責註冊並在卸載時自動移除。當引擎自身處理該請求型別產生運算結果後，您的回呼常式會被觸發並接收 `(request, result)` 參數。這允許程式庫在引擎發生特定事件時（記錄稽核日誌、發布聊天頻道通知、觸發背景匯出）執行自訂響應邏輯，而無需擁有該請求型別的定義權限。

它是 [`get_request_handlers`](#get-request-handlers) 的鏡像機制，核心差異在於擁有權歸屬：

| 比較特性 | `get_request_handlers` | `get_post_dispatch_hooks` |
| :--- | :--- | :--- |
| 是否獨佔該請求型別 | 是——全引擎唯一處理常式 | 否——任意數量的程式庫皆可掛載同一型別 |
| 觸發執行時機 | **取代** 引擎預設處理常式 | 在引擎處理常式產出結果 **之後** 觸發 |
| 是否能變更運算結果 | 其回傳值 **即為** 最終結果 | 否，僅供被動通知，無法修改結果 |

這正是為什麼您可以監聽掛載由 `WorkflowManager` 所擁有的 `SaveWorkflowRequest`，而若使用 `get_request_handlers` 則會因所有權衝突而宣告失敗。

支援同步與非同步回呼常式：

```python
def get_post_dispatch_hooks(self):
    return [(SaveWorkflowRequest, self._on_workflow_saved)]


async def _on_workflow_saved(self, request: RequestPayload, result: ResultPayload) -> None:
    if not isinstance(result, SaveWorkflowResultSuccess):
        return
    await self._append_audit_line(result.file_path)
```

機制的邊界限制：

- **僅供被動通知。** 回呼的返回值會被忽略，且回呼無法修改結果或中斷中斷操作。回呼內部拋出的例外將被記錄於日誌並直接忽略，且不會阻止同一請求上的其他掛鉤執行。
- **雙向結果皆會觸發。** 無論請求成功或失敗（包含處理常式未捕獲例外而生成的失敗結果），掛鉤皆會觸發。請如上方範例所示依據結果型別進行分支過濾。
- **通常為分離非同步任務，但偶有例外。** 當引擎具備活動中的事件迴圈時，該掛鉤會被排程為分離任務 (Detached Task)，因此結果能立即回傳前端編輯器而無需等待掛鉤。但在無事件迴圈的路徑上（CLI 命令、啟動流程執行、Worker 執行緒），掛鉤會以同步行內方式執行並**阻塞調用方直至其回傳**。若可能在這些路徑觸發，請確保掛鉤輕量迅速，或將耗時任務移至行程外執行。
- **精確型別匹配。** 為特定請求型別註冊的掛鉤僅針對該精確型別觸發，絕不會在其衍生子類別上觸發。
- **參數唯讀約束。** 切勿修改 request 或 result 物件；兩者即將被引擎序列化為結果事件。被標記為 `omit_from_result` 的欄位在傳遞給您前已被清空，因此無法透過掛鉤讀取這些敏感欄位。
- **嚴禁在掛鉤內部再次發送引擎請求。** 引擎的操作深度與節點執行狀態是全行程共用的，在掛鉤內發送請求會干擾正在傳輸中的操作。請改為執行行程外部作業（HTTP 請求、實體檔案寫入等）。
- **僅限編排程序 (Orchestrator)。** 掛鉤是註冊於載入程式庫的該行程事件管理器上的，因此 Worker 模式程式庫的掛鉤永遠看不到編排程序處理的請求。引擎在載入時會標記 `PostDispatchHooksWorkerIncompatibleProblem`。參見 [透過 Worker 進行節點隔離](node_isolation_with_workers.md)。
- **非持久化保障。** 行程結束時仍在執行中的掛鉤會被直接放棄。切勿將其用於需要嚴格交付保證的關鍵業務。

格式錯誤的二元組（不可調用的回呼常式或非請求型別的鍵）會被回報為 `PostDispatchHookRegistrationProblem` 並中止清單後續項目的註冊，導致程式庫被標記為 `FLAWED`。在錯誤項之前已註冊的掛鉤仍將維持作用，並在卸載時正常移除。

## 不透過清單宣告動態註冊節點型別 {: #registering-node-types-without-listing-them-in-the-manifest }

若您的節點集合是由外部資料驅動、動態生成，或規模過於龐大以致難以手動維護清單，您可以在清單中保留 `"nodes": []`，並在 `before_library_nodes_loaded` 回呼中動態建構節點定義。

這得益於 [載入生命週期序列](#the-load-sequence) 中的步驟 6 與步驟 7：掛鉤優先執行，`library_data` 是引擎隨後讀取的同一個實例，且 `library_data.nodes` 是標準的 Python 清單。

```python
class MyLibrary(AdvancedNodeLibrary):
    def before_library_nodes_loaded(self, library_data, library) -> None:
        library_data.nodes.extend(
            NodeDefinition(
                class_name=spec["class_name"],
                file_path="generated_nodes.py",
                metadata=NodeMetadata(
                    category="dynamic",
                    description=spec["description"],
                    display_name=spec["display_name"],
                ),
            )
            for spec in load_specs()
        )
```

編輯器所讀取的內容絕非直接來自磁碟上的 `nodes` 清單。`ListNodeTypesInLibrary` 與 `GetAllInfoForLibrary` 皆是讀取記憶體中的 `Library` 物件，因此動態生成的節點型別在節點選擇面板中的外觀與行為與靜態宣告的節點完全無異。各分類仍是自清單中讀取，因此請確保在清單中預先宣告所有欲動態掛載的分類。系統不會主動驗證節點分類是否存在於宣告清單中，分類不匹配會發生靜默失效：節點型別本身註冊成功，但編輯器將因找不到對應分類而無法將其歸類顯示。

以此方式新增的節點定義與手寫定義具備完全一致的行為特徵，繼承載入器的所有優勢：延遲模組載入、多個類別共享同一檔案時僅執行一次記憶體快取匯入、穩定的命名空間別名（確保儲存的工作流程能反序列化還原類別定義的值）、單節點問題回報與正確的健全度計算。

### 類別的動態生成來源 {: #where-the-classes-come-from }

引擎透過匯入 `NodeDefinition.file_path` 指定的檔案並調用 `getattr(module, class_name)` 來解析節點型別。模組層級的 `__getattr__` ([PEP 562](https://peps.python.org/pep-0562/)) 完美契合該需求，因此單一檔案即可在無需任何靜態 `class` 宣告語法的情況下，為程式庫中的所有動態節點型別提供後端實現：

```python
def __getattr__(name: str) -> type[DataNode]:
    spec = find_spec(name)
    if spec is None:
        msg = f"模組 {__name__!r} 未包含屬性 {name!r}"
        raise AttributeError(msg)
    node_class = build_node_class(spec)
    globals()[name] = node_class  # 快取：後續查詢直接略過 __getattr__
    return node_class
```

請務必將構建的類別快取於模組全域字典中。對同一節點型別的兩次查詢必須回傳完全相同的類別物件，因為引擎會快取解析後的類別，`isinstance` 型別檢查依賴其參照，且 pickle 序列化亦直接引用它。

!!! warning "動態建構類別時務必顯式設定 `__module__`"

    `type(name, bases, namespace)` 預設不會自動指派當前模組名稱。若命名空間中未包含 `__module__` 鍵，類別建立時會自調用框架的全域變數讀取 `__name__`。由於 `BaseNode` 子類別攜帶 `ABCMeta`，該調用框架實際上位於標準程式庫的 `abc` 模組內部，導致您的類別被賦予 `__module__ == "abc"`。

    載入階段完全不會引發任何異常。但崩潰會延後發生：反序列化還原已儲存的工作流程時會嘗試匯入 `__module__` 並在其中查找 `__qualname__`，導致包含該類別物件的任何工作流程在重新開啟時崩潰。請務必顯式傳入兩者：

    ```python
    return type(
        spec["class_name"],
        (DataNode,),
        {
            "__init__": __init__,
            "process": process,
            "__module__": __name__,
            "__qualname__": spec["class_name"],
        },
    )
    ```

### 為何不直接註冊類別物件？ {: #why-not-register-classes-directly }

雖然 `Library.register_new_node_type()` 與 `Library.register_lazy_node_type()` 皆為公開 API，且在 `after_library_nodes_loaded` 中調用它們亦能成功註冊可運作的節點型別，但依然強烈建議透過合成定義的方式進行動態註冊，原因有二：

- **健全度評估 (Fitness)。** 引擎是依據步驟 7 中清單驅動的迴圈結果來判定程式庫是否載入成功。宣告 `"nodes": []` 並在 after 掛鉤中直接註冊所有內容的程式庫會被評定為 `UNUSABLE`，且其註冊將被回報為失敗，即便節點型別本身實際上可正常使用。
- **穩定的命名空間。** 載入器會自動註冊待處理的穩定模組載入器，以便在重開存檔時順暢解析 `griptape_nodes.node_libraries.<library>.<file>`。直接註冊類別將跳過此機制，您必須自行承擔反序列化失敗的風險。

### 機制邊界與限制 {: #limitations }

- **不支援 Worker 隔離程式庫。** 當程式庫在 Worker 子程序中運行時，編排程序會依據 Worker 回傳的架構資訊重建 Stub 類別，並自清單的 `nodes` 清單解析各 Stub 的元數據。僅存在於 Worker 註冊表中的動態節點型別將被忽略並輸出警告。請將動態註冊的程式庫置於編排程序中，或在清單中靜態枚舉其節點。
- **宣告驗證僅檢視靜態清單。** 針對 `model_usage` 與 `model_provider_usage` 引用的合法性校驗是針對磁碟上的靜態清單執行的，因此動態節點上的這些宣告永遠不會被校驗。錯誤的模型引用將在執行階段而非載入時崩潰。
- **節點名稱衝突約束依然生效。** 動態生成的類別名稱同樣需經過跨程式庫命名衝突檢查，動態生成的名稱同樣容易發生衝突。請務必加上專屬前綴。

## 完整實用範例 {: #examples }

提供兩套完整可運作的示範程式庫。將任一資料夾複製至工作區的 `libraries` 目錄，透過編輯器的程式庫設定完成註冊，並重新啟動引擎即可體驗。

### 範例 1：提供自訂請求型別服務的程式庫 {: #example-a-library-that-serves-a-request-type }

該程式庫擁有 `ConvertColorspaceRequest` 請求型別，於進階程式庫中提供處理常式，並在自身的節點中發起調用：

- [`griptape_nodes_library.json`](example_request_handler_library/griptape_nodes_library.json)：宣告一個節點與進階程式庫的清單
- [`colorspace_events.py`](example_request_handler_library/colorspace_events.py)：透過 `PayloadRegistry` 註冊的請求與結果承載物件
- [`advanced_library.py`](example_request_handler_library/advanced_library.py)：自 `get_request_handlers` 回傳處理常式並實作其色彩轉換邏輯
- [`nodes.py`](example_request_handler_library/nodes.py)：在 `process()` 中發送該請求並妥善處置錯誤的節點

在畫布中新增一個 **Convert Colorspace** 節點，將 `color` 設為 `[0, 0.5, 1]`，`source` 設為 `rgb`，`target` 設為 `hsv` 並執行。同一引擎中的任何其他第三方程式庫現在皆可發送 `ConvertColorspaceRequest` 並獲取精確計算結果。

### 範例 2：動態註冊節點型別的程式庫 {: #example-a-library-that-registers-node-types-dynamically }

該程式庫依據 JSON 資料檔案動態註冊四個節點型別，且清單中完全不靜態宣告任何節點：

- [`griptape_nodes_library.json`](example_dynamic_library/griptape_nodes_library.json)：帶有 `"nodes": []` 與一個宣告分類的清單檔案
- [`node_specs.json`](example_dynamic_library/node_specs.json)：驅動節點生成的原始規格資料檔
- [`advanced_library.py`](example_dynamic_library/advanced_library.py)：在 `before_library_nodes_loaded` 中為每個規格動態建構 `NodeDefinition`
- [`generated_nodes.py`](example_dynamic_library/generated_nodes.py)：透過模組 `__getattr__` 隨選動態建構各 `DataNode` 子類別

啟動後您將在節點面板中看到包含四個節點的 **Dynamic** 分類。在 `node_specs.json` 中複用現有的 `operator` 值新增一筆資料並重啟引擎，第五個節點將立即現身，而無需修改任何清單或 Python 程式碼。
