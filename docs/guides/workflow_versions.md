# 工作流程版本控制 (Workflow Versions) {: #workflow-versions }

點選 **File → Save As New Version (另存為新版本)**（快速鍵 `Option/Alt` + `Shift` + `S`），可將工作流程的當前狀態寫入新檔案，而不是覆寫目前已開啟的檔案，並自動切換至編輯該新檔案。再次執行儲存即可取得另一個新檔案 — 每次儲存皆會循序生成下一個版本，將先前磁碟上的所有歷史版本完整保留。

## 生成規則 {: #what-it-produces }

版本化儲存會在工作流程檔名後追加固定位數補零的 `_v<number>` 後綴：以這種方式儲存 `my_workflow.py` 會生成 `my_workflow_v001.py`，接著是 `my_workflow_v002.py`，依此類推。版本號始終自動遞增前進；絕不會覆寫磁碟上現有的歷史版本檔案。

<!-- screenshot (#5166): the File menu open with Save As New Version highlighted -->

## 何時使用另存為新版本 {: #when-to-use-it }

當您希望建立一個能隨時還原回退的檢查點 (Checkpoint) 而又不遺失當前進度時，請使用另存為新版本 — 例如在進行高風險的流程重構之前、將工作流程移交給他人之前，或是單純在迭代過程中保留完整的快照歷史軌跡。因為每個版本皆是完全獨立的實體檔案，要退回舊版本只需直接開啟該檔案即可。

## 與 Save 和 Save As 的本質差異 {: #how-it-differs-from-save-and-save-as }

- **Save (儲存)**：直接覆寫目前開啟的檔案。不會建立任何新檔案。
- **Save As (另存新檔)**：以*您自訂*的新名稱寫入獨立的新檔案，並切換至編輯該檔案：這是一次性的複本建立。
- **Save As New Version (另存為新版本)**：同樣會寫入新檔案，但檔名由系統自動為您安排（序號中的下一個數字），且專為持續循環使用而設計 — 後續從該檔案出發的每次儲存皆會持續遞增同一個版本序列。

自動儲存 (Autosave) 功能在啟用時，其行為等同於 **Save**：它會按固定時間間隔靜默覆寫目前開啟的檔案，絕不會自行建立版本化檔案。設定方式請參閱[自動儲存 (Auto-save)](editor/running_workflows.md#auto-save)。

!!! note "與引擎或程式庫版本鎖定的差異"

    工作流程版本單純只是工作流程*檔案*的編號備份 — 它與專案正確執行所需的引擎或程式庫*版本*毫無關聯。若您需要將專案鎖定於經過驗證的引擎版本或特定程式庫版本集合，請參閱[鎖定引擎與程式庫版本](projects/version_pinning.md)。
