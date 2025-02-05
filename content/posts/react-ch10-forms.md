---
title: "CH10 React 表單處理"
date: 2025-02-05T23:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React 表單處理 📝

## 什麼是表單處理？
想像你在填寫一份線上表單 📋：
- 輸入姓名、Email
- 選擇性別、興趣
- 勾選同意條款
- 按下送出按鈕

這些都需要特別的處理方式，讓我們來學習吧！

## 為什麼需要特別處理表單？ 🤔
在 React 中，我們需要：
1. 追蹤使用者輸入的內容
2. 檢查輸入是否正確
3. 送出時統一處理資料

## 基本表單處理 - 認識 useState 的好朋友 👥

### 1. 文字輸入框
```jsx
function SimpleForm() {
  // 使用 useState 追蹤輸入值
  const [name, setName] = useState("");

  return (
    <input 
      type="text"
      value={name}  // 綁定 value
      onChange={(e) => setName(e.target.value)}  // 更新值
      placeholder="請輸入姓名"
    />
  );
}
```

### 2. 下拉選單
```jsx
function FruitSelector() {
  const [fruit, setFruit] = useState("apple");

  return (
    <select value={fruit} onChange={(e) => setFruit(e.target.value)}>
      <option value="apple">蘋果</option>
      <option value="banana">香蕉</option>
      <option value="orange">橘子</option>
    </select>
  );
}
```

### 3. 核取方塊
```jsx
function Checkbox() {
  const [isAgree, setIsAgree] = useState(false);

  return (
    <input 
      type="checkbox"
      checked={isAgree}
      onChange={(e) => setIsAgree(e.target.checked)}
    />
  );
}
```

## 實用表單範例 - 註冊表單 📋

```jsx
function SignupForm() {
  // 使用物件統一管理表單資料
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: "",
    agreeToTerms: false
  });

  // 統一處理表單變更
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === "checkbox" ? checked : value
    }));
  };

  // 處理表單送出
  const handleSubmit = (e) => {
    e.preventDefault();  // 防止頁面重新整理
    console.log("表單資料：", formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>
          使用者名稱：
          <input
            type="text"
            name="username"
            value={formData.username}
            onChange={handleChange}
          />
        </label>
      </div>

      <div>
        <label>
          Email：
          <input
            type="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
          />
        </label>
      </div>

      <div>
        <label>
          密碼：
          <input
            type="password"
            name="password"
            value={formData.password}
            onChange={handleChange}
          />
        </label>
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="agreeToTerms"
            checked={formData.agreeToTerms}
            onChange={handleChange}
          />
          同意服務條款
        </label>
      </div>

      <button type="submit">註冊</button>
    </form>
  );
}
```

## 表單驗證小技巧 ✅

```jsx
function ValidatedInput() {
  const [email, setEmail] = useState("");
  const [error, setError] = useState("");

  const validateEmail = (value) => {
    if (!value.includes("@")) {
      setError("請輸入有效的 Email！");
    } else {
      setError("");
    }
  };

  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => {
          setEmail(e.target.value);
          validateEmail(e.target.value);
        }}
        placeholder="請輸入 Email"
      />
      {error && <p style={{ color: "red" }}>{error}</p>}
    </div>
  );
}
```

## 重要提醒 ⚠️

1. 記得處理 `preventDefault`
   - 防止表單送出時頁面重新整理
   ```jsx
   const handleSubmit = (e) => {
     e.preventDefault();
     // 處理表單資料
   };
   ```

2. 使用 `name` 屬性
   - 幫助識別表單欄位
   - 方便統一處理表單資料

3. 記得設定初始值
   - 使用 `useState` 時要給預設值
   - 避免 undefined 造成的問題

## 常見問題 FAQ ❓

Q: 為什麼要使用 `value` 和 `onChange`？
A: 這樣可以讓 React 完全控制表單的狀態，稱為「受控元件」。

Q: 表單資料很多時怎麼辦？
A: 可以使用物件統一管理，如上面的 `formData` 範例。

Q: 要如何清空表單？
A: 只要重設 state 就可以了！例如：`setFormData({ username: "", email: "" })`


