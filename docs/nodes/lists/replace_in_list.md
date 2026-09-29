# Replace In List {: #replace-in-list }

Replace In List 節點讓您能透過元素值比對或指定索引位置，替換清單中的特定項目。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (list)：欲進行修改的目標來源清單
- **Item To Replace** (any)：欲在清單中被替換的目標元素（當 "Replace By" 設為 "item" 時顯示）
- **Index To Replace** (int)：欲被替換項目的整數索引位置（當 "Replace By" 設為 "index" 時顯示）
- **New Item** (any)：用以替換的新元素實體

## 節點屬性 (Properties) {: #properties }

- **Replace By** (str)：識別欲替換項目的方式
    - 選項：
        - "item"：依據元素實體數值比對進行替換
        - "index"：依據索引位置進行替換

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：已完成項目替換的修訂清單

## 運作範例 {: #example }

```python
# 輸入清單: [1, 2, 3, 4]

# 依元素替換 (Replace by item):
# Item To Replace: 3
# New Item: "three"
# 產出結果: [1, 2, "three", 4]

# 依索引替換 (Replace by index):
# Index To Replace: 1
# New Item: "two"
# 產出結果: [1, "two", 3, 4]
```

## 注意事項 {: #notes }

- 當採用元素比對 ("item") 模式時，僅會替換該元素在清單中**首次出現**的項目。
- 當採用索引 ("index") 模式時，所提供的索引值必須在清單的有效邊界範圍內。
- 若採用元素模式且在清單中未找到相符項目，節點將不作任何修改直接輸出原清單。
- 原始清單不會被就地竄改，節點始終輸出全新建立的獨立清單實體。
