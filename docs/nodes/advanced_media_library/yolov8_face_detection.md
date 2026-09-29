# YOLOv8 Face Detection {: #yolov8-face-detection }

!!! warning "使用 Hugging Face 模型前需完成的前置設定"

    [本整合指南](../../guides/integrations/hugging_face.md) 將引導您完成 Hugging Face 帳號註冊、存取權杖 (Access Token) 建立，以及下載必備模型權重以確保本節點順暢運作。

## 功能概述 {: #what-is-it }

YOLOv8 Face Detection 是一個電腦視覺節點，運用來自 🤗 Hugging Face 的 YOLOv8（You Only Look Once 第 8 版）物體偵測模型來識別影像中的人臉。該節點分析傳入的影像，並輸出每張偵測到之人臉的精確邊界框 (Bounding Box) 幾何座標與置信度分數 (Confidence Scores)。

本節點底層整合了專門針對人臉偵測任務進行深度微調訓練的 `arnabdhar/YOLOv8-Face-Detection` 模型，具備極高的推論速度與辨識精度，完美相容即時應用場景。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 偵測照片中的人臉以構建人臉識別管線
- 從複雜多人群照中精準自動裁切提取面部特寫
- 計算影像畫面中的出現人數
- 依據畫面是否包含人臉自動過濾篩選素材
- 打造聚焦於面部的構圖或風格化視覺特效
- 進行個人隱私去識別化保護（人臉模糊、打碼或遮罩合成）
- 開發全自動智慧相簿分類與人物歸檔系統
- 實作基於人臉定位的門禁或安全驗證流程

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. **新增節點**：

    - 將 "YOLOv8 Face Detection" 節點新增至工作流程中
    - 首次執行時節點會自動下載神經網路模型權重（需要具備 Hugging Face 連線權限）

1. **連線影像輸入**：

    - 將 `ImageArtifact` 或 `ImageUrlArtifact` 連線至 `input_image` 參數埠
    - 素材可來自檔案載入節點、影像生成節點或各類影像處理節點

1. **配置偵測參數**：

    - 設定置信度閾值 `confidence_threshold` (0.0-1.0) 以過濾雜訊候選框
    - 視需求設定膨脹比例 `dilation` (0-100%) 以等比向外擴展邊界框範圍

1. **執行偵測**：

    - 執行節點以完成人臉特徵提取
    - 偵測結果將以結構化清單形式於 `detected_faces` 輸出埠傳回

### 參數說明 {: #parameters }

#### 輸入端參數 {: #input-parameters }

- **input_image**（必填）

    - 型別：`ImageArtifact` 或 `ImageUrlArtifact`
    - 欲進行人臉偵測分析的目標來源影像

- **confidence_threshold**（置信度閾值）

    - 型別：`float` (0.0-1.0)
    - 預設值：`0.5`
    - 邊界框被採納納入最終結果的最低置信度得分
    - 數值越高 = 篩選越嚴格，誤報越少，但可能遺漏偏遠或模糊面部
    - 數值越低 = 召回率越高，但可能包含誤報候選框

- **dilation**（外擴膨脹比例）

    - 型別：`float` (0.0-100.0)
    - 預設值：`0.0`
    - 在維持邊界框中心點不變的前提下，向四周等比向外擴展的百分比
    - 非常適合用於在人臉特寫周圍保留髮型與下巴等周邊上下文環境
    - 範例：設定為 `10.0` 代表向各方向均勻外擴 10% 的尺寸

#### 輸出端參數 {: #output-parameters }

- **detected_faces**

    - 型別：`list`
    - 包含所有偵測到的人臉結構化清單，每個人臉字典包含：
        - `x`：邊界框左上角 X 像素座標
        - `y`：邊界框左上角 Y 像素座標
        - `width`：邊界框寬度像素
        - `height`：邊界框高度像素
        - `confidence`：偵測置信度得分 (0.0-1.0)

- **logs**

    - 型別：`string`
    - 偵測過程的詳細日誌，包含：
        - 模型載入與快取狀態
        - 執行偵測參數配置
        - 最終成功辨識到的人臉總數

### 輸出資料格式規範 {: #output-format }

每個偵測到的人臉皆表示為標準字典物件：

```json
{
  "x": 150,
  "y": 200,
  "width": 300,
  "height": 350,
  "confidence": 0.95
}
```

所有幾何座標皆以輸入影像的像素解析度為基準，原點 `(0,0)` 位於影像的**左上角**。

### 經典工作流程範例 {: #example-workflows }

#### 基礎人臉偵測流程 {: #basic-face-detection }

1. 使用 "Load Image" 或類似節點載入照片
1. 連線至 YOLOv8 Face Detection 的 `input_image` 埠
1. 將 `confidence_threshold` 設為 `0.5` 取得平衡的檢測率
1. `detected_faces` 輸出埠即回報所有人臉位置與置信度

#### 人臉自動裁切流水線 (Face Cropping Pipeline) {: #face-cropping-pipeline }

1. 使用 YOLOv8 Face Detection 偵測照片中的面部
1. 將 `dilation` 設為 `10.0` 以適度保留耳朵與頭髮輪廓
1. 將 `detected_faces` 連線至 "Crop Image" 節點
1. 自動批次提取獨立的面部特寫頭像以供後續進一步處理

#### 高精度無誤報偵測 {: #high-confidence-detection-only }

1. 將 `confidence_threshold` 調高至 `0.8` 以上
1. 嚴格過濾掉微弱或缺乏把握的候選框
1. 極為適合用於要求高準確率的安全核對場景

### 進階特性功能 {: #advanced-features }

- **模型本機自動快取**：首次下載的模型權重會妥善快取於本機，後續執行近乎零延遲瞬間載入。
- **畫布邊界智慧防溢截斷 (Boundary Clamping)**：膨脹外擴後的邊界框若超出影像畫布邊緣，會自動被截斷在有效像素範圍內。
- **等比居中外擴**：膨脹運算嚴格鎖定原始面部中心錨點。
- **批次相容性**：可接駁至迴圈與清單結構中批次處理海量照片。

## 效能與硬體考量 {: #performance-considerations }

- **首次執行耗時**：首次運算需自雲端下載約 6MB 的輕量化權重，耗時數秒。
- **後續推論速度**：本機快取加載後，YOLOv8 推論極其迅速，在單張影像上通常僅需數毫秒至數十毫秒。
- **記憶體開銷**：模型常駐記憶體僅需約 50MB RAM，極其輕量化。

## 常見問題排解 {: #common-issues }

- **缺少 API 金鑰**：請確認 Hugging Face 存取權杖已正確設定為環境變數 `HF_TOKEN`，詳情參閱 [整合指南](../../guides/integrations/hugging_face.md)。
- **找不到模型檔案**：若看到模型不可用警告，請點選介面連結開啟 Model Manager 重新下載權重。
- **未偵測到任何人臉**：若畫面上有人臉卻未檢出，可適度調低 `confidence_threshold`。
- **誤報率過高**：將 `confidence_threshold` 調高以排除背景陰影等雜訊。
- **邊界框過於侷促貼身**：適當提高 `dilation` 數值為邊界框四周保留空間。

## 技術架構細節 {: #technical-details }

- **模型架構**：Hugging Face 上的專用人臉辨識模型 (`arnabdhar/YOLOv8-Face-Detection`)
- **核心架構**：專精於人臉幾何特徵的單次遍歷 (Single-pass) YOLOv8 目標偵測演算法
- **依賴套件**：`ultralytics>=8.0.0`、`supervision>=0.20.0`
- **資料規範**：輸出標準 (x, y, width, height) 幾何邊界框與置信度陣列
