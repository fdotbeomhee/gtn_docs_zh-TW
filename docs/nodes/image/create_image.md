# GenerateImage {: #generateimage }

## 功能概述 {: #what-is-it }

GenerateImage 節點利用 Griptape Cloud 雲端服務或所連線的影像生成驅動程式，依據文字提示詞 (Prompts) 自動生成高畫質影像。它能無縫串聯 Griptape Cloud Image Generation Driver 與 Prompt Driver，將自然語言描述轉化為細膩的視覺影像。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用 GenerateImage 節點：

- 欲在工作流程中動態自主生成原創圖像
- 欲根據劇本或文案描述建立配套的視覺內容
- 將自然語言構想直觀視覺化

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 GenerateImage 節點新增至工作流程中
1. 提供描繪欲生成畫面細節的文字提示詞 (Prompt)
1. 執行您的工作流程！

### 參數說明 {: #parameters }

- **agent**：負責引導處理提示詞語意與排程影像任務的 Agent 代理（Agent 物件或字典）
- **image_generation_driver**：專門用於驅動影像生成模型的驅動程式物件（預設為 None，將自動使用系統預設雲端驅動）
- **prompt**：描繪欲生成影像之畫面特徵的文字提示詞（字串）
- **enhance_prompt**：是否由 AI 自動擴充並潤飾提示詞以獲得更高品質的影像（布林值，預設為 `True`）

### 輸出埠說明 {: #outputs }

- **output**：生成的影像產物 (ImageUrlArtifact)

## 範例展示 {: #example }

生成日落風景圖並自動寫入磁碟：

1. 將 GenerateImage 節點新增至工作流程中
1. 將 Agent 節點連線至 agent 參數埠
1. 將 prompt 提示詞設定為："A serene mountain landscape at sunset with a lake reflecting the orange sky"（夕陽餘暉下寧靜的雪山群峰，湖面倒映著橙金色的天際）
1. 將輸出埠連線至 SaveImage 節點以將生成的影像儲存至本機檔案

## 重要注意事項 {: #important-notes }

- 使用 Griptape Cloud 預設服務時，系統環境變數中必須正確配置 `GT_CLOUD_API_KEY` 以便通過身分驗證。
- 節點預設調用 `dall-e-3` 模型並啟用 `hd` 高清品質模式。
- 提示詞智慧強化 (enhance_prompt) 能大幅豐富畫面的光影細節，但有時可能對原始提示詞產生些微不同的發散詮釋。
