# 如何取得並使用 xAI Grok API 金鑰 {: #how-to-get-and-use-an-xai-grok-api-key }

Grok 是由 xAI 開發的一系列大型語言模型 (LLM)。您可以透過 GrokPrompt 組態節點存取這些模型。但若要使用這些模型，您必須擁有 xAI 帳戶並生成一組專屬的 API 金鑰。值得特別注意的是，xAI 屬於付費雲端服務，若要正式調用其 API，您必須在其官方網站上預先綁定信用卡帳單付款資訊。

## 建立帳戶與生成 API 金鑰 {: #account-and-api-key-creation }

在取得 xAI 帳戶的 API 金鑰之前，您*必須*先註冊 xAI 帳戶。請前往 [https://x.ai](https://x.ai)

<p align="center">
    <img src="../assets/grok_00_main_page.png" alt="Grok AI" width="500"/>
</p>

!!! info "說明"

    如果您已經擁有 xAI 帳戶，請直接跳至[步驟 2：設定帳單資訊](#2-set-up-billing)。

### 1. 建立 xAI 帳戶 {: #1-create-a-xai-account }

1. 點擊主控台登入選項，或直接前往 [https://accounts.x.ai/sign-up](https://accounts.x.ai/sign-up)

    <p align="center">
    <img src="../assets/grok_01_console_login.png" alt="Console Login" width="500"/>
    </p>

1. 按照畫面指引完成帳戶註冊程序。

    <p align="center">
    <img src="../assets/grok_02_signup.png" alt="Signup" width="500"/>
    </p>

### 2. 設定帳單資訊 {: #2-set-up-billing }

!!! warning "必須綁定帳單"

    在 Griptape Nodes 中使用 xAI 模型前，請務必注意 xAI *要求*必須先完成帳單資訊設定。若未完成此步驟，相關模型將無法在您的節點中正常使用。

    若您在未完成帳單設定的情況下嘗試執行 xAI 節點，工作流程將會報錯失敗，因為該服務會直接拒絕該帳戶憑證的請求。

<p align="center">
    <img src="../assets/grok_03_billing.png" alt="Billing" width="600"/>
</p>

### 3. 生成 API 金鑰 {: #3-generate-an-api-key }

1. 前往 API keys 區塊；主控台中有多個捷徑連結可直達該處。

    <p align="center">
    <img src="../assets/grok_04_key_links.png" alt="API Key links" width="600"/>
    </p>

1. 點擊 "Create API Key" 按鈕

    <p align="center">
    <img src="../assets/grok_05_create_api_key_button.png" alt="Create API Key Button" width="500"/>
    </p>

1. 為您的 API 金鑰命名（建議命名為：GriptapeNodes!）

    <p align="center">
    <img src="../assets/grok_06_create_api_key_page.png" alt="Create API Key Page" width="500"/>
    </p>

1. 點擊 **Save (儲存)**

1. 複製並安全保管您的 API 金鑰

    !!! danger "安全須知"

        強烈建議將此 Token 保存在密碼管理工具或加密筆記軟體中，確保日後隨時查取且維持高度安全性。

        您的存取 Token 是存取服務時代表您個人身分的憑證。切勿與任何人分享，並避免在螢幕截圖、視訊錄製或螢幕共用畫面中洩漏。

        請將其視同信用卡卡號一樣嚴密保管。

## 將 API 金鑰配置至 Griptape Nodes 設定中 {: #add-your-api-key-to-griptape-nodes-settings }

!!! info "概覽"

    現在您已備齊 xAI 帳戶與金鑰，接下來只需在 Griptape Nodes 中進行設定。設定流程非常直覺簡單。

### 1. 開啟 Griptape Nodes 設定功能表 {: #1-open-the-griptape-nodes-settings-menu }

1. 啟動 Griptape Nodes
1. 找到頂部功能表列中的 **Settings (設定)** 功能表（位於 File 與 Edit 右側）
1. 點擊 **Settings** 展開組態選項

    <p align="center">
    <img src="../assets/gtn_settings_menu.png" alt="Settings Menu" width="500"/>
    </p>

### 2. 在 API Keys & Secrets 中新增 xAI API 金鑰 {: #2-add-your-xai-api-key-in-api-keys-secrets }

1. 在組態編輯器 (Configuration Editor) 中，點擊左側面板的 **API Keys and Secrets**
1. 向下滾動找到 **GROK_API_KEY** 欄位
1. 將剛才生成的 API 金鑰貼入此欄位中
1. 關閉組態編輯器視窗，系統將自動儲存設定

<p align="center">
    <img src="../assets/grok_07_gtn_settings.png" alt="Settings 中的 Grok AI API Key" width="700"/>
</p>

!!! success "設定完成"

    完成上述步驟後，您便可在工作流程中順暢調用綁定於個人 Grok.ai 帳戶憑證之下的各款模型。
