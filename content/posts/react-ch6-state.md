# React State 入門指南 🔄

## 什麼是 State？
想像你正在使用手機：
- 電池電量會隨著使用而減少 🔋
- 未讀訊息數量會隨著收到新訊息而增加 📱
- 音量大小可以隨時調整 🔊

State 就像是這些「會改變的數值」，當這些數值改變時，畫面也會跟著更新！

## 開始使用 State 的三個步驟

### 1️⃣ 匯入 useState
```jsx
import { useState } from 'react';  // 記得要先匯入！
```

### 2️⃣ 宣告一個 State
```jsx
const [count, setCount] = useState(0);
// count：目前的值
// setCount：用來改變值的函式
// useState(0)：0 是初始值
```

### 3️⃣ 使用和更新 State
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>目前數字：{count}</p>
      <button onClick={() => setCount(count + 1)}>
        點我 +1
      </button>
    </div>
  );
}
```

## State 的基本使用範例

### 1. 簡單的開關
```jsx
function Toggle() {
  // 初始值是 false（代表「關」的狀態）
  const [isOn, setIsOn] = useState(false);

  // 處理點擊事件的函式
  const handleClick = () => {
    setIsOn(!isOn);  // 每次點擊都會把狀態反轉
  };

  return (
    <div>
      <button onClick={handleClick}>
        {isOn ? '開' : '關'}  {/* 根據狀態顯示不同文字 */}
      </button>
      <p>目前狀態：{isOn ? '開啟' : '關閉'}</p>
    </div>
  );
}

// 使用效果說明：
// 1. 初始狀態：
//    - isOn = false
//    - 按鈕顯示「關」
//    - 狀態顯示「關閉」
//
// 2. 點擊按鈕：
//    - setIsOn(!false) → setIsOn(true)
//    - 按鈕顯示「開」
//    - 狀態顯示「開啟」
//
// 3. 再次點擊：
//    - setIsOn(!true) → setIsOn(false)
//    - 按鈕顯示「關」
//    - 狀態顯示「關閉」
```

### 2. 個人資料表單
```jsx
function UserForm() {
  // 設定初始值為空字串
  const [name, setName] = useState('');
  const [age, setAge] = useState('');

  return (
    <div>
      <input 
        value={name}  // input 的值會跟著 name 狀態改變
        onChange={(e) => setName(e.target.value)}  
        // e.target.value 是使用者輸入的新值
        // 例如：輸入「小明」
        // e.target.value 就會是「小明」
        // setName('小明') 就會更新 name 狀態
        placeholder="請輸入姓名"
      />
      <input 
        value={age}
        onChange={(e) => setAge(e.target.value)}
        placeholder="請輸入年齡"
      />
      <p>你好，{name}！</p>
    </div>
  );
}

// 運作流程：
// 1. 使用者在輸入框輸入文字
// 2. 觸發 onChange 事件
// 3. e.target.value 取得輸入框的最新值
// 4. setName 更新 name 狀態
// 5. input 的 value 會跟著 name 狀態更新
// 6. 畫面上的問候語也會跟著更新
```

## State 的注意事項

### 1. ⚠️ State 的更新是非同步的
```jsx
const [count, setCount] = useState(0);

// ❌ 錯誤寫法
setCount(count + 1);
console.log(count);  // 這裡還是會顯示舊的值！

// ✅ 正確寫法：如果要用最新的值，請使用 useEffect 或放在下一次渲染
```

### 2. ⚠️ 更新物件型態的 State
```jsx
const [user, setUser] = useState({
  name: '小明',
  age: 25
});

// ❌ 錯誤寫法：直接修改
user.age = 26;  

// ✅ 正確寫法：創建新物件
setUser({
  ...user,    // 保留原本的所有資料
  age: 26     // 更新年齡
});
```

### 3. ⚠️ 更新陣列型態的 State
```jsx
const [fruits, setFruits] = useState(['蘋果', '香蕉']);

// ❌ 錯誤寫法：直接修改陣列
fruits.push('橘子');  

// ✅ 正確寫法：創建新陣列
setFruits([...fruits, '橘子']);
```

## 實用小技巧

### 1. 表單處理小幫手
```jsx
function Form() {
  const [form, setForm] = useState({
    username: '',
    email: ''
  });

  // 一個函式處理所有輸入框
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm({
      ...form,
      [name]: value
    });
  };

  return (
    <form>
      <input
        name="username"
        value={form.username}
        onChange={handleChange}
        placeholder="使用者名稱"
      />
      <input
        name="email"
        value={form.email}
        onChange={handleChange}
        placeholder="電子郵件"
      />
    </form>
  );
}
```

### 2. 計數器進階版
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(0)}>重設</button>
    </div>
  );
}
```

## 常見問題

### Q1: 為什麼我的 State 沒有立即更新？
因為 React 的 State 更新是非同步的，這是為了效能優化。如果你需要在更新後立即使用新的值，可以使用 useEffect。

### Q2: 可以在條件式中使用 useState 嗎？
不行！useState 必須在元件的最上層使用，不能放在 if 條件式或迴圈中。

### Q3: 要如何同時更新多個 State？
每個 setState 都是獨立的，你可以連續呼叫多個 setState：
```jsx
setName('小明');
setAge(25);
setCity('台北');
```

記住：State 就像是元件的記憶體，負責儲存會改變的資料。只要 State 改變，React 就會自動更新畫面！ 🎉 

## State 的檔案組織方式

### 1. 簡單元件的 State（單一檔案）
```jsx
// src/components/Counter.jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  );
}

export default Counter;
```

### 2. 共享 State 到子元件（多個檔案）
```jsx
// src/components/TodoForm.jsx
function TodoForm({ onAdd }) {
  return (
    <button onClick={() => onAdd('新待辦事項')}>
      新增待辦
    </button>
  );
}

export default TodoForm;

// src/components/TodoList.jsx
function TodoList({ items, onDelete }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          {item}
          <button onClick={() => onDelete(index)}>刪除</button>
        </li>
      ))}
    </ul>
  );
}

export default TodoList;

// src/pages/TodoPage.jsx
import { useState } from 'react';
import TodoForm from '../components/TodoForm';
import TodoList from '../components/TodoList';

function TodoPage() {
  const [todos, setTodos] = useState(['學習 React']);

  const handleAdd = (newTodo) => {
    setTodos([...todos, newTodo]);
  };

  const handleDelete = (index) => {
    setTodos(todos.filter((_, i) => i !== index));
  };

  return (
    <div>
      <h1>待辦事項</h1>
      <TodoForm onAdd={handleAdd} />
      <TodoList items={todos} onDelete={handleDelete} />
    </div>
  );
}

export default TodoPage;
```

### 3. State 管理的建議
- 將 State 放在需要共享這個資料的元件們的最近共同父元件中
- 子元件通過 Props 接收資料和操作函式
- 盡量讓元件的職責單一，例如：
  - `TodoForm`：負責輸入介面
  - `TodoList`：負責顯示列表
  - `TodoPage`：負責管理資料（State）

### 4. 檔案命名規則
- 元件檔案使用大寫開頭（例如：`TodoList.jsx`）
- State 變數使用小駝峰式命名（例如：`userInfo`）
- State 更新函式用 `set` 開頭（例如：`setUserInfo`） 

## React 的其他狀態管理方式

### 1. useReducer - 適合複雜的狀態邏輯
```jsx
import { useReducer } from 'react';

// 定義狀態的更新規則
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  // 使用 useReducer 來管理狀態
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>計數: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>重設</button>
    </div>
  );
}
```

### 2. Context - 適合跨元件共享狀態
```jsx
import { createContext, useContext, useState } from 'react';

// 創建 Context
const ThemeContext = createContext();

// 提供者元件
function ThemeProvider({ children }) {
  const [isDark, setIsDark] = useState(false);

  return (
    <ThemeContext.Provider value={{ isDark, setIsDark }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 子元件可以直接使用 Context 中的狀態
function ThemeButton() {
  const { isDark, setIsDark } = useContext(ThemeContext);

  return (
    <button onClick={() => setIsDark(!isDark)}>
      切換到{isDark ? '淺色' : '深色'}主題
    </button>
  );
}

// 使用方式
function App() {
  return (
    <ThemeProvider>
      <div>
        <h1>我的應用</h1>
        <ThemeButton />
      </div>
    </ThemeProvider>
  );
}
```
