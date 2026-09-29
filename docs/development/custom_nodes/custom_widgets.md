# 自訂視覺部件 (Custom Widgets) {: #custom-widgets }

節點可透過自訂 JavaScript 部件 (Widgets) 提供超越標準參數控制項的豐富互動式 UI 介面。部件為獨立的 `.js` 模組檔案，負責在容器 DOM 元素內渲染視覺介面，並透過回呼函式將使用者修改的數值即時傳遞回框架。

## 部件架構組成 {: #widget-architecture }

一個完整的自訂部件由以下三個組件共同構成：

1. **部件 JS 檔案** (`widgets/MyWidget.js`) —— 前端 UI 視覺元件
1. **節點 Python 檔案** —— 透過參數上的 `Widget` 特徵標籤 (Trait) 引用該部件
1. **程式庫 JSON 清單** (`griptape_nodes_library.json`) —— 註冊該部件以便框架探索與載入

```
library_name/
├── griptape_nodes_library.json
├── my_node.py
└── widgets/
    └── MyWidget.js
```

## 註冊自訂部件 {: #registering-a-widget }

在 `griptape_nodes_library.json` 頂層新增 `"widgets"` 陣列：

```json
{
  "name": "My Library",
  "widgets": [
    {
      "name": "MyWidget",
      "path": "widgets/MyWidget.js",
      "description": "部件的描述說明"
    }
  ],
  "nodes": [ ... ]
}
```

其中 `name` 必須與 Python 端 `Widget` 特徵標籤中指定的名稱完全一致，且 `library` 引數必須與 JSON 最頂層的 `"name"` 欄位字串嚴格匹配。

## 將部件綁定至參數 {: #attaching-a-widget-to-a-parameter }

使用 `Widget` 特徵標籤將參數與自訂部件綁定。參數的當前值將作為 `props.value` 傳遞至 JavaScript 部件中，數值變更則透過 `props.onChange` 回傳：

```python
from griptape_nodes.exe_types.core_types import Parameter, ParameterMode
from griptape_nodes.traits.widget import Widget

self.add_parameter(
    Parameter(
        name="my_data",
        input_types=["list"],
        type="list",
        output_type="list",
        default_value=[],
        tooltip="由自訂部件管控的資料清單",
        allowed_modes={ParameterMode.PROPERTY, ParameterMode.OUTPUT},
        traits={Widget(name="MyWidget", library="My Library")},
    )
)
```

## 部件 JS 函式簽章 {: #widget-js-function-signature }

部件應以 ES Module 預設匯出 (`default export`) 函式形式提供。該函式接收一個容器 DOM 元素與一個 props 物件，且**必須回傳一個清理函式 (Cleanup Function)**：

```javascript
export default function MyWidget(container, props) {
  const { value, onChange, disabled, height } = props;

  // 在 `container` 容器內部構建您的 UI
  // 當使用者變更資料時調用 `onChange(newValue)`
  // 依據 `disabled` 狀態適時禁用各項互動操作

  // 回傳清理函式
  return () => {
    // 移除全域事件監聽器、釋放相關資源
  };
}
```

**Props 屬性規格：**

| 屬性名稱 | 型別 | 描述說明 |
| :--- | :--- | :--- |
| `value` | `any` | 參數的當前數值（初次載入時對齊 Python 的 default_value） |
| `onChange` | `function` | 將更新後數值發送回引擎與編輯器框架的回呼函式 |
| `disabled` | `boolean` | 標記部件當前是否應處於唯讀 (Read-only) 狀態 |
| `height` | `number` | 建議的面板像素高度（若為 0 或缺省則為自適應） |

## 關鍵設計模式與常見陷阱 {: #critical-patterns-and-pitfalls }

### 謹慎觸發數值變更——切勿在每次按鍵時發送 `onChange` {: #emit-changes-sparingly-not-on-every-keystroke }

調用 `onChange` 會觸發框架的整體狀態更新，從而導致畫布重新渲染並**強行剝奪當前焦點**。對於文字輸入框而言，這意味著使用者每敲擊一個字元，輸入框就會立即失焦，導致完全無法連續打字。這並非自訂部件獨有的限制——Griptape Nodes 編輯器內建的 `TextComponent` 亦嚴格遵循此模式：

- **本機狀態 (Local State)**（您的內部資料陣列、計數器、邊框高亮）在每次 `input` 事件中即時響應更新。
- **`onChange`** 僅在 `blur` 事件觸發時（即使用者點擊外部或按 Tab 鍵移出欄位時）發送一次。
- **離散控制項**（按鈕點擊、微調計數器、拖曳結束）可立即調用 `onChange`，因為它們不依賴持續輸入焦點。

```javascript
// 本機狀態在每次按鍵時即時更新——確保 UI 響應流暢
textarea.addEventListener("input", (e) => {
  localData[index].text = e.target.value;
  // 在此更新字元計數器或邊框高亮樣式
});

// 僅在使用者焦點移開時向框架發布最終數值
textarea.addEventListener("blur", () => {
  localData[index].text = textarea.value;
  onChange(structuredClone(localData));
});

// 離散控制項（按鈕、微調計數器）可立即發布變更
button.addEventListener("pointerdown", (e) => {
  e.stopPropagation();
  localData[index].value++;
  onChange(structuredClone(localData));
  render();
});
```

> **為何無法透過 `requestAnimationFrame` 還原焦點？** 嘗試在每次按鍵時調用 `onChange` 並隨後透過 `requestAnimationFrame` 強制奪回焦點的策略極不可靠——因為框架內部的 React 渲染生命週期是非同步排程的，焦點還原操作必然會與 DOM 節點重新掛載發生競態條件。

### 防止干擾畫布節點的拖曳操作 {: #prevent-node-drag-interference }

節點編輯器畫布會在頂層監聽指標拖曳事件以實現視角平移與節點移動。部件內部的可互動元素必須阻止事件冒泡 (Event Propagation)，並宣告 `nodrag` 與 `nowheel` 樣式類別：

```javascript
// 在最外層包裹元素上宣告
const wrapper = document.createElement("div");
wrapper.className = "my-widget nodrag nowheel";

// 在各個互動子元素（textarea、滑桿等）上阻止冒泡
textarea.addEventListener("pointerdown", (e) => e.stopPropagation());
textarea.addEventListener("mousedown", (e) => e.stopPropagation());
```

### 防止干擾畫布全域快捷鍵 {: #prevent-keyboard-shortcut-interference }

節點編輯器在畫布層級綁定了全域鍵盤快捷鍵——例如按下 `Delete` 鍵會直接刪除當前選取的節點。當使用者在您部件內部的文字框中輸入時，若事件向上冒泡，全域快捷鍵將被意外觸發。請務必在 `keydown` 上調用 `stopPropagation` 以隔離文字輸入行為：

```javascript
textarea.addEventListener("keydown", (e) => e.stopPropagation());
```

這可有效防止使用者在編輯文字時因按下 Delete 鍵而意外刪除節點，並避免複製、貼上或復原等畫布層級快捷鍵干擾正常的文字編輯操作。

### 為文字輸入元素強制覆寫 `user-select: none` {: #override-user-select-none-for-text-inputs }

部件的外層包裹容器通常會設定 `user-select: none` 以避免拖曳時意外選取文字。此樣式會向下繼承至所有子元素並卡死 textarea 的游標選取行為。請務必顯式覆寫該樣式：

```css
textarea {
  user-select: text;
  -webkit-user-select: text;
}
```

### 在發送前深拷貝數值實例 {: #clone-values-before-emitting }

傳遞給 `onChange` 的數值必須是一份嶄新的複本——絕不要直接傳遞內部狀態陣列的參照。否則框架與部件將共享同一個記憶體實例，引發難以排查的連鎖反應與狀態不同步：

```javascript
onChange(localData.map((item) => ({ ...item })));
```

### 妥善清理 Document 層級的事件監聽器 {: #clean-up-document-level-listeners }

若您向 `document` 掛載了事件監聽器（例如用於實現拖放或點擊外部自動收起），必須在回傳的清理函式中予以徹底註銷：

```javascript
document.addEventListener("pointerdown", onDocumentClick, true);

return () => {
  document.removeEventListener("pointerdown", onDocumentClick, true);
};
```

### 為清單項目指派穩定不變的 ID {: #assign-stable-ids-to-list-items }

在開發管理可排序清單的部件時（例如拖曳分鏡編輯器），請為每個項目賦予一個與陣列索引完全解耦的唯一 ID。若缺少穩定 ID，當使用者拖曳重新排序時，由於部件全量重新渲染且元件識別碼綁定於陣列位置，文字框內容等子項目屬性將極易遺失錯位。

```javascript
let nextItemId = 1;

function assignId(item) {
  if (!item.id) {
    item.id = `item-${nextItemId++}`;
  } else {
    const num = parseInt(item.id.replace("item-", ""), 10);
    if (!isNaN(num) && num >= nextItemId) {
      nextItemId = num + 1;
    }
  }
  return item;
}

// 初始化時——保留自存檔資料還原的既有 ID
let items = value.map((v) => assignId({ ...v }));

// 新增項目時
items.push(assignId({ name: "New Item", text: "" }));
```

該 ID 會在多次排序、重新渲染以及經由 `onChange` 往返同步的過程中全程保持穩定。顯示名稱（如 "Shot1", "Shot2"）可依據當前視覺位置重新動態編號，而內部 `id` 則始終保持不變。

### 在 DOM 輔助函式中正確處置 `disabled` 屬性 {: #handle-the-disabled-attribute-correctly-in-dom-helpers }

若您撰寫了透過屬性物件動態建立 DOM 元素的輔助函式，請特別留意 `disabled` 屬性。使用 `setAttribute("disabled", false)` **無法**解除禁用狀態——在 HTML 規範中，只要該屬性存在（無論字串值為何），元素皆會被判定為禁用。請直接指派屬性值：

```javascript
if (key === "disabled") {
  element.disabled = !!val;
}
```

### 在清單末尾呈現拖曳置放指示條 {: #show-drop-indicators-at-end-of-list }

實作拖放重新排序時，拖曳目標指示器（例如藍色邊框線）必須在拖動至清單最後一項下方時也能正確顯現。標準處理策略：當拖曳位置低於所有項目時，在最後一個項目上呈現 `border-bottom`，而非嘗試在不存在的下一個項目上呈現 `border-top`：

```javascript
if (dragOverIndex === items.length && item.index === lastIndex) {
  item.el.style.borderBottom = "2px solid #4a9eff";
} else if (item.index === dragOverIndex) {
  item.el.style.borderTop = "2px solid #4a9eff";
}
```

### 強制約束彙總邊界（最小/最大加總配額） {: #enforce-aggregate-constraints-minmax-totals }

當清單項目的數值總和存在全域範圍限制時（例如總時長必須介於 3–15 秒之間），請在雙向維度上進行約束防禦：

- **上限防護 (Ceiling)**：當總和即將超出最大值時，禁用遞增微調按鈕與「新增項目」按鈕。
- **下限防護 (Floor)**：當減少任何項目的數值會導致總和低於最小值時，禁用遞減微調按鈕。
- **刪除時自動補償**：當刪除單一項目會導致總和跌破下限時，自動增加最後一個留存項目的數值以填補差額。

```javascript
const MIN_TOTAL = 3;
const MAX_TOTAL = 15;

// 當加總即將跌破下限時禁用遞減
const wouldGoBelow = totalValue() - 1 < MIN_TOTAL;
const canDecrease = !disabled && item.value > MIN_VALUE && !wouldGoBelow;

// 刪除項目時——自動補償以維持總和下限
trash.addEventListener("pointerdown", (e) => {
  e.stopPropagation();
  if (items.length <= 1) return;
  items.splice(index, 1);
  const total = totalValue();
  if (total < MIN_TOTAL) {
    const lastItem = items[items.length - 1];
    lastItem.value += MIN_TOTAL - total;
  }
  emitChange();
  render();
});
```

在狀態列中清晰展示上下限區間，例如 `"8s (3–15s)"`，讓使用者直觀理解可用範圍，並在超出合法區間時以紅色醒目警示。

## 實用範例：結構化清單編輯器部件 {: #example-list-based-editor-widget }

最常見的應用情境是管理支援新增、刪除、拖曳重排與行內編輯的結構化項目清單。核心實作要點：

- **穩定 ID**：為每個項目分配全域唯一的 `id`，確保在重新排序與 `onChange` 往返中資料不丟失。
- **拖放重排**：在拖動控制代碼上監聽 `pointerdown`，生成浮動複本提供即時視覺回饋，在 `pointermove` 中計算插入點，並在 `pointerup` 時完成重排。僅在拖放完成後發送一次 `onChange`。
- **微調計數控制項**：針對有界數值（例如時長 1–15 秒），採用 ▲/▼ 微調按鈕取代下拉選單。當數值即將突破約束時禁用按鈕。
- **彙總約束**：強制校驗跨項目的總和上下限。在刪除項目時自動補償數值以維持下限底線。
- **狀態列即時回饋**：在底部呈現小型狀態列標註當前總量與配額上限（例如 `"3 / 6 shots"`, `"8s (3–15s)"`），讓使用者清楚知曉控制項為何被禁用。
- **文字輸入隔離**：在 `pointerdown`、`mousedown` 與 `keydown` 上調用 `stopPropagation` 防止畫布拖曳與全域快捷鍵干擾。僅在 `blur` 時發布 `onChange`。

```python
# Python 端——帶有自訂部件的清單參數
self.add_parameter(
    Parameter(
        name="items",
        input_types=["list"],
        type="list",
        output_type="list",
        default_value=[{"name": "Item1", "duration": 2, "description": ""}],
        allowed_modes={ParameterMode.PROPERTY, ParameterMode.OUTPUT},
        traits={Widget(name="MyListEditor", library="My Library")},
    )
)
```

```javascript
// JS 端——清單編輯器部件骨架範例
export default function MyListEditor(container, props) {
  const { value, onChange, disabled } = props;

  // 穩定 ID 指派函式
  let nextId = 1;
  function assignId(item) {
    if (!item.id) item.id = `item-${nextId++}`;
    return item;
  }

  let items = Array.isArray(value)
    ? value.map((v) => assignId({ ...v }))
    : [assignId({ name: "Item1", duration: 2, description: "" })];

  function emitChange() {
    if (!disabled && onChange) {
      onChange(items.map((item) => ({ ...item })));
    }
  }

  function render() {
    container.innerHTML = "";
    const wrapper = document.createElement("div");
    wrapper.className = "nodrag nowheel";

    items.forEach((item, index) => {
      // ... 構建包含拖曳控制代碼、微調計數器、輸入框與刪除按鈕的資料列 ...

      // 文字輸入——按鍵時更新本機狀態，失焦時發送變更
      textarea.addEventListener("input", (e) => {
        items[index].description = e.target.value;
      });
      textarea.addEventListener("blur", () => {
        items[index].description = textarea.value;
        emitChange();
      });

      // 隔離文字輸入事件，防止干擾畫布
      textarea.addEventListener("pointerdown", (e) => e.stopPropagation());
      textarea.addEventListener("mousedown", (e) => e.stopPropagation());
      textarea.addEventListener("keydown", (e) => e.stopPropagation());
    });

    container.appendChild(wrapper);
  }

  render();

  return () => { /* 清理 Document 層級監聽器 */ };
}
```

## 部件專用測試載台 (Widget Testbed) {: #widget-testbed }

**widget-testbed** 是一個基於 React + Vite 構建的獨立開發測試環境，專供在脫離完整的 Griptape Nodes 主程式之外快速開發與調試自訂部件。它提供了即時熱重載 (Hot-reload) 的輕量開發體驗。

### 核心定位與價值 {: #purpose }

Griptape Nodes 的自訂部件為自主管理 DOM 與狀態的指令式 JavaScript 函式。測試載台使您能夠：

- **飛速原型驗證**：在開發過程中享受毫秒級熱重載
- **完全隔離測試**：無需啟動龐大的 Griptape Nodes 引擎即可打磨 UI/UX 細節
- **精準狀態校驗**：模擬複雜的狀態跳轉與使用者極限互動
- **跨部件無縫切換**：輕鬆切換載入不同部件進行獨立測試
- **直觀除錯排查**：在純粹乾淨的環境下審視 DOM 結構與即時 JSON 狀態

### 適用場景 {: #when-to-use }

以下情境強烈建議使用部件測試載台：

- 從零開始編寫全新的自訂視覺部件
- 調試焦點意外遺失、拖放失效或事件傳遞等隱蔽 BUG
- 驗證複雜狀態流轉與 `onChange` 回呼頻率
- 調整排版樣式與響應式佈局，免受畫布圖層干擾
- 開發管理內部多層巢狀結構的進階部件（多欄表單、分鏡管理器）

### 目錄結構 {: #file-structure }

```
widget-testbed/
├── index.html              # 入口 HTML
├── package.json            # 依賴定義 (React 19, Vite 6)
├── vite.config.js          # Vite 配置檔
├── src/
│   ├── main.jsx            # React 應用程式入口
│   ├── App.jsx             # 帶有除錯控制項的主測試面板
│   └── WidgetHost.jsx      # 負責承載指令式部件的 React 包裝組件
└── node_modules/           # 相依套件
```

### 核心組件說明 {: #key-components }

#### WidgetHost.jsx {: #widgethostjsx }

`WidgetHost` 是一個專門承載指令式部件的 React 包裝組件，其遵循與 Griptape Nodes 完全一致的 `(container, props)` 呼叫約定。它妥善管理部件的掛載、更新與卸載生命週期，並全力防範不必要的元件重新掛載。

**核心能力：**

- **指令式部件相容**：傳遞容器元素與標準 props 驅動您的原生 JS 函式
- **智慧差分重新掛載**：僅當數值由外部變更時（例如點擊 Reset 按鈕）才重新掛載部件
- **區分變更來源**：精準追蹤變更源於部件內部操作還是外部 Props 注入
- **嚴謹清理防護**：在重新掛載或卸載時切實調用清理函式

#### App.jsx {: #appjsx }

主測試面板提供：

- **部件掛載區**：透過 `WidgetHost` 載入並渲染目標部件
- **狀態切換控制**：一鍵切換 Disabled 唯讀模式
- **即時 JSON 檢視器**：即時呈現部件輸出資料的 JSON 結構
- **重設功能**：一鍵將部件還原至初始狀態
- **暗色主題風格**：完美貼合 Griptape Nodes 的視覺美學

### 啟動測試流程 {: #testing-a-widget }

**1. 安裝相依套件：**

```bash
cd widget-testbed
npm install
```

**2. 編輯 App.jsx 匯入您的部件：**

```javascript
import MyWidget from "../../my-library/widgets/MyWidget.js";

const INITIAL_VALUE = { /* 您的初始測試資料 */ };

export default function App() {
  const [value, setValue] = useState(INITIAL_VALUE);
  const [disabled, setDisabled] = useState(false);
  const [showDebug, setShowDebug] = useState(true);

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 24 }}>
      <h1 style={{ fontSize: 18, fontWeight: 600, color: "#eee" }}>
        MyWidget 測試載台
      </h1>

      <div style={{ display: "flex", gap: 12, alignItems: "center" }}>
        <label style={{ display: "flex", alignItems: "center", gap: 6, fontSize: 13 }}>
          <input
            type="checkbox"
            checked={disabled}
            onChange={(e) => setDisabled(e.target.checked)}
          />
          Disabled 唯讀模式
        </label>
        <label style={{ display: "flex", alignItems: "center", gap: 6, fontSize: 13 }}>
          <input
            type="checkbox"
            checked={showDebug}
            onChange={(e) => setShowDebug(e.target.checked)}
          />
          呈現 JSON 狀態
        </label>
        <button
          onClick={() => setValue(INITIAL_VALUE)}
          style={{
            padding: "4px 12px",
            fontSize: 12,
            background: "#333",
            border: "1px solid #555",
            borderRadius: 4,
            color: "#ccc",
            cursor: "pointer",
          }}
        >
          重設 (Reset)
        </button>
      </div>

      <div
        style={{
          border: "1px solid #333",
          borderRadius: 8,
          overflow: "hidden",
        }}
      >
        <WidgetHost
          widgetFn={MyWidget}
          value={value}
          onChange={setValue}
          disabled={disabled}
        />
      </div>

      {showDebug && (
        <pre
          style={{
            background: "#1a1a1a",
            border: "1px solid #333",
            borderRadius: 8,
            padding: 12,
            fontSize: 11,
            color: "#8c8",
            overflow: "auto",
            maxHeight: 300,
          }}
        >
          {JSON.stringify(value, null, 2)}
        </pre>
      )}
    </div>
  );
}
```

**3. 啟動本機開發伺服器：**

```bash
npm run dev
```

**4. 在瀏覽器中開啟：**

連線至 `http://localhost:5173` 進行即時調試。
