# Maya MCP 伺服器 (Maya MCP Server) {: #maya-mcp-server }

**[Maya MCP 伺服器](https://github.com/PatrickPalmer/MayaMCP)** 讓 AI Agent 能依據 Model Context Protocol (MCP) 標準，透過自然語言提示詞直接控制與操作 Autodesk Maya。這項整合支援對話式輔助 3D 建模、場景搭建、材質賦予與階層管理。本專案為開發者 [Patrick Palmer](https://github.com/PatrickPalmer) 創建之開源第三方伺服器，非由 Autodesk 或 Griptape 官方直接維護。

!!! warning "第三方開源專案提示 (Third-Party Server)"

    本 MCP 伺服器未由 Griptape 或 Autodesk Maya 官方正式背書。請自行評估使用風險，並請於操作前務必妥善備份個人專案檔案。

!!! note "事前準備 (Prerequisites)"

    在使用 Maya MCP 伺服器前，您必須具備：

    1. 安裝 [Autodesk Maya](https://www.autodesk.com/products/maya) 2023 或更新版本
    1. 安裝 Python 3.10 或更新版本
    1. 自 [專案存放庫](https://github.com/PatrickPalmer/MayaMCP) 下載並配置 Maya MCP 伺服器（請遵循下述[安裝步驟](#1-download-and-setup-maya-mcp-server)）。

## 安裝步驟指引 {: #installation }

### 1. 下載並配置 Maya MCP 伺服器 {: #1-download-and-setup-maya-mcp-server }

1. 開啟終端機並切換至欲存放專案的目錄。建議將伺服器置於容易檢索的路徑（例如 macOS/Linux 下的 `$HOME/Documents/GitHub`）：

    ```bash
    cd $HOME/Documents/GitHub
    ```

1. **複製存放庫 (Clone)**：

    ```bash
    git clone https://github.com/PatrickPalmer/MayaMCP.git
    cd MayaMCP
    ```

1. **建立獨立 Python 虛擬環境**：

    ```bash
    python -m venv .venv
    ```

1. **啟用虛擬環境**：

    - **Windows**：`.venv\Scripts\activate.bat`
    - **macOS / Linux**：`source .venv/bin/activate`

1. **安裝依賴套件**：

    ```bash
    pip install -r requirements.txt
    ```

### 2. 開啟 Maya 並啟用指令連接埠 (Command Port) {: #2-open-maya-and-enable-command-port }

1. **開啟 Autodesk Maya 應用程式**

1. **啟用 Command Port** — 這是 MCP 伺服器與 Maya 核心通訊的必備管道。在 Maya 的 Script Editor (腳本編輯器) 中執行以下 Python 代碼：

    ```python
    import maya.cmds as cmds


    def setup_maya_command_port(port=50007):
        """設定 Maya 指令連接埠並附帶例外處理"""
        try:
            # 首先嘗試關閉該埠既有的殘留連線
            try:
                cmds.commandPort(name=f"localhost:{port}", close=True)
                print(f"Closed existing command port on localhost:{port}")
            except:
                # 無既有連接埠需要關閉，正常略過
                pass

            # 正式啟用命令通訊埠
            cmds.commandPort(name=f"localhost:{port}")
            print(f"Command Port successfully enabled on localhost:{port}")
            return True

        except Exception as e:
            print(f"Error setting up command port: {e}")
            return False


    # 執行設定
    if setup_maya_command_port(50007):
        print("Maya MCP server should now be able to connect!")
    else:
        print("Failed to setup command port. Check Maya's Command Port settings in Preferences.")
    ```

!!! warning "每次 Maya 啟動時皆須啟用"

    Command Port 設定在不同 Maya 工作階段之間不會自動保留。每次重新啟動 Maya 時，皆需重新啟用該連接埠。

!!! tip "簡化操作：儲存為 Maya 自動腳本"

    為免除手動重複執行的繁瑣，可將指令連接埠設定保存為 Maya 腳本：

    1. **將以下內容保存為 `enable_mcp_command_port.py`**：

        ```python
        import maya.cmds as cmds


        def enable_mcp_command_port(port=50007):
            """設定 Maya 指令連接埠並附帶例外處理"""
            try:
                try:
                    cmds.commandPort(name=f"localhost:{port}", close=True)
                    print(f"Closed existing command port on localhost:{port}")
                except:
                    pass

                cmds.commandPort(name=f"localhost:{port}")
                print(f"Command Port successfully enabled on localhost:{port}")
                return True

            except Exception as e:
                print(f"Error setting up command port: {e}")
                return False
        ```

    1. **在 Maya 的 Script Editor 中測試腳本**：

        ```python
        import enable_mcp_command_port

        enable_mcp_command_port.enable_mcp_command_port()
        ```

    1. **選擇以下其中一種自動化方案**：

        **選項 A：建立 Shelf 工具列快捷按鈕**

        - 將步驟 2 的測試代碼拖曳至 Maya 上方 Shelf 工具列建立快捷按鈕
        - 需要使用時只需點擊該圖示按鈕即可瞬間啟用

        **選項 B：透過 userSetup.py 開機自動載入**

        - 尋找 Maya 的 userScripts 使用者目錄：
            - **Windows**：`%USERPROFILE%\Documents\maya\2025\scripts\`
            - **macOS**：`~/Library/Preferences/Autodesk/maya/2025/scripts/`
            - **Linux**：`~/maya/2025/scripts/`
        - 在現有的 `userSetup.py`（若無則直接新增該檔案）中加入以下行：
            ```python
            import enable_mcp_command_port

            enable_mcp_command_port.enable_mcp_command_port()
            ```
        - 重新啟動 Maya 後，指令連接埠即會在啟動時自動常駐就緒

!!! tip "首次存取授權"

    Maya MCP 伺服器透過 Command Port 與 Maya 內部交換訊號。當 MCP 伺服器首次嘗試發送指令時，Maya 視窗內部可能會彈出權限確認詢問框。請點擊 **"Allow All"** 允許後續所有通訊。

### 3. 在 Griptape Nodes 中配置連線 {: #3-configure-griptape-nodes }

1. **開啟 Griptape Nodes** 並前往 **Settings** → **MCP Servers**

1. **點擊 + New MCP Server**

1. **配置伺服器連線參數**：

    - **Server Name/ID (伺服器名稱/識別碼)**：`maya`
    - **Connection Type (連線類型)**：`Local Process (stdio)`
    - **Configuration JSON (組態 JSON)**（根據您的作業系統選用）：

    **Windows 環境**：

    ```json
    {
      "transport": "stdio",
      "command": "C:\\path\\to\\MayaMCP\\.venv\\Scripts\\python.exe",
      "args": [
        "C:\\path\\to\\MayaMCP\\src\\maya_mcp_server.py"
      ],
      "cwd": null,
      "encoding": "utf-8",
      "encoding_error_handler": "strict"
    }
    ```

    **macOS / Linux 環境**：

    ```json
    {
      "transport": "stdio",
      "command": "/path/to/MayaMCP/.venv/bin/python",
      "args": [
        "/path/to/MayaMCP/src/maya_mcp_server.py"
      ],
      "cwd": null,
      "encoding": "utf-8",
      "encoding_error_handler": "strict"
    }
    ```

    !!! warning "絕對路徑配置注意事項"

        請務必將範例中的路徑字串替換為您電腦中 MayaMCP 專案目錄的真實絕對路徑。在 Windows 上需使用雙反斜線轉義 (`\\`)，在 macOS/Linux 上請使用常規正斜線 (`/`)。

1. **點擊 Create Server** 完成建立

## 內建可用工具一覽 {: #available-tools }

### 基礎場景操作工具 {: #basic-tools }

- **`list_objects_by_type`** — 列舉場景中的物件清單，支援針對攝影機 (Cameras)、燈光 (Lights)、材質 (Materials) 或幾何形狀 (Shapes) 進行精確篩選
- **`create_object`** — 建立基本幾何體（立方體、圓錐、球體、圓柱、攝影機、各類燈光等）
- **`get_object_attributes`** — 取得 Maya 指定物件的所有屬性清單與目前數值
- **`set_object_attribute`** — 修改物件的特定屬性值（位移、旋轉、縮放等）
- **`scene_new`** — 在 Maya 中建立全新的空白場景
- **`scene_open`** — 自磁碟開啟載入既有的 Maya 場景檔案
- **`scene_save`** — 儲存當前場景
- **`select_object`** — 在場景視圖中選取指定物件
- **`clear_selection_list`** — 清空當前選取物件清單
- **`viewport_focus`** — 將 3D 視角中心對齊並最適化聚焦於目標物件 (Frame Selected)

### 進階建模與材質工具 {: #advanced-modeling-tools }

- **`create_advanced_model`** — 依詳細參數建立複合 3D 模型結構（汽車、樹木、建築物、杯子、椅子等）
- **`mesh_operations`** — 執行多邊形網格編輯操作（擠出 Extrude、倒角 Bevel、細分 Subdivide、布林運算 Boolean、合併 Combine、橋接 Bridge、分割 Split）
- **`create_material`** — 建立並指派各類材質著色器（Lambert、Phong、木紋 Wood、大理石 Marble、鍍鉻 Chrome、玻璃 Glass 等）
- **`create_curve`** — 產生 NURBS 曲線路徑（直線、圓形、螺旋 Spiral、星形 Star、齒輪 Gear 等）
- **`curve_modeling`** — 依曲線執行幾何形體生成（放樣 Loft、旋轉成形 Revolve、掃掠 Sweep 等）
- **`organize_objects`** — 透過群組 (Group)、父子階層綁定 (Parenting)、版面配置、對齊 (Align) 與等距分佈 (Distribute) 組織場景元件

## 進階配置選項 {: #configuration-options }

您可以透過調整組態微調通訊參數與工作目錄：

```json
{
  "transport": "stdio",
  "command": "/path/to/MayaMCP/.venv/bin/python",
  "args": [
    "/path/to/MayaMCP/src/maya_mcp_server.py"
  ],
  "cwd": "/path/to/MayaMCP",
  "encoding": "utf-8",
  "encoding_error_handler": "strict"
}
```

## 實用提示詞範例 {: #example-use-cases }

- 「建立一台擁有 4 個輪子、具備流線型跑車外觀的簡易汽車模型」
- 「搭建一棵擁有 3 根主分枝、樹葉茂密的大樹」
- 「建立一棟帶有窗戶的建築物並為外牆賦予紅磚材質」
- 「建立一個馬克杯並為其表面套用高光鍍鉻材質」
- 「建立一張椅子並將其擺放於場景中心地面」
- 「產生一條螺旋曲線並將其擠出形成彈簧結構」
- 「建立一個齒輪形狀的輪廓曲線以進行機械建模」
- 「將場景中的所有家具物件統一編組為一個群組」
- 「將所有模型物件的中心點對齊至世界座標原點 (0,0,0)」
- 「建立新場景並將其命名儲存為 'my_project.ma'」

## 進階功能剖析 {: #advanced-features }

### 客製化進階模型生成 {: #custom-model-creation }

`create_advanced_model` 工具支援透過專屬結構化參數客製化多種模型原型：

**汽車模型原型參數範例**：

```json
{
  "model_type": "car",
  "parameters": {
    "wheels": 4,
    "sporty": true,
    "convertible": false
  }
}
```

**樹木模型原型參數範例**：

```json
{
  "model_type": "tree",
  "parameters": {
    "branches": 3,
    "leaf_density": 0.8,
    "type": "pine"
  }
}
```

### 材質著色器客製化 {: #material-creation }

快速建立具備豐富物理外觀屬性的多樣化材質：

**鍍鉻金屬材質**：

```json
{
  "material_type": "chrome",
  "color": [0.8, 0.8, 0.8],
  "parameters": {
    "reflectivity": 0.9
  }
}
```

**木質紋理材質**：

```json
{
  "material_type": "wood",
  "color": [0.6, 0.4, 0.2],
  "parameters": {
    "veinSpread": 0.5,
    "veinColor": [0.3, 0.2, 0.1]
  }
}
```

## 疑難排解 {: #troubleshooting }

### 常見問題 {: #common-issues }

- **連線異常**：確認 Maya 處於執行中狀態且 Command Port 已確實開啟；確認 Griptape Nodes 設定中的 Python 直譯器精確指向已安裝依賴的 `.venv`
- **權限遭拒 (Permission Denied)**：當首次連線時，請於 Maya 內部彈出的安全性確認對話框中點擊 **"Allow All"**
- **路徑錯誤**：確保在組態 JSON 中一律使用真實絕對路徑，且斜線格式符合作業系統規範
- **Python 版本不合**：伺服器依賴 Python 3.10 或更新版本

### 除錯小技巧 {: #debug-tips }

1. 優先使用基礎查詢進行連線測試（例如：「列舉場景中所有物件」）
1. 確認虛擬環境套件已完整安裝 (`pip install -r requirements.txt`)
1. 遇到持續性逾時，嘗試同時重啟 Maya 與 MCP 伺服器

### Maya 指令連接埠運作機制 {: #maya-command-port }

Maya MCP 伺服器採用 Maya 原生內建的 Command Port 進行資料傳輸。這代表著：

- 無需編譯或安裝複雜的 C++ Maya 外掛二進位檔
- 透過 MEL 與內部 Python 腳本橋樑進行訊號通訊
- 生成的 Python 指令碼直接在 Maya 內部的 Python 直譯環境中安全運算
- 運算產出透過指令埠串流回傳給 Griptape Nodes

## 安全性考量 {: #security-considerations }

!!! danger "動態代碼執行風險 (Arbitrary Code Execution)"

    Maya MCP 伺服器會在 Maya 內部環境執行生成的 Python 代碼。雖然具備高度靈活性，但亦伴隨潛在風險：

    - **操作前務必手動備份並儲存 Maya 專案**
    - 在正式生產環境中謹慎運用大範圍自動化刪除或覆寫工具
    - 留意極端複雜的布林網格運算可能造成軟體記憶體高負載崩潰

## 相關參考資源 {: #resources }

- [Maya MCP 伺服器原始碼倉庫](https://github.com/PatrickPalmer/MayaMCP) — 官方標準存放庫
- [Autodesk Maya 官方網站](https://www.autodesk.com/products/maya) — 官方技術手冊
- [Maya Python API 參考手冊](https://help.autodesk.com/view/MAYAUL/2023/ENU/?guid=Maya_SDK_Python_ref_index_html) — 深入學習 Maya 內部指令架構
- [Model Context Protocol 官方規範](https://modelcontextprotocol.io/) — MCP 協定標準
