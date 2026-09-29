# Create Int List {: #create-int-list }

Create Int List 節點讓您能建立包含整數 (Integer) 數值的清單陣列。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (int[])：欲收納納入輸出清單的整數數值清單

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：建立完成的整數清單

## 運作範例 {: #example }

```python
# Items: [1, 2, 3]
# Output: [1, 2, 3]

# Items: [-5, 0, 42]
# Output: [-5, 0, 42]
```

## 注意事項 {: #notes }

- 輸入清單中的所有項目皆應為整數型別。
- 元素在輸出清單中的排列順序會被完整保留。
- 此節點的 `items` 參數同時支援連線輸入與節點面板屬性 (Property) 直接輸入模式。
