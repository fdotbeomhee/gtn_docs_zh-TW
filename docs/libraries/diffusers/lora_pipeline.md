# LoRA Pipeline {: #lora-pipeline }

**在既有基礎管線上動態疊加非融合 (Non-fused) 的 LoRA 適配器，讓同一個記憶體快取模型能夠同時驅動多個不同風格的並行生成分支。**

類別目錄：`ModularDiffusion/Pipeline`

## 核心速覽 {: #tldr }

- 將 Pipeline Builder 的管線輸出埠與一個或多個 Load LoRA 節點連線至此節點；將 `lora_pipeline` 輸出埠接駁至 Generate Media Latents 節點（替代直接連入 Pipeline Builder）。
- 適配器權重僅在**每次生成調用時臨時啟用**，生成完畢後隨即釋放——基礎模型**絕不會被永久性修改**，因此單一快取模型可以無縫驅動多個分支（例如一個分支套用 LoRA，另一個分支維持純原生模型），且完全無需耗時重建。
- 當需要在多次生成間動態切換適配器、微調滑桿權重，或使用上下文條件 (In-Context, IC) LoRA、蒸餾加速 (Distillation) LoRA 時，本節點為絕對推薦的最佳解法。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder ──→ [LoRA Pipeline] ←── Load LoRA
                          │
                          └──→ Generate Media Latents → Decode Media Latent
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/activate-lora-pipeline.png" alt="LoRA Pipeline" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 基礎擴散管線。從 Pipeline Builder 接駁傳入。基礎模型僅被借用複用，絕不受破壞性修改——可同時連線至其他節點。 |
| `loras` | `loras` | 是 | 來自 Load LoRA 節點的一個或多個 LoRA 封裝物件。接收清單格式；可接駁多個 Load LoRA 節點輸出以實現權重疊加。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `lora_pipeline` | `Pipeline Config` | 在每次生成調用時動態啟用指定 LoRA 的管線引用。與輸入管線共享底層記憶體快取——接駁至 Generate Media Latents 絕不會觸發管線重建。 |
| `logs` | `str` | 建構過程的日誌資訊，包含管線配置雜湊值。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **動態啟用 LoRA 與融合 LoRA 的本質差異**：融合 LoRA（透過 Pipeline Builder 的 `loras` 輸入）會永久熔接至權重張量中；動態啟用 LoRA（本節點）僅為暫態掛載。修改融合 LoRA 會迫使管線全盤釋放並重新載入，而修改動態 LoRA 則完全零重建開銷。
- **至少連接一個 LoRA 權重**：節點要求 `loras` 清單中至少具備一個有效項目——執行前請務必連線至少一個 [Load LoRA](load_lora.md) 節點。
- **共享快取機制**：`lora_pipeline` 產物與輸入的 `pipeline` 完全共享相同的實體權重記憶體快取。您可以將它們分別接駁至兩個獨立的 Generate Media Latents 節點（一個帶 LoRA，另一個純原生），兩者並行運作而絕不重複佔用雙倍顯示卡記憶體。
- **執行期權重即時生效**：在多次生成任務之間，您可以隨意在 Load LoRA 節點上調整 `weight` 數值，管線將即刻套用全新強度，杜絕重建等待。

## 相關延伸閱讀 {: #see-also }

- [Load LoRA](load_lora.md) —— 提供本節點所需的 `loras` 輸入端。
- [Modular Diffusion Pipeline Builder](pipeline_builder.md) —— 提供基礎 `pipeline` 輸入端。
- [Generate Media Latents](generate_media_latents.md) —— 接收並引用 `lora_pipeline` 輸出的擴散去噪節點。
