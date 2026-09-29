# Advanced Media 程式庫 {: #advanced-media-library }

提供可在**本機端**直接運算的高階多媒體生成與影像處理節點——包含擴散管線 (Diffusion Pipelines)、影像與視訊預處理（深度圖、邊緣檢測、人體姿態估計、語意分割）、LoRA 微調適配工具以及 YOLOv8 人臉偵測。

- **原始碼存放庫**：[griptape-ai/griptape-nodes-library-advanced-media](https://github.com/griptape-ai/griptape-nodes-library-advanced-media)
- **環境需求**：強烈建議配備獨立 GPU 加速顯示卡；需準備 Hugging Face 帳號與存取權杖 (Access Token) 以自動下載模型權重（請參閱 [Hugging Face 模型整合](../../guides/integrations/hugging_face.md)）。
- **節點分類目錄**：節點挑選器中的音訊 (audio)、擴散 (diffusion)、影像 (image)、視訊 (video)、LoRA 與工具 (utils) 等目錄。

## 安裝指南 {: #installation }

在執行 `gtn init` 初始化引導時，系統會提示您是否安裝 Advanced Media 程式庫——您可以在當下註冊安裝，或在日後重新執行 `gtn init` 並回答 `y`。詳情可參閱 [常見問題 FAQ](../../faq.md#how-do-i-install-the-advanced-media-library-after-initial-setup)。

此外，您也可以透過標準途徑進行安裝：在編輯器中，依序點選 **管理 (Manage) → 程式庫管理 (Library Management)**，點擊 **新增程式庫 (Add Library)** 並貼入以下網址：

```text
https://github.com/griptape-ai/griptape-nodes-library-advanced-media
```

有關程式庫的通用安裝、版本更新與除錯說明，請參閱 [程式庫指南](../../guides/libraries.md)。

## 節點索引參考 {: #node-reference }

本程式庫共包含 28 個功能節點。目前已撰寫專屬說明頁面的核心節點包括：

- [Diffusion Pipelines](../../nodes/advanced_media_library/diffusion_pipelines.md) —— 本機建構並快取 🤗 Diffusers 擴散管線，並進行高畫質文生圖與圖生圖生成。
- [YOLOv8 Face Detection](../../nodes/advanced_media_library/yolov8_face_detection.md) —— 即時偵測畫面人臉並輸出精準邊界框 (Bounding Boxes) 與分割遮罩。

!!! warning "與 Diffusers 程式庫並用之注意事項"

    Advanced Media 程式庫與 [Diffusers 程式庫](../diffusers/index.md) 共用許多底層依賴項（如 `diffusers`、`transformers`、`torch`），且各自在記憶體中維護獨立的管線快取。在同一個工作階段中同時調用兩者可能導致 VRAM 雙倍佔用而引發記憶體不足 (OOM) 錯誤。強烈建議在單一流程中擇一使用。

## 支援與反饋 {: #support }

若在運作中發現程式錯誤或有全新功能建議，歡迎隨時向社群回報：
[提交 GitHub Issue](https://github.com/griptape-ai/griptape-nodes-library-advanced-media/issues)。
