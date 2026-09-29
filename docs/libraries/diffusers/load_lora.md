# Load LoRA {: #load-lora }

**從電腦本機硬碟載入 LoRA 權重檔案，並將其封裝為 Pipeline Builder 或 LoRA Pipeline 可接收的輸出格式。**

類別目錄：`ModularDiffusion/Pipeline`

## 核心速覽 {: #tldr }

- 每個節點專門載入一個 LoRA。若需疊加多個 LoRA，可新增多個 `Load LoRA` 節點並共同連線至同一個接收端。
- **LoRA 的兩種使用方式：**
    - **Pipeline Builder**：LoRA 會在管線建構階段直接**融合 (Fused/Baked)** 到模型權重中。此時 `weight` 權重係數被固定；若後續修改權重將會**觸發管線重新建構**。最適合用於工作流程中全程必備的核心風格 LoRA。
    - **LoRA Pipeline**：LoRA 僅在**每次生成任務 (Per-generation)** 執行時動態啟用，運算結束後隨即釋放。基礎管線本身完全不受修改，因此在多次執行間自由微調 `weight` 數值**不會**觸發重新載入。最適合用於 IC LoRA、滑桿調節型 LoRA (Slider LoRA) 或在不同工作流程分支間切換適配器的情境。
- 支援檔案格式：`.safetensors`、`.sft`、`.pt`、`.bin`、`.json`、`.lora`。

## 典型工作流程架構 {: #typical-workflow-position }

```text
# 權重融合模式 (在建構時熔接至模型中)：
[Load LoRA] ─┐
[Load LoRA] ─┼─→ Pipeline Builder → Generate Media Latents
[Load LoRA] ─┘

# 執行期動態加載模式 (每次生成時動態套用)：
Pipeline Builder ──┐
[Load LoRA] ───────┤→ LoRA Pipeline → Generate Media Latents
[Load LoRA] ───────┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/load-lora.png" alt="Load LoRA" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `file_path` | path | 是 | LoRA 權重檔案在電腦本機上的絕對路徑。 |
| `weight` | float (0.0–1.0) | 否 | 此 LoRA 的套用強度影響力，預設值為 `1.0`。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `loras` | `loras` (dict) | 結構化字典 `{path: weight}`——接駁至 Pipeline Builder 或 LoRA Pipeline 的 `loras` 輸入埠。 |
| `trigger_phrase` | str | 建議包含於 Prompt 提示詞中的觸發詞 (Trigger Phrase)；預設處於隱藏狀態。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **此節點不支援直接填寫 Hugging Face 存放庫 ID**：請先將權重檔案下載至本機硬碟；本節點僅支援從磁碟路徑載入。
- **LoRA 架構必須與基礎管線相容**（例如 Flux LoRA 僅能用於 Flux 管線）。若架構不匹配，錯誤將在管線建構階段觸發，而非在 LoRA 檔案載入時回報。
- **融合模式下的權重固化特性**：當直接連入 Pipeline Builder 時，由於權重已被熔接至模型本體，修改 `weight` 數值將導致整條管線全盤重建。若需頻繁微調數值，請改用 LoRA Pipeline 節點。
- **觸發詞 (Trigger Phrases) 填寫**：若您的 LoRA 需要特定觸發詞才能喚醒風格，請手動將其加入 Prompt 提示詞輸入中——`trigger_phrase` 目前純粹作為透傳中繼資料使用。

## 相關延伸閱讀 {: #see-also }

- [Modular Diffusion Pipeline Builder](pipeline_builder.md) —— 靜態融合 LoRA 權重的接收端。
- [LoRA Pipeline](lora_pipeline.md) —— 動態執行期套用 LoRA 的接收端。
- 工作流程內建範本：`workflows/templates/LoRAText2Image.py`。
