---
title: "專案環境建置說明"
date: 2024-12-22
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

## 環境建置流程

以下為專案環境的完整建置步驟，涵蓋了基礎工具、前端框架與開發輔助工具的安裝與配置。

---

### 安裝 esbuild

#### 安裝指令

使用以下指令安裝 `esbuild`，用於快速打包前端資源：

```bash
npm install --save-dev esbuild
```

#### 設定 `package.json`

在 `package.json` 中新增執行指令：

```json
"scripts": {
  "dev": "esbuild frontend/scripts/app.js --bundle --outfile=static/scripts/app.js --watch",
  "build": "esbuild frontend/scripts/app.js --bundle --outfile=static/scripts/app.js",
  "css": "tailwindcss -i ./frontend/styles/app.css -o ./static/styles/app.css --watch"
}
```

#### 建立檔案結構

新增 `frontend` 資料夾以存放待引入的資源檔案。

---

### 安裝 Tailwind CSS

#### 安裝指令

執行以下指令安裝 Tailwind 與相關工具：

```bash
npm install -D tailwindcss postcss autoprefixer
```

#### 生成設定檔

執行以下指令生成 Tailwind 的 `tailwind.config.js`：

```bash
npx tailwindcss init
```

#### 修改設定檔

設定 Tailwind 的掃描範圍，新增 `content`：

```javascript
content: [
  './frontend/**/*.html',
  './frontend/**/*.js'
],
```

#### 引入 Tailwind CSS

在 `frontend/styles` 資料夾中建立 `app.css`，內容如下：

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### 生成壓縮檔

執行以下指令生成 CSS 壓縮檔：

```bash
npm run css
```

#### 配置 Django 靜態檔案

在 `settings.py` 設定靜態檔案路徑：

```python
STATIC_URL = "static/"
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
```

#### 在 HTML 引入樣式

在 HTML 檔案中，載入 Tailwind 的樣式表：

```html
{% load static %} <link rel="stylesheet" href="{% static 'styles/app.css' %}" />
```

---

### 安裝 DaisyUI

#### 安裝指令

執行以下指令安裝 DaisyUI：

```bash
npm install -D daisyui
```

#### 配置插件

在 `tailwind.config.js` 加載 DaisyUI：

```javascript
plugins: [
  require("daisyui"),
],
```

#### 啟用主題

配置 DaisyUI 主題，範例如下：

```javascript
daisyui: {
  themes: ["light", "dark", "cupcake", "emerald"], // 自定義主題
},
```

---

### 安裝 Alpine.js

#### 安裝指令

執行以下指令安裝 Alpine.js：

```bash
npm install alpinejs
```

#### 配置資源

在 `frontend/scripts` 中新增 `app.js` 並引入 Alpine.js：

```javascript
import Alpine from "alpinejs";

window.Alpine = Alpine;

Alpine.start();
```

#### 啟動開發模式

執行以下指令啟動資源監控：

```bash
npm run dev
```

---

### 安裝 HTMX

#### 安裝指令

執行以下指令安裝 HTMX：

```bash
npm install htmx.org
```

#### 配置資源

修改 `frontend/scripts/app.js`，引入 HTMX：

```javascript
import Alpine from "alpinejs";
import htmx from "htmx.org";

window.Alpine = Alpine;
window.htmx = htmx;

Alpine.start();
```

#### 測試

在瀏覽器中執行以下指令確認 HTMX 是否成功載入：

```javascript
console.log(window.htmx);
```

---

### 安裝 Commitizen

#### 安裝指令

執行以下指令安裝 Commitizen 與適配器：

```bash
npm install --save-dev commitizen cz-conventional-changelog
```

#### 配置 Commitizen

修改 `package.json` 設定：

```json
"scripts": {
  "commit": "cz"
},
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

#### 執行提交

使用以下指令進行規範化提交：

```bash
npm run commit
```

---

### 安裝 Pre-commit

#### 安裝指令

使用 Poetry 安裝 Pre-commit：

```bash
poetry add --dev pre-commit
```

#### 安裝 Git Hook

執行以下指令安裝 Git Hook：

```bash
poetry run pre-commit install
```

#### 建立設定檔

新增 `.pre-commit-config.yaml`，內容如下：

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-added-large-files
      - id: check-json

  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: "v2.4.0"
    hooks:
      - id: prettier
        args: ["--write"]
        types: [javascript, json, css, yaml]

exclude: "venv/|.venv/|node_modules/|migrations/|dist/"
```

#### 測試

執行以下指令測試 Pre-commit：

```bash
pre-commit run --all-files
```

---
