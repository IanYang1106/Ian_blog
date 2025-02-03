---
title: "CH8 React Hooks 入門指南"
date: 2025-02-03T18:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React Hooks 入門指南 🎣

## 什麼是 Hooks？
想像你在玩遊戲 🎮：
- 需要記錄分數（useState 記錄資料）
- 需要在遊戲開始時播放音樂（useEffect 處理特效）
- 需要在遊戲結束時儲存分數（useEffect 處理儲存）

Hooks 就是一組幫助你完成這些功能的工具！

## 三個基本的 Hooks

### 1. useState（記錄資料）
最常用的 Hook！我們已經用過很多次了：
```jsx
function Counter() {
  // count 是資料，setCount 是修改資料的方法
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h2>目前計數：{count}</h2>
      <button onClick={() => setCount(count + 1)}>
        點我 +1
      </button>
    </div>
  );
}
```

### 2. useEffect（處理特效）
當你需要在「特定時機」做某件事時使用：

```jsx
function Welcome() {
  const [name, setName] = useState('');

  // 🎬 網頁載入時，跟使用者打招呼
  useEffect(() => {
    alert('歡迎光臨！');
  }, []); // 空陣列代表「只在網頁載入時執行一次」

  // 👋 當名字改變時，打招呼
  useEffect(() => {
    if (name !== '') {
      alert(`哈囉，${name}！`);
    }
  }, [name]); // [name] 代表「當 name 改變時執行」

  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="請輸入你的名字"
    />
  );
}
```

### 3. useRef（操作畫面元素）
當你需要直接操作畫面上的元素時使用：

```jsx
function AutoFocus() {
  // 建立一個參考，指向輸入框
  const inputRef = useRef(null);

  // 網頁載入時，自動聚焦到輸入框
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return (
    <input 
      ref={inputRef}
      placeholder="我會自動聚焦喔！"
    />
  );
}
```

## 實用小範例

### 1. 計數器（結合 useState 和 useEffect）
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  // 當計數改變時，顯示訊息
  useEffect(() => {
    // 只在計數大於 0 時顯示
    if (count > 0) {
      alert(`你已經點了 ${count} 次！`);
    }
  }, [count]);

  return (
    <div>
      <h2>目前計數：{count}</h2>
      <button onClick={() => setCount(count + 1)}>
        點我 +1
      </button>
    </div>
  );
}
```

### 2. 自動儲存備忘錄
```jsx
function Memo() {
  // 初始化時從 localStorage 讀取之前儲存的內容
  const [text, setText] = useState(() => {
    return localStorage.getItem('memo') || '';
  });

  // 當文字改變時，自動儲存
  useEffect(() => {
    // 儲存到 localStorage，重新整理後仍會保留
    localStorage.setItem('memo', text);
  }, [text]);

  return (
    <div>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="寫些什麼吧！會自動儲存喔"
      />
      <p>目前字數：{text.length}</p>
    </div>
  );
}
```

## 需要要注意的地方

### ⚠️ 使用 useEffect 時：
1. `[]` - 空陣列 = 只在網頁載入時執行一次
2. `[資料]` - 有資料 = 當這個資料改變時執行
3. 不寫陣列 = 任何資料改變都執行（很少用到）

### ⚠️ 常見問題：
1. Hooks 只能在最上層使用，不能放在 if 或 for 迴圈裡面
2. Hooks 只能在 React 的函式裡面使用
3. Hooks 的名字一定要用 "use" 開頭

## 小提醒 💡
1. useState 用來記錄資料
2. useEffect 用來處理特效和其他操作
3. useRef 用來操作畫面元素
