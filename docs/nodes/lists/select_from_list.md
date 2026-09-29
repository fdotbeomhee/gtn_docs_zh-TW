# Select From List {: #select-from-list }

透過下拉式選單介面自字串清單中選取單一項目。

## 節點說明 {: #description }

Select From List 節點允許使用者透過直覺的下拉式選單介面，自字串清單中選取單一項目。當輸入的清單動態變更時，它會自動更新選單中的可選項目，並在可能的情況下保留當前已選定的項目。

## 參數說明 {: #parameters }

### 輸入參數 {: #input-parameters }

| 參數名稱 | 資料型別 | 說明 | 預設值 |
| --------- | ---- | ---------------------------- | ------- |
| `list`    | list | 供使用者選取的來源項目清單 | `[]`    |

### 輸出參數 {: #output-parameters }

| 參數名稱 | 資料型別 | 說明 |
| --------------- | ------ | --------------------------- |
| `selected_item` | string | 當前選定的項目字串 |

## 功能特色 {: #features }

- **動態下拉選單**：自動依據傳入的清單內容填入下拉式選單項目
- **選取狀態保留**：當清單內容更新時，若先前選取的項目依然存在於新清單中，將維持當前選取狀態
- **字串標準化轉換**：將清單中所有元素自動轉為字串型別，以確保比對的一致性
- **即時更新機制**：上游輸入清單一經變更，立刻即時反映更新
- **安全容錯處理**：優雅應對空清單或無效輸入資料

## 範例展示 {: #examples }

### 基礎清單選取 {: #basic-list-selection }

```python
# 輸入清單: ["Apple", "Banana", "Cherry", "Date"]
# 使用者選取: "Banana"
# 輸出結果: "Banana"
```

### 狀態標籤選取 {: #status-selection }

```python
# 輸入清單: ["In Progress", "Not Started", "Waiting", "Completed", "Blocked"]
# 使用者選取: "Completed"
# 輸出結果: "Completed"
```

### 動態清單更新行為 {: #dynamic-list-updates }

```python
# 初始清單: ["Option A", "Option B", "Option C"]
# 使用者選取: "Option B"
# 清單變更為: ["Option A", "Option B", "Option D", "Option E"]
# 選取狀態保留: "Option B"（依然存在於新清單中）

# 若清單變更為: ["Option X", "Option Y", "Option Z"]
# 選取狀態重設: "Option X"（自動回退選取新清單的第一項）
```

## 應用情境 {: #use-cases }

- **狀態流轉選取**：自預設的狀態集合中進行挑選（例如 "In Progress", "Completed"）
- **分類標籤選取**：自類別或型別清單中挑選目標
- **組態設定切換**：自可用的組態設定選項中指定配置
- **互動式使用者介面**：在工作流程中提供直覺的下拉式表單選項
- **資料過濾切換**：自生成的動態清單中挑選特定項目以供後續流轉

## 工作流程範例 {: #workflow-examples }

### 狀態管理工作流程 {: #status-management-workflow }

```
Create Text List → Select From List → Process Selected Status
     ↓                    ↓
["In Progress",      "Completed"    → 更新任務狀態
 "Not Started", 
 "Waiting", 
 "Completed", 
 "Blocked"]
```

### 動態選項選取工作流程 {: #dynamic-option-selection }

```
API Response → Extract Options → Select From List → Use Selection
     ↓              ↓                ↓
JSON data    → ["Option 1",    → "Option 2"    → 處理所選項目
              "Option 2", 
              "Option 3"]
```

## 運作行為機制 {: #behavior }

### 選取判定邏輯 {: #selection-logic }

1. **初始載入 (Initial Load)**：預設選取清單中的第一個項目
1. **清單更新 (List Update)**：
    - 若當前選取的項目仍存在於新清單中 → 保持現有選取狀態
    - 若當前選取的項目不在新清單中 → 自動回退選取新清單的第一個項目
1. **空清單 (Empty List)**：清除選取狀態（輸出空字串 `""`）
1. **無效輸入 (Invalid Input)**：清除選取狀態（輸出空字串 `""`）

### 字串型別轉換 {: #string-conversion }

清單內的所有元素皆會自動轉換為字串型別，以實現一致的比對運算：

- 數值：`123` → `"123"`
- 布林值：`True` → `"True"`
- 物件：`{"key": "value"}` → `"{'key': 'value'}"`

## 相關節點 {: #related-nodes }

- [Create Text List](create_text_list.md) - 建立文字字串清單
- [Create List](create_list.md) - 自多重輸入整合建立清單
- [Get From List](get_from_list.md) - 依索引位置自清單擷取元素
- [Get Index Of Item](get_index_of_item.md) - 尋找特定項目的索引位置
- [Display List](display_list.md) - 展開展示清單內容
