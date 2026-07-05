# ETF 新手教學手冊 · 專案設計與維護指南 (Codebase & Design Guide)

本文件旨在協助後續的 AI 代理人 (Agent) 與開發者，快速理解專案的 UI/UX 設計理念、雙軌閱讀架構、行動端響應式 (RWD) 實作機制，以及日常維護的程式碼規範。

---

## 📖 1. 雙軌閱讀機制 (Dual-Track Reading Flow)

本網頁採用創新且直覺的**「精簡摘要」與「完整好讀」雙軌切換架構**，讓讀者能根據時間與心情，彈性切換內容呈現方式。

### 1.1 HTML 結構規範
* **精簡摘要視圖**：所有精簡圖卡、3D 卡片、簡明時間軸等區塊，必須包覆在 `.view-summary` 容器中。
* **完整好讀視圖**：長篇詳細散文、深入解說、完整比較表格等內容，必須包覆在 `.view-full` 容器中。
* **注意**：大標題與段落標題 `.section-header` 通常維持共用，不放入上述兩個視圖容器，以防切換時標題消失。

### 1.2 CSS 切換邏輯
* 預設狀態下，`<body>` 帶有 `summary-mode` 類別，只顯示 `.view-summary`：
  ```css
  body.summary-mode .view-summary { display: block; }
  body.summary-mode .view-full { /* 預設收合或隱藏 */ }
  ```
* 切換為詳細閱讀時，`<body>` 改為 `full-mode` 類別，隱藏摘要並顯示完整內容：
  ```css
  body.full-mode .view-summary { display: none !important; }
  body.full-mode .view-full { display: block !important; ... }
  ```

---

## 🎨 2. UI/UX 與 RWD 行動端優化指南

### 2.1色彩與對比 (Matcha Wabi-sabi Style)
專案採用日系抹茶侘寂風配色。為了符合 WCAG AA 對比度標準，重要文字顏色設定如下：
* 主要文字：`--text-main: #2f4842` (極深墨綠灰)
* 次要/說明文字：`--text-muted: #425c56` (深灰綠)

### 2.2 智慧滾動隱藏導覽列 (Scroll-to-Hide Navbar)
* 頂部導覽列 `header.navbar` 具備 `transform: translateY(0)` 及平滑過渡屬性。
* 使用者向下滑動超過 `80px` 且持續向下時，動態加上 `.hide-nav` 類別（`transform: translateY(-100%)`）將其收起，增加閱讀縱深。
* 向上滑動時立刻移除 `.hide-nav` 類別，展現導覽列，便於跳轉。

### 2.3 行動端單行導覽列與自動置中 (Mobile Single-Row Navbar)
* 在寬度 `<= 768px` 的裝置上，導覽列容器 `.nav-container` 變更為單行排列（`flex-direction: row; justify-content: space-between`），隱藏文字切換按鈕。
* 導覽選單 `.nav-menu` 採用水平滑動（`overflow-x: auto; flex-shrink: 1`），並透過 `-webkit-mask-image` 及 `mask-image` 漸層遮罩實作兩側淡出（fade-out）效果，暗示可左右滑動。
* **自動置中**：當讀者滑動至不同章節，JavaScript 會偵測當前 active 的 `.nav-link`，並以 `scrollTo` 平滑地將該標籤置中對齊在水平導覽列上，大幅提升單手操作體驗。

### 2.4 行動端閱讀模式懸浮按鈕 (FAB)
* 手機版右下角常駐 `#reading-toggle-fab` 懸浮按鈕（置於拇指操作熱區，於 `bottom: 88px; right: 24px`，避開回到頂部按鈕）。
* 該按鈕採用抹茶主題色與毛玻璃質感（`backdrop-filter: blur(12px)`），且狀態（文字、圖示）與桌面版頂部 navbar 中的切換按鈕同步。

### 2.5 閱讀切換狀態延續與滾動補償 (Scroll Compensation)
* 由於精簡版與完整版的網頁長度差距極大，直接切換會造成嚴重的「版面跳躍」，讓讀者迷失。
* **補償機制**：在切換模式前，JS 會透過 `getClosestSection()` 計算目前最接近視窗頂部的章節 ID。切換後，使用 `setTimeout` 延遲 50ms，以 `scrollTo` 平滑滾動回切換前所屬章節的標頭，並扣除頂部黏性導覽列的高度（預留 `85px` 補償），使閱讀視角無縫延續。

### 2.6 表格手機端縮放與點擊燈箱 (Mobile Table Fit & Lightbox)
* **手機預設呈現**：行動版畫面上，`.table-container` 設定為 `overflow-x: hidden !important`，表格強制滿寬，並將字型縮小至 `0.58rem`、儲存格內距壓縮為 `6px 3px`，確保整張表格不需要水平滾動即可在手機寬度內完整顯現。
* **放大燈箱**：
  * 表格頂部加上 `table-hint` 提示條（`📱 點擊表格可放大閱讀`）。
  * 點擊表格會觸發 `openTableLightbox()`，將表格複製一份置入 `table-lightbox` 遮罩中。
  * 燈箱內的表格還原為正常尺寸與易讀字型，並**開啟水平滾動**，方便讀者縮放與細讀。

### 2.7 靜態實存步驟 (Static Stepper)
* 第五章「實戰步驟」已全面取消互動式的勾選 Checkbox 功能，改為靜態的純數字圓圈呈現（`.stepper-number`，顯示 `1` 到 `7`），簡化使用者的操作干擾。

### 2.8 雙軌金流流程圖 (Dual-Track Flowchart)
* **設計背景**：原第二章的「iPhone 推出時，現金與股票流動」對照表格在手機端及閱讀體驗上不夠直覺。為更好地釐清「買股票的錢是否流進公司」此核心觀念，本專案已將該表格全面替換為雙軌金流流程圖。
* **邏輯與視覺架構**：
  * **第一軌：真實商業世界**（採用日系抹茶綠色調）：呈現「消費者買手機」 -> 「現金流入 Apple 及供應鏈」 -> 「Apple 將盈餘以股利或庫藏股分配給長線投資人與 ETF」的現金與獲利循環。
  * **第二軌：證券交易市場**（採用溫和暖橘色調）：呈現「投資人看好前景」 -> 「於股市（次級市場）交易轉手」 -> 「資金於投資人間流動並推升市值與股價」的估值與情緒波動。
  * **響應式呈現 (RWD)**：在桌面版為左右雙軌對照，以漸層線區隔；在手機版（`<= 768px`）自動重排為上下堆疊的圖卡流程，並隱藏徽章標籤，保持最佳的可讀性。

---

## 🛠️ 3. 程式碼維護規範與安全守則

### 3.1 全域 Window 繫結 (Global Window Bindings)
* **規則**：由於本專案為單一 HTML 檔案，且按鈕元件包含行內事件處理器（例如 `onclick="toggleReadingMode()"` 或 `onclick="openLightbox(this)"`）。
* **規範**：**所有被行內事件呼叫的 JavaScript 函數，必須在腳本末端顯式掛載到 `window` 全域物件上**。這能徹底防止在特定沙盒、iframe 預覽或模組化作用域下，瀏覽器拋出 `ReferenceError: ... is not defined` 的編譯與執行錯誤。
  ```javascript
  window.toggleReadingMode = toggleReadingMode;
  window.openLightbox = openLightbox;
  window.closeLightbox = closeLightbox;
  window.handleContactSubmit = handleContactSubmit;
  window.initTableLightbox = initTableLightbox;
  window.openTableLightbox = openTableLightbox;
  window.closeTableLightbox = closeTableLightbox;
  ```

### 3.2 儲存 API 防禦 (LocalStorage Try-Catch)
* **規則**：在部分隱私/無痕模式、或本地端透過 `file://` 協議開啟網頁時，瀏覽器會直接限制 `localStorage` 存取並拋出 `SecurityError` 阻擋腳本執行。
* **規範**：所有針對 `localStorage.getItem` 或 `localStorage.setItem` 的呼叫，**必須包覆在 `try...catch` 區塊中**，防止網頁因儲存例外而崩潰。
  ```javascript
  try {
    localStorage.setItem('readingMode', 'full');
  } catch (e) {
    // 靜默失敗或安全回退
  }
  ```

### 3.3 滾動效能優化 (Throttle / Passive Listeners)
* 專案內有多個監聽 `'scroll'` 事件的區塊。在進行大量 DOM 樣式變更或複雜計算時，務必注意效能，必要時應使用 `requestAnimationFrame` 進行包裝，避免造成行動端頁面卡頓。
