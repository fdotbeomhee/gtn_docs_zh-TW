# Combine Lists {: #combine-lists }

Combine Lists 節點接收兩個獨立清單，並將其順序串接合併為單一扁平化清單。

## 輸入埠 (Inputs) {: #inputs }

- **List A** (list)：欲進行合併的第一個來源清單
- **List B** (list)：欲進行合併的第二個來源清單

## 輸出埠 (Outputs) {: #outputs }

- **Output** (list)：依序包含兩組輸入清單所有項目的合併清單

## 運作範例 {: #example }

```python
# List A: [1, 2]
# List B: [3, 4]
# Output: [1, 2, 3, 4]

# List A: ["a", "b"]
# List B: ["c", "d"]
# Output: ["a", "b", "c", "d"]

# List A: [1, 2]
# List B: []
# Output: [1, 2]
```

## 注意事項 {: #notes }

- 元素在輸出清單中會嚴格保留其原始先後順序。
- 若任一輸入埠傳入的不是合法清單型別，系統會安全地將其視為空清單處理。
- 原始來源清單不會被就地修改，節點一律回傳新建立的合併清單。
- 此運算在底層邏輯上等同於 Python 中的清單串接拼接操作（`list_a + list_b`）。
