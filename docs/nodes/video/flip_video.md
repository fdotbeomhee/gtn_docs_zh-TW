# FlipVideo {: #flipvideo }

## 功能概述 {: #what-is-it }

FlipVideo 節點可將視訊進行水平翻轉、垂直翻轉或雙向同時翻轉。這能產生對稱鏡像視覺特效，並可用於校正錄影方向錯誤或創造獨特藝術效果。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 FlipVideo 節點：

- 校正顛倒錄製或左右鏡像相反的視訊素材
- 創造富有藝術張力的鏡像對稱效果
- 翻轉特定畫面視角以匹配其他相鄰剪輯片段
- 為特定社群短影音製作鏡像翻轉版本
- 製作左右對稱的抽象視訊圖案
- 修正手機或相機感光元件的方向判定瑕疵

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 FlipVideo 節點新增至工作流程中
1. 將視訊來源連線至 "video" 輸入埠
1. 挑選翻轉方向（horizontal 水平、vertical 垂直或 both 雙向）
1. 執行工作流程以完成視訊鏡像翻轉

### 參數說明 {: #parameters }

- **video**：欲翻轉的目標視訊素材（相容 VideoArtifact 與 VideoUrlArtifact 格式）

- **direction**：翻轉幾何方向（預設值："horizontal"）

    - "horizontal"：水平左右翻轉（鏡像效果，左變右、右變左）
    - "vertical"：垂直上下翻轉（上下顛倒）
    - "both"：同時進行水平與垂直雙向翻轉（等同中心旋轉 180 度）

- **processing_speed**：運算速度與輸出畫質平衡模式（預設值："balanced"）

    - **fast**：極速轉碼，畫質稍有妥協（採用 ultrafast preset，CRF 30）
    - **balanced**：速度與畫質極佳平衡（採用 medium preset，CRF 23）
    - **quality**：極致清晰頂級畫質，轉碼稍慢（採用 slow preset，CRF 18）

### 輸出埠說明 {: #outputs }

- **video**：完成翻轉處理的視訊產物，可供後續節點繼續引用

## 範例展示 {: #example }

為短片製作左右對調的水平鏡像特效：

1. 將 FlipVideo 節點新增至工作流程中
1. 將來自 LoadVideo 節點的視訊輸出連線至 FlipVideo 的 "video" 輸入埠
1. 將 "direction" 設為 "horizontal" 建立水平鏡像
1. 執行工作流程——影片畫面將即刻完成水平左右對調
1. 輸出檔案名稱將自動格式化為 `{original_filename}_flipped_horizontal.{format}`

## 重要注意事項 {: #important-notes }

- FlipVideo 節點底層使用 FFmpeg 高品質影像幾何變換濾波器。
- 水平翻轉產生鏡像效果（左右對換）。
- 垂直翻轉產生上下顛倒效果。
- 雙向翻轉同時在兩個軸向上翻轉。
- **原始音訊完美保留**：原始視訊自帶的音訊軌道將完整保留且同步。
- 轉碼處理時長取決於視訊長度與解析度規格。
- 支援輸出詳細的處理日誌以供除錯與效能診斷。

## 翻轉方向推薦設定 {: #direction-recommendations }

- **鏡像視覺特效**：使用 "horizontal"——產生左右鏡像反射
- **上下顛倒校正**：使用 "vertical"——修正倒置拍攝的畫面
- **完全旋轉翻轉**：使用 "both"——在兩個維度同時翻轉
- **鏡頭方向修正**：依據實體鏡頭錄製時的實際偏差角度選取

## 常見問題排解 {: #common-issues }

- **轉碼超時 (Processing Timeout)**：超大檔案處理可能耗時較長；節點預設設有 5 分鐘安全逾時機制。
- **翻轉方向不符合預期**：請重新確認 direction 參數是否選取了正確的幾何軸向。
- **無視訊輸入**：請確認已將有效的視訊來源連接至 "video" 輸入埠。
- **FFmpeg 錯誤**：若轉碼失敗，請檢視 logs 參數以獲取底層編碼器的詳細錯誤報告。
