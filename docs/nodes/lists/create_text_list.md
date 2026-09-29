# Create Text List {: #create-text-list }

Create Text List 節點讓您能建立包含純文字字串 (String) 的清單陣列。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (string[])：欲收納納入輸出清單的字串清單

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：建立完成的文字字串清單

## 運作範例 {: #example }

```python
# Items: ["Hello", "World", "!"]
# Output: ["Hello", "World", "!"]

# Items: ["First", "Second", "Third"]
# Output: ["First", "Second", "Third"]
```

## 注意事項 {: #notes }

- 輸入清單中的所有項目皆應為文字字串型別。
- 元素在輸出清單中的排列順序會被完整保留。
- 此節點的 `items` 參數同時支援連線輸入與節點面板屬性 (Property) 直接輸入模式。
