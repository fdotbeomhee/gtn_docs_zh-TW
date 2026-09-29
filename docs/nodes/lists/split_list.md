# Split List {: #split-list }

Split List 節點允許您透過指定索引位置或比對特定元素值，將單一清單分割為兩個子清單。

## 輸入埠 (Inputs) {: #inputs }

- **Items** (list)：欲進行分割的目標來源清單
- **Split Index** (int)：用以分割清單的整數索引位置（當 "Split By" 設定為 "index" 時顯示）
- **Split Item** (any)：用以分割清單的基準元素實體（當 "Split By" 設定為 "item" 時顯示）

## 節點屬性 (Properties) {: #properties }

- **Split By** (str)：分割清單的判斷基準
    - 選項：
        - "index"：於指定的索引位置進行分割
        - "item"：於特定的元素值處進行分割
- **Keep Split Item** (bool)：是否在第二個子清單中保留該分割元素（當 "Split By" 設定為 "item" 時顯示）

## 輸出埠 (Outputs) {: #outputs }

- **Output A** (list)：分割後的前半段子清單
- **Output B** (list)：分割後的後半段子清單

## 運作範例 {: #example }

```python
# 輸入清單: [1, 2, 3, 4]

# 依索引分割 (Split by index):
# Split Index: 2
# Output A: [1, 2]
# Output B: [3, 4]

# 依元素分割並保留該元素 (Keep Split Item: True):
# Split Item: 3
# Keep Split Item: True
# Output A: [1, 2]
# Output B: [3, 4]

# 依元素分割且剔除該元素 (Keep Split Item: False):
# Split Item: 3
# Keep Split Item: False
# Output A: [1, 2]
# Output B: [4]
```

## 注意事項 {: #notes }

- 當依索引 ("index") 模式分割時，該索引位置上的元素會被歸入第二個子清單 (Output B) 中。
- 當依元素 ("item") 模式分割時，您可以自由選擇是否將該分割基準元素保留在第二個子清單中。
- 若依元素模式分割且在清單中未找到相符項目，則不會進行任何分割（Output A 包含完整原清單，Output B 為空）。
- 依索引分割時，所傳入的索引值必須在清單的有效邊界範圍內。
- 原始清單不會被就地竄改，節點始終輸出全新建立的獨立子清單。
