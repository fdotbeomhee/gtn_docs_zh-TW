<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/griptape-ai/griptape-nodes-engine/raw/main/docs/assets/img/griptape_nodes_from_foundry_white.svg">
  <img alt="Griptape Nodes" src="https://github.com/griptape-ai/griptape-nodes-engine/raw/main/docs/assets/img/griptape_nodes_from_foundry_black.svg" width="600">
</picture>

# Griptape Nodes 繁體中文技術文件 (zh-TW) {: #griptape-nodes-traditional-chinese-docs }

Griptape Nodes 是一款專為專業藝術家、技術美術 (Technical Artist) 與創意工作者打造的強大視覺化節點式 AI 工作流程構建器。透過直覺的拖曳式雲端介面 [Griptape Nodes IDE](https://app.nodes.griptape.ai/)，輕鬆編排並運行複雜的多模態 AI 工作流程。

本儲存庫收錄了經過系統性深度在地化的 **Griptape Nodes 官方技術文件全量繁體中文版 (`zh-TW` / `zh-Hant`)**，並基於 MkDocs Material 構建了完備的離線與線上文件服務。

---

## ✨ 核心特性

- **100% 官方原文件全量翻譯**：涵蓋快速上手、完整教學、使用者指南、自訂節點開發、企業級內部部署 (On-Premises) 及底層組態參考手冊（共 260 篇 Markdown 文件）。
- **專業標準繁體 IT 術語對齊**：嚴格遵循台灣/香港標準計算機資訊術語（如 `專案`、`預設`、`節點`、`快取`、`記憶體`、`介面`、`伺服器`、`用戶端`、`影片`、`音訊`、`儲存貯體`、`遮罩`、`色階`、`保留模式`），杜絕簡體字盲目轉換。
- **雙語並記對照**：核心 UI 元件與技術概念採用 `繁體中文 (English)` 標準雙語並記，無縫銜接英語原生軟體操作介面。
- **錨點全量相容 (Zero-Warning)**：透過 `attr_list` 擴充套件實現所有底層技術錨點（Anchor Slug）的 100% 互通，達成 0 個建置警告與 0 壞連結。
- **獨立部署與多語並行**：預設配置獨立通訊埠 **`8003`**，可與韓語版（通訊埠 `8000`）、簡體中文版（通訊埠 `8001`）及日語版（通訊埠 `8002`）在單機環境無縫並行預覽。

---

## 🚀 本地極速預覽

### 1. 準備環境與安裝相依套件
確保系統已安裝 Python 3.10+：

```bash
# 進入儲存庫目錄
cd gtn_docs_zh-TW

# 建立虛擬環境
python -m venv venv

# 啟用虛擬環境 (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# 啟用虛擬環境 (macOS / Linux)
source venv/bin/activate

# 安裝 MkDocs 相關相依套件
pip install -r requirements.txt
```

### 2. 啟動本地文件伺服器
為了避免通訊埠衝突，繁體中文文件推薦使用通訊埠 **8003**：

```bash
python -m mkdocs serve -a 127.0.0.1:8003
```

啟動成功後，在瀏覽器中開啟：[http://127.0.0.1:8003/zh-TW/](http://127.0.0.1:8003/zh-TW/)

### 3. 建置靜態站點
```bash
python -m mkdocs build --clean
```
建置產物將輸出至 `site/` 目錄中，可直接發布至 GitHub Pages、Nginx 或任何靜態託管平台。

---

## 📂 文件架構目錄

```text
docs/
├── index.md                 # 文件首頁總覽
├── installation.md          # 桌面端與 CLI 引擎安裝全量指南
├── architecture.md          # SaaS 與企業內部部署拓撲架構
├── tutorials/               # 官方經典案例實戰教學
├── guides/                  # 核心功能使用指南 (組態、變數、專案、多媒體)
├── enterprise/              # 企業管理儀表板、Admin Server 與 Cedar 存取控制
├── development/             # 自訂節點開發、Worker 隔離與嚴格模式
├── reference/               # 引擎系統組態與 CLI 指令完全參考手冊
└── nodes/                   # 標準節點庫完整規格參考 (200+ 節點)
```

---

## 📜 開源授權協議

本專案文件遵循 [Apache License 2.0](LICENSE) 協議發布。
原始英文文件版權歸 [Griptape, Inc.](https://www.griptape.ai/) 所有。
