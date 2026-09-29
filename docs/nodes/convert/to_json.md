# To JSON {: #to-json }

使用 `json-repair` 機制將傳入的任意數值轉換為正規標準 JSON 資料。

## 功能說明 {: #description }

To JSON 節點能將各種多樣化資料型別轉換為標準 JSON 格式。其核心採用 `json-repair` 技術，具備對格式破損、缺少引號或標點錯誤的畸形 JSON 字串進行自動容錯修復的高容錯能力。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `from` | any | 欲轉換為標準 JSON 的來源資料 | `{}` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `output` | json | 轉換完成的標準 JSON 資料 |

## 核心特性 {: #features }

- **整合 JSON 自動修復**：底層調用 `json_repair.repair_json()`，能自動修補語法錯誤的畸形 JSON 字串
- **支援多元輸入型別**：具備智慧型別辨識能力，自適應處理字典、清單、數值或字串
- **完善的例外處理**：當完全無法解析時具備優雅降級回退機制
- **高穩健轉換**：確保輸出必定符合標準 JSON 資料規範

## 轉換範例展示 {: #examples }

### 字典轉 JSON (Dictionary to JSON) {: #dictionary-to-json }

```python
# 輸入: {"name": "John", "age": 30, "active": True}
# 輸出: {"name": "John", "age": 30, "active": true}
```

### 畸形破損 JSON 字串自動修復 (Malformed JSON String Repair) {: #malformed-json-string-repair }

```python
# 輸入: '{"name": "John", age: 30, "city": "New York"}'  # age 鍵名缺少引號
# 輸出: {"name": "John", "age": 30, "city": "New York"}  # 自動修復為合法 JSON
```

### 常規 JSON 字串 (Regular JSON String) {: #regular-json-string }

```python
# 輸入: '{"user": {"name": "Alice", "active": true}}'
# 輸出: {"user": {"name": "Alice", "active": true}}
```

### 清單陣列轉 JSON (List to JSON) {: #list-to-json }

```python
# 輸入: [1, 2, 3, "four", {"nested": "value"}]
# 輸出: [1, 2, 3, "four", {"nested": "value"}]
```

### 其他基礎型別 (Other Data Types) {: #other-data-types }

```python
# 輸入: "simple string"
# 輸出: "simple string"

# 輸入: 42
# 輸出: 42

# 輸入: True
# 輸出: true
```

## 輸入型別處理原則 {: #input-type-handling }

### 字典輸入 (Dictionary Input) {: #dictionary-input }

- **行為原則**：若傳入已為字典型別，則直接原樣傳遞
- **範例**：`{"key": "value"}` → `{"key": "value"}`

### 字串輸入 (String Input) {: #string-input }

- **行為原則**：優先嘗試修復畸形語法，隨後進行標準 JSON 解析反序列化
- **範例**：`'{"name": "John", age: 30}'` → `{"name": "John", "age": 30}`

### 其他型別 (Other Types) {: #other-types }

- **行為原則**：先轉為字串表示式，再進行自動修復嘗試；若完全失敗則備援為空字典
- **範例**：`[1, 2, 3]` → `[1, 2, 3]`

## 適用情境 {: #use-cases }

- **資料標準化**：將多元非結構化格式統一收斂為標準 JSON
- **API 介面整合**：為僅接收 JSON 的外部 REST API 準備乾淨酬載
- **髒資料清洗**：修復大語言模型有時輸出的非標準畸形 JSON
- **工作流程串接**：轉換型別以便下游 JSON 專用節點讀取
- **組態設定解析**：處理各種來源的配置設定資料

## 相關節點 {: #related-nodes }

- [JSON Input](../json/json_input.md) — 自多個引數建立 JSON 資料
- [JSON Extract Value](../json/json_extract_value.md) — 自 JSON 物件中提取特定路徑鍵值
- [JSON Replace](../json/json_replace.md) — 替換或更新 JSON 中的特定數值
- [Display JSON](../json/display_json.md) — 格式化並視覺化展示 JSON 資料
