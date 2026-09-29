# Modular Diffusion Pipeline Builder {: #modular-diffusion-pipeline-builder }

**單次載入並快取 🧨 Diffusers 擴散管線，供工作流程中的所有後續節點重複複用。**

類別目錄：`ModularDiffusion/Pipeline`

## 核心速覽 {: #tldr }

- **單次建構，多次生成**：請將此節點置於工作流程前端；將其 `pipeline` 輸出埠連線至所有需要模型權重的 Modular Diffusion 節點（如 Generate Media Latent、VAE Encode/Decode、Create Noise 等）。
- **優先選定 `provider`**：模型存放庫路徑與執行階段各項參數，皆會依據您選取的模型供應商動態重新生成。
- **管線快取機制**：系統會依據管線設定值的雜湊 (Hash) 自動快取模型。修改載入期參數（如模型存放庫、量化模式、LoRA）會觸發管線重新建構與快取；而執行期參數（如 Prompt 提示詞、步數）則**不會**觸發重新載入。
- 輸出型別：`Pipeline Config`。

## 典型工作流程架構 {: #typical-workflow-position }

```text
[Pipeline Builder] → Create Noise Latents → Generate Media Latents → Decode Media Latent
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/pipeline-builder.png" alt="Modular Diffusion Pipeline Builder" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `loras` | `loras` | 否 | 連接一個或多個 [Load LoRA](load_lora.md) 節點。依連線順序依序疊加掛載。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 快取的管線設定成品。接駁傳遞至所有接收 `pipeline` 輸入的節點。 |
| `logs` | `str` | 建構過程的日誌資訊，包含解析出的配置雜湊值。 |

## 參數說明 {: #parameters }

### 管線選擇 *(動態參數——隨 `provider` 變更而自動更新)* {: #pipeline-selection }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `provider` | choice | `Flux`、`Flux2`、`Stable Diffusion`、`Stable Diffusion 3`、`Qwen`、`Z-Image`、`HunyuanVideo 1.5`、`LTX`、`LTX2`、`WAN`。切換此項會動態重置下方所有參數。 |
| `pipeline_type` | choice | 針對選定供應商的特定管線類別（例如 `FluxPipeline`、`WanImageToVideoPipeline`）。決定該管線的核心功能能力。 |
| `<model repo>` | HF repo picker | Hugging Face 存放庫 ID。僅支援 Diffusers 標準目錄格式——不直接載入單一檔案的 `.safetensors` Checkpoint 權重。 |

### 顯示卡記憶體最佳化 (Memory optimization) {: #memory-optimization }

| 名稱 | 型別 | 預設值 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `memory_optimization_strategy` | choice | `Manual` | 選為 `Automatic` 時將隱藏下方各項微調開關，改用系統為該模型配置的最佳化預設值。 |
| `attention_slicing` | bool | `False` | 注意力矩陣分塊切片：顯著降低 VRAM 佔用，速度微幅下降。 |
| `vae_slicing` | bool | `False` | VAE 逐格解碼切片：以 Batch Size = 1 解碼潛在張量；對大批次生成極為有效。 |
| `transformer_layerwise_casting` | bool | `False` | 逐層型別動態轉換：Transformer 主體常駐於低精度，僅在特定層運算時提升精度。 |
| `cpu_offload_strategy` | choice | `None` | CPU 權重卸載：可選 `Model`（整模組卸載）或 `Sequential`（逐層卸載）——閒置時移至記憶體以釋放 VRAM，會降低推論速度。 |
| `quantization_mode` | choice | `None` | 權重量化模式：`fp8` / `int8` / `int4`（透過 `optimum-quanto` / `bitsandbytes`）——大幅壓縮權重體積，但會略微妥協畫質。 |

請僅啟用工作流程實際需要的項目——每項最佳化皆是在「推論速度」與「顯示卡記憶體」之間進行權衡。

## 技巧與避坑指南 {: #tips-pitfalls }

- **引擎重新啟動後的快取生命週期**：管線快取僅存在於本機處理程序記憶體中；重新啟動引擎後首次執行時，節點會自動重新加載並快取。
- **載入時顯示卡記憶體不足 (OOM)**：模型權重超過可用 GPU 記憶體容量。請依據影響程度依序嘗試以下調整：將 `quantization_mode` 設為 `int8` 或 `int4`、將 `cpu_offload_strategy` 設為 `Model`（將閒置子模組移至 CPU），或開啟 `vae_slicing`（降低解碼期的記憶體尖峰峰值）。
- **LoRA 權重未生效**：請檢查兩點：(1) LoRA 是否針對您選取的 `pipeline_type` 相同模型架構進行訓練——例如 Flux LoRA 無法套用於 WAN 視訊管線；(2) [Load LoRA](load_lora.md) 節點是否已穩固連線至本節點的 `loras` 輸入端。
- **配置參數前務必先選定 `provider`**：更換 `provider` 會清空並重新生成針對新模型的專屬參數——同名參數的連線會盡可能保留，但數值將恢復預設狀態。

## 相關延伸閱讀 {: #see-also }

- [Configure ControlNet](configure_controlnet.md) · [ControlNet Pipeline](controlnet_pipeline.md) —— 為建構好的管線掛載 ControlNet 條件引導。
- [Load LoRA](load_lora.md) —— 載入微調權重以接駁至本節點。
- [Generate Media Latents](generate_media_latents.md) —— 接收並引用本節點 `pipeline` 產物的核心擴散生成節點。
