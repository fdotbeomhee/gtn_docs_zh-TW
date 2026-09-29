# 管理模型與程式庫 (Managing Models and Libraries) {: #managing-models-and-libraries }

某些節點在本地電腦執行本機 AI 模型，在執行前必須先將模型權重檔案下載至磁碟；另一些節點則來自尚未安裝的第三方擴充程式庫。這兩者皆由編輯器的 **Manage (管理)** 功能表統一維護，分別對應 **Model Management (模型管理)** 與 **Library Management (程式庫管理)** 兩大專屬視窗。本頁將逐一介紹這兩個視窗的操作方法。

關於擴充程式庫的核心架構概念 — 例如各安裝套件之間如何相互隔離、共用模式 (Shared) 與隔離模式 (Isolated) 執行差異，以及載入失敗時的除錯 SOP — 請參閱[節點程式庫 (Libraries)](../libraries.md)。本頁著重於介紹這兩個圖形管理視窗本身的使用方式。

## 模型管理 (Model Management) {: #model-management }

前往 **Manage → Model Management** 可搜尋、下載與清理本機 AI 模型檔案。

該視窗支援的所有功能亦可透過終端機 CLI 命令 `gtn models` 完整執行（包含 search、download、list、delete 與 download status）— 請參閱 [CLI 參考手冊](../../reference/command_line_interface.md#models)。

### 搜尋模型 {: #searching-for-a-model }

在搜尋輸入方塊中輸入關鍵字，系統會即時向 Hugging Face 查詢相符的模型項目。相符的搜尋結果會以快顯下拉選單列出，展示各模型的 ID、簡短描述、下載次數、按讚數以及部分分類標籤；您本機已經下載完成的模型會標記 **Installed (已安裝)** 徽章。點擊任一結果進行選取 — 輸入欄位隨即會顯示該模型的完整檔案大小（待引擎查詢完畢後），並附帶直達該模型 Hugging Face 頁面的超連結。

若您已確知精確的模型識別碼 (Model ID)，亦可直接完整輸入，無需依賴搜尋與下拉清單。

### 下載模型 {: #downloading-a-model }

選定模型後，點擊 **Download (下載)**。下載程序會在背景自動進行 — 您可以繼續在畫布上構建工作流程、關閉管理視窗或切換至 Library Management，背景下載作業絕不受干擾。

### 追蹤下載進度 {: #tracking-progress }

**Downloads (下載中)** 區塊詳細列出所有進行中與最近剛完成的下載任務，並標記明確的狀態徽章：

| 狀態徽章 | 具體狀態說明 |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Downloading** | 正在下載中。顯示動態進度條、目前已下載的 GB 數 / 總 GB 數，以及完成百分比。 |
| **Completed** | 下載成功；進度條呈滿格綠色顯示。 |
| **Failed** | 下載失敗。錯誤訊息會印在該條目下方（若失敗原因是尚未取得授權，會包含 Hugging Face 存取申請連結 — 詳見下文[門禁模型需配置 Token](#gated-models-need-a-token)）。 |

使用清單上方的 **All / Downloads / Models** 篩選標籤，可將檢視範圍限縮至僅顯示活躍下載任務或僅顯示本機已安裝模型。

### 取消下載 {: #cancelling-a-download }

在仍處於下載狀態的任務條目上點擊垃圾桶圖示可予以取消。系統會先彈出 **Cancel Download** 確認視窗 — 點擊 **Keep Downloading** 可恢復下載，點擊 **Cancel Download** 則立即中止傳輸並清除未完成的暫存下載記錄。

### 刪除模型 {: #deleting-a-model }

**Installed Models (已安裝模型)** 區塊列出本機磁碟中已下載的所有模型，並載明其本地存放路徑與佔用容量。點擊垃圾桶圖示可將其刪除。系統會先彈出 **Delete Model** 確認對話方塊 — 刪除操作為永久性破壞操作，檔案將從本機儲存空間中徹底抹除，任何依賴該模型的工作流程在您重新下載或改換其他模型之前都將無法正常執行。

### 門禁模型需配置 Token (Gated models need a token) {: #gated-models-need-a-token }

Hugging Face 上的部分模型屬於「門禁管制模型 (Gated models)」— 您必須先在其官方頁面提交存取授權申請，且必須在 Griptape Nodes 中預先配置 Hugging Face Access Token 才能進行下載。若系統未偵測到 Token，Model Management 視窗頂部會顯示提示橫幅，指引解決步驟：前往 Hugging Face 建立個人 Token，然後將其作為 `HF_TOKEN` 鍵值填入 **Settings → API Keys & Secrets** 中。點擊橫幅中的連結可直接跳轉至設定畫面。

詳細的操作教學與特定門禁模型的申請流程，請參閱[使用 Hugging Face 節點的前置設定](../integrations/hugging_face.md)。

## 程式庫管理 (Library Management) {: #library-management }

前往 **Manage → Library Management** 可安裝、更新與移除節點擴充程式庫。在終端機中，`gtn libraries download <git-url>` 與 `gtn libraries sync` 分別對應安裝與更新功能 — 請參閱 [CLI 參考手冊](../../reference/command_line_interface.md#libraries)。

頂部的篩選列支援依名稱搜尋已安裝的程式庫，並可透過 **All / Updates / Errors** 標籤快速縮小範圍 — 其中 **Errors** 是快速揪出載入失敗程式庫的最有效途徑。標籤旁邊的兩個圖示按鈕分別支援**檢查所有程式庫是否有更新**以及單純**重新整理 (Refresh)** 清單。

點擊任一程式庫列可將其展開，查閱其詳細說明、Git 遠端倉庫 URL 與目前參照的 Ref（針對 Git 安裝的程式庫），以及引擎回報的任何載入異常資訊。

### 安裝擴充程式庫 {: #installing-a-library }

點擊 **Add Library (新增程式庫)** 開啟對話方塊。貼入 Git 倉庫網址 — 例如託管於 GitHub 上的社群自訂程式庫 — 然後點擊 **Install**。

編輯器在正式執行 Git Clone 之前，會先連線探測該倉庫，並在對話方塊中向您展示該程式庫的正式名稱、功能描述、版本號與內含節點總數確認資訊，方便您在發現貼錯網址時及時退出。

展開 **Advanced Options (進階選項)** 可進行細部調整：

- **Branch / Tag / Commit** — 安裝指定的特定 Git Ref 分支、標籤或 Commit Hash，而非預設主分支。
- **Download Directory (下載目錄)** — 若不希望存放在預設的 libraries 目錄中，可在此自訂複製路徑。點擊資料夾圖示可手動瀏覽選取。

若不確定要安裝什麼，表單下方的 **Browse Community Libraries** 提供了官方精選推薦的社群程式庫目錄，您可以直接複製網址並貼回上方欄位。

若目標安裝目錄中已經存在檔案 — 例如先前安裝過的同名程式庫或無關的雜湊檔案 — 系統會跳出確認視窗詢問是否進行覆寫。覆寫具有破壞性：它會抹除目標資料夾中的既有內容（包含未提交至 Git 的本機修改），並以全新安裝取而代之。當您嘗試更新帶有本機修改的程式庫時，亦會觸發相同的覆寫警告。

### 更新程式庫 {: #updating-a-library }

展開帶有 Git Remote 的程式庫條目，點擊 **Check for Updates (檢查更新)** 即可比對遠端倉庫版本。若有可用更新，該列會亮起 **Update** 按鈕（若引擎已知目標版本號，會明確標註如 **Update to 1.4.0**）；點擊即可拉取更新。若更新目前處於延遲保護期 — 某些發行版設有最小存活時間門檻，避免使用者第一時間拉取數分鐘前剛發布的潛在不穩定代碼 — 該列會改為顯示倒數計時提示，告知距離允許更新還需等待多久。

您也可以直接在展開的面板中將程式庫就地切換至不同的分支、標籤或 Commit：點擊目前的 Ref 即可就地編輯並確認切換。

若某個程式庫的依賴套件未能自動安裝成功，展開列中的 **Install Dependencies (安裝相依套件)** 可單獨重試依賴套件安裝步驟，無需重新 Clone 整個倉庫。

### 移除程式庫 {: #removing-a-library }

Library Management 視窗本身不直接自磁碟中實體抹除程式庫檔案 — 關於如何實體刪除、如何在不移除檔案的前提下暫時停用程式庫，以及如何在共用 (Shared) 與隔離 (Isolated) 模式之間進行切換，請參閱 Libraries 專題指南中的[切換與移除程式庫](../libraries.md#toggling-and-removing-libraries)。
