# TranscribeAudio {: #transcribeaudio }

## 功能簡介 {: #what-is-it }

TranscribeAudio 節點調用 OpenAI 的模型將語音音訊轉換為文字。它支援多種轉錄模型，並能透過直接的模型參數設定或結合代理人 (Agent) 架構來執行語音辨識與轉錄。

## 適用時機 {: #when-would-i-use-it }

在以下情境中適合使用本節點：

- 將語音轉換為文字
- 轉錄現場或檔案錄音內容
- 從音訊檔案中提取對話文字
- 將語音輸入轉化為書面文字形式
- 為各類音訊內容建立精確的文字逐字稿

## 操作指南 {: #how-to-use-it }

### 基本設定 {: #basic-setup }

1. 將 TranscribeAudio 節點新增至工作流程中。
1. 將音訊來源連接至 "audio" 輸入端。
1. 挑選轉錄模型或連線現有的代理人。
1. 執行節點以生成文字轉錄結果。

### 參數說明 {: #parameters }

- **agent**：選填，用於轉錄任務的現有代理人物件配置。
- **model**：欲使用的語音轉錄模型（預設值為 "gpt-4o-mini-transcribe"）。
- **audio**：必填，待轉錄的音訊檔案。
- **output**：轉錄完成後的文字輸出。

### 輸出端說明 {: #outputs }

- **output**：從音訊中轉錄出來的文字內容。
- **agent**：用於執行轉錄任務的代理人物件，可繼續傳遞給後續節點。

## 範例示範 {: #example }

錄製音訊並自動轉為文字的完整工作流程：

1. 新增 Microphone 節點以錄製語音輸入。
1. 將 Microphone 節點的 "audio" 輸出端連接至 TranscribeAudio 節點的 "audio" 輸入端。
1. 挑選您偏好的語音轉錄模型。
1. 執行該工作流程。
1. 轉錄完成的文字字串即可從 "output" 參數中讀取。

## 重點注意事項 {: #important-notes }

- 本節點需要在您的環境變數中設定合法的 OpenAI API 金鑰 `OPENAI_API_KEY`。
- 可選用模型清單包括：
    - `gpt-4o-mini-transcribe`
    - `gpt-4o-transcribe`
    - `whisper-1`
- 您亦可傳入自訂的代理人配置，以實現高度客製化的語音理解行為。
- 轉錄品質取決於原始音訊清晰度與所選取的辨識模型。

## 常見問題與排解 {: #common-issues }

- **API 金鑰遺失**：請確保已正確在環境變數或憑據面板中配置了 OpenAI API 金鑰。
- **未提供音訊來源**：請確認已在 "audio" 輸入端連入合法的音訊產物。
- **轉錄品質欠佳**：嘗試切換至更高階的模型（如 `whisper-1` 或 `gpt-4o-transcribe`），或在上游改善音訊輸入品質。
- **處理逾時或異常**：過長的超大音訊檔案或極度微弱吵雜的錄音可能會導致處理時間增加或轉錄精確度下降。
