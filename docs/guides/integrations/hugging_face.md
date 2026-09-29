# 使用 Hugging Face 節點的前置設定 {: #setup-for-nodes-that-use-hugging-face }

## 帳戶註冊與存取 Token 建立 {: #account-and-token-creation }

本指南將引導您建立 Hugging Face 帳戶、生成存取權杖 (Access Token)，並下載與安裝在 Griptape Nodes 中所需的各項模型權重。

!!! info "說明"

    如果您已經擁有 Hugging Face 帳戶，請直接跳至[步驟 2：建立存取 Token](#2-create-an-access-token)。

### 1. 註冊 Hugging Face 帳戶 {: #1-create-a-new-account-on-hugging-face }

1. 前往 [https://huggingface.co/](https://huggingface.co/)
1. 點擊右上角的 **Sign Up** 按鈕
1. 完成機器人驗證程序

<p align="center">
    <img src="../assets/huggingface_00_MainPage.png" alt="HF Site" width="500"/>
</p>

<p align="center">
    <img src="../assets/huggingface_01_signup.png" alt="Signup" width="300"/>
</p>

### 2. 建立存取 Token {: #2-create-an-access-token }

1. 存取您的帳戶設定
1. 登入您的 Hugging Face 帳戶
1. 點擊右上角的個人頭像圖示
1. 在下拉選單中選擇 **Settings**（或直接前往 [Settings](https://huggingface.co/settings/profile/)）

    <p align="center">
    <img src="../assets/huggingface_02_Settings.png" alt="Settings" width="500"/>
    </p>

!!! warning "必須完成電子郵件驗證"

    若在建立 Token 時遇到問題，請確保您已完成電子郵件地址的驗證。請先完成驗證手續再繼續。

1. 在設定功能表中導覽至 **Access Tokens**

<p align="center">
  <img src="../assets/huggingface_03_AccessTokens.png" alt="Access Tokens" width="500"/>
</p>

1. 點擊右上角的 **Create new token** 按鈕
1. 權限類型選擇 **Read (讀取)**

<p align="center">
  <img src="../assets/huggingface_04_TokenRead.png" alt="Token Read" width="500"/>
</p>

1. 為您的 Token 設定一個易讀的描述性名稱（例如：GriptapeNodes）
1. 點擊 **Create Token**。系統將彈出視窗並顯示您的新 Token。請特別注意：這是您唯一能夠檢視或複製此金鑰的機會。

<p align="center">
  <img src="../assets/huggingface_05_SaveToken.png" alt="Save Token" width="400"/>
</p>

1. 複製並安全保管您的 Token
1. 點擊 **Done** 關閉視窗。

!!! danger "安全須知"

    強烈建議將此 Token 保存在密碼管理工具或加密筆記軟體中，確保日後隨時查取且維持高度安全性。

    您的存取 Token 是存取服務時代表您個人身分的憑證。切勿與任何人分享，並避免在螢幕截圖、視訊錄製或螢幕共用畫面中洩漏。

    請將其視同信用卡卡號一樣嚴密保管。

## 安裝必要檔案與 CLI 工具 {: #install-required-files }

取得帳戶 Token 後，您可以安裝 Hugging Face CLI (命令列介面)，以便在終端機中與 Hugging Face 進行互動。

### 1. 安裝 Hugging Face CLI {: #1-install-the-hugging-face-cli }

開啟終端機並執行：

```bash
pip install -U "huggingface_hub[cli]"
```

如需更多資訊，請造訪[官方 CLI 說明文件](https://huggingface.co/docs/huggingface_hub/main/en/guides/cli)。

### 2. 使用 Token 進行身分驗證登入 {: #2-login-with-your-token }

在終端機中，使用剛才建立的存取 Token 進行登入認證：

```bash
huggingface-cli login
```

系統會提示您貼入 Token。

### 3. 下載安裝特定節點所需模型 {: #3-install-models-as-required }

每個 Hugging Face 節點均針對特定模型進行設計。部分節點僅支援單一模型，另一些節點則相容於多種選擇。下表彙整了各節點相容的模型清單：

| 節點名稱 | 相容模型 |
| ----------------- | -------------------------- |
| SPAN Upscale | 4x-ClearRealityV1.pth |
| Flux | FLUX.1-dev, FLUX.1-schnell |
| Flux Post Upscale | FLUX.1-dev, FLUX.1-schnell |

## 模型安裝考量策略 {: #model-installation-considerations }

模型安裝主要有兩種策略，取決於您的工作流程需求、可用磁碟儲存空間以及網路頻寬：

1. **按需選用安裝 (Selective Installation)**：僅下載您計劃使用的特定節點所需之模型。
    - **優點**：大幅節省下載時間與本機硬碟空間。
    - **缺點**：在下載相應模型前，其餘相關節點無法立即使用。

1. **全量完整安裝 (Complete Installation)**：一次性下載所有相容模型。
    - **優點**：無需反覆等待下載，開箱即可完整調用所有節點功能。
    - **缺點**：初次下載耗時較長，且佔用顯著的硬碟容量。

!!! note "下載耗時提示"

    這些模型檔案容量龐大，依據您的網路連線速度，下載過程可能需要 30 分鐘到數小時不等。在背景下載期間，您可以繼續進行最後的設定步驟。

!!! info "下載存放位置"

    模型檔案將被儲存在您的 Hugging Face Hub 快取目錄 (HF_HUB_CACHE) 中。若要查詢具體路徑，可透過 huggingface-cli 檢視：

    在終端機中執行以下命令：

    ```bash
    huggingface-cli env
    ```

    在輸出列表中找到以 `- HF_HUB_CACHE:` 開頭的該列：

    ```
    - HF_HUB_CACHE: /Users/jason/.cache/huggingface/hub
    ```

#### 下載 SPAN Upscale 模型 {: #for-span-upscale }

```bash
huggingface-cli download skbhadra/ClearRealityV1 4x-ClearRealityV1.pth
```

#### 下載 Flux 與 Flux Post Upscale 模型 (FLUX.1-schnell) {: #for-flux-and-flux-post-upscale }

```bash
huggingface-cli download black-forest-labs/FLUX.1-schnell
```

#### 下載 FLUX.1-dev 模型 {: #for-flux1-dev }

```bash
huggingface-cli download black-forest-labs/FLUX.1-dev
```

!!! warning "下載錯誤排解"

    在下載過程中，若遇到類似以下的錯誤訊息：

    ```
    Cannot access gated repo for url https://huggingface.co...
    ```

    這表示該模型屬於門禁管制模型 (Gated model)。錯誤日誌末端會附帶申請存取權限的網址連結。

    請造訪 [https://huggingface.co/black-forest-labs/FLUX.1-schnell](https://huggingface.co/black-forest-labs/FLUX.1-schnell) 提交存取授權申請。通過審核後，系統會提示您已取得存取許可，此時重新執行下載命令即可順利完成。

    <p align="center">
      <img src="../assets/huggingface_06_gated_model.png" alt="門禁管制模型" width="350"/>
    </p>

## 將 Token 配置至 Griptape Nodes 設定中 {: #add-your-token-to-griptape-nodes-settings }

!!! info "概覽"

    完成帳戶建立與模型下載後，接下來只需在 Griptape Nodes 中填入您的 Token。設定過程非常直截了當。

### 1. 開啟 Griptape Nodes 設定功能表 {: #1-open-the-griptape-nodes-settings-menu }

1. 啟動 Griptape Nodes
1. 找到頂部功能表列中的 **Settings (設定)** 功能表（位於 File 與 Edit 右側）
1. 點擊 **Settings** 展開組態選項

<p align="center">
  <img src="../assets/huggingface_07_GN_Settings.png" alt="Settings 功能表" width="500"/>
</p>

### 2. 在 API Keys & Secrets 中填入 Hugging Face Token {: #2-add-your-hugging-face-token-in-api-keys-secrets }

1. 在組態編輯器左側導覽面板中找到 **API Keys and Secrets**
1. 點擊展開該區塊
1. 向下滾動找到 **HUGGINGFACE_HUB_ACCESS_TOKEN** 欄位
1. 將剛才建立的 Token 貼入此欄位中
1. 關閉組態編輯器視窗，系統將自動儲存設定

<p align="center">
  <img src="../assets/huggingface_08_GN_HFToken.png" alt="Token 組態配置" width="500"/>
</p>

!!! success "設定就緒"

    完成上述步驟後，您便可正式在 Griptape Nodes 的各類工作流程中盡情調用 Hugging Face 節點！
