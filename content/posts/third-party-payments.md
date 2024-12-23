---
title: "以django實現第三方支付功能串接"
date: 2024-12-22
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

### 第三方支付串接綠界金流

#### 測試環境與環境變數安裝

- 下載 ngrok 用來做測試

```
brew install ngrok
```

- 載入設定檔
  每個帳號自己都有屬於自己的設定檔

```
ngrok config add-authtoken <帳號的authtoken>
```

- 如果不想固定設定檔改輸入下列指令啟動

```
ngrok http 8000 --authtoken=<帳號的authtoken>
```

- 已固定設定檔後啟動 ngrok 產生虛擬網址

```
ngrok http 8000
```

- 安裝 dotenv
  使用 poetry 進行安裝

```
poetry add python-dotenv
```

- 使用 venv 進行安裝

```
pip install python-dotenv
```

#### 資料夾結構

```
payments/
│
├── __pycache__/                # Python 編譯檔案目錄
│
├── migrations/                 # 資料庫遷移檔案目錄
│
├── templates/                  # 模板檔案目錄
│   └── payments/
│       ├── create_payment.html # 建立付款頁面
│       ├── checkout.html       # 結帳確認頁面
│       └── complete.html       # 付款完成頁面
│
├── __init__.py                # Python 套件標示檔
├── admin.py                   # Django 後台管理設定
├── apps.py                    # Django 應用程式設定
├── models.py                  # 資料模型定義
├── tests.py                   # 測試檔案
├── urls.py                    # URL 路由設定
├── views.py                   # 視圖邏輯處理
└── ecpay_sdk.py              # 綠界金流 SDK 實作
```

#### 檔案功能

1. 資料模型 (models.py):

- 定義了 Order 模型來儲存訂單資訊
- 包含欄位：
  - order_id: 訂單編號
  - amount: 訂單金額
  - description: 訂單描述
  - created_at: 建立時間
  - paid: 支付狀態

2. 視圖邏輯 (views.py):

- create_payment: 建立新的支付訂單
  - 接收前端提交的金額和描述
  - 生成訂單編號
  - 建立綠界支付訂單
- payment_notify: 處理綠界支付回調 - 接收支付結果通知 - 更新訂單狀態
  payment_complete: 顯示支付完成頁面

3. 綠界金流整合 (ecpay_sdk.py):

- ECPayPayment 類別處理與綠界金流的整合
- 主要功能：
  - 建立訂單參數
  - 產生檢查碼
  - 處理加密和編碼

4. 其他檔案:

- urls.py: 定義支付相關的 URL 路由
- templates/: 存放支付相關的前端模板
- migrations/: 資料庫遷移檔案

##### models.py

```py
from django.db import models

class Order(models.Model):
    """
    訂單模型：用於儲存支付訂單的相關資訊
    """
    # 訂單編號，最大長度100字元，必須是唯一值
    order_id = models.CharField(max_length=100, unique=True)

    # 訂單金額，使用整數型別儲存（例：1000 表示新台幣1000元）
    amount = models.IntegerField()

    # 訂單描述，最大長度200字元，用於說明訂單內容
    description = models.CharField(max_length=200)

    # 訂單建立時間，會自動設定為當前時間
    created_at = models.DateTimeField(auto_now_add=True)

    # 支付狀態，預設為 False（未支付）
    paid = models.BooleanField(default=False)

    def __str__(self):
        """
        模型的字串表示方法，用於在Django管理後台顯示訂單資訊
        """
        return f"Order {self.order_id}"

```

##### views.py

```py
from django.shortcuts import render, redirect
from django.views.decorators.csrf import csrf_exempt
from django.http import HttpResponse
from django.conf import settings
from .models import Order
from .ecpay_sdk import ECPayPayment
from datetime import datetime
import json


# 創建支付訂單的視圖函數
def create_payment(request):
    if request.method == "POST":
        # 從 POST 請求中獲取訂單金額和描述
        amount = request.POST.get("amount")
        description = request.POST.get("description", "商品訂單")

        # 使用時間戳生成唯一的訂單編號
        order_id = f"TEST{datetime.now().strftime('%Y%m%d%H%M%S')}"

        # 在資料庫中創建新的訂單記錄
        order = Order.objects.create(
            order_id=order_id, amount=amount, description=description
        )

        # 初始化綠界支付 SDK 並創建綠界訂單
        ecpay = ECPayPayment()
        order_params = ecpay.create_order(
            order_id=order_id, amount=amount, description=description
        )

        # 渲染結帳頁面，傳入支付網址和訂單參數
        return render(
            request,
            "payments/checkout.html",
            {"payment_url": settings.ECPAY_PAYMENT_URL, "order_params": order_params},
        )

    # GET 請求時顯示創建訂單的表單頁面
    return render(request, "payments/create_payment.html")


# 處理綠界支付回調的視圖函數
# 使用 csrf_exempt 裝飾器忽略 CSRF 驗證，因為綠界的回調不會帶有 CSRF token
@csrf_exempt
def payment_notify(request):
    if request.method == "POST":
        # 獲取綠界回傳的所有參數
        notify_data = request.POST.dict()

        try:
            # 根據訂單編號查找對應的訂單
            order = Order.objects.get(order_id=notify_data.get("MerchantTradeNo"))

            # 檢查交易狀態，RtnCode=1 表示交易成功
            if notify_data.get("RtnCode") == "1":
                order.paid = True
                order.save()

            # 回應綠界，告知已收到通知
            return HttpResponse("1|OK")
        except Order.DoesNotExist:
            return HttpResponse("Order not found", status=404)

    return HttpResponse("Bad Request", status=400)


# 支付完成後的返回頁面
def payment_complete(request):
    return render(request, "payments/complete.html")

```

##### ecpay_sdk.py

```python
import urllib.parse
import hashlib
from datetime import datetime
from django.conf import settings


class ECPayPayment:
    """
    綠界金流支付整合類別
    負責處理與綠界金流服務的串接，包含訂單建立、參數加密等功能
    """

    def __init__(self):
        """
        初始化綠界金流設定
        從 Django settings 中讀取必要的金流參數
        """
        # 商店代號，由綠界提供的特店編號
        self.merchant_id = settings.ECPAY_MERCHANT_ID
        # 金流 API 串接金鑰，用於參數加密
        self.hash_key = settings.ECPAY_HASH_KEY
        # 金流 API 串接密碼，用於參數加密
        self.hash_iv = settings.ECPAY_HASH_IV
        # 綠界金流 API 的基礎 URL
        self.payment_url = settings.ECPAY_PAYMENT_URL

    def create_order(self, order_id, amount, description):
        """
        建立綠界金流訂單

        參數說明：
        order_id (str): 訂單編號，必須唯一
        amount (int/str): 訂單金額，必須為正整數
        description (str): 交易描述

        回傳：
        dict: 包含所有需要傳送給綠界的訂單參數
        """
        # 準備綠界訂單所需的基本參數
        order_params = {
            "MerchantID": self.merchant_id,  # 商店代號
            "MerchantTradeNo": order_id,  # 訂單編號
            "MerchantTradeDate": datetime.now().strftime(
                "%Y/%m/%d %H:%M:%S"
            ),  # 訂單建立時間
            "PaymentType": "aio",  # 交易類型: 綜合支付
            "TotalAmount": str(amount),  # 訂單金額
            "TradeDesc": urllib.parse.quote(description),  # URL 編碼的交易描述
            "ItemName": description,  # 商品名稱
            "ReturnURL": "https://7f2f-2001-b011-4008-1e8a-9199-5d4-645a-4b82.ngrok-free.app/notify/",  # 交易通知網址
            "ClientBackURL": "https://7f2f-2001-b011-4008-1e8a-9199-5d4-645a-4b82.ngrok-free.app/payments/complete/",  # 支付完成返回網址
            "ChoosePayment": "ALL",  # 付款方式：ALL 為所有支付方式
            "EncryptType": "1",  # 加密類型：1 為 SHA256
        }

        # 產生檢查碼並加入訂單參數中
        check_mac = self._generate_check_mac(order_params)
        order_params["CheckMacValue"] = check_mac

        return order_params

    def _generate_check_mac(self, params):
        """
        產生綠界訂單檢查碼

        參數說明：
        params (dict): 訂單參數字典

        回傳：
        str: SHA256 加密後的檢查碼

        檢查碼規則：
        1. 將參數依照參數名稱排序
        2. 將參數值用 & 符號串接
        3. 在首尾加入 HashKey 和 HashIV
        4. 將字串轉為小寫並進行 URL encode
        5. 使用 SHA256 進行加密並轉為大寫
        """
        # 依照參數名稱排序
        sorted_params = sorted(params.items())

        # 組合參數字串
        param_str = f"HashKey={self.hash_key}"
        for key, value in sorted_params:
            param_str += f"&{key}={value}"
        param_str += f"&HashIV={self.hash_iv}"

        # URL encode 並轉為小寫
        encoded_str = urllib.parse.quote_plus(param_str).lower()

        # SHA256 加密並轉為大寫
        check_mac = hashlib.sha256(encoded_str.encode("utf-8")).hexdigest().upper()

        return check_mac

```

##### 檢查碼產生原理簡易範例

1. 參數名稱的字母順序排列

```
MerchantID=2000132
MerchantTradeNo=Test123456
PaymentType=aio
```

2. 加上 HashKey 和 HashIV
   - 在最前面加上 HashKey=你的 HashKey。
   - 在最後加上 &HashIV=你的 HashIV。

```
HashKey=YourHashKey&MerchantID=2000132&MerchantTradeNo=Test123456&PaymentType=aio&HashIV=YourHashIV
```

3. URL 編碼
   - 把整個字串進行 URL encode，把特殊字元（如空格、& 等）轉換為編碼格式。

```
HashKey%3DYourHashKey%26MerchantID%3D2000132%26MerchantTradeNo%3DTest123456%26PaymentType%3Daio%26HashIV%3DYourHashIV
```

4. 加密
   - 對 URL encode 後的結果使用 SHA256 算法進行加密。
   - 結果會是一個 64 位的十六進位字串。
5. 轉換成大寫
   - 把加密後的結果轉換為大寫，得到最終的 CheckMacValue。

前面五個步驟以此範例依序會有下列變化

1. 排序後參數：

```
MerchantID=2000132&MerchantTradeNo=Test123456&PaymentType=aio
```

2. 加上 HashKey 和 HashIV：

```
HashKey=YourHashKey&MerchantID=2000132&MerchantTradeNo=Test123456&PaymentType=aio&HashIV=YourHashIV
```

3. URL encode：

```
HashKey%3DYourHashKey%26MerchantID%3D2000132%26MerchantTradeNo%3DTest123456%26PaymentType%3Daio%26HashIV%3DYourHashIV
```

4. SHA256 加密並轉大寫：

```
8A72D16F0D2D62234A5796B9B0A55D4B4EA65F8E3F8305066CFF98D8F516DAB5
```

##### urls.py

```py
from django.urls import path
from . import views

app_name = 'payments'

urlpatterns = [
    path('create/', views.create_payment, name='create_payment'),
    path('notify/', views.payment_notify, name='payment_notify'),
    path('complete/', views.payment_complete, name='payment_complete'),
]
```

##### checkout.html

```html
{% extends "shared/layout.html" %}{% load socialaccount %} {% block "content" %}
<div class="min-h-screen flex items-center justify-center">
  <div class="bg-white p-8 rounded-lg shadow-lg max-w-md w-full text-center">
    <div class="mb-6">
      <svg
        class="animate-spin h-12 w-12 text-indigo-600 mx-auto mb-4"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>

      <h2 class="text-2xl font-bold text-gray-800 mb-2">
        正在轉導至綠界支付...
      </h2>
      <p class="text-gray-600 mb-4">
        請稍候，您將在
        <span x-text="countdown" class="font-semibold text-indigo-600"></span>
        秒後自動轉導至付款頁面
      </p>
      <div class="w-full bg-gray-200 rounded-full h-2.5 mb-4">
        <div
          class="bg-indigo-600 h-2.5 rounded-full transition-all duration-1000"
          x-bind:style="'width: ' + ((3-countdown)/3 * 100) + '%'"
        ></div>
      </div>
    </div>

    <p class="text-sm text-gray-500">如果沒有自動轉導，請點擊下方按鈕</p>
    <button
      onclick="document.getElementById('ecpay-form').submit();"
      class="mt-4 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
    >
      手動前往付款
      <svg
        class="ml-2 -mr-1 h-4 w-4"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M14 5l7 7m0 0l-7 7m7-7H3"
        />
      </svg>
    </button>
  </div>
</div>

<form id="ecpay-form" method="post" action="{{ payment_url }}" class="hidden">
  {% for key, value in order_params.items %}
  <input type="hidden" name="{{ key }}" value="{{ value }}" />
  {% endfor %}
</form>

<script>
  setTimeout(() => {
    document.getElementById("ecpay-form").submit();
  }, 3000);
</script>
{% endblock %}
```

##### complete.html

```html
{% extends "shared/layout.html" %}{% load socialaccount %} {% block "content" %}
<div class="min-h-screen flex items-center justify-center">
  <div class="bg-white p-8 rounded-lg shadow-lg max-w-md w-full text-center">
    <div class="mb-8">
      <!-- 成功圖標 -->
      <div
        class="mx-auto flex items-center justify-center h-16 w-16 rounded-full bg-green-100 mb-4"
      >
        <svg
          class="h-10 w-10 text-green-500"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M5 13l4 4L19 7"
          ></path>
        </svg>
      </div>

      <h1 class="text-3xl font-bold text-gray-900 mb-2">付款完成</h1>
      <div class="text-lg text-green-600 mb-8">感謝您的購買！</div>

      <!-- 動畫效果 -->
      <div
        class="space-y-4"
        x-data="{ show: false }"
        x-init="setTimeout(() => show = true, 500)"
      >
        <div
          x-show="show"
          x-transition:enter="transition ease-out duration-300"
          x-transition:enter-start="opacity-0 transform -translate-y-4"
          x-transition:enter-end="opacity-100 transform translate-y-0"
          class="text-gray-600"
        >
          您的訂單已經成功處理
        </div>
      </div>
    </div>

    <div class="space-x-4">
      <a
        href="/"
        class="inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-md text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition-colors duration-200"
      >
        返回首頁
        <svg
          class="ml-2 -mr-1 h-5 w-5"
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
          />
        </svg>
      </a>
    </div>
  </div>
</div>
{% endblock %}
```

##### create_payment.html

```html
{% extends "shared/layout.html" %}{% load socialaccount %} {% block "content" %}
<div
  class="container mx-auto px-4 py-8 max-w-md"
  x-data="{ amount: '', description: '' }"
>
  <div class="bg-white rounded-lg shadow-lg p-6">
    <h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">建立付款</h1>

    <form method="post" class="space-y-6">
      {% csrf_token %}

      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700"> 金額 </label>
        <input
          type="number"
          name="amount"
          x-model="amount"
          required
          class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
          placeholder="請輸入金額"
        />
      </div>

      <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700"> 描述 </label>
        <input
          type="text"
          name="description"
          x-model="description"
          class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
          placeholder="請輸入商品描述"
        />
      </div>

      <div class="flex items-center justify-between mt-6">
        <div class="text-sm text-gray-600" x-show="amount">
          總金額：<span class="font-semibold" x-text="amount + ' 元'"></span>
        </div>
        <button
          type="submit"
          class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
          x-bind:disabled="!amount"
          x-bind:class="{'opacity-50 cursor-not-allowed': !amount}"
        >
          前往付款
        </button>
      </div>
    </form>
  </div>
</div>
{% endblock %}
```

##### settings.py

```python
import os
from dotenv import load_dotenv

load_dotenv()

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')

INSTALLED_APPS = [
    'payments'
]
# 環境變數
ECPAY_MERCHANT_ID = env('ECPAY_MERCHANT_ID')
ECPAY_HASH_KEY = env('ECPAY_HASH_KEY')
ECPAY_HASH_IV = env('ECPAY_HASH_IV')
ECPAY_PAYMENT_URL = env('ECPAY_PAYMENT_URL')
CSRF_TRUSTED_ORIGINS = os.getenv('CSRF_TRUSTED_ORIGINS').split(',')
```

##### .env

```python
ECPAY_MERCHANT_ID=3002607
ECPAY_HASH_KEY=pwFHCqoQZGmho4w6
ECPAY_HASH_IV=EkRm7iFT261dpevs
ECPAY_PAYMENT_URL=https://payment-stage.ecpay.com.tw/Cashier/AioCheckOut/V5
CSRF_TRUSTED_ORIGINS=https://ad33-61-220-182-115.ngrok-free.app
```

##### 當使用 ngrok 需要依照虛擬網址修改的地方

.env

```
ALLOWED_HOSTS=localhost,127.0.0.1,ad33-61-220-182-115.ngrok-free.app
CSRF_TRUSTED_ORIGINS=https://ad33-61-220-182-115.ngrok-free.app
```

ecpay_sdk.py

```python
'ReturnURL': 'https://7f2f-2001-b011-4008-1e8a-9199-5d4-645a-4b82.ngrok-free.app/notify/',  # 請修改為您的網域
'ClientBackURL': 'https://7f2f-2001-b011-4008-1e8a-9199-5d4-645a-4b82.ngrok-free.app/payments/complete/',  # 請修改為您的網域
```
