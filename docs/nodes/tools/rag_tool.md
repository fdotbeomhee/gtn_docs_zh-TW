# InfoRetriever {: #inforetriever }

## 功能概述 {: #what-is-it }

InfoRetriever 節點為您的工作流程引入檢索增強生成 (Retrieval Augmented Generation, RAG) 能力。您可以將其視為一位智慧研究員，能夠從龐大的私有資料庫中檢索高相關度資訊，藉此大幅強化並精確約束 AI 模型的回應品質。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 讓 AI 模型的回應牢固基於 (Grounding) 您的私有或專有資料來源
- 賦予 AI Agent 存取與精確引用特定企業文件的能力
- 藉由注入高關聯度上下文，消除幻覺並提升回答準確度
- 將專屬向量知識庫 (Knowledge Base) 無縫掛載至對話式代理

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 InfoRetriever 節點新增至工作流程中
1. 將其輸出引腳連線至需要 RAG 檢索能力的節點（例如 Agent 節點）

### 參數說明 {: #parameters }

- **description**：說明此工具提供何種範疇的資訊（預設為 `"Contains information"`， Agent 透過此語意判定呼叫時機）
- **off_prompt**：是否在主提示詞之外獨立執行 RAG 檢索操作（預設為 `true`，以大幅節省 Context Token）
- **rag_engine**：用於檢索相關資訊的底層 RAG 引擎物件（必填參數）

### 輸出埠說明 {: #outputs }

- **tool**：配置完備的 RAG 工具物件 (Tool Object)，可直接供其他節點引用
- **rules**：由該 RAG 工具所伴隨套用的規則集 (Ruleset)

## 範例展示 {: #example }

建立一個能參考公司內部技術手冊進行問答的專業客服 Agent：

1. 將 InfoRetriever 節點新增至工作流程中
1. 將索引有內部文件的向量檢索引擎連線至 "rag_engine" 輸入埠
1. 將 "tool" 輸出引腳連線至 Agent 節點的 "tools" 輸入埠
1. 該 Agent 在回答使用者問題時，便能自主檢索內部手冊並附帶精確的引用依據

## 實作細節 {: #implementation-details }

InfoRetriever 充分發揮了 Griptape 框架的 Off-Prompt 設計哲學。透過將大量未過濾的知識庫資料保留在模型 Context 之外，僅將檢索精煉後的關聯上下文動態回傳，能有效避免長文本造成的專注力發散與 Token 成本膨脹。
