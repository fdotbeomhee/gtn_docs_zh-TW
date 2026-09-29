# 資產與輸出管理 (Assets and Outputs) {: #assets-and-outputs }

當工作流程中的節點生成圖像、影片、音訊片段或任何其他檔案時，該檔案必須妥善存放在本機磁碟的特定位置，並能被編輯器正常存取，以便您進行即時預覽與下載。本頁說明這些檔案預設儲存於何處、如何自訂存放路徑，以及外部檔案如何匯入工作流程中（透過上傳或拖放操作）。

## 生成檔案的儲存位置 {: #where-generated-files-go }

節點絕不將檔案寫入寫死的固定絕對路徑。每次檔案儲存皆透過您的**專案 (Project)** 機制進行：專案定義了一組具名的目錄結構（`inputs`、`outputs`、`temp` 等）以及一組稱為**情境 (Situations)** 的檔案儲存規則，用來決定特定類型的檔案應寫入哪個資料夾以及該如何命名。在預設專案配置下，各類檔案的流向如下：

- 節點運算彩現的輸出成品（圖像、彩現影片、生成音訊）透過 [`save_node_output`](projects/situations.md#save_node_output) 情境規則，統一儲存至專案的 `outputs` 目錄中。
- 您直接拖入編輯器或從專案外部複製進來的檔案，透過 [`copy_external_file`](projects/situations.md#copy_external_file) 情境規則，儲存至 `inputs` 目錄中。
- 節點在運算運作期間寫入且運算完畢即清除的臨時暫存檔，存放於 `temp` 目錄。
- 編輯器為您自動生成的縮圖與預覽圖像，存放於隱藏的 `.griptape-nodes-previews` / `.griptape-nodes-thumbnails` 資料夾中。

這些路徑完全沒有硬編碼在引擎內部 — 它們皆由專案組態統一派發，且專案設定檔可以將其中任何目錄重新導向至其他位置（例如共用網路磁碟機、不同的子資料夾或跨平臺相容路徑），亦可自訂檔名衝突時的覆寫策略。各具名目錄清單請參閱[目錄架構 (Directories)](projects/directories.md)，各類檔案適用的儲存邏輯請參閱[情境規則 (Situations)](projects/situations.md)，自訂整體專案請參閱[專案系統 (Project)](projects/index.md)。

## 編輯器如何預覽與下載檔案 {: #how-the-editor-previews-and-downloads-files }

當引擎需要向您展示檔案或供您下載時，它不會直接向瀏覽器拋出裸實體檔案路徑 — 而是簽發一個專用 URL。對於工作區內部的檔案，通常直接指向本地靜態檔案伺服器（預設為 `http://localhost:8124/workspace/...`）。對於任何其他路徑 — 包含工作區外部檔案、`file://` 協議路徑或類似 `{outputs}/render.png` 的巨集路徑 — 引擎會在收到請求時即時簽發一個具備時效限制的預簽章 (Presigned) 下載 URL。無論採用哪種底層傳輸，其機制皆相同：編輯器請求 URL，引擎將其解析為檔案實際所在的實體磁碟位置，瀏覽器隨後調用該 URL 擷取預覽或觸發下載。您無需手動維護這些 URL；它們均按需動態產生，不適合長久保存或跨工作階段分享。

您可能還會在工作區中看到一個 `staticfiles` 資料夾（精確名稱由 `static_files_directory` 設定項決定）。這屬於早期版本的舊式儲存路徑，多數現代節點已不再使用 — 轉而透過上述的專案目錄系統進行儲存 — 但透過引擎舊版靜態檔案 API 儲存的檔案仍會落在此處。

## 將外部檔案上傳至工作流程 {: #uploading-files-into-a-workflow }

將外部檔案（參考照片、來源影片、音訊片段）匯入工作流程的操作邏輯正好相反：您只需將檔案拖放到節點上方，或點擊節點的檔案選取器，編輯器便會將該檔案上傳至專案內部。具體存放位置由專案的 [`copy_external_file`](projects/situations.md#copy_external_file) 情境規則決定。在預設專案中，檔案會寫入 `inputs` 目錄；若副檔名可被系統識別，還會自動歸納至對應的子資料夾（`images`、`videos`、`audio`、`text`）；若目標路徑已存在同名檔案，新上傳的檔案會自動追加數字編號後綴（如 `photo_001.png`）以避免意外覆寫。

在底層實作中，這是兩階段交接傳輸而非瀏覽器直接操作本機檔案系統：編輯器首先向引擎請求專屬的上傳 URL，隨後以 HTTP PUT 方式將檔案二進位資料推播至該 URL。

## 相關參考資源 {: #related-pages }

- [目錄架構 (Directories)](projects/directories.md) — 具名目錄（`inputs`、`outputs`、`temp` 等）完整清單與路徑自訂教學。
- [情境規則 (Situations)](projects/situations.md) — 規範各類節點輸出檔案走向與重名衝突機制的儲存規則。
- [圖像序列 (Sequences)](projects/sequences.md) — 將帶有編號的目錄序列檔案（如 `render.0001.exr`、`render.0002.exr` 等）作為單一連續序列讀取。
- [巨集語法 (Macros)](projects/macros.md) — 情境規則用於動態構建檔案路徑的範本語法（如 `{outputs}/{file_name_base}.{file_extension}`）。
- [組態配置參考手冊 (Configuration Reference)](../reference/configuration_reference.md) — 各項引擎組態參數（`static_files_directory`、`workspace_directory` 等）的預設值與環境變數覆寫規範。
