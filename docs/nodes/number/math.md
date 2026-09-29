# Math Operations {: #math-operations }

## 功能概述 {: #what-is-it }

Math 節點是一個功能全面的通用數學運算節點，能夠對數值執行各類算術與進階數學運算函式。它同時支援單元運算（Unary Operations，僅需單一輸入）與二元運算（Binary Operations，需要雙輸入）。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 執行基礎算術四則運算（加、減、乘、除）
- 計算進階數學函式（平方根、絕對值、正弦函數）
- 數值捨入與取整（四捨五入、無條件進位、無條件捨去）
- 求解最大值 (Max) 或最小值 (Min)
- 計算數值平均值 (Average)
- 執行取模 / 取餘數運算 (Modulo)
- 計算乘冪次方 (Power)

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 Math 節點新增至工作流程中
1. 自下拉式選單中選取您欲執行的運算規則
1. 將數值輸入連線至參數 A 與 B（B 參數僅在二元運算中需要）
1. 運算結果將自動於 "result" 輸出埠傳回

### 可用運算規則說明 {: #available-operations }

#### 二元運算 (Binary Operations，同時需要 A 與 B 輸入) {: #binary-operations-require-both-a-and-b-inputs }

- **加法 Add**：`A + B`
- **減法 Subtract**：`A - B`
- **乘法 Multiply**：`A * B`
- **除法 Divide**：`A / B`
- **取模 Modulo**：`A % B`
- **乘冪 Power**：`A ^ B`
- **平均值 Average**：`avg(A, B)`
- **最小值 Min**：`min(A, B)`
- **最大值 Max**：`max(A, B)`

#### 單元運算 (Unary Operations，僅需 A 輸入) {: #unary-operations-only-require-a-input }

- **平方根 Square Root**：`√A`
- **四捨五入 Round**：`round(A)`
- **無條件進位 Ceiling**：`⌈A⌉`
- **無條件捨去 Floor**：`⌊A⌋`
- **絕對值 Absolute Value**：`|A|`
- **正弦函數 Sine**：`sin(A)`

### 輸出埠說明 {: #outputs }

- **result**：所選數學運算執行後的數值計算結果

## 範例展示 {: #example }

計算兩個數值的平均值：

1. 將 Math 節點新增至工作流程中
1. 自運算下拉選單中選取 "average [avg(A, B)]"
1. 將兩個數值連線接駁至參數 A 與 B
1. "result" 輸出引腳將輸出兩數的平均值

## 實作細節 {: #implementation-details }

Math 節點會自動進行以下內部處理：

- 輸入數值的自動型別相容轉換
- 除數為零 (Division by Zero) 的安全容錯處理（傳回無窮大 `infinity`）
- 模數為零 (Modulo by Zero) 的安全容錯處理（傳回無窮大 `infinity`）
- 動態參數可見度切換（當選取單元運算時自動隱藏 B 參數引腳）
