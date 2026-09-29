# Diffusion Pipelines {: #diffusion-pipelines }

!!! warning "使用 Hugging Face Diffusion Pipeline 節點前需完成的前置設定"

    [本整合指南](../../guides/integrations/hugging_face.md) 將引導您完成 Hugging Face 帳號註冊、存取權杖 (Access Token) 建立，以及下載必備模型權重以確保本節點順暢運作。

## 功能概述 {: #what-are-they }

Diffusion Pipeline 擴散管線系統由兩個相輔相成的節點組成，共同提供高效能的影像生成架構：

- **Diffusion Pipeline Builder**：建構並快取 🤗 Diffusers 管線，以供多個執行節點重複複用
- **Generate Image (Diffusion Pipeline)**：調用快取的管線執行實際影像生成

這種模組化架構允許您「單次配置管線，多次高效生成」，大幅提升執行效能並節省硬體資源開銷。系統透過高度動態化的參數介面，全面支援豐富的供應商架構與模型。

## 支援供應商架構 {: #supported-providers }

Diffusion Pipeline Builder 支援主流的 AI 擴散模型供應商：

- **Flux**：極致清晰頂級品質的文字生圖模型
- **Qwen**：多模態視覺與語言生成能力
- **Stable Diffusion**：廣受歡迎的開源擴散模型家族（SD 1.5、SD 2.1、SDXL）
- **Allegro**：前沿視訊生成模型
- **Amused**：高效遮罩影像建模架構
- **AudioLDM**：基於文字提示詞生成高保真音訊
- **WAN**：專業特化生成模型
- **Wuerstchen**：極致壓縮效率的兩階段擴散架構
- **Custom**：支援自訂擴散管線配置與載入本機自備模型

## 適用情境 {: #when-would-i-use-it }

在以下情況使用這些節點：

- 依據自然語言提示詞，運用多元模型架構生成高品質影像
- 在創意美術專案中整合先進的生成模型
- 實驗並對比不同供應商與模型架構的生成風格
- 透過管線記憶體快取複用，極大化批次生成的吞吐量
- 探索音訊、視訊或多模態等特定領域生成模型

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

Diffusion Pipeline 系統採用經典的雙節點協同架構：

1. **配置管線建構器 (Builder)**：

    - 將 "Diffusion Pipeline Builder" 節點新增至工作流程中
    - 挑選目標供應商（Flux、Stable Diffusion 等）
    - 配置供應商專屬參數（模型存放庫、LoRA 適配器、顯示卡記憶體最佳化）
    - 執行建構器節點以在記憶體中建立並快取管線實例

1. **生成影像 (Generate)**：

    - 將 "Generate Image (Diffusion Pipeline)" 節點新增至工作流程中
    - 將建構器的 `pipeline` 輸出端連線至該執行節點的管線輸入端
    - 配置執行期生成參數（Prompt 提示詞、寬高解析度、步數等）
    - 執行該節點以渲染生成最終影像

### Pipeline Builder 參數說明 {: #pipeline-builder-parameters }

建構器節點的參數會依據選取的供應商動態變化：

- **provider**：從支援清單中挑選供應商（Flux、Stable Diffusion 等）
- **供應商專屬參數**：模型版本選取、LoRA 權重配置、記憶體最佳化選項
- **pipeline**：封裝了已快取管線設定的輸出連線埠

### 執行期節點 (Runtime) 參數說明 {: #runtime-parameters }

執行期生成節點的參數埠會根據連線的管線型別動態自我調適：

- **pipeline**：接收來自 Builder 節點的管線輸入端
- **動態生成參數**：正負提示詞、像素解析度、推論時間步數、引導係數 (CFG)
- **output_image**：生成的最終影像產物 (ImageArtifact)
- **seed**：用於亂數生成的整數種子
- **logs**：生成運算歷程的詳細時間戳記日誌

!!! note "動態響應參數"

    這兩個節點皆採用動態參數架構。當您挑選不同的供應商或連線不同的管線時，介面上的參數插槽會自動即時重新排版。

### 進階特性功能 {: #advanced-features }

- **管線智慧快取**：建構完成的管線透過設定值雜湊進行記憶體快取，供重複執行時秒級複用
- **LoRA 完整支援**：支援掛載並配置多個 LoRA 適配器以客製化風格
- **全方位記憶體最佳化**：針對有限顯示卡記憶體提供細緻的卸載與量化選項
- **即時去噪預覽**：支援在生成中途串流輸出低解析度預覽圖（會略微增加推論耗時）
- **參數連線自動保留**：更換上游管線時，執行期節點會自動保留同名參數的連線關係

## 手動顯示卡記憶體設定 (Manual Memory Settings) {: #manual-memory-settings }

Diffusion Pipeline Builder 預設將 `memory_optimization_strategy` 設定為 **Manual（手動模式）**。之所以預設採用手動模式，是因為 Automatic（自動模式）較為保守——它會預先啟用所有相容的最佳化項目以確保絕對不溢位，但在配備強大顯示卡的硬體環境下會形成效能瓶頸並拖慢生成速度。手動模式允許具備 [🤗 Diffusers 記憶體最佳化觀念](https://huggingface.co/docs/diffusers/main/en/optimization/memory) 的進階使用者精準微調各項開關。

本節詳細剖析手動模式下的核心開關：其運作原理、效能代價以及建議啟用的時機。

### `attention_slicing`（注意力分塊切片） {: #attention_slicing }

- **運作原理**：將注意力矩陣運算切分為微小的連續切片逐步計算，而非一次性全量載入，顯著壓低注意力計算時的 VRAM 尖峰。
- **效能代價**：以速度換取記憶體（推論耗時通常增加 5–20%）。
- **啟用時機**：在生成過程中遭遇顯示卡記憶體不足 (OOM) 錯誤時開啟，特別是在 VRAM 小於 8 GB 的顯示卡、統一記憶體小於 64 GB 的 Apple Silicon (MPS)，或純 CPU 環境。若顯示卡記憶體充裕請保持關閉。

### `vae_slicing`（VAE 分塊切片） {: #vae_slicing }

- **運作原理**：在 VAE 解碼階段，以小批次分塊解碼潛在張量，而非整塊一次性解碼。
- **效能代價**：顯著降低 VAE 解碼階段的記憶體尖峰，速度損耗幾乎可忽略不計。
- **啟用時機**：單次批次生成多張圖片，或在大解析度生成最後一步解碼時遭遇 VRAM 溢位。建議隨時保持開啟；對於 Batch Size = 1 的任務近乎零開銷。

### `transformer_layerwise_casting`（逐層型別動態轉換） {: #transformer_layerwise_casting }

- **運作原理**：將 Transformer（或 UNet）權重以 fp8 (`float8_e4m3fn`) 格式常駐於記憶體中，僅在特定層實際執行矩陣乘法時，才臨時向上提升精度至 bfloat16。
- **效能代價**：相較於純 bfloat16，權重佔用記憶體近乎減半；代價為逐層動態轉換帶來的微幅延遲，以及部分模型上微乎其微的數值精度偏差。
- **啟用時機**：當模型在未壓縮狀態下無法完整塞入 VRAM，但您又不希望執行全量量化時開啟。若模型本身已經過預量化或不支援逐層轉換，節點會自動記錄日誌並略過此開關。

### `cpu_offload_strategy`（CPU 權重卸載策略） {: #cpu_offload_strategy }

- **運作原理**：在主機 CPU 記憶體 (RAM) 與顯示卡記憶體 (VRAM) 之間動態換入換出管線模組，極致壓縮 VRAM 常駐佔用。
- **選項說明**：
    - **None**：所有模型模組全程常駐於 GPU。推論速度最快，但消耗最多 VRAM。
    - **Model**：GPU 每次僅常駐一個完整子模型（例如文字編碼器、Transformer 或 VAE），其餘暫存於 CPU RAM 並在需要時換入。節省可觀 VRAM，速度微幅妥協。
    - **Sequential**：更細顆粒度的卸載：個別 `nn.Module` 神經網路層按需動態串流傳輸至 GPU。節省最多 VRAM，但速度代價極大（通常慢數倍）。
- **選用策略推薦**：
    - 若顯示卡記憶體足夠容納整個管線，請一律選 **None**。
    - 若差幾 GB 即可完全載入，推薦選 **Model**。
    - 僅在極限低配硬體上嘗試運行巨型模型時，才選用 **Sequential** 作為最後手段。

### `quantization_mode`（權重量化模式） {: #quantization_mode }

- **運作原理**：在推論前透過 `optimum-quanto` 將管線權重量化為 `fp8`、`int8` 或 `int4`。
- **效能代價**：極大幅度降低記憶體佔用（`int4` 體積約為 bfloat16 的四分之一），但隨著位元深度降低，影像精細度劣化的風險逐漸提高。首次載入時需額外承受一次性量化運算耗時。
- **啟用時機**：即使開啟了卸載機制模型依然無法載入，或希望騰出寶貴 VRAM 容納更大批次或多個 LoRA 時。`fp8` 通常是無損且安全的起點；僅在萬不得已時再降至 `int8` 或 `int4`。

### 顯示卡記憶體不足 (OOM) 階梯式調校排查法 {: #if-you-run-out-of-memory }

當遭遇記憶體溢位時，強烈建議依照以下影響與速度優先級，**逐一開啟**排查：
開啟 `vae_slicing` → 開啟 `attention_slicing` → 將 `cpu_offload_strategy` 切換為 `Model` → 開啟 `transformer_layerwise_casting` → 逐步降低 `quantization_mode`（`fp8` → `int8` → `int4`） → 最後手段切換 `cpu_offload_strategy` 為 `Sequential`。

### 猶豫不決時：直接切換至 Automatic {: #when-in-doubt-switch-to-automatic }

自動模式會透過內建的硬體記憶體決策樹（參閱 `pipeline_utils.py` 中的 `_automatic_optimize_diffusion_pipeline`），自動偵測您目前的硬體規格，並精確開啟剛好能容納該模型的最佳化組合。雖然在頂級顯卡上略遜於手工精心調校的配置，但能保證在絕大多數環境下一鍵安全運作。

有關底層最佳化概念的深度技術剖析，請參閱上游官方 [🤗 Diffusers 記憶體最佳化指南](https://huggingface.co/docs/diffusers/main/en/optimization/memory)。

## 效能最佳化實踐 {: #performance-optimization }

- **管線快取複用**：單次建構，多次生成；將多個 Generate 執行節點連線至同一個 Builder 節點。
- **快取生命週期管理**：管線在工作流程多次執行間自動快取複用，免除重複加載開銷。
- **硬體相稱的記憶體設定**：依據實體顯示卡記憶體規模配置適當的最佳化開關。
- **預覽設定權衡**：正式批次生成時關閉中間步驟即時預覽以極大化推論速度。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認 Hugging Face 存取權杖已正確設定為環境變數 `HUGGINGFACE_HUB_ACCESS_TOKEN`，詳情請參閱 [整合指南](../../guides/integrations/hugging_face.md)。
- **找不到快取管線**：若遭遇快取相關錯誤，請確保上游的 Builder 節點已成功執行完畢。
- **記憶體資源受限**：巨型模型或超高解析度生成需要充沛的 GPU 記憶體支援，請善用手動最佳化階梯調校法。
- **供應商架構相容性**：請確認選取的模型存放庫格式與選定的管線類型相互相容。
