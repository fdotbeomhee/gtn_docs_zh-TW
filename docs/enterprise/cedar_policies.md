# Cedar 政策參考手冊 (Cedar Policies) {: #cedar-policies }

Griptape Nodes 權限範本採用 Amazon 開源的授權語言 [Cedar](https://www.cedarpolicy.com/)。在 [權限編輯器 (Permission Editor)](admin_dashboard.md#permission-editor) 中，**可視化權限建構器 (Permission Builder)** 會自動將您的設定選項轉換為 Cedar 語句。當您需要編寫 **Raw Cedar（原始 Cedar）** 範本或審查建構器產生的 Cedar 程式碼時，請參閱本頁面。

僅當建構器的功能能力目錄無法表達您所需的細部規則時，才選擇使用 Raw Cedar。

## 政策輸入參數 (Policy inputs) {: #policy-inputs }

Cedar 依據四個輸入要素做出每項授權決策：`(principal, action, resource, context)`。Griptape Nodes 將各授權檢查點與其授權資料映射至以下 Cedar 輸入參數：

| 構成部分 (Part) | 所包含之內容 | 用途指引 |
| :--- | :--- | :--- |
| `principal` | 固定的佔位符號，`User::"<anonymous>"`。 | 無特定用途。保持無約束條件即可。 |
| `action` | 授權檢查點，例如 `Action::"LoadLibrary"`。 | 指定規章所涵蓋的具體操作名稱。 |
| `resource` | 檢查點所解析出的程式庫、節點型別、專案、模型或轉碼器。 | 比對特定資源項目或該資源的相關屬性事實。 |
| `context` | 活動專案、引擎、程式庫與授權資料等情境事實。 | 將規章限定於特定專案範疇或特定授權類型。 |

## 決策組合規則 (Combining decisions) {: #combining-decisions }

Cedar 在組合多個範本時遵循兩大核心規則：

1. 必須至少有一個相符的 `permit` 允許該操作。
1. 相符的 `forbid` 會覆寫所有 `permit`，包含來自其他範本的 permit。

視覺化建構器產生下列標準 Cedar 範式：

- **Exploration (Allow all)**：以 `permit(principal, action, resource);` 起手，接著針對例外情況新增 `forbid` 語句。
- **Production (Deny All)**：省略通配允許語句，僅針對獲准的操作逐一新增 `permit` 語句。

!!! warning "預設拒絕與硬性阻擋 (Default deny and hard blocks)"

    Cedar 預設會拒絕操作，除非至少有一條語句允許 (permit) 它。因此，僅包含 `forbid` 語句的政策集合會直接拒絕所有操作，而不僅僅是符合該語句的操作。若要建立黑名單封鎖清單，必須將 forbid 與通配 permit 搭配使用：

    ```cedar
    permit(principal, action, resource);

    forbid(principal, action, resource)
    when { resource has lifecycle_stage && resource.lifecycle_stage == "LABS" };
    ```

    省略 permit 並非硬性阻擋。來自其他範本的 permit 仍可能允許該操作。當您希望任何 permit 皆無法覆寫拒絕決策時，請使用 `forbid`。Cedar 會將附加至某個授權金鑰的所有範本評估為單一政策集合，因此 permit 與 forbid 可以分別存在於不同的範本中。

## 授權檢查點 (Checkpoints) {: #checkpoints }

下方列出的各個動作標識了引擎所管轄的受控操作。拒絕時的表現形式取決於該檢查點：

| 動作 (Action) | 觸發時機 | 資源 (Resource) | 拒絕時的表現形式 |
| :--- | :--- | :--- | :--- |
| `Action::"LoadLibrary"` | 程式庫載入超越其中繼資料階段時。 | `Library` | 該程式庫被標記為不可用，其錯誤圖示上會註明具體原因。 |
| `Action::"InstantiateNode"` | 建立節點實例時。 | `NodeType` | 畫布上會放置 Error Proxy 節點以取代真實節點。被拒絕的節點型別亦會預先列在程式庫面板中。 |
| `Action::"LoadProject"` | 讀取專案範本時。 | `Project` | 該專案載入失敗。 |
| `Action::"ActivateProject"` | 某個專案欲成為當前活動專案時。 | `Project` | 切換失敗並維持在當前既有專案。 |
| `Action::"OfferModel"` | 構建模型選擇器選單時。 | `Model` | 該模型會從模型選擇器中被過濾隱藏。 |
| `Action::"InvokeModel"` | 節點呼叫模型時。 | `Model` | 模型調用執行失敗。 |
| `Action::"ReadVideoCodec"` | 即將讀取視訊影片，以及構建相關選擇器時。 | `VideoCodec` | 讀取被拒絕，或該轉碼器被過濾隱藏。 |
| `Action::"WriteVideoCodec"` | 即將寫入輸出視訊影片，以及構建相關選擇器時。 | `VideoCodec` | 寫入被拒絕，或該轉碼器被過濾隱藏。 |

## 資源屬性 (Resource attributes) {: #resource-attributes }

除非表格另有註明，否則所有資源屬性皆為選填。在讀取選填屬性前請務必使用 `has`。具體防禦範式請參閱 [防禦選填屬性缺失](#guard-optional-attributes)。

### `Library` {: #library }

| 屬性 (Attribute) | 型別 (Type) | 呈現時機 |
| :--- | :--- | :--- |
| `id` | string | 始終存在。程式庫名稱。 |
| `lifecycle_stage` | string | 當程式庫有宣告生命週期階段時。 |

### `NodeType` {: #nodetype }

| 屬性 (Attribute) | 型別 (Type) | 呈現時機 |
| :--- | :--- | :--- |
| `id` | string | 始終存在。節點型別名稱。 |
| `executes_arbitrary_code` | bool | 始終存在。當節點的程式庫宣告標註為執行任意 Python 程式碼時為 `true`。 |
| `lifecycle_stage` | string | 當節點有宣告階段，或自其所屬程式庫繼承階段時。 |
| `model_ids` | set<string> | 當節點有宣告模型使用時。 |
| `provider_ids` | set<string> | 當節點有宣告模型或提供者使用時。 |
| `model_families` | set<string> | 當節點宣告的模型解析為程式庫模型目錄中的家族時。 |

### `Project` {: #project }

| 屬性 (Attribute) | 型別 (Type) | 呈現時機 |
| :--- | :--- | :--- |
| `id` | string | 始終存在。專案的不透明 ID，在編輯器中建立的專案為 GUID。 |
| `name` | string | 當範本載入進度足以得知其名稱時。 |

對於人類易讀的規章請使用 `name`；當必須精確比對時請使用 `id`。在 `permit` 中建議優先使用 `id`。專案名稱僅在範本完全載入後方可使用，若 permit 未能精確匹配將導致存取被拒。

### `Model` {: #model }

| 屬性 (Attribute) | 型別 (Type) | 呈現時機 |
| :--- | :--- | :--- |
| `id` | string | 始終存在。目錄中穩定的模型鍵值。 |
| `provider_id` | string | 當該鍵值於模型目錄中成功解析時。 |
| `model_families` | set<string> | 當解析出的模型宣告有模型家族時（包含該單一家族名稱）。 |
| 若欲比對整個提供者而非單一模型，請參閱 [實體階層](#entity-hierarchy)。 | | |

### `VideoCodec` {: #videocodec }

| 屬性 (Attribute) | 型別 (Type) | 呈現時機 |
| :--- | :--- | :--- |
| `id` | string | 始終存在。探測出的轉碼器名稱，例如 `h264`、`hevc`。 |
| `container_format` | string | 當容器格式已知時，例如 `mp4`、`mov`。 |

## 實體階層 (Entity hierarchy) {: #entity-hierarchy }

### `ModelProvider` {: #modelprovider }

每個 `Model` 皆隸屬於提供它的提供者之下，因此使用 `in` 關鍵字可涵蓋該提供者旗下的所有模型，無需逐一列舉模型名稱。

範例：

```cedar
forbid(principal, action, resource)
unless { resource in ModelProvider::"anthropic" };
```

提供者 ID 即為程式庫 [`model_catalog`](../development/custom_nodes/authoring_libraries.md#model-catalog) 中的提供者鍵值，例如 `anthropic` 或 `ollama`。

## 情境事實 (Context facts) {: #context-facts }

!!! warning "防禦情境事實缺失 (Guard context facts)"

    所有的情境事實皆為選填。Griptape Nodes 僅會包含它能夠成功解析的事實並省略其餘項目，因此在讀取每個事實前請務必使用 `has` 進行防禦檢查。

    先進行記錄防禦檢查，接著再讀取其內部屬性：

    ```cedar
    when {
      context has loaded_libraries &&
      context.loaded_libraries.names.contains("My Library")
    }
    ```

| 事實 (Fact) | 型別 (Type) | 備註說明 |
| :--- | :--- | :--- |
| `active_project.id` | string | 作為引擎註冊表鍵值的不透明專案 ID。 |
| `active_project.name` | string | 其顯示名稱（當範本已載入時）。需要獨立的 `has` 防禦檢查。 |
| `engine.id` | string | 活動引擎的 ID。 |
| `loaded_libraries.names` | set<string> | 截至目前已載入的程式庫名稱集合。 |
| `license_id` | string | 授權金鑰的 ID。 |
| `org_id` | string | 授權所屬的組織 ID。 |
| `license_type` | string | `"headless"` 或 `"interactive"`。 |

## 專案範疇 (Project scope) {: #project-scope }

視覺化建構器會利用連結的專案，為專案範疇範本中的每條語句自動附加此條件：

```cedar
when { context has active_project && context.active_project.id == "<project id>" }
```

Griptape Nodes 會針對活動專案祖先鏈中的每個專案逐一評估政策。這帶來了兩項關鍵影響：

**必須包含祖先專案**：當專案繼承自父專案時，政策會針對鏈中的每個專案依序執行：首先是活動專案，接著是各個祖先專案。每次執行都必須允許該操作。因此，限定於父專案的 `forbid` 會直接封鎖其子專案，且專案白名單必須包含該鏈中的所有專案。

例如，假設 **Shot 42** 繼承自 **Studio Defaults**。若要在 Shot 42 中允許載入程式庫，必須同時為這兩個專案 ID 提供 permit 語句：

```cedar
permit(principal, action == Action::"LoadLibrary", resource)
when {
  context has active_project &&
  context.active_project.id == "<Shot 42 id>"
};

permit(principal, action == Action::"LoadLibrary", resource)
when {
  context has active_project &&
  context.active_project.id == "<Studio Defaults id>"
};
```

若僅有第一條 permit，則 Shot 42 檢查會通過，但 Studio Defaults 檢查會被拒絕。同樣地，比對 Studio Defaults 的 `forbid` 亦會在 Shot 42 中直接拒絕該操作。

**納入預設專案的考量**：除非另行設定，否則引擎預設開機進入 [預設專案 (Default Project)](../guides/projects/projects.md#the-system-defaults)，其 ID 為 `<system-defaults>`。您不需要明確為該 ID 撰寫 permit 語句，但明確的 `forbid` 規則依然適用。

## 拒絕註解 (Denial annotations) {: #denial-annotations }

Cedar 會標識拒絕操作的具體規章，但無法向使用者解釋缺少了什麼。Griptape Nodes 採用三項註解來補充該資訊。這三項皆為 Griptape 約定，Cedar 在政策評估期間會直接忽略它們。

| 註解 (Annotation) | 用途 |
| :--- | :--- |
| `@id("<slug>")` | 規章的穩定識別碼。在拒絕訊息中顯示以取代位置備援字串。 |
| `@capability("<name>")` | 使用者所欠缺的功能能力名稱，例如 `arbitrary-code-execution`。 |
| `@advice("<text>")` | 具體處置建議。這是使用者所看到的修復說明文字。 |

```cedar
@id("nodes/no-arbitrary-code")
@capability("arbitrary-code-execution")
@advice("Nodes that run arbitrary code are not available on this license. Ask your studio admin to enable them.")
forbid(principal, action == Action::"InstantiateNode", resource)
when { resource has executes_arbitrary_code && resource.executes_arbitrary_code };
```

建議為您撰寫的每條 `forbid` 附加這些註解。缺少 `@advice` 時，使用者僅能看到政策 ID，而無法獲得解決拒絕問題的明確指引。

`@capability` 的數值為自由格式，Cedar 不會對其進行驗證。建議沿用視覺化建構器所產生的標準名稱，以保持各範本間拒絕資訊的一致性：

- `library`
- `library-lifecycle`
- `node-lifecycle`
- `arbitrary-code-execution`
- `project`
- `model`
- `model-provider`
- `model-family`
- `video-codec`

## 完整實作範例 (Examples) {: #examples }

以下各範例皆為完整的 Raw Cedar 政策。

### 阻擋執行任意程式碼的節點 {: #block-arbitrary-code-nodes }

此範例包含通配 permit，構成一個完整的黑名單封鎖範本：

```cedar
permit(principal, action, resource);

@id("nodes/no-arbitrary-code")
@capability("arbitrary-code-execution")
@advice("Nodes that run arbitrary code are not available on this license.")
forbid(principal, action == Action::"InstantiateNode", resource)
when { resource has executes_arbitrary_code && resource.executes_arbitrary_code };
```

### 阻擋不穩定的實驗性程式庫與節點 {: #block-unstable-libraries-and-nodes }

建議為程式庫與節點使用獨立的語句，如此每筆拒絕訊息皆能精準指出使用者欠缺的能力名稱。

```cedar
permit(principal, action, resource);

@id("lifecycle/no-experimental-libraries")
@capability("library-lifecycle")
@advice("Experimental libraries are blocked on this license. Ask your studio admin which libraries are approved.")
forbid(principal, action == Action::"LoadLibrary", resource)
when {
  resource has lifecycle_stage &&
  (resource.lifecycle_stage == "LABS" || resource.lifecycle_stage == "ALPHA")
};

@id("lifecycle/no-experimental-nodes")
@capability("node-lifecycle")
@advice("Experimental nodes are blocked on this license. Use a STABLE or BETA alternative.")
forbid(principal, action == Action::"InstantiateNode", resource)
when {
  resource has lifecycle_stage &&
  (resource.lifecycle_stage == "LABS" || resource.lifecycle_stage == "ALPHA")
};
```

### 僅允許兩家特定的模型提供者 {: #allow-two-model-providers }

`unless` 子句反轉了匹配邏輯，阻擋下方指名兩家以外的所有提供者。當資源不存在相符祖先時，`in` 運算子會回傳 false 而非報錯，因此不需要 `has` 防禦。

限制模型時請務必指名這兩個模型檢查點。`OfferModel` 會從選擇器中移除被拒絕的模型；`InvokeModel` 則封鎖已綁定這些模型的既有節點運算。

```cedar
permit(principal, action, resource);

@id("models/approved-providers")
@capability("model-provider")
@advice("Only Anthropic and OpenAI models are approved on this license.")
forbid(principal, action in [Action::"OfferModel", Action::"InvokeModel"], resource)
unless { resource in ModelProvider::"anthropic" || resource in ModelProvider::"openai" };
```

### 阻擋特定的模型家族 {: #block-a-model-family }

由於 `model_families` 是集合 (Set)，請使用 `contains` 來匹配數值：

```cedar
permit(principal, action, resource);

@id("models/no-claude-3")
@capability("model-family")
@advice("Claude 3 models are retired on this license. Use Claude 4.")
forbid(principal, action in [Action::"OfferModel", Action::"InvokeModel"], resource)
when { resource has model_families && resource.model_families.contains("Claude 3") };
```

### 僅允許特定專案 {: #allow-specific-projects }

引擎分別對專案載入與專案啟用進行權限控管，因此規章需同時指名這兩個檢查點。

```cedar
permit(principal, action, resource);

@id("projects/approved")
@capability("project")
@advice("This project is not on your license. Ask your studio admin for access.")
forbid(principal, action in [Action::"LoadProject", Action::"ActivateProject"], resource)
unless {
  resource has id &&
  (resource.id == "8f2c1a04-9d3e-4b7a-9f10-2c5d6e8a1b33" || resource.id == "c07b5e91-4a2d-4f88-bd63-1e9f7a205c48")
};
```

### 限制視訊轉碼器寫入 {: #restrict-codec-writes }

此語句僅限制寫入檢查點。

```cedar
permit(principal, action, resource);

@id("video/no-prores-writes")
@capability("video-codec")
@advice("Writing ProRes is not permitted on this license. Render to H.264 instead.")
forbid(principal, action == Action::"WriteVideoCodec", resource)
when { resource has id && resource.id == "prores" };
```

### 將規章限定於單一專案範疇 {: #scope-a-rule-to-one-project }

Cedar 透過 AND 組合多個 `when` 子句，因此您可以將專案範疇與資源條件分別獨立編寫：

```cedar
permit(principal, action, resource);

@id("show-a/no-experimental-nodes")
@capability("node-lifecycle")
@advice("Show A is locked to stable nodes.")
forbid(principal, action == Action::"InstantiateNode", resource)
when {
  context has active_project &&
  context.active_project has name &&
  context.active_project.name == "Show A"
}
when { resource has lifecycle_stage && resource.lifecycle_stage == "LABS" };
```

### 限制 Headless 無周邊自動化授權 {: #restrict-headless-licenses }

```cedar
permit(principal, action, resource);

@id("headless/no-model-invocation")
@capability("model")
@advice("Headless licenses cannot call models on this plan.")
forbid(principal, action == Action::"InvokeModel", resource)
when { context has license_type && context.license_type == "headless" };
```

## 常見陷阱 (Gotchas) {: #gotchas }

### 防禦選填屬性缺失 {: #guard-optional-attributes }

讀取不存在的屬性會產生執行錯誤。Cedar 會將包含錯誤的條件視為未滿足，而引擎則會拒絕任何無法乾淨評估的操作。因此，未受保護的讀取可能會使 `forbid` 漏掉未宣告該屬性的資源，並封鎖不具備該事實的有效合法操作。

```cedar
// 錯誤示範。將直接拒絕所有未宣告生命週期階段的節點型別。
forbid(principal, action == Action::"InstantiateNode", resource)
when { resource.lifecycle_stage == "LABS" };

// 正確示範。
forbid(principal, action == Action::"InstantiateNode", resource)
when { resource has lifecycle_stage && resource.lifecycle_stage == "LABS" };
```

由於 `&&` 具備短路求值特性，請務必將 `has` 防禦條件置於最前方。

對於巢狀結構的事實，請在每一層皆進行防禦。`context has active_project` 並無法保證活動專案一定擁有 `name` 屬性：

```cedar
when {
  context has active_project &&
  context.active_project has name &&
  context.active_project.name == "Show A"
}
```

### 檢查名稱拼寫是否有筆誤 {: #check-names-for-typos }

引擎不會針對結構綱要預先驗證 Cedar 政策。諸如 `resource.lifecycle_stge` 或 `Action::"LoadLibrry"` 等筆誤雖然能成功通過語法剖析，但永遠無法匹配任何資源，導致規章看似完全無效。

### 務必允許所有必要的檢查點 {: #permit-every-required-checkpoint }

若 Production 範本僅允許了 `LoadLibrary`，則程式庫載入可以運作，但預設拒絕機制會阻擋其他受控檢查點的操作。請務必指名工作流程所需的所有操作，或附加另一個允許這些操作的範本。

```cedar
// 僅允許程式庫載入操作。
permit(principal, action == Action::"LoadLibrary", resource);

// 廣泛允許涵蓋所有現行檢查點。
permit(principal, action in [
  Action::"LoadLibrary",
  Action::"InstantiateNode",
  Action::"LoadProject",
  Action::"ActivateProject",
  Action::"OfferModel",
  Action::"InvokeModel",
  Action::"ReadVideoCodec",
  Action::"WriteVideoCodec"
], resource);
```
