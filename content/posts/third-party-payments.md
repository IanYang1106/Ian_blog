---
title: "以django實現第三方支付功能串接"
date: 2024-12-22
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

### 資料夾結構

```zsh
project_root/
│
├── manage.py                     # Django 專案的入口文件
├── requirements.txt              # 專案的依賴套件清單
├── .env                          # 環境變數設定文件，包含綠界的 API 金鑰與商店代號
│
├── project_name/                 # Django 核心專案目錄
│   ├── __init__.py
│   ├── settings.py               # 專案設定
│   ├── urls.py                   # 全域路由設定
│   ├── wsgi.py                   # 部署相關設定
│   ├── asgi.py                   # 部署相關設定
│   ├── static/                   # 靜態檔案目錄
│
├── payments/                     # 綠界金流專用的 app 目錄
│   ├── __init__.py
│   ├── admin.py                  # 用於管理後台查看訂單紀錄
│   ├── apps.py                   # App 註冊配置
│   ├── models.py                 # 資料庫模型，定義訂單資訊與交易狀態
│   ├── views.py                  # 處理支付請求與回調的邏輯
│   ├── urls.py                   # 金流相關的路由設定
│   ├── forms.py                  # 用於前端的支付表單處理
│   ├── services.py               # 綠界 API 的封裝邏輯
│   ├── utils.py                  # 工具函數（如簽名生成與驗證）
│   ├── templates/                # 金流相關的 HTML 模板
│   │   ├── payments/
│   │       ├── checkout.html     # 支付頁面
│   │       ├── success.html      # 支付成功頁面
│   │       ├── failure.html      # 支付失敗頁面
│   └── static/                   # 該 app 專用的靜態資源
│       ├── payments/
│           ├── styles.css        # 支付相關樣式
│           ├── scripts.js        # 支付相關腳本
│
└── migrations/                   # 資料庫遷移記錄
```

### 檔案與功能詳細說明

#### 核心檔案

1. models.py
   定義支付相關的資料表，如：
   • 訂單資訊（Order）。
   • 交易記錄（Transaction）。
   • 可擴展其他金流處理記錄。
2. views.py
   包含處理支付的邏輯，如：
   • 生成支付表單。
   • 接收綠界的回調（NotifyURL 和 ReturnURL）。
   • 驗證支付簽名。
3. urls.py
   定義金流相關的 URL 路由，例如：

```py
from django.urls import path
from . import views

urlpatterns = [
    path('checkout/', views.checkout, name='checkout'),  # 支付頁面
    path('callback/', views.callback, name='callback'),  # 綠界的回調
    path('success/', views.success, name='success'),    # 支付成功頁
    path('failure/', views.failure, name='failure'),    # 支付失敗頁
]
```

4. services.py
   • 封裝綠界金流 API 的邏輯，如簽名生成、交易請求等。
   • 例如：

```py
import hashlib
import requests

def generate_signature(params, hash_key, hash_iv):
    raw = f"HashKey={hash_key}&{params}&HashIV={hash_iv}"
    return hashlib.sha256(raw.encode('utf-8')).hexdigest().upper()

def create_payment_request(order_data):
    url = "https://payment.ecpay.com.tw/Cashier/AioCheckOut/V5"
    response = requests.post(url, data=order_data)
    return response.json()
```

5. utils.py
   • 工具函數，例如：
   • 驗證綠界簽名。
   • 格式化金流參數。
