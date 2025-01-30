---
title: "CH5 React Props 說明"
date: 2025-01-30
draft: false
categories: ["React"]
tags: ["學習筆記"]
---


# React Props 說明 🎁

## 什麼是 Props？
想像你在點飲料：
- 你會告訴店員要什麼飲料 🥤
- 要大杯還是小杯 📏
- 要加什麼配料 🧋
- 要幾分糖、幾分冰 🧊

Props 就像是這些「點餐選項」，讓你可以傳遞資料給元件，讓元件根據這些資料顯示不同的內容！

## 開始使用 Props 的三個步驟

### 1️⃣ 在父元件傳遞 Props
```jsx
// 父元件
function App() {
  return (
    <Greeting name="小明" />  // 傳遞 name 這個 prop
  );
}
```

### 2️⃣ 在子元件接收 Props
```jsx
// 子元件
function Greeting(props) {
  return <h1>哈囉，{props.name}！</h1>;
}
```

### 3️⃣ 使用解構讓程式碼更簡潔
```jsx
// 更簡潔的寫法
function Greeting({ name }) {
  return <h1>哈囉，{name}！</h1>;
}
```

## Props 的基本使用範例

### 1. 父子元件的基本溝通
```jsx
// 子元件：顯示問候語
function Greeting({ name, time }) {
  return (
    <div className="greeting-card">
      <h2>哈囉，{name}！</h2>
      <p>現在是{time}好</p>
    </div>
  );
}

// 父元件：可以重複使用子元件
function ParentComponent() {
  return (
    <div className="container">
      <h1>我是父元件</h1>
      
      {/* 第一個問候卡片 */}
      <Greeting 
        name="小明"
        time="早"
      />

      {/* 第二個問候卡片 */}
      <Greeting 
        name="小華" 
        time="午"
      />

      {/* 我們可以重複使用相同的元件，只要傳入不同的 props */}
      <Greeting 
        name="小美" 
        time="晚"
      />
    </div>
  );
}

// 實際顯示的結果：
// 哈囉，小明！
// 現在是早好
//
// 哈囉，小華！
// 現在是午好
//
// 哈囉，小美！
// 現在是晚好
```

這個例子展示了 React 元件重複使用的概念：
1. `Greeting` 是一個可重複使用的子元件
2. 父元件可以多次使用這個子元件
3. 每次使用時可以傳入不同的 props
4. 每個 Greeting 都會根據收到的 props 顯示不同的內容

### 2. 父元件傳遞事件處理函式
```jsx
// 子元件：按鈕
function Button({ text, color, onClick }) {
  return (
    <button 
      style={{ 
        backgroundColor: color,
        color: 'white',        // 文字顯示白色
        padding: '10px 20px',  // 加入內距
        margin: '5px',         // 加入外距
        border: 'none',        // 移除邊框
        borderRadius: '5px'    // 圓角
      }}
      onClick={onClick}
    >
      {text}
    </button>
  );
}

// 父元件：管理所有按鈕的行為
function ButtonContainer() {
  // 定義處理函式
  const handleSave = () => {
    alert('儲存成功！');
  };

  const handleDelete = () => {
    alert('刪除成功！');
  };

  return (
    <div>
      <h1>按鈕區域</h1>
      
      {/* 儲存按鈕 */}
      <Button 
        text="儲存" 
        color="blue" 
        onClick={handleSave}
      />
      {/* 點擊會跳出「儲存成功！」*/}

      {/* 刪除按鈕 */}
      <Button 
        text="刪除" 
        color="red" 
        onClick={handleDelete}
      />
      {/* 點擊會跳出「刪除成功！」*/}
    </div>
  );
}

// 實際顯示效果：
// +------------------+
// |    按鈕區域     |
// |                 |
// | [儲存] [刪除]   |
// +------------------+
//   藍色   紅色
```

這個例子展示了：
1. 如何建立一個可重複使用的按鈕元件
2. 如何透過 props 傳遞：
   - 外觀設定（顏色、文字）
   - 行為（點擊事件）
3. 如何在父元件中管理不同按鈕的行為

### 3. 父元件傳遞資料物件
```jsx
// 子元件：顯示使用者資訊
function UserProfile({ user }) {
  // user 是一個物件，包含了所有使用者資訊
  // 可以用 user.name, user.age 等方式取得資料
  return (
    <div className="user-card">
      <h2>{user.name}的個人資料</h2>
      <p>年齡：{user.age}</p>
      <p>職業：{user.job}</p>
      <p>城市：{user.city}</p>
    </div>
  );
}

// 父元件：管理使用者資料
function UserContainer() {
  // 把所有相關的資料整理成一個物件
  const userData = {
    name: '小明',
    age: 25,
    job: '工程師',
    city: '台北'
  };

  return (
    <div>
      <h1>使用者資訊</h1>
      {/* 把整個 userData 物件透過 user 這個 prop 傳給子元件 */}
      <UserProfile user={userData} />
    </div>
  );
}

// 實際顯示效果：
// +------------------------+
// |    使用者資訊         |
// | 小明的個人資料        |
// | 年齡：25              |
// | 職業：工程師          |
// | 城市：台北            |
// +------------------------+

// 也可以傳遞多個使用者資料：
function MultipleUsers() {
  const user1 = {
    name: '小明',
    age: 25,
    job: '工程師',
    city: '台北'
  };

  const user2 = {
    name: '小華',
    age: 28,
    job: '設計師',
    city: '台中'
  };

  return (
    <div>
      <h1>多位使用者資訊</h1>
      <UserProfile user={user1} />
      <UserProfile user={user2} />
    </div>
  );
}
```

這個例子展示了：
1. 如何將多個相關的資料組織成一個物件
2. 如何將整個物件作為一個 prop 傳遞
3. 在子元件中如何存取物件中的資料
4. 如何重複使用同一個元件來顯示不同的資料

優點：
- 程式碼更整潔（不用分別傳遞 name, age, job, city）
- 資料更有組織性（相關的資料都放在同一個物件中）
- 方便擴展（想加新欄位時只需修改物件）

## Props 的注意事項

### 1. ⚠️ Props 是唯讀的（不能修改）
```jsx
function Greeting(props) {
  // ❌ 錯誤寫法：不能修改 props
  props.name = '新名字';  // 這會報錯！
  
  // ✅ 正確寫法：只能讀取 props
  return <h1>哈囉，{props.name}！</h1>;
}
```

### 2. ⚠️ 傳遞不同類型的資料
```jsx
// 字串：直接寫
<Card name="小明" />

// 數字：用大括號
<Card age={25} />

// 布林值：用大括號
<Card isStudent={true} />

// 陣列：用大括號
<Card hobbies={['打籃球', '看電影']} />

// 物件：用大括號
<Card info={{ city: '台北', country: '台灣' }} />

// 函式：用大括號
<Card onClick={() => alert('點擊')} />
```

### 3. ⚠️ Props 的預設值
```jsx
function Greeting({ name = '訪客' }) {
  return <h1>哈囉，{name}！</h1>;
}

// 使用元件
function App() {
  return (
    <div>
      <Greeting />  {/* 會顯示：哈囉，訪客！*/}
      <Greeting name="小明" />  {/* 會顯示：哈囉，小明！*/}
    </div>
  );
}
```

## 實用小技巧

### 1. 使用展開運算子傳遞多個 Props
```jsx
function UserCard({ name, age, job, city }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>年齡：{age}</p>
      <p>職業：{job}</p>
      <p>城市：{city}</p>
    </div>
  );
}

// 父元件
function App() {
  const userInfo = {
    name: '小明',
    age: 25,
    job: '工程師',
    city: '台北'
  };

  // ✅ 方便的寫法：使用展開運算子
  return <UserCard {...userInfo} />;

  // 等同於下面這種寫法：
  return (
    <UserCard 
      name={userInfo.name}
      age={userInfo.age}
      job={userInfo.job}
      city={userInfo.city}
    />
  );
}
```

### 2. 使用 children prop
```jsx
// 子元件：可以包住其他內容的卡片
function Card({ title, children }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}  {/* 這裡會顯示被包住的內容 */}
    </div>
  );
}

// 父元件：使用卡片包住不同內容
function App() {
  return (
    <div>
      {/* 卡片 1：包住文字 */}
      <Card title="簡介">
        <p>這是一段說明文字</p>
        <p>可以放入多個段落</p>
      </Card>

      {/* 卡片 2：包住表單 */}
      <Card title="登入">
        <input placeholder="帳號" />
        <input type="password" placeholder="密碼" />
        <button>送出</button>
      </Card>
    </div>
  );
}
```

### 3. Props 的解構賦值
```jsx
// ❌ 較繁瑣的寫法
function UserInfo(props) {
  return (
    <div>
      <p>名字：{props.name}</p>
      <p>年齡：{props.age}</p>
    </div>
  );
}

// ✅ 使用解構的寫法
function UserInfo({ name, age }) {
  return (
    <div>
      <p>名字：{name}</p>
      <p>年齡：{age}</p>
    </div>
  );
}

// 🌟 解構時設定預設值
function UserInfo({ name = '訪客', age = 0 }) {
  return (
    <div>
      <p>名字：{name}</p>
      <p>年齡：{age}</p>
    </div>
  );
}
```