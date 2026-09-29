# Nuke End Flow {: #nuke-end-flow }

**針對 Nuke 工作流程的終點節點；回報流程是否成功執行。**

類別目錄：`Foundry Nuke`

## 核心速覽 {: #tldr }

- 放置於任何預計發布為 `.gizmo` 工具組的工作流程之最頂層流程終點。
- 提供 `was_successful` 與 `result_details` 輸出，讓發布後的 Gizmo 能在 Nuke 內部直接回報執行狀態與詳細結果。
- 必須與流程另一端的 [Nuke Start Flow](nuke_start_flow.md) 配對使用。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Nuke Start Flow → (工作流程節點) → [Nuke End Flow]
```

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `exec_in` | control | 是 | 源自流程中最後一個運算節點的控制信號連線。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `was_successful` | `bool` | 流程是否圓滿執行成功。 |
| `result_details` | `str` | 關於流程執行結果的詳細訊息。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **發布為 Gizmo 的必要條件**：Gizmo 發布工具在驗證時，嚴格要求頂層流程必須以 [Nuke Start Flow](nuke_start_flow.md) 為起點，並以 Nuke End Flow 為終點；使用一般的普通 End Flow 將無法通過發布驗證。

## 相關延伸閱讀 {: #see-also }

- [Nuke Start Flow](nuke_start_flow.md) —— 與本節點配對的流程入口節點。
- [Nuke Script](nuke_script.md) —— 在流程中直接執行 `.nk` 腳本。
