# Evaluate Text Result {: #evaluate-text-result }

Evaluate Text Result 節點允許您利用 Griptape 的評估引擎 (Eval Engine)，依據特定評估準則對文字輸出進行精確量化評估。此節點非常適合用於驗證 AI 生成的文字內容、檢查事實準確性，或評定文字產出品質。

## 輸入與屬性 (Inputs) {: #inputs }

- **Examples** (屬性 Property)：從預設範例中挑選評估範本或建立自訂評估
    - 選項：
        - Choose a preset.. (選擇預設範本..)
        - Paraphrase (改寫評估)
        - Factual (事實準確性評估)
        - Analogy (類比評估)

- **Input** (輸入埠/屬性 Input/Property)：作為評估基準的原始輸入文字
    - 支援多行文字輸入

- **Expected Output** (輸入埠/屬性 Input/Property)：預期或作為黃金標準的參考輸出文字
    - 單行文字輸入

- **Actual Output** (輸入埠/屬性 Input/Property)：待評估的實際模型輸出文字
    - 單行文字輸入

- **Criteria** (輸入埠/屬性 Input/Property)：用於評定分數的具體評估準則 (Criteria)
    - 支援多行文字輸入
    - 範例："Does the output accurately paraphrase the input without losing meaning?"（輸出是否準確改寫了輸入且未遺失核心語意？）

## 輸出埠 (Outputs) {: #outputs }

- **Score** (輸出埠 Output)：介於 `0` 到 `1` 之間的浮點數數值，代表評估得分
    - `1.0` 代表完美吻合
    - `0.0` 代表完全不符合

- **Reason** (輸出埠 Output)：評估結果的詳細推理說明文字
    - 提供給予該分數的具體反饋依據
    - 詳細解釋發現的任何歧異或缺陷

## 使用範例展示 {: #example-usage }

### 語意改寫評估 (Paraphrase Evaluation) {: #paraphrase-evaluation }

```python
Input: "The quick brown fox jumps over the lazy dog."
Expected Output: "A swift brown fox leaps above a sleeping dog."
Actual Output: "A fast fox jumps over a dog that's not awake."
Criteria: "Does the output accurately paraphrase the input without losing meaning?"
```

### 事實準確性評估 (Factual Evaluation) {: #factual-evaluation }

```python
Input: "The capital of France is Paris."
Expected Output: "Paris is the capital city of France."
Actual Output: "France's capital is Paris."
Criteria: "Is the output factually correct based on the input?"
```

### 類比邏輯評估 (Analogy Evaluation) {: #analogy-evaluation }

```python
Input: "A bird is to sky as a fish is to ______."
Expected Output: "water"
Actual Output: "concrete"
Criteria: "Does the output correctly complete the analogy?"
```

## 注意事項 {: #notes }

- 本節點採用 Griptape 原生 Eval Engine 執行高精度評估。
- 評估過程完全基於您所提供的評估準則 (Criteria) 進行推理判定。
- 評估分數統一正規化為 `0.0` 至 `1.0` 之間的數值。
- Reason 引腳提供條理清晰的分析反饋，便於診斷提示詞效果。
- 您可直接套用現成範本或靈活撰寫高度自訂的評估題目。
