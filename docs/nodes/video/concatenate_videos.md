# ConcatenateVideos {: #concatenatevideos }

## 功能概述 {: #what-is-it }

ConcatenateVideos 節點允許您將多段視訊素材拼接合併為單一連貫的視訊檔案，並提供高度靈活的格式封裝與編碼器 (Codec) 選項。它具備智慧解析度適配能力，會自動將不同尺寸的視訊縮放對齊至第一段視訊的規格，確保拼接畫面平滑無縫且杜絕視覺跳變。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 ConcatenateVideos 節點：

- 將多個獨立的短片片段拼接為單一完整影片
- 製作精彩集錦 (Montages)、作品回顧或視訊合輯
- 將先前被裁切或分段錄製的影片區段重新整合
- 將多個由 AI 視訊模型生成的短秒數片段組裝為長篇故事影片
- 串聯各個獨立視訊素材建立播放清單序列
- 拼接不同解析度的視訊片段（自動縮放演算法無縫處理尺寸不匹配問題）

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 ConcatenateVideos 節點新增至工作流程中
1. 將多個視訊輸入連線至 **video_inputs** 參數埠（支援個別連線或直接傳入視訊清單）
1. 依據需求配置輸出封裝格式與編解碼器設定
1. 將 **output** 輸出埠接駁至後續需要視訊輸入的節點

### 參數說明 {: #parameters }

- **video_inputs**：欲拼接的視訊清單（相容 VideoArtifact、VideoUrlArtifact 及兩者的混合清單）
- **output_format**：輸出視訊封裝格式——可選 mp4、avi、mov、mkv 或 webm（預設值：mp4）
- **video_codec**：輸出視訊編碼器——可選 libx264、libx265、libvpx-vp9 或 copy（預設值：libx264）
- **audio_codec**：輸出音訊編碼器——可選 aac、mp3、libmp3lame 或 copy（預設值：aac）
- **output_frame_rate**：畫格率控制（繼承自 BaseVideoProcessor）
- **processing_speed**：轉碼畫質與運算速度取捨（繼承自 BaseVideoProcessor）

### 輸出埠說明 {: #outputs }

- **output**：拼接完成的最終視訊檔案，封裝為 VideoUrlArtifact
- **logs**：處理過程的詳細日誌，記錄檔案下載、尺寸檢測與自適應縮放的完整歷程

## 視訊自適應自動縮放機制 {: #automatic-video-resizing }

ConcatenateVideos 節點可智慧應對多段輸入視訊解析度各異的情況：

### 運作原理 {: #how-it-works }

1. **尺寸自動偵測**：節點底層透過 FFprobe 自動檢測所有輸入視訊的幾何寬高
1. **基準尺寸選定**：**第一段輸入視訊的解析度**將自動作為整體拼接目標的基準規格
1. **智慧高品質縮放**：尺寸不吻合的後續視訊將由 FFmpeg 自動進行高品質等比縮放
1. **長寬比嚴格保護**：縮放時嚴格保護原始畫面長寬比例，必要時自動添加黑邊 (Black Padding / Pillarbox)
1. **完整日誌回報**：所有縮放轉換操作皆會詳細記錄前後解析度變化

### 縮放處理策略 {: #resizing-behavior }

- **尺寸全數一致**：若所有視訊解析度完全相同，直接進行極速無損拼接，不觸發重新縮放
- **混合尺寸素材**：所有非基準視訊自動縮放對齊至第一段視訊的規格
- **畫質抗失真保護**：採用高品質重採樣插值濾波與比例保護
- **音訊軌道保護**：在畫面幾何尺寸調整過程中，音訊串流直接無損複製，絕不重複轉碼

### 運算日誌範例 {: #log-examples }

當所有視訊解析度一致時：

```
✅ All videos have matching dimensions - no resizing needed
```

當需要自適應調整尺寸時：

```
🔄 Videos have different dimensions - resizing to match first video...
Target dimensions (from first video): 1920x1080
• Video 1: 1920x1080 (reference)
• Video 2: 1280x720 → 1920x1080 (resizing...)
  ✅ Video 2 resized successfully
• Video 3: 1920x1080 (no resize needed)
```

## 範例展示 {: #example }

將多段 AI 生成的短片段合併為長篇流暢影片：

1. 將 ConcatenateVideos 節點新增至工作流程中
1. 將多個 AI 視訊生成節點的產物連線聚合至 **video_inputs** 參數埠
1. 將 **output_format** 設為 "mp4"，將 **video_codec** 設為 "libx264" 以獲得最廣泛的裝置相容性
1. 將 **audio_codec** 設為 "aac" 以保留高保真音質
1. 執行節點，所有短片片段將自動組裝為單一連貫的完整影片

## 進階使用指南 {: #advanced-usage }

### 格式與編解碼器選用建議 {: #format-and-codec-selection }

- **mp4 + libx264**：網路播放、社群媒體與通用設備的最佳相容組合
- **mov + libx264**：專業影音後製與剪輯軟體工作流程的理想選擇
- **mkv + libx265**：極致壓縮效率，大幅節省硬碟儲存空間
- **webm + libvpx-vp9**：針對現代網頁 HTML5 串流播放深度最佳化
- **copy 編碼器**：當所有輸入片段的格式與編碼規格完全一致時，可實現秒級極速無損封裝

### 效能最佳化建議 {: #performance-tips }

- 若所有輸入視訊具備相同的編碼與規格，將視訊/音訊編碼器選為 "copy" 可免除重複轉碼開銷，速度最快。
- 快速預覽可選 "fast" 處理速度，最終發布渲染建議選 "quality"。
- 在品質相仿的前提下，libx265 (HEVC) 產出的檔案體積顯著小於 libx264。
- **最佳效能實踐**：盡量使用解析度一致的素材，以避免觸發額外的縮放轉碼管線。
- **視訊排序策略**：**將最高畫質或具備目標目標解析度的影片放置於清單首位**，確立正確的基準規格。

## 重要注意事項 {: #important-notes }

- **最低輸入數量限制**：拼接操作**至少需要 2 段視訊**。
- **多格式廣泛支援**：支援 mp4、avi、mov、mkv、webm 等主流視訊格式。
- **雲端/遠端資源自動下載**：自動下載遠端 URL 視訊資源並預先快取以備拼接。
- **音畫同步鎖定**：在整段拼接影片中精準鎖定並維持音畫同步。
- **Web 串流預載最佳化**：自動注入 `faststart` 標誌 (moov atom 前置)，確保網頁串流秒開播放。

## 常見問題排解 {: #common-issues }

- **提示「At least 2 videos required」**：請確認連線至節點的視訊數量至少達到兩段。
- **格式規格衝突**：若各片段來源格式差異極大，請避免選用 "copy" 編碼器，應讓節點重新轉碼。
- **超長視訊處理耗時**：針對超大檔案，可切換為 "fast" 處理模式以加快運算。
- **音訊格式衝突**：若各片段音訊編碼不同，請使用 aac 或 mp3 重新編碼，避免選用 copy。
- **記憶體資源佔用過高**：同時處理多段超大解析度視訊需要足夠的系統記憶體支援。
