---
title: "CH9 React Router 說明"
date: 2025-02-04T21:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React Router 說明 🛣️

## 什麼是 Router？
想像你在瀏覽一個購物網站 🛍️：
- 首頁看商品列表
- 點擊商品看詳細資訊
- 進入購物車結帳
- 查看個人訂單記錄

Router 就是幫你管理這些「頁面切換」的工具！

## 專案資料夾結構 📁
建議的資料夾結構如下：

```
src/
├── components/        # 共用元件
│   ├── Navbar/       # 導覽列元件
│   │   ├── Navbar.jsx
│   │   └── Navbar.module.css
│   └── Footer/       # 頁尾元件
│       ├── Footer.jsx
│       └── Footer.module.css
│
├── pages/            # 頁面元件
│   ├── Home/         # 首頁
│   │   ├── Home.jsx
│   │   └── Home.module.css
│   ├── About/        # 關於我們
│   │   ├── About.jsx
│   │   └── About.module.css
│   └── Products/     # 商品頁面
│       ├── ProductList.jsx    # 商品列表
│       ├── ProductDetail.jsx  # 商品詳情
│       └── Products.module.css
│
├── routes/           # 路由設定
│   └── index.jsx     # 主要路由設定檔
│
└── App.jsx           # 根元件
```

## 開始使用 Router

### 1. 安裝套件
```bash
# 安裝 React Router 套件
npm install react-router-dom
```

### 2. 基本設定
在 `App.jsx` 中設定 Router：

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import Home from './pages/Home'
import About from './pages/About'
import NotFound from './pages/NotFound'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  )
}

export default App
```

💡 **提醒：**
- Router 的設定直接寫在 `App.jsx` 中
- 不需要修改 `main.jsx`
- 記得：
  1. 安裝 `react-router-dom`
  2. 在 `App.jsx` 中引入需要的元件
  3. 用 `BrowserRouter` 包住 `Routes`

### 3. 建立頁面
```jsx
// pages/Home.jsx - 首頁元件
function Home() {
  return <h1>首頁</h1>;
}

// pages/About.jsx - 關於我們頁面
function About() {
  return <h1>關於我們</h1>;
}

// pages/NotFound.jsx - 404 找不到頁面
// 當使用者訪問不存在的路徑時顯示
function NotFound() {
  return <h1>找不到頁面 😢</h1>;
}
```

## 實用小範例

### 1. 導航列
```jsx
// components/Navbar.jsx
function Navbar() {
  return (
    <nav>
      {/* Link 元件用來做頁面跳轉，不會重新整理頁面 */}
      <Link to="/">首頁</Link>
      <Link to="/about">關於我們</Link>
    </nav>
  );
}
```

### 2. 商品詳細頁（動態路由）
動態路由就像是可以變化的網址，例如：
- `/product/1` - 顯示商品 1 的資料
- `/product/2` - 顯示商品 2 的資料
- `/product/abc` - 顯示商品 abc 的資料

#### 步驟一：設定路由
```jsx
// App.jsx - 設定動態路由
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import Product from './pages/Product';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/product/:id" element={<Product />} />
      </Routes>
    </BrowserRouter>
  );
}
```

#### 步驟二：建立商品頁面
```jsx
// pages/Product.jsx - 商品詳細頁
import { useParams } from 'react-router-dom';

function Product() {
  // id 會從網址中取得
  // 例如：當使用者訪問 /product/123 時
  // id 的值就會是 "123"
  // 當使用者訪問 /product/abc 時
  // id 的值就會是 "abc"
  const { id } = useParams();
  
  return (
    <div>
      <h1>商品 {id} 的詳細資料</h1>
    </div>
  );
}
```

💡 **id 的來源說明：**
1. 當我們在步驟一設定 `path="/product/:id"` 時，`:id` 就是一個參數
2. 使用者訪問不同的網址時：
   - 訪問 `/product/123` → id 是 "123"
   - 訪問 `/product/abc` → id 是 "abc"
   - 訪問 `/product/iphone` → id 是 "iphone"
3. 使用 `useParams()` 可以取得這個網址中的參數

#### 步驟三：加入連結
```jsx
// pages/Home.jsx - 在首頁加入商品連結
import { Link } from 'react-router-dom';

function Home() {
  // 假設這是從 API 取得的商品列表
  const products = [
    { id: 1, name: '商品一' },
    { id: 2, name: '商品二' },
    { id: 3, name: '商品三' },
  ];

  return (
    <div>
      <h1>商品列表</h1>
      <ul>
        {products.map(product => (
          <li key={product.id}>
            {/* 動態產生商品連結 */}
            <Link to={`/product/${product.id}`}>
              {product.name}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### 完整的商品頁面範例
```jsx
// pages/Product.jsx - 完整的商品頁面
import { useParams, useNavigate } from 'react-router-dom';

function Product() {
  const { id } = useParams();  // 取得商品 ID
  const navigate = useNavigate();  // 用於頁面導航

  // 假設這是商品資料
  const product = {
    id: id,
    name: `商品 ${id}`,
    price: 100 * id,
    description: `這是商品 ${id} 的說明`,
  };

  return (
    <div>
      <h1>{product.name}</h1>
      <p>價格：${product.price}</p>
      <p>說明：{product.description}</p>
      
      {/* 返回按鈕 */}
      <button onClick={() => navigate(-1)}>
        返回上一頁
      </button>
    </div>
  );
}
```

💡 **動態路由使用提醒：**
1. `:參數名` 可以設定多個，例如：`/product/:category/:id`
2. 用 `useParams()` 取得的參數都是字串型態
3. 記得處理參數不存在的情況
4. 可以搭配 `useNavigate()` 做頁面導航

## 常用的 Router 功能

### 1. Link：連結到其他頁面
```jsx
// components/Navbar.jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      {/* Link 比 <a> 標籤好，不會重新整理整個頁面 */}
      <Link to="/">首頁</Link>
      <Link to="/about">關於我們</Link>
    </nav>
  );
}
```

### 2. useNavigate：程式切換頁面
```jsx
// components/LoginButton.jsx
function LoginButton() {
  // useNavigate 用來做程式化的頁面跳轉
  const navigate = useNavigate();
  
  const handleLogin = () => {
    // 可以在登入成功、表單送出等情況下使用
    // 例如：登入成功後跳轉到首頁
    navigate('/');
    
    // 也可以使用 replace 來取代目前的歷史記錄
    // navigate('/', { replace: true });
    
    // 或是返回上一頁
    // navigate(-1);
  };

  return <button onClick={handleLogin}>登入</button>;
}
```

### 3. useParams：取得網址參數
```jsx
// pages/Product.jsx
function Product() {
  // 用 useParams 取得網址參數
  // 例如：/product/123 會得到 { id: '123' }
  const { id } = useParams();
  
  return <h1>商品 {id}</h1>;
}
```

## 要注意的地方

### ⚠️ 路由設定：
1. 要把 `<BrowserRouter>` 放在最外層
2. `<Routes>` 裡面只能放 `<Route>`
3. `path="*"` 是「找不到頁面」的路由

### ⚠️ 常見問題：
1. Link 和 a 標籤的差別
   - `<Link>` 是 React Router 專用，不會重新整理頁面
   - `<a>` 會重新整理整個頁面，不要用在 React 內部連結

2. 路徑要加斜線
   - ✅ `to="/about"`
   - ❌ `to="about"`

## 小提醒 💡
1. 記得先安裝 `react-router-dom`
2. 用 `<Link>` 不要用 `<a>` 做內部連結
3. 動態路由用 `:參數名` 的方式寫
4. 找不到頁面用 `path="*"` 設定 