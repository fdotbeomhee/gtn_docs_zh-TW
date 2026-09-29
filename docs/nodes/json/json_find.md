# JSON Find {: #json-find }

依據搜尋條件在 JSON 陣列中尋找匹配的項目。

## 功能說明 {: #description }

JSON Find 節點讓您能檢索 JSON 陣列並精確找出符合指定條件的資料項目。它支援多種搜尋模式、大小寫敏感度切換，並可設定傳回首個符合項目或所有匹配項目清單。是在複雜資料結構中進行篩選與搜尋的理想工具。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `json` | json, str, dict | 欲進行檢索的 JSON 資料（應為陣列或包含陣列的物件） | `[]` |
| `search_field` | str | 欲比對搜尋的欄位點記號路徑（例如 `'attributes.content'`） | `""` |
| `search_value` | str | 欲尋找的比對目標數值 | `""` |
| `search_mode` | str | 搜尋模式：`'exact'` (完全相符)、`'contains'` (包含子字串) 或 `'starts_with'` (以此開頭) | `"exact"` |
| `return_mode` | str | 回傳模式：`'first'` 回傳首個匹配項目，`'all'` 回傳所有符合項目清單 | `"first"` |
| `case_sensitive` | bool | 是否區分英文大小寫 | `true` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `found_item` | json | 找到的項目（若 `return_mode` 為 'first' 則為單一物件，若為 'all' 則為陣列） |
| `found_count` | int | 匹配成功的總項目數量 |
| `found_index` | int | 第一個匹配項目的原始索引位置（若未找到則為 `-1`） |

## 核心特性 {: #features }

- **靈活的搜尋模式**：支援完全一致 (Exact)、包含 (Contains) 以及開頭符合 (Starts With)
- **大小寫敏感控制**：自由切換是否嚴格區分英文大小寫
- **多元回傳模式**：可取單一首個結果或傳回完整清單
- **多層巢狀欄位存取**：利用點記號法 (Dot Notation) 深入比對深層欄位
- **陣列自動探測**：當傳入為根物件時，會自動嘗試在常見陣列欄位名稱中搜尋
- **即時響應**：當輸入資料變動時自動即時重新篩選

## 搜尋模式解析 {: #search-modes }

### 完全相符 (`exact`) {: #exact-match-exact }

欄位數值必須與搜尋目標 100% 完全一致。

```python
# 搜尋: "Design"
# 匹配: "Design"
# 不匹配: "Design Task", "design", "My Design"
```

### 包含子字串 (`contains`) {: #contains-contains }

欄位數值只要包含該子字串即視為匹配。

```python
# 搜尋: "Design"
# 匹配: "Design", "Design Task", "My Design Work"
# 不匹配: "design"（若 case_sensitive=true）
```

### 以此開頭 (`starts_with`) {: #starts-with-starts_with }

欄位數值的前綴必須與搜尋文字吻合。

```python
# 搜尋: "Design"
# 匹配: "Design", "Design Task", "Designer"
# 不匹配: "My Design", "design"
```

## 欄位路徑語法 {: #field-path-syntax }

`search_field` 採用與 JSON Extract Value 相同的點記號路徑語法：

```python
search_field = "name"  # 搜尋根層級的 name 欄位
search_field = "attributes.content"  # 搜尋多層巢狀的 content 欄位
search_field = "user.profile.email"  # 搜尋深層巢狀的 email 欄位
```

## 實用範例展示 {: #examples }

### 依內容搜尋任務 (Find Task by Content) {: #find-task-by-content }

```python
# 輸入 JSON: [
#   {"attributes": {"content": "Design", "status": "wtg"}},
#   {"attributes": {"content": "Model", "status": "wtg"}},
#   {"attributes": {"content": "Design Review", "status": "ip"}}
# ]
# search_field: "attributes.content"
# search_value: "Design"
# search_mode: "exact"
# return_mode: "first"
# 產出: {"attributes": {"content": "Design", "status": "wtg"}}
```

### 依狀態篩選所有任務 (Find All Tasks with Status) {: #find-all-tasks-with-status }

```python
# 輸入 JSON 同上
# search_field: "attributes.status"
# search_value: "wtg"
# search_mode: "exact"
# return_mode: "all"
# 產出: [{"attributes": {"content": "Design", "status": "wtg"}},
#        {"attributes": {"content": "Model", "status": "wtg"}}]
```

### 忽略大小寫搜尋 (Case-Insensitive Search) {: #case-insensitive-search }

```python
# 輸入 JSON: [
#   {"name": "Design Task"},
#   {"name": "design review"},
#   {"name": "DESIGN WORK"}
# ]
# search_field: "name"
# search_value: "design"
# search_mode: "contains"
# case_sensitive: false
# return_mode: "all"
# 產出: 全部三個項目皆匹配成功
```

## 陣列自動探測機制 {: #array-auto-detection }

若傳入的根節點為包含陣列的 JSON 物件，節點會自動嘗試尋找以下常見鍵名以進行陣列比對：

- `data`
- `items`
- `results`
- `list`

## 適用情境 {: #use-cases }

- **任務管理**：依內容、狀態或指派人員篩選任務條目
- **大數據過濾**：根據特定指標自巨量資料集提取精準子集
- **API 回應過濾**：在外部 API 傳回的物件清單中定位目標記錄
- **組態項尋找**：自大型配置清單中檢索特定項目的參數值

## 相關節點 {: #related-nodes }

- [JSON Extract Value](json_extract_value.md) — 依路徑提取單一鍵值
- [JSON Input](json_input.md) — 自多個引數建立 JSON 資料
- [JSON Replace](json_replace.md) — 替換或更新 JSON 中的特定數值
- [Display JSON](display_json.md) — 格式化並視覺化展示 JSON 資料
- [To JSON](../convert/to_json.md) — 將其他資料型別轉換為標準 JSON
