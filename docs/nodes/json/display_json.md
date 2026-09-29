# Display JSON {: #display-json }

具備自動修復與美化排版功能的 JSON 資料展示節點。

## 功能說明 {: #description }

Display JSON 節點接收 JSON 資料並將其排版以便於在畫布中直觀預覽。它底層結合 `json-repair` 技術自動修補破損畸形的 JSON 字串，為工作流程中的資料檢視與除錯提供整潔、高可讀性的排版介面。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `json` | json, str, dict | 欲展示檢視的 JSON 資料 | `{}` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `json` | json | 格式化美化後的標準 JSON 資料（直通傳遞） |

## 核心特性 {: #features }

- **自動 JSON 修復**：整合 `json-repair`，能自動修補缺少引號或標點錯誤的畸形字串
- **支援多元輸入型別**：無縫相容原生字典 (dict)、純文字字串 (str) 與其他資料型別
- **完善的例外處理**：當完全無法解析時具備安全回退機制
- **視覺化呈現最佳化**：結構化縮排排版，大幅提升開發除錯體驗
- **即時反應處理**：輸入端數值變更時自動同步刷新展示面板

## 轉換與展示範例 {: #examples }

### 基礎 JSON 展示 (Basic JSON Display) {: #basic-json-display }

```python
# 輸入: {"name": "John", "age": 30, "active": true}
# 輸出: {"name": "John", "age": 30, "active": true}
```

### 畸形字串修復展示 (Malformed JSON Repair and Display) {: #malformed-json-repair-and-display }

```python
# 輸入: '{"name": "John", age: 30, "city": "New York"}'  # age 缺少雙引號
# 輸出: {"name": "John", "age": 30, "city": "New York"}  # 自動修復並格式化
```

### 複雜巢狀 JSON 結構 (Complex JSON Structure) {: #complex-json-structure }

```python
# 輸入: {"user": {"name": "Alice", "profile": {"email": "alice@example.com", "preferences": {"theme": "dark"}}}}
# 輸出: {"user": {"name": "Alice", "profile": {"email": "alice@example.com", "preferences": {"theme": "dark"}}}}
```

### 字串轉 JSON 展示 (String to JSON Conversion and Display) {: #string-to-json-conversion-and-display }

```python
# 輸入: '{"items": [{"title": "Book", "price": 25}, {"title": "Magazine", "price": 10}]}'
# 輸出: {"items": [{"title": "Book", "price": 25}, {"title": "Magazine", "price": 10}]}
```

## 適用情境 {: #use-cases }

- **除錯排錯**：在工作流程建置期間即時檢視與檢查 JSON 產物
- **資料驗證**：驗證大語言模型產出的結構與內容是否合規
- **API 回應檢視**：檢查外部 REST 服務端點傳回的 Payload 內容
- **組態配置檢視**：確認系統設定與專案 YAML/JSON 檔案解析結果
- **資料流向監控**：即時掌握結構化數據在跨節點傳遞時的動態變化

## 相關節點 {: #related-nodes }

- [JSON Input](json_input.md) — 自多個引數建立 JSON 資料
- [JSON Extract Value](json_extract_value.md) — 自 JSON 物件中提取特定路徑鍵值
- [JSON Replace](json_replace.md) — 替換或更新 JSON 中的特定數值
- [To JSON](../convert/to_json.md) — 將其他資料型別轉換為標準 JSON
