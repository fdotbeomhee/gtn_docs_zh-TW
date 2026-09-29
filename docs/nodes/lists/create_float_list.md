# Create Float List {: #create-float-list }

Create Float List 節點讓您能建立包含浮點數 (Float) 數值的清單陣列。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (float[])：欲收納納入輸出清單的浮點數值清單

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：建立完成的浮點數清單

## 運作範例 {: #example }

```python
# Items: [1.5, 2.7, 3.14]
# Output: [1.5, 2.7, 3.14]

# Items: [-0.5, 0.0, 42.0]
# Output: [-0.5, 0.0, 42.0]
```

## 注意事項 {: #notes }

- 輸入清單中的所有項目皆應為浮點數型別。
- 元素在輸出清單中的排列順序會被完整保留。
- 此節點的 `items` 參數同時支援連線輸入與節點面板屬性 (Property) 直接輸入模式。
