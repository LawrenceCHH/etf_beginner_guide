# Web Design & Front-End Design Preferences (網頁與前端設計規範)

本文件整理並提取本專案（日系抹茶侘寂風 ETF 新手教學）的網頁設計與前端開發邏輯，歸納出**抽象設計理念（原則）**與**具體實作規範（技術）**。此規範旨在協助未來的專案或網頁設計，維持一致的高質感視覺美學、雙軌閱讀架構與行動端極致體驗。

---

## 🍃 1. 抽象設計理念 (Abstract Design Principles)

### 1.1 日系侘寂風視覺哲學 (Wabi-sabi Aesthetics)
* **低飽和與大自然色彩**：避免使用刺眼、高飽和度的純紅、純藍或純綠。採用源於大自然的色調（如森林綠、泥土灰、米黃、開心果奶霜），呈現溫和、平靜、有呼吸感的視覺基調。
* **留白與呼吸感**：區塊之間預留足夠的內距（`padding`）與外距（`margin`），並利用陰影與極細的框線（`border`）建立輕量級層次，避免資訊過度擁擠。
* **情緒對比色設計**：若需使用警告或市場情緒色彩，應同樣經過「侘寂化」的低飽和微調（例如使用溫和的暖橘色 `#d4a373` 代替刺眼的紅色，以表現市場估值波動或警示資訊）。

### 1.2 雙軌閱讀哲學 (Dual-Track Content Architecture)
* **尊重讀者的專注力**：現代讀者時間破碎且耐心有限。網頁應提供兩種閱讀層次：
  1. **精簡摘要軌**：以高視覺化圖卡、流程圖、核心數據為主，適合快速瀏覽、5分鐘抓取重點。
  2. **完整好讀軌**：以系統化散文、深度解說、脈絡對照為主，適合安靜深讀。
* **無縫切換體驗**：切換這兩種軌道時，必須做到「不迷失位置」、「不閃爍跳躍」，提供極致平滑的體驗。

### 1.3 互動細節與微互動 (Micro-interactions)
* **生命力（Aliveness）**：卡片、按鈕在 Hover 時應有輕微的位移（如向上平移 `2px`）與陰影加深，讓使用者感受到元件的活性。
* **視覺線索暗示**：對於可滑動或可展開的區塊，提供淡出遮罩（fade-out mask）或旋轉箭頭等視覺暗示，免去文字說明，符合直覺。

---

## 🛠️ 2. 具體技術規範 (Concrete Specifications)

### 2.1 配色與語意 Tokens (Color Tokens)
未來專案可參照此 HSL/Hex 微調之調色盤：
```css
:root {
  /* 基礎配色 */
  --primary: #659287;          /* 主色：深海綠 / 抹茶綠 */
  --primary-hover: #4e736a;    /* 主色懸浮 */
  --secondary: #88bda4;        /* 次色：中海綠 */
  --secondary-light: #e6f2dd;  /* 強調背景：開心果奶油綠 */
  
  --bg: #f4f8f1;               /* 網頁底色：極淡抹茶灰白 */
  --surface: #ffffff;          /* 卡片背景 */
  --border: #b1d3b9;           /* 輕量框線 */
  --border-light: #e6f2dd;     /* 超輕量框線/分隔線 */
  
  /* 文字顏色 - 符合 WCAG AA 對比標準 */
  --text-main: #2f4842;        /* 主要文字：極深墨綠灰 */
  --text-muted: #425c56;       /* 說明文字：深灰綠 */
  --text-inverse: #ffffff;     /* 反白文字 */
  
  /* 警告與情緒色 (低飽和微調) */
  --warning-bg: #fdfae6;       /* 暖橘/黃背景 */
  --warning-border: #d4a373;   /* 暖橘框線 */
  --warning-text: #8c5e3c;     /* 暖橘文字 */
}
```

### 2.2 雙軌閱讀切換機制 (Dual-Track HTML/CSS/JS)
* **HTML 架構**：
  * 精簡卡片、摘要內容放在 `.view-summary` 內。
  * 長篇詳解放在 `.view-full` 內。
  * 章節標題及大框架維持共用，不放入上述兩個容器。
* **CSS 切換**：
  ```css
  /* 預設：精簡模式 */
  body.summary-mode .view-summary { display: block; }
  body.summary-mode .view-full { /* 作為手風琴隱藏，或預設 display: none */ }
  
  /* 詳細模式 */
  body.full-mode .view-summary { display: none !important; }
  body.full-mode .view-full {
    display: block !important;
    background-color: var(--surface) !important;
    border: 1px solid var(--border) !important;
    border-radius: var(--radius-lg) !important;
  }
  ```
* **JS 滾動補償 (Scroll Compensation)**：
  當讀者切換模式時，網頁總長度會劇烈變化。必須利用 JS 計算當前視窗頂部的 Section ID，切換後滾動定位回去（扣除 Sticky Navbar 的高度，一般預留 `85px` 補償），以確保閱讀不中斷：
  ```javascript
  const closestSec = getClosestSection(); // 偵測當前章節
  setReadingMode(newMode);
  if (closestSec) {
    setTimeout(() => {
      const rect = closestSec.getBoundingClientRect();
      window.scrollTo({
        top: window.scrollY + rect.top - 85,
        behavior: 'smooth'
      });
    }, 50);
  }
  ```

### 2.3 雙軌流程圖設計規範 (Dual-Track Flowchart)
針對流程圖（如 Apple 實戰金流），應放棄傳統縱深過長、在手機難以閱讀的複雜表格，改用**卡片式雙軌流程圖**：
* **桌面端**：採用 `display: grid; grid-template-columns: 1fr 1fr;` 雙欄對照。中間使用 `::after` 絕對定位繪製垂直漸層分隔線。
* **行動端 (`<= 768px`)**：
  * 自動折疊成 `grid-template-columns: 1fr;` 單欄上下堆疊。
  * 隱藏中間分隔線 (`display: none`)。
  * 隱藏非必要的徽章標籤（如 `.flow-card-badge`）以最大化行內空間。
  * 將連線箭頭 (`.flow-arrow`) 轉為向下指引。
* **卡片 Hover 互動**：
  * Hover 卡片時加上 `transform: translateY(-2px);` 及 `box-shadow`。
  * 核心企業或市場節點加上特別邊框色（如 `border-left: 4px solid var(--primary)`）以強化視覺起點。

### 2.4 行動端導覽列極致體驗 (Mobile Navbar Smooth-Scroll)
* **水平滑動與淡出**：手機版導覽列應單行水平排列，允許 `overflow-x: auto;`。使用 CSS `mask-image` 漸層遮罩，讓兩側邊緣呈現半透明淡出，提示可左右滑動。
* **Active 項目自動置中**：監聽滾動與點擊，在切換章節時，以 JS 自動將 Active 的連結平滑滾動至導覽列的中心位置：
  ```javascript
  const activeLink = document.querySelector('.nav-link.active');
  if (activeLink) {
    const container = document.querySelector('.nav-menu');
    const containerWidth = container.offsetWidth;
    const linkLeft = activeLink.offsetLeft;
    const linkWidth = activeLink.offsetWidth;
    container.scrollTo({
      left: linkLeft - (containerWidth / 2) + (linkWidth / 2),
      behavior: 'smooth'
    });
  }
  ```

### 2.5 程式安全性防禦 (Code Defense)
* **全域掛載**：對於單一 HTML 檔案且使用行內 `onclick="foo()"` 呼叫的函式，必須在 JS 尾端顯式掛載到 `window` 上（如 `window.foo = foo`），防止在 sandbox、iframe 或模組化編譯環境中因作用域問題發生 `ReferenceError`。
* **LocalStorage 防禦**：任何讀取/寫入 `localStorage` 的行為，必須包覆在 `try...catch` 中，避免使用者在無痕模式或在本地以 `file://` 開啟時拋出安全例外導致整個腳本崩潰。
* **效能防禦**：對滾動監聽（`scroll`）使用 `throttle` 或 `requestAnimationFrame` 以防效能卡頓。

---

## 📋 3. 新專案設計檢核表 (New Project Checklist)

* [ ] **色彩檢驗**：主文字對比度是否符合 WCAG AA 標準（建議墨綠灰/深灰綠）？是否避免使用高飽和純色？
* [ ] **雙軌規劃**：首頁或主要頁面是否規劃了 `.view-summary` 與 `.view-full` 機制？
* [ ] **滾動補償**：切換閱讀模式時，是否有做滾動定位補償，防止頁面跳躍？
* [ ] **流程呈現**：複雜的多角色或前後順序邏輯，是否以「雙軌卡片流程圖」取代了傳統長表格？
* [ ] **手機版導覽**：手機版導覽列是否能水平滑動？Active 項目是否能平滑自動置中？
* [ ] **安全防禦**：所有的 `localStorage` 寫入是否有包覆 `try-catch`？行內事件函式是否已綁定至 `window`？
