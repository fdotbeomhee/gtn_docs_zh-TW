# Create Bool List {: #create-bool-list }

Create Bool List 節點讓您能建立包含布林值 (`True` / `False`) 的清單陣列。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (bool[])：欲收納納入輸出清單的布林值陣列清單

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：建立完成的布林值清單

## 運作範例 {: #example }

```python
# Items: [True, False, True]
# Output: [True, False, True]

# Items: [False, True]
# Output: [False, True]
```

## 注意事項 {: #notes }

- 輸入清單中的所有項目皆應為標準布林值。
- 元素在輸出清單中的排列順序會被完整保留。
- 此節點的 `items` 參數同時支援常規引腳連線輸入與節點面板屬性 (Property) 直接勾選編輯模式。
