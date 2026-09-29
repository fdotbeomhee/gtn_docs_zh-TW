# Nuke Start Flow {: #nuke-start-flow }

**針對 Nuke 工作流程的專屬入口節點——將工作流程發布為 Nuke Gizmo 工具組的必備前提。**

類別目錄：`Foundry Nuke`

## 核心速覽 {: #tldr }

- 放置於任何預計發布為 `.gizmo` 工具組的工作流程之最頂層流程起點。
- 運作行為與標準 [Start Flow](../../nodes/execution/start_flow.md) 節點相同；Nuke 專屬變體能向 Gizmo 發布工具標記此工作流程專門以 Nuke 為目標環境。
- 必須與流程另一端的 [Nuke End Flow](nuke_end_flow.md) 配對使用。

## 典型工作流程架構 {: #typical-workflow-position }

```text
[Nuke Start Flow] → (工作流程節點) → Nuke End Flow
```

## 輸入端說明 {: #inputs }

無。

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `exec_out` | control | 連接至流程中第一個運算節點的控制信號連線。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **發布為 Gizmo 的必要條件**：Gizmo 發布工具在驗證時，嚴格要求頂層流程必須以 Nuke Start Flow 為起點，並以 [Nuke End Flow](nuke_end_flow.md) 為終點；使用一般的普通 Start Flow 將無法通過發布驗證。

## 相關延伸閱讀 {: #see-also }

- [Nuke End Flow](nuke_end_flow.md) —— 與本節點配對的流程終點節點。
- [Nuke Script](nuke_script.md) —— 在流程中直接執行 `.nk` 腳本。
