# LM Studio 本機模型供應商 {: #lm-studio-local }

[LM Studio](https://lmstudio.ai) 是一款能在本機電腦下載並運行開源大型語言模型的桌面端應用程式。完全無需 API 金鑰，且任何資料絕不會上傳或離開您的個人電腦。Griptape Nodes 能夠動態探測並自動抓取您在 LM Studio 中載入的所有模型。

## 事前準備 {: #prerequisites }

1. 造訪 [lmstudio.ai](https://lmstudio.ai) 下載並安裝 LM Studio。

1. 在 LM Studio 內部，利用其模型瀏覽器下載至少一款 AI 模型。

1. 在 LM Studio 中啟動 **Local Server (本地伺服器)** — Griptape Nodes 會連線至該伺服器端點，因此每當您想使用 LM Studio 作為模型供應商時，該伺服器必須處於運行狀態。

## 新增服務供應商 {: #add-the-provider }

1. 前往 **Settings → Agent Settings**
1. 點擊 **+ Add Provider**
1. 選擇 **LM Studio (local)**

填寫組態參數：

- **Name** — 保持預設的 `LM Studio` 或輸入自訂標籤名稱。
- **Base URL** — 預設為 `http://localhost:1234/v1`；點擊重新整理按鈕可測試連線狀態。
- 當 LM Studio 伺服器處於運行狀態時，介面會以綠色字體顯示 **Connected — N models available**。
- **Model (optional)** — 選取預設模型，或保持空白於對話時再行挑選。

點擊 **Create Provider**。

## 連線測試 {: #test }

建立完成後，精靈會顯示確認畫面。挑選一個模型，輸入簡短測試訊息，並確認能夠順暢取得回覆，最後點擊 **Done**。

## 選擇模型 {: #choosing-models }

在 LM Studio 中載入的模型會自動出現於：

- [對話智慧體](../index.md)的 **Model** 下拉選單中，歸納於 LM Studio 供應商名稱之下。
- 當 **Agent 節點**的 `provider` 參數指定為 LM Studio 時的 `prompt model` 下拉清單中。

若要新增更多模型，只需在 LM Studio 中下載它們，隨後便會即時出現在 Griptape Nodes 的介面中。

## 相關參考資源 {: #related }

- [AI 模型服務供應商總覽](./index.md)
- [Ollama 本機模型供應商](./ollama.md) — 另一款強大的本機模型執行工具
