# BloomEffect {: #bloomeffect }

## 功能概述 {: #what-is-it }

BloomEffect 節點為您的影像套用唯美的泛光 (Bloom) 或發光 (Glow) 特效，營造出夢幻、空靈的視覺氛圍。它模擬了強光源在周圍空氣中形成的柔和光暈 (Luminous Halo)，為影像增添奇幻與電影質感的藝術魅力。

## 適用情境 {: #when-would-i-use-it }

在以下情況使用此節點：

- 營造夢幻、空靈的超現實視覺質感
- 為畫面中的亮部區域增添柔和的向外擴散光暈
- 增強奇幻 (Fantasy) 或科幻 (Sci-Fi) 場景中發光物體的視覺張力
- 營造浪漫、溫暖或戲劇化的環境氛圍光效
- 為人像攝影或風景照片增添柔焦與藝術感
- 模擬相機鏡頭光暈 (Lens Flare) 與泛光過曝效果

## 使用方式說明 {: #how-to-use-it }

### 基礎設定步驟 {: #basic-setup }

1. 將 BloomEffect 節點新增至工作流程中
1. 將影像來源連線至 "input_image" 輸入埠
1. 調整泛光強度 (bloom_amount) 與擴散半徑 (bloom_radius) 滑桿
1. 處理完成的泛光影像將於 "output" 輸出埠傳回

### 參數說明 {: #parameters }

#### 泛光參數設定 (Bloom Settings) {: #bloom-settings }

- **bloom_amount**（泛光強度，範圍 0.0-2.0，預設值：0.5）：控制泛光發光的強度比例

    - 0.0 = 無泛光效果
    - 數值越高，光暈亮度越強烈奪目
    - 2.0 = 最大泛光強度

- **bloom_radius**（擴散半徑，範圍 1-20，預設值：5）：控制光暈向外擴散模糊的範圍大小

    - 較小數值產生緊湊、聚焦的高對比光暈
    - 較大數值產生範圍更廣、柔和擴散的瀰漫光霧
    - 較大的半徑數值會耗費略多的運算時間

### 輸出埠說明 {: #outputs }

- **output**：套用泛光效果後的完成影像產物

## 範例展示 {: #example }

為霓虹夜景照片添加電影感泛光：

1. 使用 LoadImage 載入照片或使用 GenerateImage 生成畫面
1. 將影像連線至 BloomEffect 節點的 "input_image" 參數埠
1. 調整泛光滑桿參數：
    - 將 bloom_amount 設為 0.8 以獲得適中醒目的光暈
    - 將 bloom_radius 設為 8 形成柔和適度的光霧擴散
1. 將 "output" 引腳連線至 DisplayImage 檢視最終渲染成果

## 重要注意事項 {: #important-notes }

- **即時預覽 (Live Preview)**：移動滑桿時畫面會即時呈現光效變化。
- **運算耗時考量**：較大的 bloom_radius 半徑需要執行更大半徑的高斯模糊卷積，運算時間會隨之增加。
- **最佳視覺場景**：在包含高光點、燈光或強反光高對比區域的畫面上效果最為顯著。
- **RGBA 透明度完整保留**：若輸入為帶有透明通道的 RGBA 影像，將完整保留其透明度數值。

## 常見問題排解 {: #common-issues }

- **看不出光效**：請確認 bloom_amount 數值已大於 0，且畫面上具有足夠明亮的像素點。
- **效果過於微弱**：適度提高 bloom_amount 強度以增強可見度。
- **處理速度偏慢**：適度降低 bloom_radius 半徑以加速運算。

## 技術實作細節 {: #technical-details }

泛光效果遵循專業影視後製處理流程：

1. **亮部閾值檢索 (Brightness Detection)**：透過高光隔離濾波器提取畫面中的高光像素。
1. **高斯模糊卷積 (Gaussian Blur)**：對高光圖層執行精確的雙向高斯模糊以生成柔和的光暈擴散層。
1. **濾色混合疊加 (Screen/Additive Blending)**：將模糊後的光暈圖層與原始影像以特定光學混合模式進行合成。
1. **強度加權調控**：根據 bloom_amount 係數加權調整最終光效的疊加佔比。
