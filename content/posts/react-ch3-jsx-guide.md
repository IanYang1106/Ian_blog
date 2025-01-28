---
title: "CH3 JSX 簡單說明"
date: 2025-01-28
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# JSX 簡單說明 🚀

## 什麼是 JSX？
JSX 是一種特殊的 JavaScript 語法，讓你可以在 JavaScript 中寫 HTML。
這樣說可能有點抽象，讓我們看個例子：

```jsx
// 一般的 HTML
<div>
  <h1>歡迎光臨</h1>
  <p>這是我的網站</p>
</div>

// 用 JSX 寫
function Welcome() {
  return (
    <div>
      <h1>歡迎光臨</h1>
      <p>這是我的網站</p>
    </div>
  );
}
```

## React 函式元件的結構
React 的函式元件主要分為兩個區域：

### 1. 邏輯區域（return 之前）
這裡寫所有的 JavaScript 程式邏輯：
- 宣告變數
- 定義函式
- 處理資料
- 使用 Hooks（如 useState）

### 2. 渲染區域（return 之後）
這裡寫要顯示的 JSX 內容：
- HTML 結構
- 插入變數和運算結果
- 使用條件渲染
- 使用迴圈顯示列表

讓我們看個完整的例子：
```jsx
function UserProfile() {
  // 邏輯區域：
  const [name, setName] = useState('小明');  // 宣告 state
  const age = 25;                           // 宣告一般變數
  
  // 定義處理函式
  const handleClick = () => {
    setName('小華');
  };
  
  // 資料處理
  const displayText = `${name} 今年 ${age} 歲`;

  // 渲染區域：
  return (
    <div>
      <h1>{displayText}</h1>
      <button onClick={handleClick}>
        改名字
      </button>
    </div>
  );
}
```

### 常見用法

1. **在邏輯區域處理資料，渲染區域顯示結果**
```jsx
function NumberList() {
  // 邏輯區域：處理資料
  const numbers = [1, 2, 3, 4, 5];
  const doubledNumbers = numbers.map(num => num * 2);
  
  // 渲染區域：顯示結果
  return (
    <ul>
      {doubledNumbers.map(num => (
        <li key={num}>{num}</li>
      ))}
    </ul>
  );
}
```

2. **在邏輯區域定義事件處理函式**
```jsx
function Form() {
  // 邏輯區域：定義處理函式
  const handleSubmit = (e) => {
    e.preventDefault();
    alert('表單送出！');
  };
  
  // 渲染區域：使用處理函式
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">送出</button>
    </form>
  );
}
```

3. **條件渲染的準備工作**
```jsx
function Greeting() {
  // 邏輯區域：準備條件判斷
  const isLoggedIn = true;
  const userName = '小明';
  const welcomeText = isLoggedIn 
    ? `歡迎回來，${userName}！`
    : '請先登入';
  
  // 渲染區域：顯示結果
  return (
    <div>
      <h1>{welcomeText}</h1>
    </div>
  );
}
```

### 注意事項
1. 邏輯區域可以寫任何 JavaScript 程式碼
2. 渲染區域只能包含 JSX 語法和簡單的表達式
3. 渲染區域不能直接寫 if 判斷，要改用三元運算子
4. 複雜的邏輯建議在邏輯區域處理完，再在渲染區域顯示結果

## JSX 基本使用方法

### 1. 在 JSX 中放入變數
```jsx
function Greeting() {
  const name = '小明';
  const age = 25;
  
  return (
    <div>
      <h1>你好，{name}！</h1>
      <p>你今年 {age} 歲</p>
    </div>
  );
}
```

### 2. 使用簡單的運算
```jsx
function Counter() {
  const count = 5;
  
  return (
    <div>
      <p>目前數字：{count}</p>
      <p>數字加十：{count + 10}</p>
      <p>數字乘二：{count * 2}</p>
    </div>
  );
}
```

### 3. 設定 CSS 樣式
```jsx
// 方法一：使用 className（注意不是 class！）
function StyleDemo1() {
  return (
    <div className="title">
      這是標題
    </div>
  );
}

// 方法二：使用 style 屬性
function StyleDemo2() {
  return (
    <div style={{
      color: 'blue',           // 文字顏色
      fontSize: '20px',        // 文字大小
      backgroundColor: 'pink'  // 背景顏色
    }}>
      這是藍色文字、粉紅背景
    </div>
  );
}
```

### 4. 處理點擊事件
```jsx
function Button() {
  // 當按鈕被點擊時會執行這個函式
  const handleClick = () => {
    alert('你點擊了按鈕！');
  };

  return (
    <button onClick={handleClick}>
      點我試試
    </button>
  );
}
```

### 5. 條件顯示內容
```jsx
function Greeting() {
  const isLoggedIn = true;  // 可以改成 false 看看

  return (
    <div>
      {isLoggedIn ? (
        <h1>歡迎回來！</h1>
      ) : (
        <h1>請先登入</h1>
      )}
    </div>
  );
}
```

### 6. 顯示列表
```jsx
function FruitList() {
  const fruits = ['蘋果', '香蕉', '橘子'];

  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

## JSX 注意事項

### 1. 常見錯誤
```jsx
// ❌ 錯誤：JSX 只能有一個根元素
function Wrong() {
  return (
    <h1>標題</h1>
    <p>內容</p>
  );
}

// ✅ 正確：用一個 div 包起來
function Correct() {
  return (
    <div>
      <h1>標題</h1>
      <p>內容</p>
    </div>
  );
}

// ✅ 更好：使用 <> </> 包起來（不會產生多餘的 div）
function Better() {
  return (
    <>
      <h1>標題</h1>
      <p>內容</p>
    </>
  );
}
```

### 2. HTML 和 JSX 的不同
```jsx
// HTML 寫法
<div class="container">
  <label for="name">姓名：</label>
  <input type="text">
</div>

// JSX 寫法
<div className="container">
  <label htmlFor="name">姓名：</label>
  <input type="text" />
</div>
```

## 實用小技巧

### 1. 製作一個簡單的計數器
```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>目前數字：{count}</p>
      <button onClick={() => setCount(count + 1)}>
        +1
      </button>
      <button onClick={() => setCount(count - 1)}>
        -1
      </button>
    </div>
  );
}
```

### 2. 製作一個簡單的待辦事項
```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    '學習 React',
    '學習 JSX',
    '製作專案'
  ]);

  return (
    <div>
      <h2>待辦事項</h2>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 可能遇到的問題

### 1. 為什麼我的變數沒有顯示出來？
- 記得要用大括號 `{}` 包住變數
- 例如：`<p>{myVariable}</p>`

### 2. 為什麼我的樣式沒有生效？
- 記得用 `className` 而不是 `class`
- style 要用兩層大括號：`style={{ color: 'red' }}`

### 3. 為什麼我的列表要加 key？
- key 幫助 React 識別哪些項目改變了
- 通常用資料的 id 或索引（index）當作 key
