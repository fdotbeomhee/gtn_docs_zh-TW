# ReverseVideo {: #reversevideo }

## 功能概述 {: #what-is-it }

ReverseVideo 節點可顛倒視訊的時間播放方向，使其完全倒放 (Play Backwards)。它同時支援對音訊軌道進行同步倒放或靜音處理，以滿足多樣化的創作需求。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 ReverseVideo 節點：

- 創造時光倒流或倒放視覺特效
- 出於藝術構思或創意剪輯倒播特定片段
- 製作需要逆向動態的視覺特效（如物體復原、水流逆流）
- 為視訊創作注入超現實主義或夢境般的神秘質感
- 為特定影音後製流程倒轉局部影片片段
- 針對高速動作或物理運動進行逆向軌跡運動分析

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ReverseVideo 節點新增至工作流程中
1. 將視訊來源連線至 "video" 輸入埠
1. 選擇音訊處理模式（倒放 reverse、靜音 mute 或保留原音 keep）
1. 執行工作流程以產生倒放視訊

### 參數說明 {: #parameters }

- **video**：欲倒放的目標視訊素材（相容 VideoArtifact 與 VideoUrlArtifact 格式）

- **audio_handling**：音訊軌道的處理策略（預設值："reverse"）

    - "reverse"：視訊與音訊同時同步倒放（推薦，呈現最自然的倒帶體驗）
    - "mute"：僅倒放畫面，完全靜音剔除音軌
    - "keep"：倒放畫面，但維持原始音訊正向播放（不建議，極易產生嚴重違和感）

- **processing_speed**：運算速度與輸出畫質平衡模式（預設值："balanced"）

    - **fast**：極速轉碼，畫質稍有妥協（採用 ultrafast preset，CRF 30）
    - **balanced**：速度與畫質極佳平衡（採用 medium preset，CRF 23）
    - **quality**：極致清晰頂級畫質，轉碼稍慢（採用 slow preset，CRF 18）

### 輸出埠說明 {: #outputs }

- **video**：完成倒放處理的視訊產物，可供後續節點繼續引用

## 範例展示 {: #example }

為普通短片營造時間倒流視覺特效：

1. 將 ReverseVideo 節點新增至工作流程中
1. 將來自 LoadVideo 節點的視訊輸出連線至 ReverseVideo 的 "video" 輸入埠
1. 將 "audio_handling" 設為 "reverse"，實現視訊與音訊雙重倒帶
1. 執行工作流程——影片將即刻呈現逆向播放
1. 輸出檔案名稱將自動格式化為 `{original_filename}_reversed_reverse.{format}`

## 重要注意事項 {: #important-notes }

- ReverseVideo 節點底層使用 FFmpeg 高品質反向時間戳與重排濾波演算法。
- 同步倒放視訊與音訊能呈現最原汁原味的錄音帶倒帶感。
- 若倒放音訊聽感過於怪異，建議選用 "mute" 靜音模式。
- "keep" 選項因時間軸徹底錯位，極易造成音畫嚴重脫節。
- 運算時長取決於視訊長度（倒放需要將畫格反向載入緩衝區）。
- 節點會自動智慧檢測來源影片是否自帶音軌，並做出相應處理；對於純無聲視訊也能正常順利轉碼。
- 支援輸出詳細的處理日誌以供除錯與效能診斷。

## 音訊處理推薦設定 {: #audio-handling-recommendations }

- **自然倒帶特效**：使用 "reverse"——音畫同步倒放
- **無聲純視覺倒放**：使用 "mute"——純畫面倒放，消除雜音干擾
- **自訂背景音樂配樂**：使用 "mute" 靜音後，於後續管線獨立混流自訂音軌

## 常見問題排解 {: #common-issues }

- **轉碼超時 (Processing Timeout)**：超大視訊檔案倒放處理耗時較長；節點預設設有 5 分鐘安全逾時機制。
- **倒放聲音怪異難聽**：若逆向音訊不適應，請將 audio_handling 改選為 "mute"。
- **音畫不同步脫節**：請勿在倒放時選用 "keep"，請改用 "reverse" 或 "mute"。
- **無視訊輸入**：請確認已將有效的視訊來源連接至 "video" 輸入埠。
- **FFmpeg 錯誤**：若轉碼失敗，請檢視 logs 參數以獲取底層編碼器的詳細錯誤報告。
