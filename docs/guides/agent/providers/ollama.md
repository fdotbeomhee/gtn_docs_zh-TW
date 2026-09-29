# Ollama 本機模型供應商 {: #ollama-local }

[Ollama](https://ollama.com) 允許您在個人電腦本機直接運行開源 AI 模型。完全無需 API 金鑰，且任何資料絕不離開您的本機電腦。Griptape Nodes 能夠動態探測並自動列出您本機已安裝的模型。

## 事前準備 {: #prerequisites }

1. 造訪 [ollama.com](https://ollama.com) 下載並安裝 Ollama。

1. 拉取至少一個模型。例如：

    ```bash
    ollama pull llama3.2
    ```

1. 驗證 Ollama 是否正常運行且模型已就緒：

    ```bash
    ollama list
    ```

    每當您需要在 Griptape Nodes 中調用時，Ollama 服務必須保持在背景運行。

## 新增服務供應商 {: #add-the-provider }

1. 前往 **Settings → Agent Settings**
1. 點擊 **+ Add Provider**
1. 選擇 **Ollama (local)**

填寫組態參數：

- **Name** — 保持預設的 `Ollama` 或輸入自訂標籤名稱。
- **Base URL** — 預設為 `http://localhost:11434/v1`；點擊重新整理按鈕可測試連線狀態。
- 當 Ollama 處於運行狀態時，介面會以綠色字體顯示 **Connected — N models available**。
- **Model (optional)** — 選取預設模型，或保持空白於對話時再行挑選。

點擊 **Create Provider**。

## 連線測試 {: #test }

建立完成後，精靈會顯示確認畫面。挑選一個模型，輸入簡短測試訊息，並確認能夠順暢取得回覆，最後點擊 **Done**。

## 選擇模型 {: #choosing-models }

所有透過 `ollama pull` 下載的模型都會自動出現於：

- [對話智慧體](../index.md)的 **Model** 下拉選單中，歸納於 Ollama 供應商名稱之下。
- 當 **Agent 節點**的 `provider` 參數指定為 Ollama 時的 `prompt model` 下拉清單中。

若要新增更多模型，只需在終端機中執行 `ollama pull <model-name>`，下載完成後新模型便會即時出現在介面中。

## 相關參考資源 {: #related }

- [AI 模型服務供應商總覽](./index.md)
- [LM Studio 本機模型供應商](./lm_studio.md) — 另一款強大的本機模型執行工具
- [智慧體本機模型應用 (MCP 方案)](../../mcp/advanced_local_models.md) — 在工作流程中透過 MCP 節點調用 Ollama 的替代進階架構
