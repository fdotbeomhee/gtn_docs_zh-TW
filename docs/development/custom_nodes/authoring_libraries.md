# 程式庫編寫與發布 (Authoring Libraries) {: #authoring-libraries }

節點是以程式庫 (Libraries) 的形式進行散布與分享的。本頁面詳細介紹了程式庫資訊清單檔 (`griptape_nodes_library.json`)、宣告語法、依賴套件管理、文件規範，以及向標準程式庫貢獻節點的完整流程。

## 建立節點程式庫 {: #creating-node-libraries }

將多個節點打包為程式庫以便於分享。建立 `griptape_nodes_library.json`：

```json
{
  "name": "Library Name",
  "library_schema_version": "0.11.0",
  "settings": [
    {
      "description": "API keys required by nodes in this library",
      "category": "app_events.on_app_initialization_complete",
      "contents": {
        "secrets_to_register": ["MY_SERVICE_API_KEY", "MY_OTHER_API_KEY"]
      }
    }
  ],
  "metadata": {
    "author": "Author Name",
    "description": "Library description",
    "library_version": "1.0.0",
    "engine_version": "0.55.0",
    "tags": ["AI", "Image Processing"],
    "dependencies": {
      "pip_dependencies": ["pillow", "requests"],
      "pip_install_flags": ["--upgrade"]
    },
    "declarations": [
      { "type": "lifecycle_stage", "stage": "STABLE" },
      {
        "type": "model_catalog",
        "providers": {
          "anthropic": {
            "display_name": "Anthropic",
            "terms_url": "https://www.anthropic.com/legal/commercial-terms",
            "models": {
              "claude_opus_byok": {
                "display_name": "Claude Opus 4 (BYOK)",
                "family": "Claude 4",
                "provider_model_id": "claude-opus-4",
                "key_support": "REQUIRES_CUSTOMER_KEY"
              }
            }
          }
        }
      }
    ]
  },
  "widgets": [
    {
      "name": "MyWidget",
      "path": "widgets/MyWidget.js",
      "description": "Custom UI component for the node"
    }
  ],
  "categories": [
    {
      "image": {
        "title": "Image Processing",
        "description": "Image manipulation nodes",
        "color": "border-purple-500",
        "icon": "Image"
      }
    }
  ],
  "nodes": [
    {
      "class_name": "MyImageNode",
      "file_path": "image/my_image_node.py",
      "metadata": {
        "category": "image",
        "description": "Process images with AI",
        "display_name": "AI Image Processor",
        "icon": "image",
        "group": "processing",
        "declarations": [
          { "type": "model_usage", "model_ids": ["claude_opus_byok"] }
        ]
      }
    }
  ],
  "workflow_nodes": [
    {
      "node_type": "UpscaleAndTag",
      "workflow_path": "workflows/upscale_and_tag.py",
      "metadata": {
        "category": "image",
        "description": "Upscales an image and tags it",
        "display_name": "Upscale and Tag"
      }
    }
  ],
  "workflows": ["workflows/example_workflow.py"],
  "is_default_library": false
}
```

### 程式庫結構 {: #library-structure }

- **settings**：註冊程式庫內節點所需的敏感金鑰與 API 金鑰
    - 使用 `secrets_to_register` 陣列宣告必要的金鑰清單
    - 分類 category 應設定為 `app_events.on_app_initialization_complete`
    - 金鑰應透過 `GriptapeNodes.handle_request(GetSecretValueRequest(key=...))` 進行安全讀取
- **metadata.dependencies**：載入程式庫時自動透過 PIP 安裝的相依套件
- **metadata.declarations** / 單一節點 **metadata.declarations**：具備型別的識別屬性（生命週期階段、任意 Python 執行權限）以及程式庫層級的模型目錄與各節點對應的引用。詳見下方的 [程式庫與節點宣告](#library-and-node-declarations)。
- **beta_features**：使用者可在編輯器的測試功能頁面開啟的實驗性特性。詳見下方的 [測試功能 (Beta Features)](#beta-features)。
- **widgets**：註冊自訂 JS 視覺部件元件（參見 [自訂部件 (Custom Widgets)](custom_widgets.md)）
- **categories**：在 UI 上為節點分組，並指派專屬顏色與圖示
- **nodes**：列出節點類別、檔案路徑與元數據
- **advanced_library_path**：可選的 Python 檔案路徑，用於宣告 `AdvancedNodeLibrary` 子類別，適用於需在載入與卸載時執行自訂程式碼、掌管自訂請求型別或動態註冊資訊清單未列出之節點型別的進階程式庫（參見 [進階程式庫 (Advanced Libraries)](advanced_libraries.md)）
- **workflow_nodes**：由儲存的工作流程檔案動態生成的節點，而非傳統 Python 類別。詳見下方的 [來自工作流程檔案的節點](#nodes-from-workflow-files)。
- **workflows**：範本工作流程檔案清單
- **is_default_library**：是否為預設啟用的核心程式庫

**重要提示：** `secrets_to_register` 陣列向系統宣告您的程式庫需要哪些機密憑據。使用者將在 UI 介面上或透過環境變數被引導完成這些金鑰的配置。

請使用扁平的目錄結構。引擎會自動探索、註冊並載入程式庫。

### 來自工作流程檔案的節點 {: #nodes-from-workflow-files }

節點不一定必須撰寫為 Python 類別。只需將 `workflow_nodes` 項目指向已儲存的工作流程檔案，引擎便會自動為您生成對應的節點型別：

```jsonc
"workflow_nodes": [
  {
    "node_type": "UpscaleAndTag",
    "workflow_path": "workflows/upscale_and_tag.py",
    "metadata": {
      "category": "image",
      "description": "Upscales an image and tags it",
      "display_name": "Upscale and Tag"
    }
  }
]
```

| 欄位 | 意涵 |
| :--- | :--- |
| `node_type` | 節點註冊於系統中的名稱。此名稱將被寫入儲存的工作流程檔中。 |
| `workflow_path` | 指向已儲存的 `.py` 工作流程檔案之路徑，相對於 `griptape_nodes_library.json`。 |
| `metadata` | 與 `nodes` 陣列相同的節點元數據區塊：包含 category、description、display_name 等。 |

**該工作流程必須具備 Start Flow 節點與 End Flow 節點。** 它們定義了該生成節點的各項參數：

- **Start Flow** 節點上除控制流之外的所有非控制參數，皆轉化為該節點的**輸入端 (Input)**。
- **End Flow** 節點上除控制流之外的所有非控制參數，皆轉化為該節點的**輸出端 (Output)**。
- 該節點自身會提供獨立的 Flow In 與 Flow Out 連接埠，因此原工作流程內部的控制參數不會暴露在外。
- End Flow 節點內建的 **Status** 狀態參數（`was_successful`、`result_details`）亦不會外顯。它們僅回報 End Flow 節點本身的運行狀態，而非您業務中暴露的產物。僅當您自行定義的同名參數同時被歸入名為 `Status` 的參數分組時，才會被過濾忽略。

若參數名稱在所有 Start Flow 或 End Flow 節點中唯一，則維持其原始名稱。若兩個 Start Flow 節點同時暴露了 `prompt`，則這兩個參數將自動附加各自的節點名稱作為前置修飾（`Start_Flow.prompt`、`Start_Flow_2.prompt`）以避免衝突。由於參數名稱不允許空格，節點名稱中的空格會自動替換為底線。若同一個參數名稱同時出現在 Start Flow 與 End Flow 節點上，則該名稱將聚合為一個兼具輸入與輸出的單一參數。

當該節點執行時，引擎會將原工作流程載入為該節點所屬流程的子流程 (Child Subflow)，將節點輸入端的數值複製至各 Start Flow 節點上，執行該子流程，並將 End Flow 的輸出值取回賦予該節點的輸出端。該子流程在同一個工作階段中會被重複快取複用，且永遠不會被直接序列化寫入使用者的外層存檔中——因此使用者儲存的工作流程中僅記錄該節點本體，而非背後完整工作流程的一份拷貝。

**在正式交付釋出前，請務必先在編輯器中儲存該工作流程。** 節點的參數規格是直接自工作流程檔案頂部的元數據標頭 (Metadata Header) 解析讀取的，編輯器在存檔時會自動產生此標頭。若工作流程缺少已儲存的參數結構（例如未包含 Start Flow 與 End Flow 節點，或純手工編寫而缺乏標頭），系統將回報為程式庫問題，且該節點將無法被註冊載入。

該工作流程本身亦會被註冊為獨立的工作流程，因此會出現在編輯器的工作流程選擇器清單中。若希望隱藏僅作為節點後端實現的工作流程，可在其元數據標頭中宣告 `is_internal = true`。

### 程式庫與節點宣告 (Declarations) {: #library-and-node-declarations }

宣告 (Declarations) 用於為程式庫或個別節點附加結構化的型別元數據。`declarations` 陣列中的每個項目皆為一個攜帶 `type` 鑑別器的物件，用於選取對應的宣告類別。目前的詞彙表涵蓋了生命週期階段屬性、包含各節點引用的程式庫層級模型目錄，以及任意 Python 程式碼執行權限宣告；未來的引擎版本將在此欄位下擴充更多宣告型別。

程式庫層級的 `metadata.declarations` 與節點層級的 `metadata.declarations` 皆接受清單格式。清單內項目的順序無關緊要。該欄位預設為 `[]`，因此採用舊版本結構規格（`0.6.0`, `0.4.0`, `0.1.0`）的程式庫無需任何修改即可正常載入。

#### `lifecycle_stage` {: #lifecycle-stage }

宣告程式庫或特定節點所處的生命週期成熟度階段。可選值包括：

| 數值 | 意涵 |
| :--- | :--- |
| `STABLE` | 穩定；成熟可靠，適合生產環境使用。 |
| `BETA` | 測試版；功能已齊備，但仍在進行健全性強化。 |
| `ALPHA` | 早期預覽版；處於初期實現階段，預期可能發生重大破壞性變更 (Breaking changes)。 |
| `LABS` | 實驗室特性；探索性功能，未來可能被移除。 |
| `DEPRECATED` | 已淘汰；已排定移除計畫，現有使用者應儘速遷移至替代方案。 |

語意規範：

- **程式庫層級的缺省與 `STABLE` 具備刻意的語意區分。** 未宣告 `lifecycle_stage` 的程式庫視為「未指明 (unstated)」——使用端應在介面上顯式標註（`<No lifecycle stage provided by library author>`），而非靜默假設其為 `STABLE`。
- **節點層級的缺省代表「繼承程式庫的生命週期階段」。** 節點層級的 `lifecycle_stage` 設定將覆寫程式庫層級的值。

#### `model_catalog` {: #model-catalog }

在程式庫層級宣告其內部節點可存取的第三方模型註冊表，組織為 `provider → model` 的層級結構。這兩層的識別碼皆為字典的鍵（該鍵即為節點引用與管理原則所依據的**穩定控制代碼**）；每個項目皆包含 UI 呈現用的 `display_name`，以及可選的 `terms_url` 服務條款連結與 `notes` 附註。每個 `Model` 模型額外強制宣告 `key_support`，以及可選的 `family` 系列分組標籤與上游 `provider_model_id`。

`key_support` 欄位明確告知系統管理員該模型調用所需的 API 金鑰型別：

| 數值 | 意涵 |
| :--- | :--- |
| `REQUIRES_CUSTOMER_KEY` | 僅支援客戶自行提供的 API 金鑰 (BYOK)。 |
| `SUPPORTS_CUSTOMER_KEY_OR_GRIPTAPE_KEY` | 同時支援客戶自行提供的金鑰或 Griptape 官方金鑰。 |
| `REQUIRES_GRIPTAPE_KEY` | 僅支援 Griptape 官方提供的 API 金鑰。 |
| `NO_KEY_REQUIRED` | 本機運行或無需 API 金鑰（例如由 Ollama 託管的模型）。 |

`notes`（在提供商與模型層級皆可配置）是隨項目一同渲染的自由格式開發者指引。適用於無法歸入其他欄位的特殊說明，例如「BYOK 需要注入特定提供商的提示驅動器」。

```jsonc
{
  "type": "model_catalog",
  "providers": {
    "anthropic": {
      "display_name": "Anthropic",
      "terms_url": "https://www.anthropic.com/legal/commercial-terms",
      "models": {
        "claude_opus_byok": {
          "display_name": "Claude Opus 4 (BYOK)",
          "family": "Claude 4",
          "provider_model_id": "claude-opus-4",
          "key_support": "REQUIRES_CUSTOMER_KEY"
        },
        "claude_opus_griptape": {
          "display_name": "Claude Opus 4 (Griptape Key)",
          "family": "Claude 4",
          "provider_model_id": "claude-opus-4",
          "key_support": "REQUIRES_GRIPTAPE_KEY"
        }
      }
    },
    "kling": {
      "display_name": "Kling",
      "terms_url": "https://app.klingai.com/global/about/terms",
      "models": {
        "kling_v2": {
          "display_name": "Kling v2",
          "provider_model_id": "kling-v2-master",
          "key_support": "REQUIRES_GRIPTAPE_KEY"
        }
      }
    },
    "ollama": {
      "display_name": "Ollama",
      "key_support": "NO_KEY_REQUIRED",
      "notes": "本機執行環境；模型於執行階段動態列舉，此處無靜態宣告。"
    }
  }
}
```

重要規則與原則：

- **`family` 僅為顯示分組標籤。** 它將相關模型群組化以便在 UI 上展示（例如上述的兩款 Claude 4 模型）。它不是容器，亦不構成模型唯一身分識別的一部分，因此缺乏意義分組的提供商直接省略即可。
- **`key_support` 預設直接隸屬於模型。** 每個 `Model` 皆宣告自己的授權模式。同一個上游模型若具備兩種不同的金鑰要求，應拆分為兩個不同鍵名的模型（參見上述的 `claude_opus_byok` 與 `claude_opus_griptape`）。`ModelProvider` 亦可選填 `key_support`，但僅在其完全未宣告任何模型時生效（例如像 Ollama 這種本機環境提供商，`key_support=NO_KEY_REQUIRED` 是唯一有意義的訊號）。
- **模型 ID 在整個程式庫中必須保持全域唯一。** Pydantic 會自動約束各提供商內部 `models` 字典鍵的唯一性；跨不同提供商的命名衝突則會在程式庫載入階段被攔截為 `DuplicateModelIdProblem`。
- **每個程式庫最多宣告一個 `model_catalog`。** 宣告兩個目錄會在驗證時被拒絕；請將所有提供商整合至單一目錄中。

節點的模型選擇下拉選單儲存的是提供商自身的模型識別碼（如 `kling-v2-master`，而非顯示名稱 `Kling v2`），隨後 `ModelAccessComponent` 會將其解析為許可權限層所管制的目錄鍵。若改為儲存顯示名稱，將導致選取項目無法正確解析，進而無法對照授權原則進行查驗。若某個下拉選單歷史上儲存了其他值，請使用該元件的 `deprecated_values` 映射表自動遷移舊存檔，切勿為了迎合舊值而重新命名目錄項目：

```python
self._model_access = ModelAccessComponent(
    node=self,
    parameter=model_param,
    model_choices=["kling-v2-master", "kling-v2-1-master"],
    default_model="kling-v2-master",
    deprecated_values={"Kling v2": "kling-v2-master", "Kling v2.1": "kling-v2-1-master"},
)
```

在指派數值的任何地方（包含工作流程載入時），每個歷史舊鍵皆能被相容接收並自動遷移為映射後的選項。舊鍵絕不會出現在下拉選單中，因此淘汰模型意味著將其自 `model_choices` 移至 `deprecated_values` 並指向其替代方案。每個映射目標值必須存在於 `model_choices` 中，且舊鍵本身不能已是現有選項；任何一項錯誤都會在節點建構時拋出異常。若您正要淘汰的模型恰好也是 `default_model`，請在同一次修改中將 `default_model` 改為指向替代模型——`default_model` 必須永遠指向現有的有效選項，絕不能指向已淘汰的鍵，否則元件會拋出異常。

##### `model_usage` {: #model-usage }

節點可透過字典鍵引用目錄中的一個或多個具體模型。當節點綁定至特定具名模型集合時使用。在程式庫載入時，每個項目必須能解析至目錄中的某個模型；無法解析的引用將暴露為 `UnresolvedModelUsageReferenceProblem`。

```jsonc
{ "type": "model_usage", "model_ids": ["claude_opus_byok", "kling_v2"] }
```

##### `model_provider_usage` {: #model-provider-usage }

節點可引用整個提供商。當節點需要在執行階段動態列舉某提供商旗下的所有可用模型時使用。每個項目必須能解析至目錄中已宣告的提供商；無法解析的引用將暴露為 `UnresolvedModelProviderUsageReferenceProblem`。

```jsonc
{ "type": "model_provider_usage", "provider_ids": ["anthropic", "ollama"] }
```

這兩類使用宣告彼此獨立。單一節點可攜帶任意組合——例如：「該提供商旗下的所有模型，外加來自另一提供商的這兩款特定模型」。

#### `arbitrary_python_execution` {: #arbitrary-python-execution }

宣告節點會執行於執行階段傳入的任意 Python 程式碼（例如由創作者自行編寫的腳本）。僅限節點層級宣告。這是一項攸關安全性的核心身分屬性：使用端 UI 可以在節點運作前向創作者發出醒目的安全警示。缺少此宣告即代表該節點保證不執行任意 Python 程式碼。

| 欄位 | 意涵 |
| :--- | :--- |
| `executes_arbitrary_python` | 為 `true` 時代表節點會執行未經審查、執行階段傳入的 Python 程式碼。 |

```jsonc
"declarations": [
  { "type": "arbitrary_python_execution", "executes_arbitrary_python": true }
]
```

#### 組合多項宣告 {: #combining-declarations }

節點可以靈活攜帶多種宣告組合。例如，一個使用兩款模型的實驗室 (Labs) 階段節點：

```jsonc
"metadata": {
  "category": "labs",
  "description": "Labs node demonstrating multiple declarations.",
  "display_name": "Labs Node",
  "declarations": [
    { "type": "lifecycle_stage", "stage": "LABS" },
    { "type": "model_usage", "model_ids": ["claude_opus_byok", "kling_v2"] }
  ]
}
```

未來引擎版本中新增的宣告型別皆會以累加擴充的方式納入此 `declarations` 欄位中，而無需升級架構規格版本號。

### 測試功能 (Beta Features) {: #beta-features }

測試功能機制允許您在程式庫中發布全新功能，但在使用者主動開啟前維持停用狀態。您的功能將呈現在編輯器 [測試功能 (Beta Features)](../../guides/editor/beta_features.md) 設定頁面的 **Engine** 群組中，供使用者隨時切換開關。請在各功能的 `description` 中明確提及所屬節點的名稱，以便使用者識別其歸屬。

在 `griptape_nodes_library.json` 的最頂層（與 `name`、`metadata` 和 `nodes` 並列）以 `beta_features` 清單宣告各項特性。請注意它不是 `metadata` 的子項：

```jsonc
"beta_features": [
  {
    "id": "sharpen_after_upscale",
    "name": "Sharpen after upscaling",
    "description": "Adds a Sharpen setting to the Upscale Image node.",
    "owner": "@your-github-handle",
    "remove_by": "2027-03-31"
  }
]
```

將 `remove_by` 設定為具體日期，距離您新增該功能當日起算不得超過 180 天。

| 欄位 | 意涵 |
| :--- | :--- |
| `id` | 以小寫字母與數字組成、由單個底線分隔並以字母開頭的字串。在您的程式庫內必須保持唯一。 |
| `name` | 顯示於測試功能設定頁面上的簡潔標籤名稱。 |
| `description` | 用一至兩句話清晰告知使用者在何處發生了何種改變。 |
| `default` | 選填。尚未主動設定之使用者的預設開關狀態。預設為 `false`。 |
| `owner` | 負責完善該功能或進行最終清理移除的人員識別碼。 |
| `remove_by` | `YYYY-MM-DD` 格式的過期截止日期，屆時您必須將該功能轉為標準功能或將其徹底移除。最長不得超過新增日起算 180 天。 |

在節點中透過 `self.is_beta_feature_enabled("<id>")` 檢查功能狀態。它會回傳使用者的選取狀態，若使用者未作選擇則回傳 `default` 值。**務必始終宣告建立該功能所使用的所有參數，並僅依據開關狀態動態隱藏或顯示它們。** 如此一來，由開啟該功能的使用者所儲存的工作流程，在未開啟該功能的使用者端依然能順暢開啟並還原，反之亦然。

```python
from typing import Any

from griptape_nodes.exe_types.core_types import Parameter
from griptape_nodes.exe_types.node_types import DataNode


class UpscaleImage(DataNode):
    def __init__(self, name: str, metadata: dict[str, Any] | None = None) -> None:
        super().__init__(name, metadata)

        self.add_parameter(
            Parameter(name="image", input_types=["ImageArtifact"], type="ImageArtifact", tooltip="Image to upscale")
        )
        self.add_parameter(Parameter(name="upscaled_image", output_type="ImageArtifact", tooltip="The upscaled image"))

        # 為所有環境建立該參數，但僅對開啟測試功能的使用者呈現
        self.add_parameter(
            Parameter(
                name="sharpen",
                input_types=["float"],
                type="float",
                default_value=0.0,
                tooltip="How much to sharpen the image after upscaling",
            )
        )
        if not self.is_beta_feature_enabled("sharpen_after_upscale"):
            self.hide_parameter_by_name("sharpen")

    def process(self) -> None:
        image = self.get_parameter_value("image")
        upscaled = upscale(image)

        if self.is_beta_feature_enabled("sharpen_after_upscale"):
            upscaled = sharpen(upscaled, self.get_parameter_value("sharpen"))

        self.parameter_output_values["upscaled_image"] = upscaled
```

關鍵規範與注意事項：

- **單一項目的設定錯誤不會阻止整個程式庫的載入。** 缺失必要欄位、非法 `id` 或重複的 `id` 會被回報為程式庫問題，且該項測試功能會被忽略。您的其他功能與節點仍能正常運作。
- **檢查未宣告的 id 會回傳 `false`** 並在日誌中輸出帶有該特徵名稱的警告訊息。
- **在 `__init__` 中的檢查僅在節點初次建立時執行一次。** 若使用者中途開啟或關閉測試功能，畫布上已存在的節點會保持其建立時的參數呈現狀態，直到使用者重新整理節點、重新新增節點或重開工作流程。而在 `process` 中的檢查則會在下一次執行運算時立即反映最新值。
- **超過 `remove_by` 期限後，該功能將一律強制採用 `default` 值。** 它將自測試功能頁面消失，且程式庫會持續回報問題警示，直到您將該特性轉正為標準功能或徹底刪除。宣告超過 180 天的 `remove_by` 日期亦會被立即報警。
- **使用者的選項設定是依程式庫隔離儲存的**，記錄於其設定檔的 `library_beta_features` 下，鍵名為您程式庫名稱轉換為小寫並將空格及標點轉為底線後的字串。例如 "Acme Image Tools" 會轉化為 `library_beta_features.acme_image_tools.sharpen_after_upscale`。僅保留英文字母 a 至 z 與數字，因此若程式庫名稱完全不含英數字將無法支援測試功能；兩個載入的程式庫若轉換為相同鍵名亦會被回報為程式庫衝突。
- **在程式庫架構版本 `0.14.0` 之前發布的引擎會自動忽略 `beta_features`。** 新增測試特性時，請將 `library_schema_version` 設定為 `0.14.0` 或更高版本。

## 採用 uv 進行相依套件管理的程式庫架構 {: #library-structure-with-uv-dependency-management }

**現代推薦做法**：依循 Minimax 程式庫範式，採用 `uv` 實現飛快、可重現的現代相依套件管理。

### 目錄結構 {: #directory-structure }

```
library-name/
├── pyproject.toml              # uv 專案設定檔
├── uv.lock                     # 鎖定檔案（自動生成）
├── LICENSE                     # 授權協定
├── README.md                   # 說明文件
├── CHANGELOG.md                # 版本更新歷程
├── .gitignore                  # Git 忽略清單
└── library_name/
    ├── griptape_nodes_library.json  # 程式庫資訊清單檔
    └── node_file.py
```

### pyproject.toml 配置範例 {: #pyprojecttoml-configuration }

```toml
[project]
name = "library-name"
version = "1.0.0"
description = "Description of your library"
authors = [
    {name = "Your Name", email = "email@example.com"}
]
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests",
    # 在此新增節點執行階段匯入的其他第三方套件
]

[dependency-groups]
dev = ["griptape-nodes-engine", "pytest", "pyright", "ruff"]

[tool.uv.sources]
griptape-nodes-engine = { git = "https://github.com/griptape-ai/griptape-nodes-engine", rev = "latest" }

[tool.hatch.build.targets.wheel]
packages = ["library_name"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

#### 切勿將引擎列為執行階段依賴套件 {: #do-not-list-the-engine-as-a-runtime-dependency }

引擎是負責載入您程式庫的宿主環境，而非您程式庫所引入的依賴套件。請嚴格將 `griptape-nodes-engine` 排除在 `[project] dependencies` 之外：

- 否則在安裝您的程式庫時，會額外安裝**第二份引擎實例**。引擎會將程式庫的虛擬環境置於其自身匯入路徑的最前端，導致這份副本遮蔽正在運行的真實引擎，引發看似引擎核心崩潰、實為程式庫遮蔽的詭異錯誤。
- `[dependency-groups] dev` 依然能為您的單元測試、型別檢查器與 IDE 編輯器提供完整的引擎型別解析支援，因為 `uv sync` 預設會自動安裝開發依賴組。您的開發體驗絲毫不受影響。
- 程式庫所需的具體引擎版本應明確宣告於資訊清單 JSON 的 `engine_version` 欄位中。這是引擎載入時實際查驗的真值；pyproject 中的版號規範在載入時完全不會被查核。

在兩處同時宣告意味著維護同一事實兩次，且兩者極易發生漂移脫節。

!!! note "程式庫打包機制演進中的預告"

    未來當程式庫全面支援打包為獨立隔離環境時，引擎將成為常規的有界依賴套件（`griptape-nodes-engine>=X,<Y`），屆時將由單一的套件解析機制取代 `engine_version` 檢查。在此機制正式發布前，請嚴格遵守上述配置方式。

### 程式庫內部配置（位於子目錄內） {: #library-configuration-inside-subdirectory }

將 `griptape_nodes_library.json` 置於程式庫子目錄內部：

```json
{
  "name": "Library Name",
  "library_schema_version": "0.1.0",
  "settings": [
    {
      "description": "API keys required by nodes",
      "category": "app_events.on_app_initialization_complete",
      "contents": {
        "secrets_to_register": ["API_KEY_NAME"]
      }
    }
  ],
  "nodes": [
    {
      "class_name": "NodeClassName",
      "file_path": "node_file.py", // 相對於程式庫子目錄
      "metadata": {
        "category": "category_name",
        "description": "Node description",
        "display_name": "Node Display Name"
      }
    }
  ]
}
```

### README 中的安裝說明規範 {: #installation-instructions-in-readme }

同時提供 uv（推薦首選）與 pip（備用方案）兩種安裝途徑：

````markdown
## 安裝方式

### 選項 1：使用 uv（推薦首選）

1. Clone 或下載本程式庫：

2. 安裝依賴套件：

   ```bash
   cd library-name
   uv sync
   ```

3. 將資料夾置於 Griptape Nodes 的 libraries 目錄中

### 選項 2：自動安裝

1. 將資料夾直接放置於 libraries 目錄中
2. 啟動時將透過 pip 自動安裝各項依賴
````

### 產生鎖定檔案 (Lock File) {: #generate-lock-file }

```bash
cd library-name
uv sync
```

**採用 uv 的優勢：**

- 極致飛快的安裝速度（基於 Rust 構建）
- 透過 lock 檔案實現高度一致、可重現的建置環境
- 直接支援 griptape-nodes 的 GitHub 整合
- 完美向下相容 pip 安裝模式

## 節點程式庫文件編寫範式 {: #documentation-patterns-for-node-libraries }

### 完整 README 結構範本 {: #comprehensive-readme-structure }

```markdown
# 程式庫名稱

簡要描述與核心功能亮點。

## 功能特性

- 核心能力項目清單
- 涵蓋的模型支援
- 突顯獨特優勢

## 安裝指南

### 選項 1：使用 uv（推薦首選）

uv 安裝步驟

### 選項 2：自動安裝

pip 安裝步驟

## 快速上手

### 簡易模式（初學者推薦）

附帶說明的極簡範例

### 自訂模式（進階）

展示所有高階特性的完整範例

## 參數說明

### 基礎參數

包含 名稱、型別、說明的表格

### 進階參數（預設折疊）

包含 名稱、型別、預設值、說明的表格

### 輸出端參數

輸出端清單表格

## 模型對比評估

模型對比分析表格：
| 模型 | 最大生成時長 | 品質表現 | 運算速度 | 字元限制 |

## 字元配額限制

清晰展示各模型與模式下的字元限制表格

## API 頻率限制 (Rate Limits)

記錄說明：

- 並發請求上限
- 預期生成等待時間
- 檔案保留政策

## 範例工作流程

涵蓋常見情境的 3–5 個完整範例

## 錯誤處置

常見錯誤代碼與排查指南

## 疑難排解 (Troubleshooting)

常見問答 FAQ 式排錯指引

## API 參考連結

官方 API 開發者文件連結

## 最佳實踐

最佳化效能與成本的操作技巧

## 技術支援

獲取社群支援的途徑

## 版本歷史

指向 CHANGELOG 的連結
```

### 必備的經典排錯條目說明 {: #troubleshooting-entries-worth-documenting }

有兩項錯誤極為普遍，強烈建議在程式庫 README 的疑難排解章節中收錄：

#### 錯誤 1："Missing required variables: file_extension, file_name_base" {: #error-missing-required-variables-file-extension-file-name-base }

**完整報錯資訊：**

```text
ERROR: Attempted to resolve macro path. Failed because missing required variables: file_extension, file_name_base
ERROR: Attempted to create download URL. Failed with file_path='{outputs}/{node_name?:_}{file_name_base}{_index?:03}.{file_extension}'
```

**根本成因：** 未能正確捕獲 `write_bytes()` 的回傳值，誤用了未經完全解析的 `dest.location` 而非保存完成的 `saved.location`。

**錯誤寫法：**

```python
dest = self._output_file.build_file()
dest.write_bytes(video_bytes)  # ❌ 未捕獲回傳值
artifact = VideoUrlArtifact(dest.location)  # 錯誤：使用的是 dest 而非已存檔的物件
```

**修復方案：**

```python
dest = self._output_file.build_file()
saved = dest.write_bytes(video_bytes)  # ✅ 正確捕獲存檔物件
artifact = VideoUrlArtifact(saved.location)  # 使用 saved 物件所解析的正確 location
```

**技術原理解析：** 巨集變數是在 `write_bytes()` 實際執行存檔寫入時才動態填入完成的。`write_bytes()` 回傳的 `saved` 物件才真正包含完全解析就緒的絕對路徑。

#### 錯誤 2：影像參數型別轉換異常 {: #error-type-conversion-issues-with-image-parameters }

**症狀表徵：** 影像參數無法一致相容處理不同的輸入型別，或在節點間傳遞 URL、實體路徑或產物時引發型別崩潰。

**問題根源：** 使用泛用的 `Parameter` 搭配手動型別配置，缺乏標準化的型別適配轉換邏輯：

```python
# ❌ 不一致的型別處理
Parameter(
    name="image",
    input_types=["ImageArtifact", "ImageUrlArtifact", "str"],
    type="ImageArtifact",
)
```

**修復方案：** 使用 `ParameterImage` 達成全自動標準化型別轉換：

```python
# ✅ 標準化型別處理
ParameterImage(
    name="image",
    tooltip="Input image",
    allow_output=False,
)
```

**核心效益：**

- 宣告 `ImageUrlArtifact` 取代傳統的 `ImageArtifact`，嚴格控制參數記憶體體積——參見 [參數資料承載體積](error_handling.md#parameter-payload-size)
- 一致無縫相容 ImageArtifact、ImageUrlArtifact 與純文字字串
- 內建開箱即用地相容遠端 URL、本機檔案路徑與 Base64 Data URI
- 針對各類輸入格式提供優雅的容錯保護
- 大幅降低複雜工作流程中的型別轉換異常

### 模型對比表格範例 {: #model-comparison-table }

針對支援多種模型的服務，務必提供清晰的橫向對比表：

```markdown
| 模型 | 最大時長 | 品質評等 | 運算速度 | 字元配額上限 |
| :--- | :--- | :--- | :--- | :--- |
| V5 | 4 分鐘 | 極致 | 最快 | 提示詞: 5000, 風格: 1000 |
| V4_5 | 8 分鐘 | 高優 | 快 | 提示詞: 5000, 風格: 1000 |
| V4 | 4 分鐘 | 優良 | 中等 | 提示詞: 3000, 風格: 200 |
```

## 向官方標準程式庫貢獻節點 {: #contributing-to-the-standard-library }

當您欲向核心 `griptape_nodes_library` 貢獻節點（而非發布為獨立外掛程式庫）時，請遵循以下標準貢獻流程：

### 1. 建立功能分支 (Feature Branch) {: #1-create-a-feature-branch }

```bash
cd griptape-nodes
git checkout -b feature/add-color-match-node
```

### 2. 新增節點原始碼檔案 {: #2-add-the-node-file }

將您的節點檔案置於適當的分類子目錄中：

```
libraries/griptape_nodes_library/griptape_nodes_library/
├── image/
│   ├── color_match.py      # 新節點檔案
│   ├── load_image.py
│   └── save_image.py
├── text/
├── audio/
└── ...
```

### 3. 更新 griptape_nodes_library.json {: #3-update-griptape-nodes-library-json }

在 `libraries/griptape_nodes_library/griptape_nodes_library.json` 進行三處同步修改：

#### a. 遞增程式庫版本號 {: #a-increment-the-library-version }

```json
{
  "metadata": {
    "library_version": "0.59.0" // 原為 0.58.0
  }
}
```

#### b. 登錄新增的 pip 依賴套件 {: #b-add-any-new-pip-dependencies }

```json
{
  "metadata": {
    "dependencies": {
      "pip_dependencies": [
        "existing-dep",
        "color-matcher" // 全新相依套件
      ]
    }
  }
}
```

#### c. 新增節點條目登記 {: #c-add-the-node-entry }

```json
{
  "nodes": [
    {
      "class_name": "ColorMatch",
      "file_path": "griptape_nodes_library/image/color_match.py",
      "metadata": {
        "category": "image",
        "description": "Transfer color characteristics from a reference image to a target image",
        "display_name": "Color Match",
        "icon": "palette",
        "group": "edit"
      }
    }
  ]
}
```

### 4. 編寫技術文件 {: #4-add-documentation }

在 `docs/nodes/<category>/<node_name>.md` 建立技術文件頁面：

```markdown
# 色彩匹配 (Color Match)

將參考影像的色彩特徵遷移至目標影像中。

## 功能概述

將參考影像的色調調色盤套用至目標影像...

## 參數說明

### 輸入端 (Inputs)

| 參數名稱 | 型別 | 描述說明 |
| :--- | :--- | :--- |
| reference_image | ImageUrlArtifact | 色彩調色盤來源影像 |
| target_image | ImageUrlArtifact | 欲進行色彩套用的目標影像 |

### 輸出端 (Outputs)

| 參數名稱 | 型別 | 描述說明 |
| :--- | :--- | :--- |
| output_image | ImageUrlArtifact | 完成色彩匹配的結果影像 |

## 使用範例

1. 連接具備理想色調的參考影像
2. 連接欲變換顏色的目標影像
3. 執行節點運算

## 技術細節

底層採用 color-matcher 函式庫進行直方圖色彩映射配對...
```

### 5. 更新 mkdocs.yml 導航結構 {: #5-update-mkdocs-yml-navigation }

在 `mkdocs.yml` 的導航區塊中掛載您的文件頁面：

```yaml
nav:
  - Nodes Reference:
      - Image:
          - Load Image: nodes/image/load_image.md
          - Save Image: nodes/image/save_image.md
          - Color Match: nodes/image/color_match.md # 新增條目
```

### 6. 執行本機程式碼品質檢查 {: #6-run-quality-checks }

在提交 Commit 前，請務必執行格式化與靜態檢查：

```bash
make format        # 自動格式化程式碼
make check/lint    # 檢查程式碼風格
make check/types   # 檢查靜態型別
```

修正所有潛在報錯後再繼續。

### 7. 提交變更並發起 Pull Request {: #7-commit-and-create-pr }

```bash
git add .
git commit -m "feat(image): add ColorMatch node for color transfer"
git push -u origin HEAD
gh pr create --title "Add ColorMatch node" --body "## Summary
- Adds ColorMatch node for transferring colors between images
- Uses color-matcher library
- Includes documentation

## Test plan
- [ ] Load two images
- [ ] Run color match
- [ ] Verify output has reference colors"
```

### 標準核心程式庫 vs 外部獨立程式庫 {: #standard-library-vs-external-library }

| 比較維度 | 標準核心程式庫 (Standard Library) | 外部獨立程式庫 (External Library) |
| :--- | :--- | :--- |
| 存儲倉庫 | `griptape-nodes` 主倉庫 | 獨立第三方 Git 倉庫 |
| 安裝方式 | 隨 Griptape 官方預設內建 | 使用者手動下載安裝 |
| 程式碼審查 | 必須通過官方 PR 審查合併 | 自行發布與維護 |
| 依賴管理 | 登記至核心 `griptape_nodes_library.json` | 獨立維護專屬的 `griptape_nodes_library.json` |
| 版本控制 | 遵循官方核心程式庫版本發布 | 擁有完全獨立的版本發布週期 |
| 說明文件 | 整合收錄於官方技術文件站 | 維護於自有的 README 與 Wiki 中 |

**推薦貢獻至標準程式庫的情境：**

- 該節點對廣大使用者具備廣泛的通用價值
- 無私有化/需付費的強制性外部 API 依賴
- 實現架構穩定成熟、具備充分的測試覆蓋
- 嚴格符合官方的所有程式碼品質規範

**推薦建立外部獨立程式庫的情境：**

- 高度特化或利基型專案需求
- 依賴昂貴或私有的商業 API 金鑰
- 尚處實驗性階段、API 變更頻繁劇烈
- 希望保持自主敏捷的獨立發行節奏
