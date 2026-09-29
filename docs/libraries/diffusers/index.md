# Diffusers 程式庫 {: #diffusers-library }

運用模組化 🧨 [Diffusers](https://huggingface.co/docs/diffusers/index) 管線，建構高度靈活的多媒體生成工作流程。

Diffusers 程式庫專為渴望深度掌控擴散運算細節的創作者而設計。不同於傳統將整體生成封裝為單一、黑箱固定步驟的作法，本程式庫將擴散生成過程拆解為多個獨立、可自由接駁的各階段節點——讓您能夠即時監控、複用並深度自訂管線中的每一個細節。

!!! warning "實驗性功能——處於積極開發階段"

    API 介面、節點結構、工作流程範本與程式庫架構可能隨時變更，恕不另行通知或提供遷移支援。若您的工作流程高度追求穩定性，建議固定釘選至特定 Git Commit 版本，並在後續升級時留意可能的相容性變更。

- **原始碼存放庫**：[griptape-ai/griptape-nodes-library-diffusers](https://github.com/griptape-ai/griptape-nodes-library-diffusers)
- **環境需求**：具備 GPU 加速能力的環境（**CUDA** 或 **MPS**）
- **節點分類目錄**：節點挑選器中的 `ModularDiffusion/…` 分類

## 安裝指南 {: #installation }

在編輯器中，依序開啟 **管理 (Manage) → 程式庫管理 (Library Management)**，點擊 **新增程式庫 (Add Library)** 並貼上以下存放庫網址：

```text
https://github.com/griptape-ai/griptape-nodes-library-diffusers
```

利用對話框中的 **進階選項 (Advanced Options)** 可指定釘選特定分支 (Branch)、標籤 (Tag) 或 Commit 雜湊值。亦可直接透過 CLI 命令列執行安裝：

```bash
gtn libraries download https://github.com/griptape-ai/griptape-nodes-library-diffusers
```

有關通用的程式庫安裝、版本更新與常見問題排解，請參閱 [程式庫指南](../../guides/libraries.md)。

!!! note "模型本機下載與快取"

    所有模型權重在首次使用時，皆會自動下載並快取至本機 Hugging Face 快取目錄中（Linux/macOS 預設為 `~/.cache/huggingface/hub`，Windows 預設為 `%USERPROFILE%\.cache\huggingface\hub`）。若欲重定向至其他磁碟儲存位置，可在啟動引擎前設定 `HF_HOME` 環境變數。有關 Hugging Face 帳號綁定與 Token 金鑰配置，請參閱 [Hugging Face 模型整合](../../guides/integrations/hugging_face.md)。

## 運作原理 {: #how-it-works }

典型的模組化生成流程如下所示：

1. **單次建構管線**：使用 [Pipeline Builder](pipeline_builder.md) 節點建立擴散管線實例，並在多次生成任務間重複複用。
1. **建立或載入潛在空間張量 (Latents)**：生成噪訊 (Noise)、建立空白張量、將影像/視訊編碼為潛在張量，或直接載入已儲存的張量檔案。
1. **執行擴散去噪**：使用 [Generate Media Latents](generate_media_latents.md) 產生全新潛在張量——可多次級聯以實現多階段細化 (Multi-stage) 或二次擴散重繪 (Rediffusion)。
1. **潛在空間變換**：在各生成階段之間，運用張量運算、遮罩合成或潛在放大 (Upsampling) 對 Latents 進行加工。
1. **解碼為視覺成果**：使用 [Decode Media Latent](decode_media_latent.md) 將最終潛在張量解碼還原為實體影像或視訊畫面。

因為每個環節皆為獨立節點，您可以任意分支、串聯或重新排列各個步驟——實現傳統端到端黑箱節點無法企及的進階玩法，例如多階段畫質細化、ControlNet 堆疊疊加、潛在空間遮罩合成、首尾幀視訊條件引導以及高階潛在放大。

## 支援模型清單 {: #supported-models }

模型可在 Pipeline Builder 節點介面上的 `provider` 下拉選單中自由選取。目前全面支援：

- **Flux** 與 **Flux2**（包含 Flux2-Klein）
- **Stable Diffusion XL**
- **Qwen-Image**（與 Qwen-Edit）
- **Z-Image**
- **LTX**（視訊）
- **LTX-2.x**（文字/影像/視訊生視訊，支援影像與視訊條件控制、IC-LoRA 以及用於輸出線性 HDR 的 HDR IC-LoRA）
- **WAN**（文字生視訊與圖生視訊）

模型直接從 Hugging Face 存放庫以標準 Diffusers 格式載入（不直接載入單一檔案的 `.safetensors` Checkpoint——請填寫 Hugging Face 存放庫 ID）。透過 Builder 還可向同一管線同時掛載多個 **LoRA** 權重。

## 節點索引參考 {: #node-reference }

節點分組嚴格對應編輯器節點挑選器中的目錄分類：

### 管線建構 (Pipeline) {: #pipeline }

- [Modular Diffusion Pipeline Builder](pipeline_builder.md)
- [ControlNet Pipeline](controlnet_pipeline.md)
- [Load LoRA](load_lora.md)
- [LoRA Pipeline](lora_pipeline.md)

### 潛在張量建立 (Create) {: #create }

- [Create Noise Latents](create_noise_latents.md)
- [Create Empty Latents](empty_latents.md)

### 擴散處理 (Processing) {: #processing }

- [Generate Media Latents](generate_media_latents.md)
- [Latent Upsampler](latent_upsampler.md)

### 潛在空間變換 (Transform) {: #transform }

- [Add Latents](add_latents.md)
- [Subtract Latents](subtract_latents.md)
- [Multiply Latents](multiply_latents.md)
- [Latents Composite Mask](latents_composite_mask.md)

### 條件控制引導 (Conditioning) {: #conditioning }

- [Configure ControlNet](configure_controlnet.md)
- [Media Generation Conditioning](media_gen_conditioning.md)

### 編解碼器 (Encode / Decode) {: #encode-decode }

- [Encode Media Latent](encode_media_latent.md)
- [Encode Masked Media Latent](encode_masked_media_latent.md)
- [Decode Media Latent](decode_media_latent.md)
- [Decode HDR Latents](decode_hdr_latents.md)

### 檔案輸出入 (IO) {: #io }

- [Save Latent Tensor](save_latent_tensor.md)

## 即時生成預覽 (Live Previews) {: #live-previews }

開啟即時影像預覽功能可在生成運算過程中，串流傳回各個中間步驟解碼出的低解析度預覽畫面。非常適合用於即時監控長步數渲染進度（但會輕微犧牲部分推論速度）：

1. 在編輯器中開啟 **設定 (Settings) → 程式庫設定 (Library Settings)**。
1. 向下捲動至 **Modular Diffusion Library** 區塊。
1. 開啟 **Enable Image Preview Intermediates** 開關。

## 效能與顯示卡記憶體最佳化 {: #performance-and-memory }

Pipeline Builder 節點會自動將載入的管線快取至記憶體中以供重複使用，僅在管線參數設定變更時才會重新初始化建構。

針對低顯示卡記憶體 (VRAM) 的硬體環境，Builder 提供了 **顯示卡記憶體最佳化策略 (Memory Optimization Strategy)** 選項：

- **Automatic**：由 Griptape 依據選定的模型自動套用合理的預設最佳化策略。
- **Manual**：允許您手動精細掌控每一項微調開關：
    - **量化模式 (Quantization mode)**：`fp8`、`int8` 或 `int4`（透過 `optimum-quanto` / `bitsandbytes` 實現）——大幅壓縮 Transformer 權重體積，但會略微損失部分畫質精細度。
    - **CPU 卸載策略 (CPU offload strategy)**：`Model`（整模組卸載）或 `Sequential`（逐層卸載）——在模型層級閒置時將權重卸載至主記憶體以釋放 VRAM，會略微增加推論延遲。
    - **注意力分塊切片 (Attention slicing)**：將注意力矩陣切分為微小區塊分批計算；顯著降低記憶體佔用且速度損耗極低。
    - **VAE 分塊切片 (VAE slicing)**：以 Batch Size = 1 逐格解碼潛在張量；有助於避免大解析度生成時的 VRAM 瞬間暴增。
    - **Transformer 逐層型別轉換 (Layerwise casting)**：讓 Transformer 主體常駐於低精度，僅在特定層計算時動態向上提升精度。

請依據硬體條件量力啟用——每項最佳化皆是在「推論速度」與「記憶體容量」之間進行權衡。Pipeline Builder 中的每個參數標籤皆附有詳細說明圖示。

!!! warning "與 Advanced Media 程式庫並用之注意事項"

    Diffusers 程式庫與 [Advanced Media 程式庫](../../nodes/advanced_media_library/diffusion_pipelines.md) 共用許多底層上游依賴項（如 `diffusers`、`transformers`、`torch`），且各自維護獨立的記憶體管線快取。若在同一個工作階段中同時載入並使用這兩個程式庫，可能會導致多份大型模型權重同時常駐於 VRAM/RAM 中，引發顯示卡記憶體不足 (OOM) 錯誤。強烈建議在單一工作階段中僅啟用其中一個程式庫，或透過上述記憶體最佳化機制積極降低 VRAM 負擔。

## 內建工作流程範本 {: #workflow-templates-included }

- Text2Image（文生圖標準工作流）
- MultistageText2Image（多階段文生圖細化工作流）
- LoRAText2Image（掛載 LoRA 之文生圖工作流）
- ControlnetText2Image（ControlNet 精確控制文生圖工作流）
- Image2Image（圖生圖風格重繪工作流）
- FirstAndLastFrameImage2Video（首尾雙幀引導視訊生成工作流）
- LTX23-HDR-Text2Video-Upsample-Two-Stage（LTX-2.x HDR 兩階段文字生視訊放大工作流）

## 支援與反饋 {: #support }

若在運作中發現程式錯誤或有全新功能建議，歡迎隨時向社群回報：
[提交 GitHub Issue](https://github.com/griptape-ai/griptape-nodes-library-diffusers/issues)。
