---
title: "使用esbuild編譯，安裝Font Awesome"
date: 2024-12-30
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

### 安裝 Font Awesome

使用 npm 進行安裝三種套件，三種套件作用將於文末說明

```zsh
npm install -D @fortawesome/fontawesome-svg-core @fortawesome/free-solid-svg-icons @fortawesome/fontawesome-free
```

### 修改預備編譯的 JavaScript 檔案

```js
// Font Awesome 引入
import { library, dom } from "@fortawesome/fontawesome-svg-core"; //引入核心模型
import { faSpinner, faHouse, faHeart } from "@fortawesome/free-solid-svg-icons"; // 引入所需的圖示
import "@fortawesome/fontawesome-svg-core/styles.css"; //引入 Font Awesome 的樣式

// 將圖示添加到資料庫
library.add(faSpinner, faHouse, faHeart);

// 自動掃描 DOM 並渲染圖示
dom.watch();
```

### 透過 esbuild 處理 JavaScript 檔案編譯

```json
"scripts": {
  "dev": "esbuild frontends/scripts/app.js --bundle --outfile=static/scripts/app.js --watch --platform=browser"
}
```

1. esbuild：
   - 使用 esbuild 這個工具來進行打包構建。
2. frontends/scripts/app.js：
   - 指定入口檔案，這裡是 app.js，它是包含主要邏輯的主檔案。
3. --bundle：
   - 將入口檔案及其依賴（包括從 node_modules 中引用的套件）一起打包成單一檔案，減少瀏覽器需要載入的請求數。
4. --outfile=static/scripts/app.js：
   - 指定輸出檔案的位置，這裡是將打包後的檔案輸出到 static/scripts/app.js。
5. --watch：
   - 啟用文件監聽模式。如果檔案發生變更，esbuild 會自動重新構建，非常適合開發模式。
6. --platform=browser：
   - 設置目標運行環境為 瀏覽器。
   - 告訴 esbuild 不使用 Node.js 模式的模組（例如 fs 或 path），而是優化為瀏覽器友好的格式。

### 依照 Font Awesome 的 icon 名稱帶入 JavaScript 檔案，有幾個就需要帶幾個

```js
import { <帶入Font Awesome名稱> } from "@fortawesome/free-solid-svg-icons"
library.add( <帶入Font Awesome名稱> );
```

### 執行編譯

```zsh
npm run dev
```

### 安裝的三個套件內容與用途

| 模組                                | 包含內容                                           | 使用場景                                                       |
| ----------------------------------- | -------------------------------------------------- | -------------------------------------------------------------- |
| `@fortawesome/fontawesome-free`     | 全部圖標（Solid、Regular、Brands）及 CSS 文件      | 通過 CSS 使用圖標；專案需要完整的圖標庫。                      |
| -------------------------------     | -----------------------------------------          | -----------------------------------------------------------    |
| `@fortawesome/fontawesome-svg-core` | 圖標核心功能，管理和渲染圖標，不包含具體的圖標資源 | 在 JavaScript 中動態載入、按需渲染圖標；需搭配其他圖標庫使用。 |
| -------------------------------     | -----------------------------------------          | -----------------------------------------------------------    |
| `@fortawesome/free-solid-svg-icons` | 僅包含實心風格的圖標                               | 按需載入實心圖標，優化效能。                                   |
