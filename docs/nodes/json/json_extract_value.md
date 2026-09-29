# JSON Extract Value {: #json-extract-value }

使用 [JMESPath](https://jmespath.org) 表達式自 JSON 資料中精準提取特定鍵值。

## 功能說明 {: #description }

JSON Extract Value 節點讓您能利用宣告式的 JMESPath 表達式自 JSON 結構中提取特定欄位。它支援多層巢狀物件存取、陣列索引定位以及強大的萬用字元過濾語法，讓您能輕而易舉地在複雜 JSON 結構中檢索數據。

## 參數規格 {: #parameters }

### 輸入參數 (Input Parameters) {: #input-parameters }

| 參數名稱 | 資料型別 | 描述說明 | 預設值 |
| :--- | :--- | :--- | :--- |
| `json` | json, str, dict | 欲提取數據的來源 JSON 資料 | `{}` |
| `path` | str | 用於提取目標資料的 JMESPath 表達式（例如 `'user.name'`、`'items[0].title'`、`'[*].assignee'`） | `""` |

### 輸出參數 (Output Parameters) {: #output-parameters }

| 參數名稱 | 資料型別 | 描述說明 |
| :--- | :--- | :--- |
| `output` | json | 提取出的目標數值物件 |

## 核心特性 {: #features }

- **支援 JMESPath 表達式**：具備強大且具彈性的宣告式資料查詢能力
- **點記號法 (Dot Notation)**：使用直觀的 `.` 符號遍歷多層物件結構
- **陣列索引定位**：使用 `[index]` 語法精確存取特定陣列元素
- **萬用字元投影**：使用 `[*]` 語法自陣列中批次提取所有物件的同名屬性
- **原生 Python 型別回傳**：直接回傳原生資料型別（字串、字典、清單、數值等），無需繁瑣的二次序列化反序列化
- **即時響應**：當輸入資料變更時自動同步重新運算
- **安全防護**：若查詢路徑不存在，安全備援回傳空字典 `{}` 而非拋出例外

## JMESPath 查詢語法速查 {: #jmespath-syntax }

> **📚 延伸閱讀**：欲深入了解完整的進階語法規範，請參閱 [JMESPath 官方教學文件](https://jmespath.org/tutorial.html)。

### 基礎物件存取 (Basic Object Access) {: #basic-object-access }

```python
# 存取巢狀物件屬性
path = "user.name"  # 取得使用者的姓名
path = "user.profile.email"  # 取得深層巢狀的 Email
```

### 陣列索引存取 (Array Indexing) {: #array-indexing }

```python
# 存取陣列元素
path = "items[0]"  # 取得陣列中的第一個元素
path = "items[0].title"  # 取得第一個項目的 title 屬性
path = "users[2].name"  # 取得第三個使用者的姓名
```

### 萬用字元批次提取 (Wildcard Operations) {: #wildcard-operations }

```python
# 自陣列中批次提取特定屬性值
path = "items[*].title"  # 取得 items 陣列中所有物件的 title
path = "users[*].name"  # 取得 users 陣列中所有使用者的姓名
path = "[*].assignee"  # 自根層級陣列中提取所有 assignee 屬性
```

### 複合進階路徑 (Complex Paths) {: #complex-paths }

```python
# 混合搭配物件與陣列索引
path = "orders[0].items[1].price"  # 取得第一筆訂單中第二個項目的價格
path = "projects[*].tasks[*].assignee"  # 取得所有專案中所有任務的指派人
```

## 實用範例展示 {: #examples }

### 基礎物件提取 {: #basic-object-extraction }

```python
# 輸入 JSON: {"user": {"name": "John", "age": 30}}
# Path: "user.name"
# 輸出: "John"  # 回傳原生 Python 字串
```

### 陣列元素提取 {: #array-element-extraction }

```python
# 輸入 JSON: {"items": [{"title": "Book", "price": 25}, {"title": "Magazine", "price": 10}]}
# Path: "items[0].title"
# 輸出: "Book"  # 回傳原生 Python 字串
```

### 巢狀陣列提取 {: #nested-array-access }

```python
# 輸入 JSON: {"orders": [{"items": [{"name": "Product A"}, {"name": "Product B"}]}]}
# Path: "orders[0].items[1].name"
# 輸出: "Product B"  # 回傳原生 Python 字串
```

### 路徑不存在時的回退表現 {: #non-existent-path }

```python
# 輸入 JSON: {"user": {"name": "John"}}
# Path: "user.email"
# 輸出: {}  # 路徑不存在時回傳空字典
```

### 多樣化回傳型別 {: #different-return-types }

節點直接回傳 JMESPath 解析後的原生 Python 實體：

```python
# 字串型別提取
# 輸入 JSON: {"product": {"name": "Widget", "category": "Electronics"}}
# Path: "product.name"
# 輸出: "Widget"  # Python str

# 物件型別提取
# 輸入 JSON: {"user": {"name": "John", "age": 30}}
# Path: "user"
# 輸出: {"name": "John", "age": 30}  # Python dict

# 陣列型別提取
# 輸入 JSON: {"items": [{"title": "Book"}, {"title": "Magazine"}]}
# Path: "items"
# 輸出: [{"title": "Book"}, {"title": "Magazine"}]  # Python list
```

## 適用情境 {: #use-cases }

- **API 回應過濾**：自外部複雜的 API JSON Payload 中精準剔除無用欄位
- **資料縮減過濾**：在大數據物件中精確提取感興趣的業務欄位
- **組態設定讀取**：提取多層結構化設定檔中的特定參數
- **資料轉換管道**：為下游專業節點裁切準備最適格式的資料

## 相關節點 {: #related-nodes }

- [JSON Input](json_input.md) — 自多個引數建立 JSON 資料
- [JSON Replace](json_replace.md) — 替換或更新 JSON 中的特定數值
- [Display JSON](display_json.md) — 格式化並視覺化展示 JSON 資料
- [To JSON](../convert/to_json.md) — 將其他資料型別轉換為標準 JSON
