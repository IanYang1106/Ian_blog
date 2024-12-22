### 環境建置

#### 安裝 esbuild

```
npm install --save-dev esbuild
```

package.json
設定執行檔

```json
  "scripts": {
    "dev": "esbuild frontend/scripts/app.js --bundle --outfile=static/scripts/app.js --watch",
    "build": "esbuild frontend/scripts/app.js --bundle --outfile=static/scripts/app.js",
    "css": "tailwindcss -i ./frontend/styles/app.css -o ./static/styles/app.css --watch"
  },
```

創建 frontends 資料夾放置準備引入的檔案

#### 安裝 tailwind

安裝指令

```
npm install -D tailwindcss postcss autoprefixer
```

生成 tailwind config 檔案

```
npx tailwindcss init
```

修改 config 檔案，主要讓 tailwind 知道哪些檔案需要掃描，這樣就知道在這一層裡的 frontend 裡的資料夾底下有 html 與 js 副檔名檔案都需要被掃描

```js
  content: [
    './frontend/**/*.html',
    './frontend/**/*.js'
  ],
```

在 css 中引入 tailwind，在資料夾 frontend/styles 建立 app.css 檔案如下

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

輸入指令讓 esbuild 產生 css 壓縮檔，這邊在前面 package.json 已有設定關於 css 的指令，所以直接執行即可

```
npm run css
```

django settings.py 需要做 static 的資料夾讀取設定，以確保能正確讀到 tailwind link 的檔案

```python
STATIC_URL = "static/"
STATICFILES_DIRS = [
    BASE_DIR / "static",  # 確保這裡指向包含 styles 的目錄
]
```

html 最前面先加上下列指令，啟用 static 標籤，使其可以用 `{% static "文件路徑" %}`語法來生成靜態文件的 URL

```
{% load static %}
```

在 html 檔案中加入以下 link 載入 tailwind 樣式

```html
<link rel="stylesheet" href="{% static 'styles/app.css' %}" />
```

#### 安裝 daisyUI

使用 npm 安裝

```
npm install -D daisyui
```

配置 Tailwind CSS 使用 DaisyUI，修改 tailwind.config.js，增加 DaisyUI 到 plugins

```js
  plugins: [
    require("daisyui"), // 添加 DaisyUI 插件
  ],
```

也可以設定一些主題直接套用在裡面，一樣修改 tailwind.config.js，增加 daisyui 項目，依照希望套入的主題選擇

```js
  daisyui: {
    themes: ["light", "dark", "cupcake", "emerald"], // 指定啟用的主題
  },
```

假設想要把目前所有提供的主題都放入，大致如下

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./frontend/**/*.html",
    "./frontend/**/*.js",
    "./templates/**/*.html",
    "./*/templates/**/*.html",
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require("daisyui"), // 使用 require 加載 DaisyUI 插件
  ],
  daisyui: {
    themes: [
      "light",
      "dark",
      "cupcake",
      "bumblebee",
      "emerald",
      "corporate",
      "synthwave",
      "retro",
      "cyberpunk",
      "valentine",
      "halloween",
      "garden",
      "forest",
      "aqua",
      "lofi",
      "pastel",
      "fantasy",
      "wireframe",
      "black",
      "luxury",
      "dracula",
      "cmyk",
      "autumn",
      "business",
      "acid",
      "lemonade",
      "night",
      "coffee",
      "winter",
      "dim",
      "nord",
      "sunset",
    ], // 指定啟用的主題
  },
};
```

在 html 中測試選擇啟用的主題是否能正常套用，如下方，套用的是 dark，應用在全局環境

```html
<html data-theme="dark">
  <body>
    <button class="btn btn-primary">Dark Theme Button</button>
  </body>
</html>
```

#### 安裝 alpinejs

使用 npm 安裝，輸入下列指令

```
npm install alpinejs
```

在 frontend/scripts 下創建 app.js，引入 alpinejs 檔案，內容如下

```js
import Alpine from "alpinejs";

window.Alpine = Alpine;

Alpine.start();
```

使用前面已在 package.json 中建立的指令

```
npm run dev
```

在 html 載入 alpinejs 的檔案，有加 type="module"就不需要自己加上 defer

```html
<script src="{% static "scripts/app.js" %}" type="module"></script>
```

#### 安裝 HTMX

```
npm install htmx.org
```

修改 frontend 路徑下 scripts 資料夾中的 app.js

```js
import Alpine from "alpinejs";
import htmx from "htmx.org";

window.Alpine = Alpine;
window.htmx = htmx;

Alpine.start();
```

測試是否產生效果，在中控台輸入下列指令確定有帶出東西出來

```
console.log(window.htmx);
```

#### 安裝 cz commit

進行安裝 commitizen 和 cz-conventional-changelog

```
npm install --save-dev commitizen cz-conventional-changelog
```

設定 package.json 檔案中的 scripts

```json
  "scripts": {
    "commit": "cz"
  },
```

設定 package.json 檔案中的 config，這段用來告訴 commitizen 使用的適配器來源，這個來源也是前面與 commitizen 一起裝設的

```json
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

使用 npm run commit 取代 git commit

```
npm run commit
```

### 下面關於 Pre-commit 這次並未更新，若有興趣，仍可以在自己電腦嘗試使用

#### 安裝 pre-commit

使用 poetry 安裝，並且指定為開發階段使用

```
poetry add --dev pre-commit
```

確認 pre-commit 安裝裝態

```
poetry show pre-commit
```

安裝 Git Hook

```
poetry run pre-commit install
```

創建一個.pre-commit-config.yaml 檔案，設定 commit 前格式整理的方法

```
touch .pre-commit-config.yaml
```

.pre-commit-config.yaml 檔案內容

```yaml
repos:
  # 通用 Pre-commit hooks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-added-large-files
      - id: check-json

  # Python 格式化工具 Black
  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black

  # Python 導入排序工具 isort
  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  # JavaScript 格式化工具 Prettier
  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: "v2.4.0" # 最新版本
    hooks:
      - id: prettier
        args: ["--write"]
        types: [javascript, json, css, yaml]

exclude: "venv/|.venv/|node_modules/|migrations/|dist/"
```

測試是否有正常工作

```
pre-commit run --all-files
```
