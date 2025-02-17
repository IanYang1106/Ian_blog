---
title: "CH12 React 樣式管理"
date: 2025-02-17T23:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# Todo List 專案實作 ✅

## 專案說明
我們要做一個待辦事項清單：
- 可以新增待辦事項 ➕
- 可以刪除待辦事項 ❌
- 可以標記完成/未完成 ✓
- 可以篩選顯示項目 🔍

## 第一步：建立基本結構

### 1. 建立專案資料夾結構
```bash
src/
  ├── components/
  │   ├── TodoForm.jsx    # 新增待辦表單
  │   ├── TodoItem.jsx    # 單個待辦項目
  │   ├── TodoList.jsx    # 待辦清單
  │   └── TodoFilter.jsx  # 篩選器
  └── App.jsx
```

### 2. 設定基本狀態
```jsx
// App.jsx
import { useState } from 'react';
import TodoForm from './components/TodoForm';
import TodoList from './components/TodoList';
import TodoFilter from './components/TodoFilter';

function App() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all'); // 'all', 'active', 'completed'

  return (
    <div className="app">
      <h1>待辦事項清單</h1>
      <TodoForm onAdd={(text) => {/* 待實作 */}} />
      <TodoFilter filter={filter} onFilterChange={setFilter} />
      <TodoList todos={todos} onToggle={(id) => {/* 待實作 */}} />
    </div>
  );
}
```

## 第二步：實作各個元件

### 1. 新增待辦表單
```jsx
// components/TodoForm.jsx
function TodoForm({ onAdd }) {
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');  // 清空輸入框
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="新增待辦事項..."
      />
      <button type="submit">新增</button>
    </form>
  );
}
```

### 2. 待辦項目元件
```jsx
// components/TodoItem.jsx
function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{ 
        textDecoration: todo.completed ? 'line-through' : 'none' 
      }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>刪除</button>
    </li>
  );
}
```

### 3. 待辦清單元件
```jsx
// components/TodoList.jsx
function TodoList({ todos, onToggle, onDelete }) {
  if (todos.length === 0) {
    return <p>目前沒有待辦事項</p>;
  }

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
          onDelete={onDelete}
        />
      ))}
    </ul>
  );
}
```

### 4. 篩選器元件
```jsx
// components/TodoFilter.jsx
function TodoFilter({ filter, onFilterChange }) {
  return (
    <div>
      <button 
        onClick={() => onFilterChange('all')}
        disabled={filter === 'all'}
      >
        全部
      </button>
      <button 
        onClick={() => onFilterChange('active')}
        disabled={filter === 'active'}
      >
        未完成
      </button>
      <button 
        onClick={() => onFilterChange('completed')}
        disabled={filter === 'completed'}
      >
        已完成
      </button>
    </div>
  );
}
```

## 第三步：完成主要功能

### 1. 新增功能
```jsx
// App.jsx
const addTodo = (text) => {
  const newTodo = {
    id: Date.now(),  // 用時間戳當作 id
    text,
    completed: false
  };
  setTodos([...todos, newTodo]);
};
```

### 2. 切換完成狀態
```jsx
// App.jsx
const toggleTodo = (id) => {
  setTodos(todos.map(todo => 
    todo.id === id 
      ? { ...todo, completed: !todo.completed }
      : todo
  ));
};
```

### 3. 刪除功能
```jsx
// App.jsx
const deleteTodo = (id) => {
  setTodos(todos.filter(todo => todo.id !== id));
};
```

### 4. 篩選功能
```jsx
// App.jsx
const filteredTodos = todos.filter(todo => {
  if (filter === 'active') return !todo.completed;
  if (filter === 'completed') return todo.completed;
  return true;  // 'all' 的情況
});
```

## 加分功能

### 1. 本地儲存
```jsx
// App.jsx
// 從 localStorage 讀取初始資料
const [todos, setTodos] = useState(() => {
  const saved = localStorage.getItem('todos');
  return saved ? JSON.parse(saved) : [];
});

// 當 todos 改變時儲存
useEffect(() => {
  localStorage.setItem('todos', JSON.stringify(todos));
}, [todos]);
```

### 2. 拖拉排序
```jsx
// 需要安裝 react-beautiful-dnd
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

// 處理拖拉結束事件
const handleDragEnd = (result) => {
  if (!result.destination) return;

  const items = Array.from(todos);
  const [reorderedItem] = items.splice(result.source.index, 1);
  items.splice(result.destination.index, 0, reorderedItem);

  setTodos(items);
};
```

## 要注意的地方

### ⚠️ 狀態管理：
1. 用 `useState` 管理待辦事項列表
2. 每個待辦事項都要有唯一的 id
3. 不要直接修改 state，要用 `setTodos`

### ⚠️ 常見問題：
1. 忘記處理空白輸入
2. 忘記加上 `key` 屬性
3. 直接修改陣列而不是創建新的
