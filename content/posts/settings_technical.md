---
title: "Django專案中使用的settings.py 功能說明"
date: 2025-01-26
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

```python
# 導入必要的模組
from pathlib import Path  # 用於處理文件路徑
import environ           # 用於處理環境變數
import os               # 用於操作系統功能
from dotenv import load_dotenv  # 用於載入 .env 文件

# 載入 .env 文件中的環境變數
load_dotenv()

# 初始化環境變數
BASE_DIR = Path(__file__).resolve().parent.parent  # 獲取專案根目錄路徑
env = environ.Env()  # 創建環境變數處理器
environ.Env.read_env(env_file=BASE_DIR / ".env")  # 讀取 .env 文件

# 再次定義 BASE_DIR（這是一個重複定義，可能是歷史遺留）
BASE_DIR = Path(__file__).resolve().parent.parent

# 核心安全設置
SECRET_KEY = env("SECRET_KEY", default="your-default-secret-key")  # Django 密鑰
DEBUG = env.bool("DEBUG", default=True)  # 調試模式開關
ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")  # 允許的主機列表

# 應用程序定義
INSTALLED_APPS = [
    # Django 內建應用
    "django.contrib.admin",          # 管理後台
    "django.contrib.auth",           # 認證系統
    "django.contrib.contenttypes",   # 內容類型框架
    "django.contrib.sessions",       # 會話管理
    "django.contrib.messages",       # 消息框架
    "django.contrib.staticfiles",    # 靜態文件
    "django.contrib.humanize",       # 數據人性化顯示
    
    # 自定義應用
    "accounts",                      # 帳戶管理
    "homepages",                    # 首頁
    "projects",                     # 專案管理
    "faqs",                        # 常見問題
    "users",                       # 用戶管理
    
    # Django sites 框架
    "django.contrib.sites",
    
    # 第三方應用
    "allauth",                     # 認證框架
    "allauth.account",            # 帳戶管理
    "allauth.socialaccount",      # 社交帳戶
    "allauth.socialaccount.providers.google",  # Google 登入
    "storages",                   # 文件存儲
    "comments",                   # 評論系統
    "categories",                 # 分類系統
    "rewards",                    # 回饋系統
    "update_records",             # 更新記錄
    "payments",                   # 支付系統
    "chats",                      # 聊天系統
    "anymail",                    # 郵件系統
    "csp",                        # 內容安全策略
]

# 站點 ID
SITE_ID = 1

# 中間件配置
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",         # 安全中間件
    "csp.middleware.CSPMiddleware",                         # 內容安全策略
    "django.contrib.sessions.middleware.SessionMiddleware",  # 會話處理
    "django.middleware.common.CommonMiddleware",            # 通用功能
    "django.middleware.csrf.CsrfViewMiddleware",            # CSRF 保護
    "django.contrib.auth.middleware.AuthenticationMiddleware", # 用戶認證
    "django.contrib.messages.middleware.MessageMiddleware",   # 消息處理
    "django.middleware.clickjacking.XFrameOptionsMiddleware", # 點擊劫持保護
    "allauth.account.middleware.AccountMiddleware",          # Allauth 中間件
]

# 安全設置
SECURE_BROWSER_XSS_FILTER = True           # 開啟瀏覽器 XSS 過濾
SECURE_CONTENT_TYPE_NOSNIFF = True         # 防止內容類型嗅探
SECURE_HSTS_SECONDS = 31536000             # HSTS 有效期（1年）
SECURE_HSTS_INCLUDE_SUBDOMAINS = True      # HSTS 包含子域名
SECURE_HSTS_PRELOAD = True                 # HSTS 預載入

# 內容安全策略 (CSP) 設置
CSP_DEFAULT_SRC = ("'self'",)                          # 默認只允許同源資源
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")          # 樣式來源
CSP_SCRIPT_SRC = ("'self'",)                          # 腳本來源
CSP_IMG_SRC = ("'self'", "data:", "https:")           # 圖片來源
CSP_FONT_SRC = ("'self'", "https:", "data:")          # 字體來源

# Cookie 設置
SESSION_COOKIE_HTTPONLY = True              # Session Cookie 禁止 JS 訪問
CSRF_COOKIE_HTTPONLY = True                 # CSRF Cookie 禁止 JS 訪問
if not DEBUG:                               # 生產環境額外安全設置
    CSRF_COOKIE_SECURE = True               # CSRF Cookie 只通過 HTTPS 傳輸
    SESSION_COOKIE_SECURE = True            # Session Cookie 只通過 HTTPS 傳輸

# URL 配置
ROOT_URLCONF = "core.urls"                  # 根 URL 配置文件

# 模板配置
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",  # 模板引擎
        "DIRS": ["templates"],                                        # 模板目錄
        "APP_DIRS": True,                                            # 搜索應用模板目錄
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",           # 調試信息
                "django.template.context_processors.request",         # 請求信息
                "django.contrib.auth.context_processors.auth",        # 認證信息
                "django.contrib.messages.context_processors.messages", # 消息信息
            ],
            "autoescape": True,                                      # 自動轉義
        },
    },
]

# WSGI 應用配置
WSGI_APPLICATION = "core.wsgi.application"

# 資料庫配置
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",     # 使用 MySQL 引擎
        "NAME": env("DB_NAME"),                   # 數據庫名稱
        "USER": env("DB_USER"),                   # 數據庫用戶
        "PASSWORD": env("DB_PASSWORD"),           # 數據庫密碼
        "HOST": env("DB_HOST"),                   # 數據庫主機
        "PORT": env("DB_PORT"),                   # 數據庫端口
    }
}

# 密碼驗證配置
AUTH_PASSWORD_VALIDATORS = [
    {
        # 檢查密碼與用戶資料相似度
        "NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator",
        "OPTIONS": {
            "user_attributes": ("username", "email"),  # 檢查的屬性
            "max_similarity": 0.8,                     # 最大相似度
        },
    },
    {
        "NAME": "django.contrib.auth.password_validation.MinimumLengthValidator",  # 最小長度
    },
    {
        "NAME": "django.contrib.auth.password_validation.CommonPasswordValidator",  # 常見密碼
    },
    {
        "NAME": "django.contrib.auth.password_validation.NumericPasswordValidator", # 純數字密碼
    },
]

# 國際化配置
LANGUAGE_CODE = "en-us"                # 語言代碼
TIME_ZONE = "Asia/Taipei"              # 時區設置
USE_I18N = True                        # 開啟國際化
USE_TZ = True                          # 使用時區

# 靜態文件配置
STATIC_URL = "/static/"                # 靜態文件 URL
STATIC_ROOT = BASE_DIR / "staticfiles" # 靜態文件收集目錄
STATICFILES_DIRS = [                   # 額外的靜態文件目錄
    BASE_DIR / "static",
    BASE_DIR / "frontends",
]

# 默認主鍵類型
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"

# 認證後端配置
AUTHENTICATION_BACKENDS = (
    "django.contrib.auth.backends.ModelBackend",           # Django 默認後端
    "allauth.account.auth_backends.AuthenticationBackend", # Allauth 後端
)

# 社交帳戶配置
SOCIALACCOUNT_PROVIDERS = {
    "google": {
        "APPS": [
            {
                "client_id": env("GMAIL_CLIENT_ID"),      # Google 客戶端 ID
                "secret": env("GMAIL_API_SECRET"),        # Google API 密鑰
                "settings": {
                    "scope": ["profile", "email"],        # 請求權限範圍
                    "auth_params": {"access_type": "online"},
                },
            },
        ],
    }
}

# 登入相關配置
LOGIN_REDIRECT_URL = "/"                # 登入後重定向 URL
LOGIN_URL = "/accounts/login"           # 登入 URL

# 社交帳戶設置
SOCIALACCOUNT_LOGIN_ON_GET = True      # GET 請求時直接登入
ACCOUNT_EMAIL_REQUIRED = True          # 要求電子郵件
ACCOUNT_UNIQUE_EMAIL = True            # 電子郵件唯一
ACCOUNT_AUTHENTICATION_METHOD = "email" # 使用郵件認證

# 文件存儲配置
DEFAULT_FILE_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"  # 使用 S3 存儲
MEDIA_URL = os.getenv("MEDIA_URL")     # 媒體文件 URL

# 存儲後端配置
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",  # S3 存儲後端
        "OPTIONS": {
            "access_key": os.getenv("AWS_ACCESS_KEY_ID"),      # AWS 訪問密鑰
            "secret_key": os.getenv("AWS_SECRET_ACCESS_KEY"),  # AWS 密鑰
            "bucket_name": os.getenv("AWS_STORAGE_BUCKET_NAME"), # S3 桶名
            "region_name": os.getenv("AWS_S3_REGION_NAME"),    # AWS 區域
        },
    },
    "staticfiles": {
        "BACKEND": "django.contrib.staticfiles.storage.StaticFilesStorage", # 靜態文件後端
    },
}

# CSRF 信任源
CSRF_TRUSTED_ORIGINS = os.getenv("CSRF_TRUSTED_ORIGINS").split(",")

# ASGI 應用
ASGI_APPLICATION = "core.asgi.application"

# 郵件配置
ANYMAIL = {
    "MAILGUN_API_KEY": os.getenv("MAILGUN_API_KEY"),           # Mailgun API 密鑰
    "MAILGUN_SENDER_DOMAIN": os.getenv("MAILGUN_SENDER_DOMAIN"), # 發送域名
}
EMAIL_BACKEND = "anymail.backends.mailgun.EmailBackend"         # 郵件後端
DEFAULT_FROM_EMAIL = "吱吱 CHICHI <CHICHI@mg.chichii.com>"      # 默認發件人

# AWS S3 配置
AWS_STORAGE_BUCKET_NAME = env("AWS_STORAGE_BUCKET_NAME")        # S3 桶名
AWS_S3_CUSTOM_DOMAIN = f"{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com"  # S3 域名

# CSRF 信任源（重複定義，可能需要整合）
CSRF_TRUSTED_ORIGINS = [
    "https://chichii.com",
]

# Google Maps API 配置
GOOGLE_MAPS_API_KEY = os.getenv("GOOGLE_MAPS_API_KEY")         # Google Maps API 密鑰
GOOGLE_MAPS_MAP_ID = os.getenv("GOOGLE_MAPS_MAP_ID")          # Google Maps 地圖 ID
``` 