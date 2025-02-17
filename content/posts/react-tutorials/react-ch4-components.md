---
title: "CH4 React 元件說明"
date: 2025-01-29
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React 元件說明 🎯

## 什麼是 React 元件？
想像你在玩樂高，每個樂高積木就像是一個元件。你可以：
- 把小積木組合成大積木
- 重複使用相同的積木
- 每個積木都有自己的功能和外觀

## 元件的基本架構

### 最簡單的元件
```jsx
function Welcome() {
  return (
    <div>
      <h1>歡迎光臨！</h1>
    </div>
  );
}
```
這就是一個最基本的元件！它只會顯示「歡迎光臨！」。

### 帶有資料的元件
```jsx
function UserCard() {
  // 這裡放資料
  const name = '小明';
  const age = 25;

  // 這裡放要顯示的內容
  return (
    <div>
      <h2>{name}</h2>
      <p>年齡：{age}</p>
    </div>
  );
}
```

## 元件的兩個重要部分

### 1. 資料和邏輯部分（return 之前）
```jsx
function Counter() {
  // 這裡放資料
  const number = 0;
  
  // 這裡放函式
  const handleClick = () => {
    alert('你點擊了按鈕！');
  };

  // 下面是顯示的部分...
```

### 2. 顯示部分（return 之後）
```jsx
  // ...接上面的程式碼
  return (
    <div>
      <p>目前數字：{number}</p>
      <button onClick={handleClick}>
        點我
      </button>
    </div>
  );
}
```

## 常見的元件類型

### 1. 純顯示元件
只負責顯示內容，沒有互動功能：
```jsx
function Header() {
  return (
    <header>
      <h1>我的網站</h1>
      <nav>
        <a href="/">首頁</a>
        <a href="/about">關於</a>
      </nav>
    </header>
  );
}
```

### 2. 互動元件
可以跟使用者互動：
```jsx
function LikeButton() {
  const [likes, setLikes] = useState(0);

  return (
    <div>
      <p>按讚數：{likes}</p>
      <button onClick={() => setLikes(likes + 1)}>
        👍 按讚
      </button>
    </div>
  );
}
```

### 3. 表單元件
處理使用者輸入：
```jsx
function SimpleForm() {
  const [name, setName] = useState('');

  return (
    <div>
      <input 
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="請輸入名字"
      />
      <p>你好，{name}！</p>
    </div>
  );
}
```

## 如何使用元件

### 1. 在其他元件中使用
```jsx
function App() {
  return (
    <div>
      <Header />
      <UserCard />
      <LikeButton />
    </div>
  );
}
```

### 2. 重複使用元件
```jsx
function App() {
  return (
    <div>
      <LikeButton />  {/* 第一個按讚按鈕 */}
      <LikeButton />  {/* 第二個按讚按鈕 */}
      <LikeButton />  {/* 第三個按讚按鈕 */}
    </div>
  );
}
```

## 常見問題

### 1. 元件名稱要注意什麼？
- 必須以大寫字母開頭（例如：`UserCard` 而不是 `userCard`）
- 名稱要有意義（例如：`LoginForm` 而不是 `Form1`）

### 2. 為什麼我的元件沒有顯示？
檢查以下幾點：

#### a. 是否有正確匯出元件（export）
```jsx
// ❌ 錯誤：沒有匯出元件
function Welcome() {
  return <h1>歡迎光臨</h1>;
}

// ✅ 正確：記得加上 export
export function Welcome() {
  return <h1>歡迎光臨</h1>;
}

// ✅ 也可以這樣寫
function Welcome() {
  return <h1>歡迎光臨</h1>;
}
export default Welcome;
```

#### b. 是否有正確匯入元件（import）
```jsx
// ❌ 錯誤：路徑或檔名寫錯
import Welcome from './welcom';  // 檔名寫錯
import Welcome from '../Welcome';  // 路徑寫錯

// ✅ 正確：確認路徑和檔名
import Welcome from './Welcome';
import { Welcome } from './Welcome';  // 如果是用 export function
```

#### c. 元件名稱是否正確
```jsx
// ❌ 錯誤：元件名稱小寫開頭
function welcome() {
  return <h1>歡迎光臨</h1>;
}

// ❌ 錯誤：使用時名稱寫錯
function App() {
  return (
    <div>
      <welcome />  {/* 錯誤：小寫 */}
      <Welcom />   {/* 錯誤：拼錯 */}
    </div>
  );
}

// ✅ 正確：
function Welcome() {  // 大寫開頭
  return <h1>歡迎光臨</h1>;
}

function App() {
  return (
    <div>
      <Welcome />  {/* 正確 */}
    </div>
  );
}
```

#### d. return 是否有包含內容
```jsx
// ❌ 錯誤：沒有 return 內容
function Welcome() {
  const name = '小明';
  // 忘記 return
}

// ❌ 錯誤：return 後沒有內容
function Welcome() {
  return;  // 空的 return
}

// ❌ 錯誤：return 後有多個元素但沒有包裹
function Welcome() {
  return (
    <h1>標題</h1>
    <p>內容</p>    // 這樣會報錯
  );
}

// ✅ 正確：
function Welcome() {
  return (
    <div>
      <h1>標題</h1>
      <p>內容</p>
    </div>
  );
}

// ✅ 也可以用 Fragment
function Welcome() {
  return (
    <>
      <h1>標題</h1>
      <p>內容</p>
    </>
  );
}
```

#### e. 檔案結構範例
```
src/
├── components/
│   ├── Welcome.jsx       // 元件檔案
│   └── Welcome.css       // 樣式檔案（如果需要）
├── App.jsx
└── main.jsx
```

完整的使用範例：
```jsx
// src/components/Welcome.jsx
export function Welcome() {
  return <h1>歡迎光臨</h1>;
}

// src/App.jsx
import { Welcome } from './components/Welcome';

function App() {
  return (
    <div>
      <Welcome />
    </div>
  );
}
```

### 3. 我的元件要寫在哪裡？
- 建議每個元件都建立一個獨立的檔案
- 檔案名稱和元件名稱相同
- 放在 `src/components` 資料夾內



