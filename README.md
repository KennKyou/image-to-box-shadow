# Image to Box-Shadow 技術文件

## 專案簡介

這是一個純前端的圖片轉 CSS Box-Shadow 像素藝術工具。使用者可以上傳任意圖片，系統會將圖片的每個像素轉換為 CSS box-shadow 屬性，最終生成一個可下載的 HTML 檔案，展示由純 CSS 製作的像素藝術。

## 技術架構

### 核心技術棧
- **HTML5 Canvas API** - 圖片像素數據處理
- **File API** - 檔案上傳處理
- **Blob API** - 動態 HTML 檔案生成
- **CSS3 Box-Shadow** - 像素渲染技術
- **原生 JavaScript** - 邏輯控制
- **響應式 CSS** - 跨裝置相容性

### 檔案結構
```
├── index.html          # 主應用程式（包含 HTML/CSS/JS）
├── README.md          # 技術文件
└── CLAUDE.md          # AI 開發指南
```

## 詳細執行流程

### 1. 圖片上傳階段

#### 1.1 使用者操作
- **拖拽上傳**：直接將圖片拖拽到上傳區域
- **點擊上傳**：點擊「選擇圖片」按鈕開啟檔案選擇器

#### 1.2 技術實現
```javascript
// 事件監聽器設定
fileInput.addEventListener('change', handleFileSelect);
uploadArea.addEventListener('dragover', handleDragOver);
uploadArea.addEventListener('drop', handleDrop);

// 檔案處理流程
function handleFileSelect(event) {
  const file = event.target.files[0];
  if (file && file.type.startsWith('image/')) {
    loadImage(file);
  }
}
```

#### 1.3 檔案驗證
- 檢查檔案類型是否為圖片格式（`image/*`）
- 支援 JPG、PNG、GIF 等常見格式
- 不支援的檔案會被自動過濾

### 2. 圖片載入與預覽

#### 2.1 檔案讀取
```javascript
function loadImage(file) {
  const reader = new FileReader();
  reader.onload = function(e) {
    const img = new Image();
    img.onload = function() {
      // 儲存圖片資訊
      currentImageData = {
        src: e.target.result,
        width: img.width,
        height: img.height
      };
      
      showImagePreview(e.target.result);
      showControls();
    };
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}
```

#### 2.2 預覽顯示
- 使用 `FileReader` 將檔案轉換為 Data URL
- 在預覽區域顯示原始圖片
- 顯示操作按鈕供使用者進行下一步

### 3. 圖片像素分析

#### 3.1 Canvas 初始化
```javascript
const canvas = document.getElementById('hiddenCanvas');
const ctx = canvas.getContext('2d');

// 設定 Canvas 尺寸為圖片原始尺寸
canvas.width = img.width;
canvas.height = img.height;
```

#### 3.2 圖片繪製到 Canvas
```javascript
// 將圖片繪製到 Canvas 上
ctx.drawImage(img, 0, 0, width, height);

// 取得像素數據
const imageData = ctx.getImageData(0, 0, width, height);
const pixels = imageData.data;
```

#### 3.3 像素數據結構
- `imageData.data` 是一個 `Uint8ClampedArray`
- 每個像素佔用 4 個元素：[R, G, B, A]
- 數據按列優先順序排列（從左到右，從上到下）

### 4. Box-Shadow 轉換

#### 4.1 像素遍歷算法
```javascript
let boxShadows = [];

for (let y = 0; y < height; y++) {
  for (let x = 0; x < width; x++) {
    const index = (y * width + x) * 4;
    const r = pixels[index];     // 紅色分量
    const g = pixels[index + 1]; // 綠色分量
    const b = pixels[index + 2]; // 藍色分量
    const a = pixels[index + 3] / 255; // 透明度（0-1）
    
    // 只處理不透明的像素
    if (a > 0.1) {
      boxShadows.push(`${x}px ${y}px 0 rgba(${r},${g},${b},${a})`);
    }
  }
}
```

#### 4.2 Box-Shadow 語法說明
```css
/* box-shadow: offset-x offset-y blur-radius color */
box-shadow: 0px 0px 0 rgba(255,0,0,1),
           1px 0px 0 rgba(0,255,0,1),
           2px 0px 0 rgba(0,0,255,1);
```

- `offset-x`：水平偏移（像素 X 座標）
- `offset-y`：垂直偏移（像素 Y 座標）  
- `blur-radius`：模糊半徑（設為 0，產生銳利邊緣）
- `color`：像素顏色（RGBA 格式）

#### 4.3 性能優化
- 跳過透明度低於 0.1 的像素，減少 CSS 規則數量
- 使用字串陣列收集，最後一次性 join，提升效能
- 保持原始像素尺寸，確保 1:1 像素對應

### 5. HTML 檔案生成

#### 5.1 模板結構
```javascript
function generateHTML(css, width, height) {
  return `<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Box-Shadow Pixel Art</title>
    <style>
        /* 樣式定義 */
    </style>
</head>
<body>
    <div class="container">
        <div class="pixel-art"></div>
    </div>
</body>
</html>`;
}
```

#### 5.2 關鍵 CSS 設計
```css
.container {
    width: ${width}px;
    height: ${height}px;
    background: white;
    overflow: hidden;
    position: relative;
}

.pixel-art {
    width: 1px;
    height: 1px;
    background: transparent;
    box-shadow: ${css};
    position: absolute;
    top: 0;
    left: 0;
}
```

#### 5.3 設計原理
- **容器尺寸**：精確設定為圖片原始尺寸
- **像素元素**：1x1px 的絕對定位元素
- **Box-Shadow**：包含所有像素的偏移和顏色資訊
- **定位原點**：左上角 (0,0) 為起始點

### 6. 檔案下載

#### 6.1 Blob 物件創建
```javascript
function downloadHTML() {
  // 創建 Blob 物件
  const blob = new Blob([generatedCSS], { type: 'text/html' });
  
  // 創建下載 URL
  const url = URL.createObjectURL(blob);
  
  // 創建隱藏的下載連結
  const a = document.createElement('a');
  a.href = url;
  a.download = 'boxshadow-pixel-art.html';
  
  // 觸發下載
  document.body.appendChild(a);
  a.click();
  
  // 清理資源
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}
```

#### 6.2 下載機制
1. 使用 `Blob` API 將 HTML 字串轉換為檔案物件
2. `URL.createObjectURL()` 創建臨時下載連結
3. 動態創建 `<a>` 元素並設定下載屬性
4. 程式化觸發點擊事件開始下載
5. 清理 DOM 元素和釋放 URL 資源

## 瀏覽器相容性

### 必要 API 支援
- **Canvas API**：IE9+
- **File API**：IE10+
- **Blob API**：IE10+
- **URL.createObjectURL**：IE10+

### 建議瀏覽器版本
- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

## 性能考量

### 檔案大小限制
- **小圖片（< 100x100）**：瞬間處理
- **中等圖片（100x100 - 500x500）**：1-3 秒處理
- **大圖片（> 500x500）**：可能產生巨大 CSS 檔案

### 記憶體使用
- 每個像素約佔用 25-30 bytes CSS 代碼
- 1000x1000 像素圖片約產生 25-30MB CSS
- 建議限制圖片尺寸在 500x500 以內

### 優化建議
1. **圖片壓縮**：上傳前適當縮小圖片尺寸
2. **透明度過濾**：跳過透明像素減少 CSS 規則
3. **進度顯示**：大圖片處理時顯示進度條

## 使用限制

### 技術限制
- 純前端處理，受瀏覽器記憶體限制
- 大圖片可能導致瀏覽器當機
- 生成的 HTML 檔案可能非常大

### 實際應用
- 適合小型像素藝術和 Logo
- 不適合高解析度照片
- 主要用於展示和教學目的

## 開發指南

### 本地開發
```bash
# 直接在瀏覽器開啟
open index.html

# 或使用本地伺服器
python -m http.server 8000
# 訪問 http://localhost:8000
```

### 程式碼結構
- **HTML 結構**：語義化標籤和無障礙設計
- **CSS 樣式**：Mobile-First 響應式設計
- **JavaScript 邏輯**：模組化函數設計

### 擴展功能建議
1. **圖片尺寸限制**：增加檔案大小檢查
2. **顏色量化**：減少顏色數量優化效能
3. **格式選擇**：支援 SVG 或其他向量格式輸出
4. **批次處理**：支援多檔案同時處理

## 授權與使用

本專案為開源專案，可自由使用、修改和分發。適合用於：
- 學習 Canvas API 和檔案處理
- 了解前端圖像處理技術  
- 創建有趣的視覺效果
- 教學和展示用途

---

*最後更新：2025年7月*