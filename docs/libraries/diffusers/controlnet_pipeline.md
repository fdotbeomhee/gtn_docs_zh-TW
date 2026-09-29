# ControlNet Pipeline {: #controlnet-pipeline }

**為已建構的基礎擴散管線封裝 ControlNet 控制能力，使其在生成階段接受精確的條件引導。**

類別目錄：`ModularDiffusion/Pipeline`

## 核心速覽 {: #tldr }

- 節點位置介於 [Pipeline Builder](pipeline_builder.md) 與 [Generate Media Latents](generate_media_latents.md) **之間**——它將快取的基礎管線重新封裝為具備 ControlNet 驅動能力的衍生管線。
- 接收來自一個或多個 [Configure ControlNet](configure_controlnet.md) 節點的 `control_net` 設定清單。在底層模型架構支援的前提下（例如 Flux Union），全面相容多個 ControlNet 疊加控制。
- 輸出型別：`Pipeline Config`——可直接無縫替換下游節點原有的基礎管線輸入。

## 典型工作流程架構 {: #typical-workflow-position }

```text
Pipeline Builder → [ControlNet Pipeline] → Generate Media Latents → Decode Media Latent
Configure ControlNet ─┘
Configure ControlNet ─┘
```

## 節點介面預覽 {: #node-preview }

<img src="../assets/nodes/controlnet-pipeline.png" alt="ControlNet Pipeline" width="480">

## 輸入端說明 {: #inputs }

| 名稱 | 型別 | 必填 | 說明備註 |
| :--- | :--- | :--- | :--- |
| `pipeline` | `Pipeline Config` | 是 | 來自基礎 [Pipeline Builder](pipeline_builder.md)。**絕對不能**已經是封裝過的 ControlNet 管線。 |
| `control_nets` | `list[control_net]` | 是 | 來自 [Configure ControlNet](configure_controlnet.md) 的一個或多個設定項目。所有項目的供應商必須與基礎管線完全匹配。 |

## 輸出端說明 {: #outputs }

| 名稱 | 型別 | 說明備註 |
| :--- | :--- | :--- |
| `controlnet_pipeline` | `Pipeline Config` | 注入 ControlNet 驅動邏輯後的全新管線產物。 |
| `control_parameters` | `control_parameters` | 所有已連線 ControlNet 配置的透傳集合。請接駁至 Generate Media Latents 節點的 `controlnet_parameters` 輸入埠。 |
| `logs` | `str` | 建構過程的日誌資訊，包含解析出的配置雜湊值。 |

## 技巧與避坑指南 {: #tips-pitfalls }

- **供應商必須嚴格匹配**：每個 ControlNet 的 `provider` 必須與基礎管線的供應商完全相同——節點在執行前會進行自動前置校驗。
- **管線類別相容性限制**：並非所有基礎擴散模型皆支援多 ControlNet 疊加；節點會在運算前驗證底層驅動架構，若發現不相容組合將主動回報錯誤資訊。
- **請在單一節點內疊加多個 ControlNet**：若需同時使用多個條件控制，請將多個 Configure ControlNet 節點聚合連線至同一個 ControlNet Pipeline 節點的 `control_nets` 清單輸入埠，**切勿串聯兩個 ControlNet Pipeline 節點**（第二個節點在檢測到輸入已為 ControlNet 產物時將報錯拒絕）。
- **管線快取複用**：系統僅為選定的 ControlNet 動態加載額外的控制層權重；基礎擴散管線本身仍會優先直接複用記憶體快取。

## 相關延伸閱讀 {: #see-also }

- [Configure ControlNet](configure_controlnet.md) —— 生成本節點所需的 `control_net` 控制條目。
- [Modular Diffusion Pipeline Builder](pipeline_builder.md) —— 本節點不可或缺的上游管線來源。
- 工作流程內建範本：`workflows/templates/ControlnetText2Image.py`。
