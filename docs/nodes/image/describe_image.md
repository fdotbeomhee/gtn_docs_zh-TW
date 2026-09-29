# DescribeImage {: #describeimage }

## 功能概述 {: #what-is-it }

DescribeImage 節點利用人工智慧視覺模型為您傳入的影像生成精準細緻的文字描述。它結合先進的多模態視覺理解模型，依據您指定的提示詞指令深入分析畫面並輸出結構化或自然語言描述。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 自動為影像生成多段落自然語言描述
- 自視覺畫面中萃取結構化資料或關鍵資訊
- 為網頁無障礙設計 (Accessibility) 自動生成 Alt Text 替代文字
- 深度分析照片、圖表、工程圖紙或各類視覺媒體
- 將視覺圖案轉換為結構化文字以供下游 Agent 進行邏輯推理

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 DescribeImage 節點新增至工作流程中
1. 將來源影像連線至 "image" 輸入埠
1. 視需要於 "prompt" 欄位中提供明確具體的分析指引
1. 執行節點以生成對該影像的詳細解讀報告

!!! info "明確具體地陳述您的分析需求"

    撰寫提示詞時，請清楚指出您最關注的畫面元素。例如：

    - **色彩分析需求**：建議使用「請詳細描述畫面整體的調色盤配置、光影色溫、明暗度與色彩飽和度表現」
    - **人物角色細節**：建議使用「請詳細描述人物的性別、推定年齡、服裝穿著、肢體儀態、表情神韻與當前動作」

    提示詞越聚焦明確，AI 模型產出的描述就越符合您的業務預期。

### 參數說明 {: #parameters }

- **agent**：選填的既有 Agent 代理配置，可用於延續對話上下文與規則集
- **model**：選取支援視覺能力的提示詞模型（或連線 Prompt Model Config 驅動配置）。當已連線 **agent** 時此項將被忽略
- **image**：欲進行辨識與解讀的目標影像（必填）
- **prompt**：指導模型如何描述分析該影像的提示詞指令（預設為 "Describe the image"）
- **description_only**：若啟用，將僅傳回純粹的畫面描述內容（不包含對話式客套語）
- **output_schema**：選填的結構化 JSON Schema。接收 JSON schema 物件或 JSON 字串。連線此項後，**output** 輸出埠將切換為結構化 JSON 輸出

### 輸出埠說明 {: #outputs }

- **output**：影像的文字描述內容。預設輸出為純文字字串；若配置了 **output_schema** 則傳回嚴格符合 Schema 的 JSON 物件
- **agent**：執行此次描述分析任務的底層 Agent 物件，可連線至下游其他對話節點

## 範例展示 {: #example }

深度分析風景照片中的自然景觀要素：

1. 將 DescribeImage 節點新增至工作流程中
1. 將上游影像節點的輸出連線至 "image" 輸入埠
1. 於 "prompt" 欄位中輸入："Describe this landscape in detail, including colors, features, and mood"（請詳細描述此處風景的地貌特徵、色彩搭配與整體意境氛圍）
1. 執行節點
1. "output" 引腳將傳回結構清晰、詞藻豐富的景觀文字說明

### 結構化 JSON 輸出實戰 {: #structured-output-example-json }

若您需要以固定的 JSON 資料規格接收分析結果：

1. 新增 `CreateAgentSchema` 節點，並輸入您期望傳回的範例 JSON 結構
1. 將 `CreateAgentSchema.schema` 連線至 `DescribeImage.output_schema`
1. 執行 `DescribeImage` 節點
1. 節點產出的 `output` 將會是嚴格吻合該規格的標準 JSON 資料

## 重要注意事項 {: #important-notes }

- 使用 Griptape Cloud API 預設服務時，環境中需具備合法的 `GT_CLOUD_API_KEY` 金鑰。
- 節點預設調用具備強大多模態能力的 `gpt-5.2` 視覺模型。
- 若未提供 prompt，將自動套用預設的 "Describe the image"。
- 若未傳入影像輸入，輸出將直接傳回 "No image provided" 提示。
- 輸出的精確度高度依賴於傳入影像的清晰度與光照條件。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認系統環境變數中已正確設定 `GT_CLOUD_API_KEY`。
- **未傳入影像 (No Image Provided)**：請檢查 "image" 輸入引腳連線是否正確。
- **描述品質未達預期**：請嘗試細化 prompt 提示詞，指明需要分析的重點面向（例如構圖、光影、色彩或主體動作）。
