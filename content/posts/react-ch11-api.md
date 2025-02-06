---
title: "CH10 React API 串接"
date: 2025-02-06T23:25:00+08:00
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React API 串接 🌐

## 什麼是 API 串接？
想像你在點餐 🍔：
- 你跟服務生說要點什麼（發送請求）
- 服務生去廚房拿餐點（等待回應）
- 服務生把餐點送來（接收資料）

API 串接就是跟伺服器要資料的過程！

## 為什麼需要 API？ 🤔
- 取得即時資料（例如：天氣資訊）
- 儲存資料到伺服器（例如：註冊帳號）
- 更新資料（例如：修改個人資料）

## 使用 fetch 抓取資料 🎣

### 基本 GET 請求
```jsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 定義非同步函式
    const fetchUsers = async () => {
      try {
        setLoading(true);  // 開始載入
        const response = await fetch('https://api.example.com/users');
        const data = await response.json();
        setUsers(data);    // 儲存資料
      } catch (err) {
        setError(err.message);  // 記錄錯誤
      } finally {
        setLoading(false); // 結束載入
      }
    };

    fetchUsers();  // 執行函式
  }, []);  // 空陣列代表只在元件載入時執行一次

  // 根據狀態顯示不同內容
  if (loading) return <div>載入中...</div>;
  if (error) return <div>錯誤：{error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### POST 請求範例
```jsx
function CreatePost() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const response = await fetch('https://api.example.com/posts', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          title,
          content,
        }),
      });
      
      const data = await response.json();
      console.log('成功建立文章：', data);
      
      // 清空表單
      setTitle("");
      setContent("");
    } catch (error) {
      console.error('發生錯誤：', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="文章標題"
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="文章內容"
      />
      <button type="submit">發布文章</button>
    </form>
  );
}
```

## 實用的 API 串接小技巧 💡

### 1. 載入狀態處理
```jsx
function LoadingExample() {
  const [loading, setLoading] = useState(false);

  return (
    <div>
      {loading ? (
        <div>載入中...</div>
      ) : (
        <div>資料已載入</div>
      )}
    </div>
  );
}
```

### 2. 錯誤處理
```jsx
function ErrorExample() {
  const [error, setError] = useState(null);

  return (
    <div>
      {error && (
        <div style={{ color: 'red' }}>
          發生錯誤：{error}
        </div>
      )}
    </div>
  );
}
```

## 重要提醒 ⚠️

1. 記得處理錯誤
   - 使用 try-catch 捕捉錯誤
   - 顯示友善的錯誤訊息給使用者

2. 注意載入狀態
   - 讓使用者知道正在載入
   - 避免重複發送請求

3. 資料格式檢查
   - 確認 API 回傳的資料格式
   - 處理可能的空值或未定義

## 常見問題 FAQ ❓

Q: 為什麼要用 async/await？
A: 讓非同步程式碼更容易閱讀和維護，避免 callback hell。

Q: API 回傳錯誤怎麼辦？
A: 檢查 response.ok 或 status code，並顯示適當的錯誤訊息。

Q: 要如何取消請求？
A: 可以使用 AbortController 或在元件卸載時清理。

## 練習時間 💪
試著串接免費的 API：
1. [JSONPlaceholder](https://jsonplaceholder.typicode.com/)
   - 取得文章列表
   - 顯示文章內容
   - 新增一篇文章

