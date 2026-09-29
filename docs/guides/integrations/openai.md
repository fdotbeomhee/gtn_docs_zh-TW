# 如何取得並使用 OpenAI API 金鑰 {: #how-to-get-and-use-an-openai-api-key }

雖然 Griptape Nodes 預設已透過您的 Griptape Cloud 帳戶及其內建金鑰提供了存取 OpenAI 模型的便捷管道（在安裝過程中均已自動配置妥當），但您可能希望直接連線至個人專屬的 OpenAI 商業帳戶。這能讓您對 API 用量、模型參數與存取權限享有 100% 的主導控制權。若這符合您的需求，申請 OpenAI API 金鑰是輕而易舉的第一步。

## 建立帳戶與生成 API 金鑰 {: #account-and-api-key-creation }

在取得 OpenAI 帳戶的 API 金鑰之前，您必須先擁有一個 OpenAI 帳戶。請前往 [https://openai.com](https://openai.com)

<p align="center">
    <img src="../assets/openai_00_main_page.png" alt="Open AI" width="500"/>
</p>

!!! info "說明"

    如果您已經擁有 OpenAI 帳戶，請直接跳至[步驟 2：建立 API 金鑰](#2-create-an-api-key)。

### 1. 建立 OpenAI 帳戶 {: #1-create-an-openai-account }

1. 點擊登入按鈕選擇 "Sign up"，或直接造訪 [https://auth.openai.com/create-account](https://auth.openai.com/create-account)
1. 依照指引完成註冊驗證流程

<p align="center">
    <img src="../assets/openai_01_login_api_platform.png" alt="Login>Platform" width="300"/>
</p>

### 2. 建立 API 金鑰 {: #2-create-an-api-key }

1. 若您不在該頁面，請返回 [https://openai.com/](https://openai.com/)，點擊右上角的 **Log In** 按鈕並選擇 **API Platform** 選項
1. 進入 "Dashboard (主控台資訊主頁)" 區域 — 登入後可在頁面頂部附近找到它。

<p align="center">
    <img src="../assets/openai_02_dashboard.png" alt="Dashboard" width="400"/>
</p>

1. 導覽至 **API Keys** 區塊 — 開啟資訊主頁後可在左側導覽列找到。

    <p align="center">
    <img src="../assets/openai_03_api_keys.png" alt="API Keys" width="200"/>
    </p>

    !!! warning "必須完成帳戶驗證"

        若在建立金鑰時遇到問題，請確保您已完成帳戶的手機與電子郵件**驗證**手續。請先完成驗證再繼續。

    進入該頁面後，右上角會顯示如下操作介面：

    <p align="center">
    <img src="../assets/openai_04_create_new_secret_key_button.png" alt="Create New Secret Key Button" width="500"/>
    </p>

1. 點擊 **Create New Secret Key** 按鈕，將彈出建立視窗：

    <p align="center">
    <img src="../assets/openai_05_create_new_secret_key_modal.png" alt="Create New Secret Key Modal" width="400"/>
    </p>

1. 將金鑰權限設定為 "Read-Only"（您亦可依需求自訂權限範圍，此處建議僅為方便快速示範）
1. 點擊 **Create Secret Key**。系統將彈出視窗並顯示您的全新 API 金鑰。請仔細閱讀並理解該畫面提示：這是您唯一一次能夠檢視或複製此金鑰的機會。
1. 複製並安全保管您的 API 金鑰

    !!! danger "安全須知"

        強烈建議將此 Token 保存在密碼管理工具或加密筆記軟體中，確保日後隨時查取且維持高度安全性。

        您的存取 Token 是存取服務時代表您個人身分的憑證。切勿與任何人分享，並避免在螢幕截圖、視訊錄製或螢幕共用畫面中洩漏。

        請將其視同信用卡卡號一樣嚴密保管。

1. 點擊 **Done** 關閉視窗。

## 將密鑰配置至 Griptape Nodes 設定中 {: #add-your-secret-key-to-griptape-nodes-settings }

!!! info "概覽"

    現在您已備齊 OpenAI 帳戶與金鑰，接下來只需在 Griptape Nodes 中填入您的金鑰。設定流程非常直截了當。

### 1. 開啟 Griptape Nodes 設定功能表 {: #1-open-the-griptape-nodes-settings-menu }

1. 啟動 Griptape Nodes
1. 找到頂部功能表列中的 **Settings (設定)** 功能表（位於 File 與 Edit 右側）
1. 點擊 **Settings** 展開組態選項

    <p align="center">
    <img src="../assets/gtn_settings_menu.png" alt="Settings Menu" width="500"/>
    </p>

### 2. 在 API Keys & Secrets 中新增 OpenAI 金鑰 {: #2-add-your-openai-secret-key-in-api-keys-secrets }

1. 在組態編輯器中，點擊左側面板的 **API Keys and Secrets**
1. 向下滾動找到 **OPENAI_API_KEY** 欄位
1. 將剛才生成的 API 金鑰貼入此欄位中
1. 關閉組態編輯器視窗，系統將自動儲存設定

<p align="center">
    <img src="../assets/openai_06_gtn_settings.png" alt="Settings 中的 OpenAI Secret Key" width="600"/>
</p>

!!! success "設定就緒"

    完成上述步驟後，您便可正式在各類工作流程中調用綁定於個人 OpenAI 商業帳戶憑證下的所有模型。
