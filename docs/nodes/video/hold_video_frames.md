# HoldVideoFrames {: #holdvideoframes }

## 功能概述 {: #what-is-it }

HoldVideoFrames 節點透過將每個畫面畫格定格保留指定的畫格數，創造出步進式 (Stepped) 定格動畫視訊特效。這能營造出逐格動畫 (Stop-motion) 或抽格定格的視覺衝擊，為您的視訊創作增添獨特的藝術張力與復古質感。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 HoldVideoFrames 節點：

- 打造定格動畫 (Stop-motion) 或黏土動畫風格特效
- 降低高速移動畫面的頻率以方便肉眼清晰辨識細節
- 透過畫格抽格與定格操控創造前衛藝術視覺
- 為現代高流暢度影片增添復古 8-bit 或早期電影的抽格質感
- 製作極富戲劇張力的慢動作或卡頓節奏特效
- 減慢動態畫面以便進行逐畫格視覺分析與審查

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 HoldVideoFrames 節點新增至工作流程中
1. 將視訊來源連線至 "video" 輸入埠
1. 設定 "hold_frames" 參數以指定每個畫格定格持續的畫格數
1. 執行工作流程以生成抽格定格特效視訊

### 參數說明 {: #parameters }

- **video**：欲施加定格抽格效果的目標視訊（相容 VideoArtifact 與 VideoUrlArtifact 格式）

- **hold_frames**（定格畫格數，範圍 1-10，預設值：2）：

    - 1 = 無特效（原始流暢播放速度）
    - 2 = 每個畫格定格停留 2 畫格（呈現輕微抽格降速感）
    - 5 = 每個畫格定格停留 5 畫格（呈現顯著定格動畫步進感）
    - 10 = 每個畫格定格停留 10 畫格（極度緩慢且強烈的逐格步進）

- **processing_speed**：運算速度與輸出畫質平衡模式（預設值："balanced"）

    - **fast**：極速轉碼，畫質稍有妥協（採用 ultrafast preset，CRF 30）
    - **balanced**：速度與畫質極佳平衡（採用 medium preset，CRF 23）
    - **quality**：極致清晰頂級畫質，轉碼稍慢（採用 slow preset，CRF 18）

### 輸出埠說明 {: #outputs }

- **video**：完成抽格定格特效處理的視訊產物，可供後續節點繼續引用

## 範例展示 {: #example }

為普通短片營造定格逐格動畫 (Stop-motion) 特效：

1. 將 HoldVideoFrames 節點新增至工作流程中
1. 將來自 LoadVideo 節點的視訊輸出連線至 HoldVideoFrames 的 "video" 輸入埠
1. 將 "hold_frames" 參數設為 3，讓每幀畫面停留 3 個畫格時長
1. 執行工作流程——影片將展現出充滿節奏感的定格步進風格
1. 輸出檔案名稱將自動格式化為 `{original_filename}_hold_3_frames.{format}`

## 重要注意事項 {: #important-notes }

- HoldVideoFrames 節點底層使用 FFmpeg 高精度畫格重採樣濾波演算法。
- 該特效實際上會依照 hold_frames 的倍數等比縮減有效畫格更新率。
- **音訊軌道保留特性**：原始音訊軌道會被保留，但由於視覺畫面進行了抽格拉長，音畫可能無法保持嚴格對齊。
- 運算耗時取決於視訊長度與 hold_frames 數值大小。
- 支援輸出詳細的處理日誌以供除錯與效能診斷。

## 特效調校推薦設定 {: #effect-recommendations }

- **微弱定格減速**：使用 hold_frames 2-3，營造輕柔的電影抽格感
- **標準逐格動畫 (Stop-motion)**：使用 hold_frames 4-6，呈現鮮明的逐格步進特色
- **強烈戲劇性慢放**：使用 hold_frames 7-10，呈現極度緩慢的卡點視覺衝擊
- **動作分析審查**：使用 hold_frames 2-3，將極快動作放慢以便於視覺檢查

## 常見問題排解 {: #common-issues }

- **轉碼超時 (Processing Timeout)**：超大視訊檔案處理耗時較長；節點預設設有 5 分鐘安全逾時機制。
- **特效過於強烈或卡頓**：若畫面過於停滯，請將 hold_frames 調小至 2 或 3。
- **音畫不同步問題**：畫格定格操作會改變視覺的時間感知，請注意音軌對齊效果。
- **無視訊輸入**：請確認已將有效的視訊來源連接至 "video" 輸入埠。
- **FFmpeg 錯誤**：若轉碼中斷，請檢視 logs 參數以獲取底層編碼器的詳細錯誤報告。
