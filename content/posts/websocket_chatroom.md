---
title: "WebSocket 聊天功能"
date: 2025-01-06
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

## 啟動服務

如果完成開發做測試，請確保啟動所有必要的服務：

1. 啟動 Redis 服務：

```bash
# 啟動 Redis
brew services start redis

# 檢查 Redis 狀態
brew services list
```

2. 啟動 Django 開發伺服器（終端機 1）：

```bash
poetry run python manage.py runserver
```

3. 啟動 Daphne WebSocket 伺服器（終端機 2）：

```bash
DJANGO_SETTINGS_MODULE=core.settings poetry run daphne -b 0.0.0.0 -p 8001 core.asgi:application
```

## 必要套件

以下套件是實作 WebSocket 聊天功能所需要的：

1. `channels`: Django 的 WebSocket 框架

   - 用於處理 WebSocket 連線和訊息傳遞
   - 提供非同步支援

2. `daphne`: ASGI 伺服器

   - 用於執行 WebSocket 服務
   - 處理 WebSocket 協定

3. `channels_redis`: Redis 通道層後端

   - 用於多伺服器部署時的訊息傳遞
   - 提供訊息佇列功能

4. `redis`: 訊息佇列伺服器
   - 用於儲存聊天訊息和管理 WebSocket 連線
   - 提供高效能的訊息傳遞

## 1. 環境準備

### 1.1 安裝必要套件

```bash
# 安裝 Django Channels
poetry add channels

# 安裝 Daphne（WebSocket 伺服器）
poetry add daphne

# 安裝 channels_redis
poetry add channels_redis

# 安裝 Redis（macOS）
brew install redis
```

### 1.2 檔案結構

```
chats/
├── __init__.py          # Python 套件標識檔
├── admin.py             # Django 後台介面設定
├── apps.py              # 應用程式設定
├── consumers.py         # WebSocket 消費者（處理 WebSocket 連線和訊息）
├── models.py            # 資料模型定義
├── routing.py           # WebSocket 路由設定
├── urls.py              # HTTP URL 路由設定
├── views.py             # HTTP 視圖函式
└── templates/           # 模板目錄
    └── chats/
        ├── chat_list.html  # 聊天室列表頁面
        └── room.html       # 聊天室頁面
```

### 1.3 設定 Django 設定檔

在 `core/settings.py` 中加入以下設定：

```python
import os  # 加入 os 模組匯入

INSTALLED_APPS = [
    ...
    'channels',  # 加入 channels
    'chats',     # 加入 chats 應用程式
]

# Channels 設定
ASGI_APPLICATION = "core.asgi.application"

# Channel Layers 設定（使用 Redis 作為後端）
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [(
                os.getenv("REDIS_HOST"),  # Redis 主機地址
                os.getenv("REDIS_PORT"),  # Redis 端口
            )],
        },
    },
}
```

### 1.3.1 環境變數範例

在 `.env.sample` 檔案中加入以下範例設定：

```plaintext
# Redis 設定
REDIS_HOST=  # Redis 伺服器主機
REDIS_PORT=  # Redis 伺服器連接埠
```

### 1.4 基礎檔案設定

#### 1.4.1 應用程式設定 (`chats/apps.py`)

```python
from django.apps import AppConfig

class ChatsConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'  # 設定主鍵欄位類型
    name = 'chats'                                        # 應用程式名稱
```

#### 1.4.2 套件初始化 (`chats/__init__.py`)

```python
# 此檔案可以為空，用於標識這是一個 Python 套件
# 如果需要在應用程式載入時執行某些初始化操作，可以在這裡加入
```

#### 1.4.3 後台介面設定 (`chats/admin.py`)

```python
from django.contrib import admin
from .models import ChatRoom, Message

@admin.register(ChatRoom)
class ChatRoomAdmin(admin.ModelAdmin):
    list_display = [
        "project",      # 顯示關聯的專案
        "visitor",      # 顯示訪客使用者
        "created_at",   # 顯示建立時間
        "updated_at"    # 顯示更新時間
    ]
    list_filter = [
        "created_at",   # 可依建立時間篩選
        "updated_at"    # 可依更新時間篩選
    ]
    search_fields = [
        "project__title",    # 可搜尋專案標題
        "visitor__username", # 可搜尋訪客使用者名稱
        "slug"              # 可搜尋 slug
    ]
    readonly_fields = [
        "slug"              # slug 為唯讀欄位
    ]

@admin.register(Message)
class MessageAdmin(admin.ModelAdmin):
    list_display = [
        "chat_room",    # 顯示所屬聊天室
        "sender",       # 顯示寄件者
        "content",      # 顯示訊息內容
        "is_read",      # 顯示是否已讀
        "created_at"    # 顯示發送時間
    ]
    list_filter = [
        "is_read",      # 可依已讀狀態篩選
        "created_at"    # 可依發送時間篩選
    ]
    search_fields = [
        "content",          # 可搜尋訊息內容
        "sender__username"  # 可搜尋寄件者使用者名稱
    ]
    readonly_fields = [
        "created_at"    # 發送時間為唯讀欄位
    ]
```

#### 1.4.4 視圖函式 (`chats/views.py`)

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.http import HttpResponseForbidden
from .models import ChatRoom, Message
from projects.models import Project

@login_required  # 確保使用者已登入
def chat_list(request):
    """顯示使用者的所有聊天室列表"""
    # 取得使用者作為專案擁有者的聊天室，按最後更新時間排序
    owner_chats = ChatRoom.objects.filter(
        project__account=request.user
    ).order_by("-updated_at")

    # 取得使用者作為訪客的聊天室，按最後更新時間排序
    visitor_chats = ChatRoom.objects.filter(
        visitor=request.user
    ).order_by("-updated_at")

    # 渲染聊天室列表模板
    return render(
        request,
        "chats/chat_list.html",
        {
            "owner_chats": owner_chats,     # 作為擁有者的聊天室
            "visitor_chats": visitor_chats,  # 作為訪客的聊天室
        },
    )

@login_required  # 確保使用者已登入
def create_chat(request, project_id):
    """建立新的聊天室或重新導向到現有聊天室"""
    # 取得專案資訊，如果不存在則回傳 404
    project = get_object_or_404(Project, id=project_id)

    # 檢查使用者是否已經有與該專案的聊天室
    existing_chat = ChatRoom.objects.filter(
        project=project,
        visitor=request.user
    ).first()

    if existing_chat:
        # 如果已經存在聊天室，重新導向到該聊天室
        return redirect(
            "chats:room",
            project_id=project.id,
            room_slug=existing_chat.slug
        )

    # 建立新的聊天室
    chat_room = ChatRoom.objects.create(
        project=project,
        visitor=request.user
    )

    # 重新導向到新建立的聊天室
    return redirect(
        "chats:room",
        project_id=project.id,
        room_slug=chat_room.slug
    )

@login_required  # 確保使用者已登入
def chat_room(request, project_id, room_slug):
    """顯示聊天室頁面"""
    # 取得專案和聊天室資訊，如果不存在則回傳 404
    project = get_object_or_404(Project, id=project_id)
    chat_room = get_object_or_404(ChatRoom, project=project, slug=room_slug)

    # 確保使用者是聊天室的參與者（專案擁有者或訪客）
    if request.user not in [project.account, chat_room.visitor]:
        return HttpResponseForbidden("您沒有權限訪問此聊天室")

    # 取得聊天室的歷史訊息，按時間正序排列
    chat_messages = Message.objects.filter(
        chat_room=chat_room
    ).order_by("created_at")

    # 渲染聊天室模板
    return render(
        request,
        "chats/room.html",
        {
            "chat_room": chat_room,       # 聊天室資訊
            "project": project,           # 專案資訊
            "chat_messages": chat_messages,  # 聊天記錄
        },
    )
```

### 1.4.5 資料模型 (`chats/models.py`)

```python
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify

class ChatRoom(models.Model):
    """聊天室模型

    用於存儲專案擁有者和訪客之間的聊天室資訊
    每個聊天室關聯一個專案和一個訪客使用者
    """
    # 關聯到專案模型，當專案被刪除時，相關的聊天室也會被刪除
    project = models.ForeignKey(
        "projects.Project",
        on_delete=models.CASCADE,
        related_name="chat_rooms"  # 可通過 project.chat_rooms 訪問相關聊天室
    )

    # 訪客使用者，當使用者被刪除時，相關的聊天室也會被刪除
    visitor = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="visited_chat_rooms"  # 可通過 user.visited_chat_rooms 訪問相關聊天室
    )

    # 用於 URL 的唯一標識符
    slug = models.SlugField(unique=True, max_length=100)

    # 時間戳記
    created_at = models.DateTimeField(auto_now_add=True)  # 創建時間
    updated_at = models.DateTimeField(auto_now=True)      # 最後更新時間

    def save(self, *args, **kwargs):
        """重寫 save 方法以自動生成 slug"""
        # 自動生成 slug，格式為 "專案ID-訪客ID"
        if not self.slug:
            self.slug = slugify(f"{self.project.id}-{self.visitor.id}")
        super().save(*args, **kwargs)

    @property
    def participants(self):
        """返回聊天室參與者列表"""
        # 返回專案擁有者和訪客的列表
        return [self.project.account, self.visitor]

    def __str__(self):
        """返回聊天室的文字字串表示"""
        # 用於管理界面和調試的文字字串表示
        return f"Chat {self.slug} - Project: {self.project.title}, Visitor: {self.visitor.username}"

    class Meta:
        """模型的元數據"""
        ordering = ["-updated_at"]  # 按最後更新時間倒序排列
        verbose_name = "聊天室"
        verbose_name_plural = "聊天室"

class Message(models.Model):
    """聊天訊息模型

    用於存儲聊天室中的訊息記錄
    每條訊息關聯一個聊天室和一個寄件者
    """
    # 關聯到聊天室，當聊天室被刪除時，相關的訊息也會被刪除
    chat_room = models.ForeignKey(
        ChatRoom,
        on_delete=models.CASCADE,
        related_name="messages"  # 可通過 chat_room.messages 訪問相關訊息
    )

    # 寄件者，當使用者被刪除時，相關的訊息也會被刪除
    sender = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="sent_messages"  # 可通過 user.sent_messages 訪問寄送的訊息
    )

    # 訊息內容
    content = models.TextField(
        verbose_name="內容",
        help_text="聊天訊息的內容"
    )

    # 訊息是否已讀
    is_read = models.BooleanField(
        default=False,
        verbose_name="已讀狀態",
        help_text="標記訊息是否已被接收者閱讀"
    )

    # 發送時間
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name="發送時間"
    )

    class Meta:
        """模型的元數據"""
        ordering = ["created_at"]  # 按創建時間正序排列
        verbose_name = "聊天訊息"
        verbose_name_plural = "聊天訊息"

    def __str__(self):
        """返回訊息的文字字串表示"""
        # 用於管理界面和調試的文字字串表示，顯示前50個字符
        return f"{self.sender.username}: {self.content[:50]}"
```

### 1.4.6 WebSocket 路由 (`chats/routing.py`)

````python
from django.urls import re_path
from . import consumers

"""WebSocket 路由設定

此檔案定義了 WebSocket 連接的 URL 模式
使用 re_path 來支援正則表達式匹配
"""

websocket_urlpatterns = [
    # WebSocket URL 格式：ws/chat/<project_id>/<room_slug>/
    # project_id: 專案ID (數字)
    # room_slug: 聊天室標識符 (字母、數字、連字符)
    re_path(
        r"ws/chat/(?P<project_id>\d+)/(?P<room_slug>[\w-]+)/$",
        consumers.ChatConsumer.as_asgi(),
    ),
]

## 2. 啟動服務

在開始開發之前，請確保啟動所有必要的服務：

1. 啟動 Redis 服務：
```bash
# 啟動 Redis
brew services start redis

# 檢查 Redis 狀態
brew services list
````

2. 啟動 Django 開發伺服器（終端機 1）：

```bash
poetry run python manage.py runserver
```

3. 啟動 Daphne WebSocket 伺服器（終端機 2）：

```bash
DJANGO_SETTINGS_MODULE=core.settings poetry run daphne -b 0.0.0.0 -p 8001 core.asgi:application
```

## 3. 功能測試

1. 使用兩個不同的瀏覽器或隱私視窗登入不同帳號
2. 一個帳號為專案擁有者，另一個為訪客
3. 訪客點擊「聯絡提案者」按鈕創建聊天室
4. 雙方可以在聊天室中即時發送和接收訊息

## 4. 常見問題排解

### 4.1 WebSocket 連接問題

- 確保 Redis 服務正在運行
- 確保 Daphne 伺服器正在運行在正確的端口（8001）
- 檢查瀏覽器控制台的錯誤信息
- 確認 WebSocket URL 格式正確

### 4.2 訊息發送問題

- 確保使用者已經登入
- 檢查 Redis 連接狀態
- 查看 Daphne 伺服器的日誌輸出
