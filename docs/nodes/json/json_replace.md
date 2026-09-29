# JSON Replace {: #json-replace }

使用點記號路徑 (Dot Notation) 替換或更新 JSON 中的特定數值。

## 功能說明 {: #description }

JSON Replace 節點允許您利用直觀的點記號語法，定位並更新 JSON 資料中特定路徑的數值。節點內部會建立原始資料的深層複本 (Deep Copy)，確保來源資料不受非預期污染，並且若指定的目標路徑原本不存在，它會自動建立缺失的中間物件或陣列節點。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `json` | json, str, dict | 欲修改調整的來源 JSON 資料 | `{}` |
| `path` | str | 欲替換目標欄位的點記號路徑（例如 `'user.name'`、`'items[0].title'`） | `""` |
| `replacement_value` | json, str, dict | 欲寫入目標路徑的新數值（可為各類型別） | `""` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `output` | json | 完成替換更新後的全新 JSON 資料物件 |

## 核心特性 {: #features }

- **深層複本防護 (Deep Copy)**：建立獨立複本進行寫入，避免副作用影響工作流程上游資料
- **路徑自動擴充**：若路徑中的中繼屬性尚未存在，會自動建立相應層次
- **支援陣列索引**：支援透過 `items[0].name` 精確更新陣列中的特定項目的欄位
- **即時響應**：當任何輸入參數發生異動時自動重新計算
- **安全容錯機制**：面對無效路徑或異常型別時具備優雅處理原則

## 路徑語法規範 {: #path-syntax }

### 巢狀物件屬性替換 {: #basic-object-replacement }

```python
path = "user.name"  # 替換使用者的姓名
path = "user.profile.email"  # 替換深層巢狀的 email
```

### 陣列元素替換 {: #array-element-replacement }

```python
path = "items[0]"  # 替換陣列的第一個元素
path = "items[0].title"  # 替換第一個項目的 title 屬性
path = "users[2].name"  # 替換第三個使用者的姓名
```

### 複合巢狀陣列替換 {: #complex-path-replacement }

```python
path = "orders[0].items[1].price"  # 替換第一筆訂單中第二個項目的價格
```

## 實用範例展示 {: #examples }

### 基礎數值替換 (Basic Value Replacement) {: #basic-value-replacement }

```python
# 原始 JSON: {"user": {"name": "John", "age": 30}}
# Path: "user.name"
# Replacement Value: "Jane"
# 產出: {"user": {"name": "Jane", "age": 30}}
```

### 陣列項目屬性更新 (Array Element Replacement) {: #array-element-replacement_1 }

```python
# 原始 JSON: {"items": [{"title": "Book", "price": 25}, {"title": "Magazine", "price": 10}]}
# Path: "items[0].title"
# Replacement Value: "Novel"
# 產出: {"items": [{"title": "Novel", "price": 25}, {"title": "Magazine", "price": 10}]}
```

### 自動建立缺失的新路徑 (Creating New Paths) {: #creating-new-paths }

```python
# 原始 JSON: {"user": {"name": "John"}}
# Path: "user.email"
# Replacement Value: "john@example.com"
# 產出: {"user": {"name": "John", "email": "john@example.com"}}
```

### 陣列動態擴充 (Extending Arrays) {: #extending-arrays }

```python
# 原始 JSON: {"items": [{"title": "Book"}]}
# Path: "items[2].title"
# Replacement Value: "Magazine"
# 產出: {"items": [{"title": "Book"}, null, {"title": "Magazine"}]}  # 自動以 null 填補間隙
```

## 適用情境 {: #use-cases }

- **資料欄位更新**：在工作流程中動態更新使用者檔案或系統組態
- **API Payload 組裝**：在發送網路請求前動態抽換特定參數值
- **資料轉換與豐富化**：在現有 JSON 結構中追加 AI 模型分析產出的衍生屬性
- **樣板填充**：以動態計算數值覆寫既有 JSON 樣板的佔位欄位

## 相關節點 {: #related-nodes }

- [JSON Input](json_input.md) — 自多個引數建立 JSON 資料
- [JSON Extract Value](json_extract_value.md) — 依路徑提取單一鍵值
- [Display JSON](display_json.md) — 格式化並視覺化展示 JSON 資料
- [To JSON](../convert/to_json.md) — 將其他資料型別轉換為標準 JSON
