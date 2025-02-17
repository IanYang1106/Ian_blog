---
title: "聊天功能說明文件(Django 標準視圖 + AJAX 輪詢)"
date: 2025-01-04
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

# 聊天功能說明文件(Django 標準視圖 + AJAX 輪詢)

## 目錄結構

```
chats/
├── migrations/          # 資料庫遷移檔案
├── templates/
│   └── chats/          # 聊天相關樣板
│       ├── chat_room_detail.html  # 聊天室詳細頁面
│       └── chat_room_list.html    # 聊天室列表頁面
├── __init__.py
├── admin.py            # 後台管理設定
├── apps.py             # 應用程式設定
├── models.py           # 資料模型
├── urls.py             # 網址路由設定
└── views.py            # 視圖函式
```

## 資料模型 (models.py)

### ChatRoom 模型

- 用於儲存聊天室資訊
- 欄位：
  - `project`: 關聯的專案 (ForeignKey)
  - `visitor`: 訪客/聊天者 (ForeignKey)
  - `slug`: 聊天室唯一識別碼
  - `created_at`: 建立時間
  - `updated_at`: 最後更新時間
- 特性：
  - `participants`: 回傳聊天室參與者清單（專案提案者和訪客）

### Message 模型

- 用於儲存聊天訊息
- 欄位：
  - `chat_room`: 所屬聊天室 (ForeignKey)
  - `sender`: 發送者 (ForeignKey)
  - `content`: 訊息內容
  - `is_read`: 是否已讀
  - `created_at`: 發送時間

## 視圖函式 (views.py)

### project_chat

- 功能：顯示或建立與專案提案者的聊天室
- 網址: `/chats/project/<project_slug>/[<chat_room_slug>/]`
- 方法：GET/POST
- 邏輯：
  1. 取得專案資訊
  2. 處理聊天室存取權限
  3. 處理訊息發送
  4. 標記未讀訊息為已讀
  5. 渲染聊天室頁面

### project_chat_list

- 功能：專案提案者查看所有聊天室列表
- 網址: `/chats/project/<project_slug>/list/`
- 方法：GET
- 邏輯：
  1. 驗證使用者是否為專案提案者
  2. 取得該專案的所有聊天室
  3. 計算未讀訊息數量
  4. 渲染列表頁面

### send_message (API)

- 功能：發送訊息 API
- 網址: `/chats/api/send-message/<slug>/`
- 方法：POST
- 邏輯：
  1. 驗證請求方法和權限
  2. 建立新訊息
  3. 回傳訊息資料

### get_messages (API)

- 功能：取得新訊息 API
- 網址: `/chats/api/get-messages/<slug>/`
- 方法：GET
- 邏輯：
  1. 驗證權限
  2. 取得指定 ID 之後的訊息
  3. 回傳訊息清單

## 樣板檔案

### chat_room_detail.html

- 功能：聊天室詳細頁面
- 主要元件：
  - 聊天室標題
  - 訊息清單
  - 發送訊息表單
- JavaScript 功能：
  - 自動捲動到最新訊息
  - 定期檢查新訊息
  - 表單提交處理

### chat_room_list.html

- 功能：聊天室列表頁面
- 主要元件：
  - 專案標題
  - 聊天室清單
  - 未讀訊息計數
  - 最後更新時間

## 網址設定 (urls.py)

```python
urlpatterns = [
    path("project/<slug:project_slug>/", views.project_chat, name="project_chat"),
    path("project/<slug:project_slug>/list/", views.project_chat_list, name="project_chat_list"),
    path("project/<slug:project_slug>/<slug:chat_room_slug>/", views.project_chat, name="project_chat"),
    path("api/send-message/<slug:slug>/", views.send_message, name="send_message"),
    path("api/get-messages/<slug:slug>/", views.get_messages, name="get_messages"),
]
```

## 實作細節

### 訊息即時更新

- 使用 AJAX 輪詢機制
- 每 5 秒檢查一次新訊息
- 使用 `lastMessageId` 追蹤最新訊息

### 未讀訊息處理

- 進入聊天室時自動標記訊息為已讀
- 在列表頁面顯示未讀訊息數量
- 僅計算對方發送的未讀訊息

### 權限控制

- 使用 `@login_required` 裝飾器確保使用者登入
- 驗證使用者是否為聊天室參與者
- 專案提案者可以存取所有相關聊天室

### 安全性考量

- CSRF 保護
- 使用者身分驗證
- 聊天室存取權限驗證

## 使用方式

1. 訪客查看專案時可以點擊「與提案者聊天」按鈕
2. 自動建立聊天室或進入現有聊天室
3. 提案者可以在專案聊天列表中查看所有聊天
4. 訊息會自動更新，無需手動重新整理頁面

## 程式碼詳解

### models.py

```python
from django.db import models
from django.contrib.auth.models import User
from projects.models import Project
from autoslug import AutoSlugField

class ChatRoom(models.Model):
    # 關聯到專案模型，當專案被刪除時，相關聊天室也會被刪除
    project = models.ForeignKey(
        Project,
        on_delete=models.CASCADE,
        related_name='chat_rooms'
    )

    # 訪客，與專案提案者聊天的使用者
    visitor = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='visited_chats'
    )

    # 自動產生的唯一識別碼，用於網址
    slug = AutoSlugField(
        populate_from='id',
        unique=True
    )

    # 時間戳記錄
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    @property
    def participants(self):
        # 回傳聊天室的參與者清單（專案提案者和訪客）
        return [self.project.account, self.visitor]

class Message(models.Model):
    # 關聯到聊天室，聊天室刪除時相關訊息也會被刪除
    chat_room = models.ForeignKey(
        ChatRoom,
        on_delete=models.CASCADE,
        related_name='messages'
    )

    # 訊息發送者
    sender = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='sent_messages'
    )

    # 訊息內容
    content = models.TextField()

    # 是否已讀
    is_read = models.BooleanField(default=False)

    # 發送時間
    created_at = models.DateTimeField(auto_now_add=True)
```

### views.py

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.http import JsonResponse
from django.utils import timezone
from django.views.decorators.http import require_http_methods
from .models import ChatRoom, Message
from projects.models import Project

@login_required
def project_chat(request, project_slug, chat_room_slug=None):
    # 取得專案資訊，如果不存在則回傳404
    project = get_object_or_404(Project, slug=project_slug)

    if chat_room_slug:
        # 如果提供了聊天室 slug，取得特定聊天室
        chat_room = get_object_or_404(ChatRoom, slug=chat_room_slug, project=project)
        # 確保使用者有權限存取此聊天室
        if request.user != project.account and request.user != chat_room.visitor:
            return redirect("chats:project_chat", project_slug=project_slug)
    else:
        # 如果是提案者且沒有指定聊天室，重新導向到列表頁
        if request.user == project.account and request.method != "POST":
            return redirect("chats:project_chat_list", project_slug=project_slug)
        # 否則取得或建立訪客的聊天室
        chat_room, _ = ChatRoom.objects.get_or_create(
            project=project,
            visitor=request.user,
        )

    # 處理發送訊息的 POST 請求
    if request.method == "POST":
        content = request.POST.get("content", "").strip()
        if content:
            try:
                # 建立新訊息
                message = Message.objects.create(
                    chat_room=chat_room,
                    sender=request.user,
                    content=content
                )
                # 更新聊天室的最後活動時間
                chat_room.updated_at = timezone.now()
                chat_room.save()

                # 如果是 AJAX 請求，回傳 JSON 格式的訊息資料
                if request.headers.get("X-Requested-With") == "XMLHttpRequest":
                    return JsonResponse({
                        "id": message.id,
                        "content": message.content,
                        "sender": message.sender.username,
                        "created_at": message.created_at.isoformat(),
                        "is_read": message.is_read,
                    })
            except Exception as e:
                if request.headers.get("X-Requested-With") == "XMLHttpRequest":
                    return JsonResponse({"error": str(e)}, status=500)

            # 一般 POST 請求則重新導向回聊天室
            return redirect(
                "chats:project_chat",
                project_slug=project_slug,
                chat_room_slug=chat_room.slug,
            )

    # 取得聊天記錄並按時間排序
    chat_messages = chat_room.messages.all().order_by("created_at")

    # 將未讀訊息標記為已讀
    chat_room.messages.filter(is_read=False).exclude(sender=request.user).update(
        is_read=True
    )

    # 渲染樣板
    return render(
        request,
        "chats/chat_room_detail.html",
        {
            "chat_room": chat_room,
            "chat_messages": chat_messages,
            "project": project,
            "chat_room_slug": chat_room.slug,
        },
    )

@login_required
def project_chat_list(request, project_slug):
    # 取得專案資訊
    project = get_object_or_404(Project, slug=project_slug)

    # 確保只有專案提案者可以存取列表頁
    if request.user != project.account:
        return redirect("chats:project_chat", project_slug=project_slug)

    # 取得所有聊天室並計算未讀訊息數
    chat_rooms = [
        {
            "room": room,
            "unread_count": room.messages.filter(is_read=False)
            .exclude(sender=request.user)
            .count(),
        }
        for room in ChatRoom.objects.filter(project=project).order_by("-updated_at")
    ]

    return render(
        request,
        "chats/chat_room_list.html",
        {
            "project": project,
            "chat_rooms": chat_rooms,
        },
    )
```

### chat_room_detail.html 中的 JavaScript

```javascript
// 取得聊天室識別碼
const roomSlug = "{{ chat_room.slug }}";
// 記錄最後一則訊息的 ID
let lastMessageId = "{{ chat_messages.last.id|default:0 }}";

// 處理訊息發送表單提交
document.getElementById("messageForm").addEventListener("submit", function (e) {
  e.preventDefault();

  const form = e.target;
  const messageInput = form.querySelector("#messageInput");
  const message = messageInput.value.trim();

  if (message) {
    // 提交表單並清空輸入框
    form.submit();
    messageInput.value = "";
  }
});

// 捲動到最新訊息
function scrollToBottom() {
  const messageContainer = document.getElementById("messageContainer");
  messageContainer.scrollTop = messageContainer.scrollHeight;
}

// 頁面載入時捲動到最底部
scrollToBottom();

// 定期檢查新訊息（每5秒）
setInterval(async function () {
  try {
    // 發送 API 請求取得新訊息
    const response = await fetch(
      `/chats/api/get-messages/${roomSlug}/?last_id=${lastMessageId}`
    );
    const data = await response.json();

    if (data.messages && data.messages.length > 0) {
      // 遍歷並新增訊息到聊天室
      data.messages.forEach((message) => {
        const messageContainer = document.getElementById("messageContainer");
        const isCurrentUser = message.sender === "{{ user.username }}";

        // 建立訊息元素
        const messageDiv = document.createElement("div");
        messageDiv.className = `mb-4 flex ${
          isCurrentUser ? "justify-end" : ""
        }`;

        const messageContent = document.createElement("div");
        messageContent.className = `message rounded-lg p-3 ${
          isCurrentUser ? "sent" : "received"
        }`;

        // 設定訊息內容
        let html = `
                    <p class="text-gray-800">${message.content}</p>
                    <p class="text-xs text-gray-500 text-right mt-1">
                        ${new Date(message.created_at).toLocaleTimeString(
                          "zh-TW",
                          {
                            hour: "2-digit",
                            minute: "2-digit",
                          }
                        )}
                    </p>
                `;

        messageContent.innerHTML = html;
        messageDiv.appendChild(messageContent);
        messageContainer.appendChild(messageDiv);

        // 更新最後訊息 ID
        lastMessageId = message.id;
      });

      // 捲動到最新訊息
      scrollToBottom();
    }
  } catch (error) {
    console.error("Error fetching messages:", error);
  }
}, 5000);
```

## 前端實作

### 依賴項

```javascript
import Alpine from "alpinejs";
import htmx from "htmx.org";

window.Alpine = Alpine;
window.htmx = htmx;
```

### JavaScript 程式碼 (frontends/scripts/app.js)

```javascript
// 聊天室 Alpine.js 元件
Alpine.data("chatRoom", (roomSlug, initialLastMessageId, currentUsername) => ({
  // 元件狀態
  newMessage: "",
  messages: [],
  lastMessageId: initialLastMessageId,
  pollingInterval: null,

  // 初始化
  init() {
    this.startPolling();
    this.scrollToBottom();
  },

  // 捲動到底部
  scrollToBottom() {
    const messageContainer = document.getElementById("messageContainer");
    if (messageContainer) {
      messageContainer.scrollTop = messageContainer.scrollHeight;
    }
  },

  // 開始輪詢新訊息
  startPolling() {
    this.pollingInterval = setInterval(() => this.checkNewMessages(), 5000);
  },

  // 檢查新訊息
  async checkNewMessages() {
    try {
      const response = await fetch(
        `/chats/api/get-messages/${roomSlug}/?last_id=${this.lastMessageId}`
      );
      const data = await response.json();

      if (data.messages && data.messages.length > 0) {
        data.messages.forEach((message) => {
          const messageContainer = document.getElementById("messageContainer");
          const isCurrentUser = message.sender === currentUsername;

          const messageDiv = document.createElement("div");
          messageDiv.className = `mb-4 flex ${
            isCurrentUser ? "justify-end" : ""
          }`;

          const messageContent = document.createElement("div");
          messageContent.className = `max-w-[70%] rounded-lg p-3 ${
            isCurrentUser ? "bg-green-100 ml-auto" : "bg-gray-100"
          }`;

          let html = `
                        <p class="text-gray-800">${message.content}</p>
                        <p class="text-xs text-gray-500 text-right mt-1">
                            ${new Date(message.created_at).toLocaleTimeString(
                              "zh-TW",
                              {
                                hour: "2-digit",
                                minute: "2-digit",
                              }
                            )}
                        </p>
                    `;

          messageContent.innerHTML = html;
          messageDiv.appendChild(messageContent);
          messageContainer.appendChild(messageDiv);

          this.lastMessageId = message.id;
        });

        this.scrollToBottom();
      }
    } catch (error) {
      console.error("Error fetching messages:", error);
    }
  },

  // 處理表單提交
  handleSubmit(event) {
    if (this.newMessage.trim()) {
      event.target.submit();
      this.newMessage = "";
      this.$nextTick(() => this.scrollToBottom());
    }
  },
}));
```

### 使用的技術

- Alpine.js: 用於前端狀態管理和互動
- Tailwind CSS: 用於樣式設計
- HTMX: 用於增強的 HTML 互動
- Fetch API: 用於非同步資料請求
