---
title: "CH14 購物車專案實作"
date: 2025-02-17T11:30:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# 購物車專案實作 🛒

## 專案說明
我們要做一個簡單的購物車系統：
- 可以瀏覽商品列表 📋
- 可以將商品加入購物車 ➕
- 可以調整商品數量 🔢
- 可以計算總金額 💰

## 第一步：建立基本結構

### 1. 建立專案資料夾結構
```bash
src/
  ├── components/
  │   ├── ProductList.jsx    # 商品列表
  │   ├── ProductItem.jsx    # 單個商品
  │   ├── Cart.jsx          # 購物車
  │   └── CartItem.jsx      # 購物車項目
  └── App.jsx
```

### 2. 設定基本資料
```jsx
// App.jsx
import { useState } from 'react';
import ProductList from './components/ProductList';
import Cart from './components/Cart';

// 商品資料
const products = [
  { id: 1, name: '超級漢堡', price: 50, image: '🍔' },
  { id: 2, name: '美式薯條', price: 30, image: '🍟' },
  { id: 3, name: '冰淇淋', price: 25, image: '🍦' },
  { id: 4, name: '可樂', price: 20, image: '🥤' },
];

function App() {
  const [cartItems, setCartItems] = useState([]);

  return (
    <div className="app">
      <h1>美食商店</h1>
      <div className="container">
        <ProductList products={products} onAddToCart={(product) => {/* 待實作 */}} />
        <Cart items={cartItems} onUpdateQuantity={(id, qty) => {/* 待實作 */}} />
      </div>
    </div>
  );
}
```

## 第二步：實作各個元件

### 1. 商品列表
```jsx
// components/ProductList.jsx
function ProductList({ products, onAddToCart }) {
  return (
    <div className="product-list">
      <h2>商品列表</h2>
      <div className="products">
        {products.map(product => (
          <ProductItem
            key={product.id}
            product={product}
            onAddToCart={onAddToCart}
          />
        ))}
      </div>
    </div>
  );
}
```

### 2. 單個商品
```jsx
// components/ProductItem.jsx
function ProductItem({ product, onAddToCart }) {
  return (
    <div className="product-item">
      <div className="product-image">{product.image}</div>
      <h3>{product.name}</h3>
      <p>NT$ {product.price}</p>
      <button onClick={() => onAddToCart(product)}>
        加入購物車
      </button>
    </div>
  );
}
```

### 3. 購物車
```jsx
// components/Cart.jsx
function Cart({ items, onUpdateQuantity }) {
  // 計算總金額
  const total = items.reduce((sum, item) => 
    sum + item.price * item.quantity, 0
  );

  if (items.length === 0) {
    return (
      <div className="cart">
        <h2>購物車</h2>
        <p>購物車是空的</p>
      </div>
    );
  }

  return (
    <div className="cart">
      <h2>購物車</h2>
      {items.map(item => (
        <CartItem
          key={item.id}
          item={item}
          onUpdateQuantity={onUpdateQuantity}
        />
      ))}
      <div className="cart-total">
        <h3>總計：NT$ {total}</h3>
        <button onClick={() => alert('謝謝購買！')}>
          結帳
        </button>
      </div>
    </div>
  );
}
```

### 4. 購物車項目
```jsx
// components/CartItem.jsx
function CartItem({ item, onUpdateQuantity }) {
  return (
    <div className="cart-item">
      <div className="item-info">
        <span className="item-image">{item.image}</span>
        <span className="item-name">{item.name}</span>
        <span className="item-price">NT$ {item.price}</span>
      </div>
      <div className="item-quantity">
        <button 
          onClick={() => onUpdateQuantity(item.id, item.quantity - 1)}
          disabled={item.quantity <= 1}
        >
          -
        </button>
        <span>{item.quantity}</span>
        <button 
          onClick={() => onUpdateQuantity(item.id, item.quantity + 1)}
        >
          +
        </button>
      </div>
    </div>
  );
}
```

## 第三步：完成主要功能

### 1. 加入購物車
```jsx
// App.jsx
const addToCart = (product) => {
  setCartItems(items => {
    // 檢查商品是否已在購物車中
    const exists = items.find(item => item.id === product.id);
    
    if (exists) {
      // 如果已存在，增加數量
      return items.map(item =>
        item.id === product.id
          ? { ...item, quantity: item.quantity + 1 }
          : item
      );
    }
    
    // 如果不存在，新增到購物車
    return [...items, { ...product, quantity: 1 }];
  });
};
```

### 2. 更新商品數量
```jsx
// App.jsx
const updateQuantity = (id, quantity) => {
  setCartItems(items => {
    // 如果數量是 0，從購物車移除
    if (quantity === 0) {
      return items.filter(item => item.id !== id);
    }
    
    // 更新數量
    return items.map(item =>
      item.id === id
        ? { ...item, quantity }
        : item
    );
  });
};
```

## 加分功能

### 1. 本地儲存
```jsx
// App.jsx
// 從 localStorage 讀取購物車資料
const [cartItems, setCartItems] = useState(() => {
  const saved = localStorage.getItem('cart');
  return saved ? JSON.parse(saved) : [];
});

// 當購物車改變時儲存
useEffect(() => {
  localStorage.setItem('cart', JSON.stringify(cartItems));
}, [cartItems]);
```

### 2. 加入動畫效果
```jsx
// 安裝 framer-motion
import { motion } from 'framer-motion';

// 在 CartItem 中使用
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  className="cart-item"
>
  {/* 內容 */}
</motion.div>
```

## 要注意的地方

### ⚠️ 狀態管理：
1. 購物車資料要用 `useState` 管理
2. 不要直接修改 state
3. 計算總金額時注意數字精確度

### ⚠️ 常見問題：
1. 忘記處理商品已存在的情況
2. 忘記處理數量為 0 的情況
3. 沒有加上載入中狀態

## 小提醒 💡
1. 先完成基本功能再加特效
2. 注意金額計算的精確度
3. 適時加入錯誤處理
4. 可以加入動畫提升體驗
``` 