# BoolInput {: #boolinput }

BoolInput 節點允許您在工作流程中建立布林值 (Boolean)。

## 輸入埠 (Inputs) {: #inputs }

- **Value** (bool)：欲建立的布林值
    - 預設值：`False`

## 輸出埠 (Outputs) {: #outputs }

- **Output** (bool)：所建立的布林值結果

## 運作範例 {: #example }

```python
# Value: True
# Output: True

# Value: False
# Output: False
```

## 注意事項 {: #notes }

- 此節點的 `value` 參數同時支援連線輸入埠與節點屬性面板 (Property) 兩種數值設定模式。
- 若未特別指定，預設數值為 `False`。
- 數值一經變更，會即時立即轉發輸出至下游節點。
