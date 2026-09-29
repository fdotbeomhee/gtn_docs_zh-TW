# Random Text {: #random-text }

Random Text 節點允許您自輸入文字中隨機抽樣選取內容，或在未提供輸入時自動隨機生成文字。它支援隨機選取字元 (Character)、單字 (Word)、句子 (Sentence) 或段落 (Paragraph)。

## 參數說明 {: #parameters }

### Input Text (輸入文字) {: #input-text }

- **資料型別**：String (字串)
- **模式**：輸入埠 / 屬性面板 (Input/Property)
- **說明**：供隨機抽樣的來源文字。若為空，節點將根據選定的抽樣型別自動隨機生成內容。
- **UI 選項**：啟用多行文字編輯 (Multiline enabled)

### Seed (隨機種子) {: #seed }

- **資料型別**：Integer (整數)
- **模式**：屬性面板 (Property)
- **說明**：用於確保隨機抽樣結果具備可重現性的種子數值 (0-10,000)。使用相同的種子數值將產生完全一致的隨機抽樣結果。
- **UI 選項**：滑動桿控制器，範圍 0 至 10,000

### Selection Type (抽樣型別) {: #selection-type }

- **資料型別**：String (字串)

- **模式**：屬性面板 (Property)

- **說明**：欲抽樣或生成的文字內容層級：

    - `character`：隨機選取單一字元
    - `word`：隨機選取單一單字
    - `sentence`：隨機選取單一句子，或自動生成新句子
    - `paragraph`：隨機選取單一段落，或自動生成新段落

### Output (輸出) {: #output }

- **資料型別**：String (字串)
- **模式**：輸出埠 (Output)
- **說明**：隨機抽樣或生成產出的文字內容
- **UI 選項**：啟用多行文字展示 (Multiline enabled)

## 運作行為機制 {: #behavior }

- 當提供了輸入文字 (Input Text) 時：

    - 節點會根據設定的抽樣型別，自傳入文字中隨機選取對應的片段
    - 對於句子與段落層級，節點會以適當的標點符號與換行符作為邊界進行切分
    - 若在來源文字中未找到相符結構，節點將自動回退為直接生成全新內容

- 當未提供輸入文字（留空）時：

    - 字元 (`character`)：從字母、數字及標點符號中隨機挑選生成
    - 單字 (`word`)：隨機生成常見英文單字
    - 句子 (`sentence`)：利用內建 AI Agent 生成語法通順自然的句子
    - 段落 (`paragraph`)：利用內建 AI Agent 生成條理連貫的文字段落

## 範例展示 {: #examples }

### 隨機抽樣現有文字內容 {: #selecting-random-content }

```python
# 輸入文字: "Hello world! This is a test. How are you today?"
# 抽樣型別: word
# 輸出結果: "world"（自文字中隨機選出的單字）
```

### 自動隨機生成全新文字內容 {: #generating-random-content }

```python
# 輸入文字: "" (留空)
# 抽樣型別: sentence
# 輸出結果: "The quick brown fox jumps over the lazy dog."（由 AI 生成的自然句子）
```

## 注意事項 {: #notes }

- 設定特定的 `seed` 參數能確保每次工作流程執行時得到確定性、可重現的隨機結果。
- 當由 AI Agent 生成段落或句子時，節點介面上會短暫顯示載入中訊息。
