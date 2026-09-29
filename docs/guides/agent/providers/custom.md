# 自訂端點 (相容 OpenAI) {: #custom-openai-compatible }

使用此服務供應商類型，可將 Griptape Nodes 連線至任何實作了 OpenAI Chat Completions API 規範的標準端點 — 無論是第三方託管服務、自建模型伺服器還是企業內部的轉發代理 (Proxy)。

## 事前準備 {: #prerequisites }

您需要準備好：

- 該端點的 **Base URL 基礎位址**（例如：`https://api.openai.com/v1`）
- 該端點專用的 **API 金鑰**
- 欲調用的 **模型名稱**（例如：`google/gemma-4-e4b`）— 此項為必填，因為自訂端點通常不支援自動化的模型動態探測清單

## 新增服務供應商 {: #add-the-provider }

1. 前往 **Settings → Agent Settings**
1. 點擊 **+ Add Provider**
1. 選擇 **Custom (OpenAI-compatible)**

填寫組態參數：

- **Name** — 用於在模型下拉選單中識別該供應商的自訂標籤名稱。

- **Icon** — 選取代表該供應商的顯示圖示（選填）。

- **Base URL** — 該端點的根路徑 URL。

- **API Key Secret** — 選取現有的密鑰變數，或點擊 **+** 號就地建立新密鑰：

    - **Secret Name** — 密鑰環境變數名稱（例如：`OPENAI_API_KEY`）
    - **Value** — 實際的金鑰明文字串

    儲存後，從下拉選單中選取該新建立的密鑰項目。

- **Model** — 輸入與該端點搭配使用的具體模型名稱。

點擊 **Create Provider**。

## 連線測試 {: #test }

建立完成後，精靈會展示確認畫面。驗證能夠正常取得模型回覆後，點擊 **Done** 結束設定。

## 相關參考資源 {: #related }

- [AI 模型服務供應商總覽](./index.md)
- [Ollama 本機模型供應商](./ollama.md) — 本機執行方案，無需 API 金鑰
- [LM Studio 本機模型供應商](./lm_studio.md) — 本機執行方案，無需 API 金鑰
