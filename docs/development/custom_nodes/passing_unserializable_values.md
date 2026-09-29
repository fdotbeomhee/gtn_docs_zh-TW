# 傳遞不可序列化值 (Passing Values That Cannot Be Serialized) {: #passing-values-that-cannot-be-serialized }

某些 Python 物件根本無法被轉換為通用資料格式。例如 Diffusers 運算管線 (Pipeline)、PyTorch 潛在張量 (Latent Tensor)、已開啟的檔案控制代碼或活躍的硬體驅動器——它們無法被轉換為 JSON 格式。若您的程式庫在獨立的 Worker 子程序中隔離運作（參見 [透過 Worker 進行節點隔離 (Node Isolation with Workers)](node_isolation_with_workers.md)），參數數值會在編排程序 (Orchestrator) 與 Worker 之間透過 JSON 進行傳遞，因此將這些非資料物件從一個節點傳遞至下游節點需要特殊支援。

極簡原則：**將生產該物件的輸出端宣告為 `serializable=False` 並直接為其賦值。消費該物件的輸入端無需進行任何特殊宣告，如常讀取即可。**

```python
class LoadPipeline(ControlNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        self.add_parameter(
            Parameter(
                name="pipeline",
                output_type="Pipeline",
                tooltip="載入完成的運算管線",
                serializable=False,
                allowed_modes={ParameterMode.OUTPUT},
            )
        )

    def process(self) -> None:
        self.parameter_output_values["pipeline"] = load_pipeline(...)


class Generate(ControlNode):
    def __init__(self, **kwargs) -> None:
        super().__init__(**kwargs)
        # 消費端完全無需任何特殊宣告
        self.add_parameter(Parameter(name="pipeline", input_types=["Pipeline"], tooltip="欲執行的運算管線"))

    def process(self) -> None:
        pipeline = self.get_parameter_value("pipeline")  # 直接獲取真實的 Python 物件實例
        ...
```

## 底層運作機制解析 {: #what-actually-happens }

真實的 Python 物件始終保留在建立它的進程內部。當該數值即將跨越行程邊界傳輸時，引擎會自動替換為一個不透明的**引用鍵 (Key)**——一段簡短的字串——並傳遞該字串。當您的下游消費節點讀取該參數時，該鍵會被自動透明地還原為原始的 Python 物件。

三項至關重要的核心認知：

- **您節點自身的輸出字典始終持有原始物件。** 將管線指派給 `parameter_output_values["pipeline"]` 並立即讀取，您拿到的是真實管線實例而非字串鍵。賦值寫入時完全不會發生任何替換。
- **從未跨越行程的拓撲圖永遠不執行此轉換。** 若您的程式庫在共享模式 (Shared Mode) 下運行，數值一律如往常般以記憶體參照 (Pass by Reference) 直接傳遞。
- **僅生產端需要進行宣告。** 引用鍵會沿著連線傳遞給未進行任何特殊宣告的消費節點，這正是上方範例中消費端使用常規 `Parameter` 即可正常工作的原因。

## `serializable=False` 的深層意涵 {: #what-serializablefalse-means }

宣告此屬性可防止數值被寫入儲存的工作流程 `.py` 檔案中。在輸出端上，它還能將非純資料物件保留在生產它的進程內部，僅將引用鍵跨越行程邊界傳輸。**在已宣告的參數上傳遞純資料時，依然會作為資料正常傳輸**：

| `serializable=False` 輸出端上的數值型別 | 跨越邊界的內容 | 原因說明 |
| :--- | :--- | :--- |
| 運算管線、張量、硬體驅動器 | 引用鍵 (Key)；實體物件留在本機進程 | 根本無法被結構化為資料 |
| API 金鑰字串 | 原始字串 | 本身即可完美傳輸，若轉為鍵在對端反而無法解析 |
| 數值字典、字串清單 | 原始資料結構 | 本身即為純資料格式 |
| `ImageUrlArtifact` | 引用鍵 (Key)；實體物件留在本機進程 | 詳見下方說明 |

最後一項常讓人感到意外。程式庫自訂的產物 (Artifact) 在解構為欄位字典的過程中會遺失其資料承載體，因此引擎不會在往返序列化上冒險：若您宣告了該參數，您的物件會被保留於進程中。若您希望產物以常規資料的形式在行程間傳播（對於包含 URL 網址的任何物件通常皆屬此類需求），**請不要宣告 `serializable=False`**。未經宣告的產物會像往常一樣正常完成序列化與還原重構。

## 在多次執行間複用高開銷資源 {: #reusing-an-expensive-resource-across-runs }

上述內容針對的是在節點之間流動的數值。若某項**系統資源**希望建立一次並在後續運算中重複複用——例如冷啟動載入需要耗時 30 秒的大型管線——則需要一個可重複推導的自訂快取鍵。節點對此提供了專屬的輕量 API：

```python
def process(self) -> None:
    key = self.local_objects.key_for(self._config_hash())
    pipeline = self.local_objects.get(key)
    if pipeline is None:
        pipeline = build_pipeline(...)
        self.local_objects.put(pipeline, key=self._config_hash(), on_drop=release_vram)
    ...
```

| 方法調用 | 具體行為 |
| :--- | :--- |
| `local_objects.put(value, *, key, on_drop=None)` | 在您的自訂鍵下快取保留 `value`，回傳完整的命名空間鍵 |
| `local_objects.get(key)` | 獲取該物件；若本進程未持有則回傳 `None` |
| `local_objects.key_for(suffix)` | 依據您指定的後綴計算完整命名空間鍵，而不實際放入任何物件 |
| `local_objects.drop(key)` | 釋放單一快取物件；回傳本進程先前是否持有該物件 |
| `local_objects.drop_all()` | 徹底釋放您的程式庫在此進程快取的所有資源；相當於「清除快取」節點的行為 |

使用已存在的鍵調用 `put` 會先釋放原先保留的舊物件（除非新舊為同一物件）——因此在相同雜湊值下重建資源不會造成舊物件的記憶體孤立。

## 釋放物件佔用的底層硬體資源 {: #freeing-what-the-object-holds }

釋放最後一個 Python 參照並不會自動清除 GPU 顯示記憶體 (VRAM)。請在參數宣告時傳入 `on_local_object_drop` 回呼（或在 `put` 時傳入 `on_drop`），當物件被釋放時引擎會自動調用該回呼：

```python
Parameter(
    name="pipeline",
    output_type="Pipeline",
    tooltip="載入完成的運算管線",
    serializable=False,
    allowed_modes={ParameterMode.OUTPUT},
    on_local_object_drop=lambda pipeline: pipeline.to("cpu"),
)
```

該掛鉤歸屬於快取系統管控，因此在快取丟棄物件時觸發：

- **您的節點再次運算**，快取為該參數接收了全新物件——原先持有的舊物件被釋放；
- **您的節點自畫布刪除**，且無其他組件繼續引用該物件；
- **您的程式庫被卸載**，或所有程式庫被全域重新載入；
- **工作流程被關閉或清空。**

每個物件僅執行一次釋放掛鉤，即便該物件同時掛載於多個輸出端。

此機制**不適用於**從未進入快取系統的物件。若您的程式庫在共享模式 (Shared Mode) 下運行，由於不存在跨行程邊界，所有內容皆不會被快取，數值純粹依記憶體參照傳遞——快取系統無從釋放任何東西，記憶體管理必須由您如以往一樣手動處置。在中途運算覆寫的物件亦同：僅在節點執行結束時參數持有的最終物件才會進入快取。此外，更新單一程式庫或切換其 Git 分支目前不會重啟 Worker，因此 Worker 持有的物件會在更新後的程式碼中繼續留存。

## 絕對禁止的操作模式 {: #what-you-cannot-do }

**清單 (List) 或字典 (Dictionary) 參數無法保留不可序列化物件。** `ParameterList` 與 `ParameterDictionary` 是依據其子項目動態組裝數值的，因此不存在單一的實體物件可供快取，亦無處掛載釋放回呼。在容器上宣告 `serializable=False` 僅具備其傳統行為——防止清單寫入工作流程存檔——但不會加入任何快取機制。

若需批次傳遞，請使用宣告為 `serializable=False` 的一般 `Parameter` 輸出整批資料——就快取系統而言，包含多個張量的清單本質上也是一個單一物件，這可以完美運作。而**消費端**使用 `ParameterList` 讀取受保護物件是完全合法的：每一列攜帶自己的鍵，在容器上調用 `get_parameter_value` 即可獲取物件清單。針對此類物件，請優先使用該方法而非 `get_parameter_list_value`，因為後者會強制展開所有可迭代物件，將張量清單拆解為個別列。

**物件絕對無法離開建立它的原程。** 快取系統隸屬於 Worker 子程序而非單一程式庫：單一 Worker 可同時託管多個程式庫並共享該快取，因此受託管於同一 Worker 的程式庫接收到引用鍵可正常還原。但**跨越不同進程**讀取鍵（例如來自另一個 Worker 或編排程序）將徹底失敗：

> Attempted to read the value for parameter 'pipeline' on node 'Generate'. Failed due to: it is held in another process, and an object cannot leave the process that built it. Read it from a node that runs in the same place as the one that made it, or have that node output a saved file instead.

僅當兩個程式庫皆未宣告自訂執行虛擬環境 (venv) 時才會共享同一個 Worker，而這對工作流程創作者而言是無法直接預見的。因此若您發布的程式庫需要將底層物件傳遞給**其他第三方程式庫**的節點，切勿依賴跨程式庫共享進程——請寫入實體磁碟並傳遞其檔案路徑或 URL 網址。在您自己的程式庫內部，所有節點永遠處於同一個 Worker 中。

透過 `put` 自訂的鍵在 Worker 內部會由您的程式庫自動進行命名空間隔離，因此同一 Worker 內的共存程式庫即便使用相同的字串也不會發生鍵名衝突。

**輸入端絕不執行快取。** 快取輸入端會在對端產生一個無法解析的懸空鍵：因為當節點在別處運算時，實體物件依然滯留在發送進程中。僅輸出端會進入快取系統。若節點需要對端的複雜資料，請讓生產節點與消費節點位於同一程式庫中，或透過實體檔案傳遞。

**重新載入後快取立即失效。** 引用鍵指向的是執行中行程的虛擬記憶體位址，無法持久化保存。

## 常見報錯資訊與故障排查 {: #errors-you-may-see-and-what-they-mean }

| 錯誤資訊關鍵字 | 意涵解析 |
| :--- | :--- |
| `it is no longer available, which happens after the workflow is reloaded or the node that made it is re-run` | 該鍵生成於本進程中，但其引用的物件隨後已被釋放。重新運行生產節點即可修復；這是正常的預期行為而非物件遺失故障 |
| `held in another process` | 您正在嘗試自建構物件之外的進程讀取它——最常見於驗證器 (Validator) 或數值掛鉤中，因為它們運行於編排程序而非 Worker 內部 |
| `nothing is connected to it` | 該輸入端未建立任何拓撲連線 |

第二項錯誤實際上是一項防護設計。每次賦值生成的引用鍵皆全域唯一，下游消費節點若持有上一輪運算的舊鍵將會被判定為懸空，而不會靜默錯誤地綁定至它從未接收過的**更新版**物件。

## 存檔與元數據處置 {: #saving-and-metadata }

受快取保護的值永遠不會被寫入工作流程 `.py` 存檔中，包含此類節點的工作流程重新開啟時狀態會呈現為 **UNRESOLVED**，以便驅動其上游生產節點重新運算並替換物件。工作流程元數據與影像附隨檔案 (Sidecars) 會將該參數記錄為省略 (Omitted) 而非記錄無意義的引用鍵。您無需為此編寫任何額外程式碼；宣告屬性會全自動處理這一切。

## 開發自我檢查檢核清單 {: #checklist }

- 生產端輸出參數明確宣告了 `serializable=False`；若釋放記憶體需要超越引用計數的額外操作，則配置了 `on_local_object_drop`。
- 消費端輸入參數維持標準常規宣告，無多餘設定。
- 生產節點與消費節點處於同一個 Worker 程序中（在同一程式庫內部會自動保證）。
- 包含 URL 網址的任何物件（`ImageUrlArtifact` 等）保持未宣告狀態，以確保其作為資料正常跨行程傳輸。
- 若節點支援在共享模式 (Shared Mode) 下運行，切勿單純依賴釋放掛鉤：因為該模式不產生快取，釋放掛鉤不會被調用。
- 批次傳遞使用常規參數輸出，切勿在 `ParameterList` 輸出端上嘗試快取不可序列化物件。
