# Web Design & Front-End Design Preferences (網頁與前端設計規範)

本文件整理並歸納本專案的網頁設計與前端開發邏輯。為了實現「極簡程式碼」、「手機體驗優先（Mobile-First）」且在「桌面版完美相容」的設計，本規範採用**行動端核心一體化架構（Mobile-Centric Unified Design）**，避免複雜的精簡/詳細雙軌 HTML 切換，大幅度降低開發與維護難度。

---

## 🍃 1. 抽象設計理念 (Abstract Design Principles)

### 1.1 日系侘寂風視覺哲學 (Wabi-sabi Aesthetics)
* **低飽和與大自然色彩**：避免使用刺眼、高飽和度的純紅、純藍或純綠。採用溫和的自然色調（如森林綠、泥土灰、米黃、開心果奶霜），呈現溫和、平靜、有呼吸感的視覺基調。
* **留白與呼吸感**：區塊之間預留足夠的內距（`padding`）與外距（`margin`），並利用陰影與極細的框線（`border`）建立輕量級層次。
* **情緒對比色設計**：警告或情緒色彩同樣經過低飽和微調（例如使用溫和的暖橘色 `#d4a373` 代替刺眼的紅色，以表現市場估值波動或警示資訊）。

### 1.2 行動核心一體化哲學 (Mobile-Centric Unified Philosophy)
* **手機優先設計 (Mobile-First)**：所有 UI/UX 完全針對手機（單手拇指操作、垂直滾動流、觸控目標尺寸）進行原生設計。
* **桌面端卡片化相容 (Desktop Adaptive Framing)**：桌面版不重新規劃複雜的多欄排版，而是透過限制內容容器最大寬度（如 `max-width: 540px` 或 `600px`）將其置中，兩側搭配柔和背景或毛玻璃感，讓桌面版呈現精美的「卡片式 Web App」質感。
* **漸進式揭露 (Progressive Disclosure)**：捨棄「精簡版」與「詳細版」兩套 HTML 程式碼的雙軌切換架構。改為**單一 HTML 架構**，預設僅呈現核心重點與大綱，詳細內容則透過「點擊展開（Accordion）」、「底部彈出抽屜（Drawer/Bottom Sheet）」或「橫向滑動圖卡（Carousel）」讓使用者依需揭露，從而免除複雜的 JS 滾動定位補償邏輯。

---

## 🛠️ 2. 具體技術規範 (Concrete Specifications)

### 2.1 配色與語意 Tokens (Color Tokens)
```css
:root {
  --primary: #659287;          /* 主色：深海綠 / 抹茶綠 */
  --primary-hover: #4e736a;    /* 主色懸浮 */
  --secondary: #88bda4;        /* 次色：中海綠 */
  --secondary-light: #e6f2dd;  /* 強調背景：開心果奶油綠 */
  
  --bg: #f4f8f1;               /* 網頁底色：極淡抹茶灰白 */
  --surface: #ffffff;          /* 卡片背景 */
  --border: #b1d3b9;           /* 輕量框線 */
  --border-light: #e6f2dd;     /* 超輕量框線/分隔線 */
  
  --text-main: #2f4842;        /* 主要文字：極深墨綠灰 */
  --text-muted: #425c56;       /* 說明文字：深灰綠 */
  
  --warning-bg: #fdfae6;       /* 暖橘/黃背景 */
  --warning-border: #d4a373;   /* 暖橘框線 */
  --warning-text: #8c5e3c;     /* 暖橘文字 */

  --radius-md: 12px;
  --radius-lg: 20px;
  --shadow-sm: 0 1px 3px rgba(15, 23, 42, 0.05);
  --shadow-md: 0 10px 25px -5px rgba(15, 23, 42, 0.08);
}
```

### 2.2 行動核心置中容器排版 (Centered App-like Layout)
* 透過外層容器限制最大寬度，在桌面端保持如手機 App 的精美版面，免去多套 RWD 切換代碼：
```css
.app-container {
  width: 100%;
  max-width: 540px;            /* 限制黃金閱讀寬度，在桌面端保持單欄手機感 */
  margin: 0 auto;              /* 水平置中 */
  background-color: var(--surface);
  min-height: 100vh;
  box-shadow: var(--shadow-md); /* 在桌面端與外圍背景區分 */
  border-left: 1px solid var(--border-light);
  border-right: 1px solid var(--border-light);
}
body {
  background-color: #e2ebd9;   /* 桌面端外圍背景：使用稍深抹茶色烘托置中卡片 */
}
```

### 2.3 單一 HTML 漸進式揭露實作 (Unified Progressive Components)
不用複製兩套 DOM 結構，使用單一結構配合 CSS/JS 隱藏細節：
* **折疊詳解卡片 (Progressive Accordion)**：
  ```html
  <div class="info-block">
    <div class="info-summary">
      <h6>💡 你買股票的錢，是給公司嗎？</h6>
      <p>一般股市交易中，資金是在投資人之間流轉，企業一毛錢也拿不到。</p>
    </div>
    
    <!-- 點擊此按鈕在下方展開詳解，不改變整體閱讀位置，免滾動補償 -->
    <button class="expand-btn" onclick="toggleDetails(this)">閱讀完整解析</button>
    
    <div class="info-details" style="display: none;">
      <p>詳細說明：只有在 IPO（初次公開發行）或企業增資時...</p>
    </div>
  </div>
  ```
* **行動端底部抽屜 (Bottom Sheet/Drawer)**：
  在手機端，點擊詳細按鈕時，從螢幕底部滑出半透明毛玻璃質感的抽屜面板，提供沉浸式但易收合的詳細閱讀，桌面端則自動降級為居中對話框。

### 2.4 行動端水平滑動與自動置中 (Mobile Navbar & Swipeable Flow)
* **水平滑動導覽 (Overflow Slider)**：手機版導覽列與長卡片組允許橫向滑動 (`overflow-x: auto;`)，配合 `mask-image` 淡出效果。
* **導覽項目置中**：點擊或滾動觸發時，JS 自動以 `scrollTo` 將 Active 項目滑動至容器中軸，提升單手點擊率。
  ```javascript
  const activeLink = document.querySelector('.nav-link.active');
  if (activeLink) {
    const container = document.querySelector('.nav-menu');
    container.scrollTo({
      left: activeLink.offsetLeft - (container.offsetWidth / 2) + (activeLink.offsetWidth / 2),
      behavior: 'smooth'
    });
  }
  ```

### 2.5 程式碼極簡防禦 (Code Minimization & Safety)
* **避免複製 DOM**：例如，行動端表格直接在原容器上透過 CSS 或 Scale 縮放，或以原生的 `dialog` 元素開啟，避免使用 JS 複製整個 Table 的 `cloneNode` 操作，以求程式碼最精簡。
* **LocalStorage 與全域綁定防禦**：
  * LocalStorage 操作一律包覆於 `try...catch` 區塊中。
  * 行內事件綁定之函式一律顯式掛載至 `window` 全域物件上。

---

## 📋 3. 新專案設計檢核表 (New Project Checklist)

* [ ] **一體化容器**：是否使用 `.app-container` 限制最大寬度（如 540px），使網頁在桌面與手機端有高度一致的 App 體驗？
* [ ] **避免代碼重複**：是否移除了 `.view-summary` 與 `.view-full` 的複製邏輯，改用「單一 DOM + 漸進式展開」？
* [ ] **原位展開**：所有的詳細內容是否使用原位展開（Accordion）或 Bottom Sheet，且無版面劇烈跳躍？
* [ ] **滑動置中**：水平導覽列在切換時，Active 項目是否會自動滾動置中？
* [ ] **防禦編寫**：`localStorage` 是否有 `try-catch`？行內事件是否全域掛載？
