# 鍵盤快速鍵 (Keyboard Shortcuts) {: #keyboard-shortcuts }

編輯器中所有鍵盤快速鍵的參考清單，依使用情境分門別類彙整。修飾鍵在不同作業系統有所差異時，macOS 與 Windows / Linux 的快速鍵分欄列出。

!!! note "文字輸入絕不受干擾"

    當您在文字輸入方塊、節點參數欄位或 Code 程式碼面板中打字時，大部分畫布單鍵快速鍵（例如 `R` 或 `N`）會被自動忽略，確保輸入的一般字母能夠正確填入欄位，而不會意外觸發快速鍵動作。

關於這些快速鍵所作用的介面區塊完整導覽，請參閱[編輯器概述 (The Editor)](index.md)。

## 添加與編輯節點 (Adding & editing nodes) {: #adding-editing-nodes }

| 操作動作 | macOS | Windows / Linux |
| ---------------------------------------------------------- | ------------------ | ------------------ |
| 在游標處添加節點（僅限游標位於畫布上方時） | Tab | Tab |
| 在游標處添加節點，若游標不在畫布上則為全選 | Shift+A | Shift+A |
| 在游標處建立便簽節點 (Note) | N | N |
| 重新命名選取的節點 | R | R |
| 複製副本選取的節點 (Duplicate) | Cmd+D | Ctrl+D |
| 鎖定 / 解鎖選取的節點 | L | L |
| 刪除選取的節點或連線 | Delete / Backspace | Delete / Backspace |

## 選取控制 (Selection) {: #selection }

| 操作動作 | macOS | Windows / Linux |
| ----------------------------------------------------- | -------------------- | -------------------- |
| 全選所有節點 | Cmd+A | Ctrl+A |
| 清除目前選取狀態 | Escape | Escape |
| 在選取集合中加入或移除節點 | Shift+點擊節點 | Shift+點擊節點 |
| 框選多個節點 | 在空白畫布上拖曳滑鼠 | 在空白畫布上拖曳滑鼠 |
| 將選取焦點移至該方向最近的節點 | 方向鍵 | 方向鍵 |

## 剪貼簿 (Clipboard) {: #clipboard }

| 操作動作 | macOS | Windows / Linux |
| ------------------------------------------------------ | ----- | --------------- |
| 複製選取的節點 (Copy) | Cmd+C | Ctrl+C |
| 貼上節點，或從剪貼簿貼上圖像/多媒體檔案 (Paste) | Cmd+V | Ctrl+V |

## 視圖巡覽與縮放 (Navigation & view) {: #navigation-view }

| 操作動作 | macOS | Windows / Linux |
| ------------------------------------------------------------------ | ------------------- | ------------------- |
| 將視圖置中縮放至選取節點；若未選取則為完整流程視圖 (Frame) | F | F |
| 將選取的群組框貼合其內部節點（否則為聚焦選取物件） | Shift+F | Shift+F |
| 平移畫布（即使游標懸停在節點上方依然有效） | 按住 Space + 拖曳滑鼠 | 按住 Space + 拖曳滑鼠 |
| 平移畫布 | 按住滾輪中鍵 + 拖曳滑鼠 | 按住滾輪中鍵 + 拖曳滑鼠 |
| 滑動流暢縮放 — 向右拖曳放大，向左拖曳縮小 (Scrubby zoom) | 按住 Z + 拖曳滑鼠 | 按住 Z + 拖曳滑鼠 |
| 放大視圖 | Cmd+= | Alt+= |
| 縮小視圖 | Cmd+- | Alt+- |
| 放大 / 縮小視圖 | 滾動滾輪或觸控板捏合 | 滾動滾輪或觸控板捏合 |

## 群組與版面配置 (Groups & layout) {: #groups-layout }

| 操作動作 | macOS | Windows / Linux |
| ------------------------------------------------ | ----------- | --------------- |
| 將選取的節點建立為群組（使用預設群組類型） | Cmd+G | Ctrl+G |
| 建立群組（每次皆跳出選擇群組類型） | Shift+Cmd+G | Shift+Ctrl+G |
| 自動重排圖形版面 (Auto-arrange) | Shift+G | Shift+G |

各群組類型的具體含義以及自動重排的排版規則，請參閱[節點分組 (Node Groups)](node_groups.md)。

## 連線控制 (Connections) {: #connections }

| 操作動作 | macOS | Windows / Linux |
| -------------------------------------------------------------- | -------- | --------------- |
| 切換顯示/隱藏已建立連線的參數端點 | Shift+H | Shift+H |
| 在游標懸停/選取的連線上插入中繼點節點 (Dot / Reroute) | I | I |
| 游標懸停在連線上時顯示該連線的操作按鈕群組 | 按住 Cmd | 按住 Ctrl |

## 工作流程與檔案操作 (Workflow / file) {: #workflow-file }

| 操作動作 | macOS | Windows / Linux |
| --------------------------------- | --------------- | ---------------- |
| 儲存工作流程 (Save) | Cmd+S | Ctrl+S |
| 另存新檔 (Save As...) | Shift+Cmd+S | Shift+Ctrl+S |
| 另存為新版本 (Save As New Version) | Option+Shift+S | Alt+Shift+S |
| 開啟工作流程 (Open Workflow) | Cmd+O | Ctrl+O |
| 建立新工作流程 (New Workflow) | Shift+Cmd+O | Shift+Ctrl+O |
| 運行至選取的目標節點 (Run To Selected) | Cmd+Enter | Ctrl+Enter |
| 完整運行整個工作流程 (Run Workflow) | Shift+Cmd+Enter | Shift+Ctrl+Enter |
| 將工作流程匯出下載為圖片 (Download as Image) | Cmd+I | Ctrl+I |
| 切換顯示/隱藏右側側邊欄 | Cmd+B | Ctrl+B |
| 切換顯示/隱藏左側側邊欄 | Shift+Cmd+B | Shift+Ctrl+B |
| 顯示此快速鍵參考對話方塊 | Shift+? | Shift+? |
