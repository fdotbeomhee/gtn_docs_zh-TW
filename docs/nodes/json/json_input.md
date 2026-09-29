# JSON Input {: #json-input }

自輸入資料建立標準 JSON 節點，並具備自動容錯修復能力。

## 功能說明 {: #description }

JSON Input 節點接收多種輸入型別並將其標準化轉換為合規的 JSON 資料物件。它整合了 `json-repair` 庫，能自動修復格式殘缺的 JSON 字串，為不同型別的資料提供強大的轉換防護。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `json` | json, str, dict | 欲轉換為標準 JSON 的輸入資料 | `{}` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `json` | json | 處理完成的正規標準 JSON 資料物件 |

## 核心特性 {: #features }

- **自動 JSON 修復**：整合 `json-repair`，能自動修補格式有缺陷的 JSON 字串
- **支援多元輸入型別**：相容原生字典 (dict)、字串 (str) 與其他資料型別
- **完善的例外處理**：當完全無法解析時具備安全回退機制
- **即時反應處理**：輸入端數值異動時自動同步刷新輸出結果

## 轉換範例展示 {: #examples }

### 基礎用法 (Basic Usage) {: #basic-usage }

```python
# 輸入: {"name": "John", "age": 30}
# 輸出: {"name": "John", "age": 30}
```

### 畸形字串自動修復 (Malformed JSON Repair) {: #malformed-json-repair }

```python
# 輸入: '{"name": "John", age: 30}'  # age 鍵名缺少引號
# 輸出: {"name": "John", "age": 30}  # 自動修復為標準 JSON
```

### 字串轉 JSON (String to JSON Conversion) {: #string-to-json-conversion }

```python
# 輸入: '{"user": {"name": "Alice", "active": true}}'
# 輸出: {"user": {"name": "Alice", "active": true}}
```

### 字典原樣直通 (Dictionary Input) {: #dictionary-input }

```python
# 輸入: {"status": "active", "count": 5}
# 輸出: {"status": "active", "count": 5}  # 原樣保留並傳遞
```

## 適用情境 {: #use-cases }

- **資料驗證**：確保工作流程輸入端為結構合規的標準 JSON
- **API 回應串接**：處理來自外部網路服務的回應資料
- **資料清洗**：修復來源不可靠的破損 JSON
- **工作流程型別轉換**：將非結構化資料收斂轉化為 JSON 物件以供下游處理

## 相關節點 {: #related-nodes }

- [JSON Extract Value](json_extract_value.md) — 依路徑提取單一鍵值
- [JSON Replace](json_replace.md) — 替換或更新 JSON 中的特定數值
- [Display JSON](display_json.md) — 格式化並視覺化展示 JSON 資料
- [To JSON](../convert/to_json.md) — 將其他資料型別轉換為標準 JSON
