---
title: "CH7 React 事件處理"
date: 2025-02-03T14:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React 事件處理 🖱️

## 什麼是事件？
想像你在使用手機 App：
- 點擊按鈕發送訊息 👆
- 在輸入框打字 ⌨️
- 按下送出鈕 ✉️

這些都是「事件」！就是使用者對你的 App 做的動作。

## 最常用的三種事件

### 1. 點擊事件（最簡單！）
```jsx
function App() {
  // 當按鈕被點擊時要做的事
  const sayHello = () => {
    alert('哈囉！');
  };

  return (
    <button onClick={sayHello}>
      點我打招呼
    </button>
  );
}
```

### 2. 輸入文字事件
```jsx
function App() {
  const [text, setText] = useState('');  // 記得要 import useState

  return (
    <div>
      <input 
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="請輸入文字"
      />
      <p>你輸入的文字：{text}</p>
    </div>
  );
}
```

### 3. 表單送出事件
```jsx
function App() {
  const handleSubmit = (e) => {
    e.preventDefault();  // 防止網頁重新整理
    alert('表單已送出！');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="請輸入名字" />
      <button type="submit">送出</button>
    </form>
  );
}
```

## 實用小範例

### 1. 按讚按鈕
```jsx
function LikeButton() {
  const [likes, setLikes] = useState(0);

  return (
    <div>
      <button onClick={() => setLikes(likes + 1)}>
        👍 按讚 ({likes})
      </button>
    </div>
  );
}
```

### 2. 簡單的待辦清單
```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);  // 儲存所有待辦事項
  const [input, setInput] = useState('');  // 儲存輸入框的文字

  // 新增待辦事項
  const addTodo = () => {
    if (input !== '') {  // 確認有輸入文字
      setTodos([...todos, input]);  // 新增到清單
      setInput('');  // 清空輸入框
    }
  };

  return (
    <div>
      {/* 輸入框和新增按鈕 */}
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="輸入待辦事項"
      />
      <button onClick={addTodo}>新增</button>

      {/* 顯示待辦清單 */}
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
    </div>
  );
}
```
## 常見問題

### Q1: onClick 的大小寫要注意嗎？
是的！React 中的事件名稱都要用駝峰式命名：
- ✅ `onClick`
- ✅ `onChange`
- ❌ `onclick`
- ❌ `onchange`

### Q2: 為什麼要用箭頭函式？
因為這樣寫最簡單，不用擔心 `this` 的問題：
```jsx
// ✅ 建議的寫法
<button onClick={() => alert('hi')}>

// ❌ 不建議的寫法
<button onClick={function() { alert('hi') }}>
```
### Q3: e.preventDefault() 是什麼？
它可以防止表單送出時網頁重新整理，在 React 中很常用！

## 小提醒
1. 記得要 `import { useState } from 'react'`

## 附註：關於 onChange 事件
`onChange` 是 HTML input 元素內建的事件，在 React 中經常使用。以下是重要說明：

1. **原生 HTML vs React**
   - 原生 HTML：`onchange`（全小寫）
   - React 版本：`onChange`（駝峰式命名）

2. **觸發時機**
   - 每次輸入框的值改變時觸發
   - 包含：打字、貼上、刪除等操作

3. **事件物件 (e)**
   - `e.target`：觸發事件的元素（這裡是 input）
   - `e.target.value`：可以取得輸入框的最新值

4. **相關的其他 input 事件**
   - `onFocus`：輸入框獲得焦點（被點擊）時觸發
   - `onBlur`：輸入框失去焦點時觸發
   - `onKeyDown`：鍵盤按下時觸發
   - `onKeyUp`：鍵盤放開時觸發


