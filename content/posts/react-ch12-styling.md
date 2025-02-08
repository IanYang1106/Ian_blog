---
title: "CH12 React 樣式管理"
date: 2025-02-08T23:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React 樣式管理 💅

## 什麼是樣式管理？
想像你在裝飾房間 🏠：
- 選擇牆壁顏色（背景色）
- 擺放家具位置（版面配置）
- 調整燈光明暗（特效）

React 也需要管理這些視覺效果！

## 專案資料夾結構 📁
建議的資料夾結構如下：

```
src/
├── styles/           # 樣式相關檔案
│   ├── global.css   # 全域樣式
│   ├── variables.css # CSS 變數
│   └── themes/      # 主題相關
│       ├── light.css
│       └── dark.css
│
├── components/       # 元件
│   ├── Button/
│   │   ├── Button.jsx
│   │   └── Button.module.css
│   └── Card/
│       ├── Card.jsx
│       └── Card.module.css
│
└── App.jsx          # 根元件
```

💡 **說明：**
- `styles/`: 存放全域樣式和共用的樣式設定
- `components/`: 每個元件都有自己的資料夾，包含元件檔和對應的樣式檔
- 使用 `.module.css` 作為 CSS Modules 的檔案命名規則

## 三種常見的樣式管理方式 🎨

### 1. 傳統 CSS 檔案
最簡單的方式，建立 `.css` 檔案並引入：

```css
/* styles.css */
/* 使用 .button 是 CSS class 選擇器 */
.button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.button:hover {
  background-color: #0056b3;
}

/* 如果要選擇所有的 button 標籤，則不需要點號 */
button {
  font-family: Arial, sans-serif;
}
```

```jsx
// App.jsx
import './styles.css';

function App() {
  return (
    {/* className="button" 使用 CSS class */}
    <button className="button">
      點擊我
    </button>
  );
}
```

### 2. CSS Modules
避免樣式衝突的好方法：

```css
/* Button.module.css */
.button {
  background-color: #28a745;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}

.primary {
  background-color: #007bff;
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ primary }) {
  return (
    <button className={`${styles.button} ${primary ? styles.primary : ''}`}>
      點擊我
    </button>
  );
}
```

### 3. Styled Components
直接在 JavaScript 中寫 CSS：

💡 **套件安裝說明：**
```bash
# 使用 npm 安裝
npm install styled-components
```

使用範例：

```jsx
// 從 styled-components 套件引入 styled 物件
import styled from 'styled-components';

// styled.button 會創建一個新的樣式化按鈕元件
// styled.button`` 使用樣板字符串(Template Literals)來寫 CSS
// 這裡的 button 代表要樣式化的 HTML 標籤
const StyledButton = styled.button`
  background-color: ${props => props.primary ? '#007bff' : '#28a745'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    opacity: 0.8;
  }
`;

// 使用創建好的樣式化元件
function Button({ primary }) {
  return (
    <StyledButton primary={primary}>
      點擊我
    </StyledButton>
  );
}
```

💡 **styled 物件說明：**
- `styled` 可以用來樣式化任何 HTML 標籤，例如：
  - `styled.div` - 樣式化 div 標籤
  - `styled.button` - 樣式化 button 標籤
  - `styled.input` - 樣式化 input 標籤
  - `styled.p` - 樣式化 p 標籤
- 使用樣板字符串 `` 來寫 CSS
- 可以使用 props 來動態改變樣式

## 實用樣式範例 🌈

### 1. 響應式卡片
首先，我們需要建立一個 CSS Module 檔案：

1. 在元件資料夾中建立 `Card.module.css`
2. 檔案命名必須使用 `.module.css` 結尾
3. 這個檔案裡面寫入我們的 CSS 樣式

```css
/* Card.module.css - 這是我們自己建立的檔案 */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  margin: 10px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .card {
    padding: 10px;
    margin: 5px;
  }
}
```

然後在 React 元件中引入並使用：

```jsx
// Card.jsx
import styles from './Card.module.css';  // 引入我們建立的 CSS Module

function Card({ title, content }) {
  return (
    <div className={styles.card}>
      <h2 className={styles.title}>{title}</h2>
      <p className={styles.content}>{content}</p>
    </div>
  );
}
```

💡 **CSS Module 檔案說明：**
- 檔案名稱必須用 `.module.css` 結尾
- 可以自由命名 class，React 會自動產生唯一的 class 名稱
- 使用 `styles.類別名稱` 的方式來引用樣式
- 這種方式可以避免樣式衝突

### 2. 主題切換
```jsx
import { useState } from 'react';
import styled from 'styled-components';

// 定義主題
const themes = {
  light: {
    background: '#ffffff',
    text: '#000000'
  },
  dark: {
    background: '#222222',
    text: '#ffffff'
  }
};

// 建立樣式化的容器
const Container = styled.div`
  background-color: ${props => themes[props.theme].background};
  color: ${props => themes[props.theme].text};
  padding: 20px;
  transition: all 0.3s ease;
`;

function ThemeSwitch() {
  const [theme, setTheme] = useState('light');

  return (
    <Container theme={theme}>
      <h1>主題切換示範</h1>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        切換主題
      </button>
    </Container>
  );
}
```

## 重要提醒 ⚠️

1. 選擇適合的方式
   - 小專案：傳統 CSS
   - 中型專案：CSS Modules
   - 大型專案：Styled Components

2. 保持一致性
   - 盡量使用同一種樣式管理方式
   - 建立統一的命名規則

3. 避免內聯樣式
   - 除非真的必要，否則避免使用 style 屬性
   - 內聯樣式難以維護且效能較差

## 常見問題 FAQ ❓

Q: CSS Modules 和一般 CSS 有什麼不同？
A: CSS Modules 會自動產生唯一的類別名稱，避免樣式衝突。

Q: 要如何選擇樣式管理方式？
A: 考慮專案大小、團隊偏好和維護性來選擇。

Q: Styled Components 需要額外安裝嗎？
A: 是的，需要安裝 `styled-components` 套件。

## 練習時間 💪
試著製作一個簡單的部落格卡片：
1. 使用 CSS Modules
2. 包含標題、內容和按鈕
3. 加入響應式設計
4. 製作暗色主題版本
