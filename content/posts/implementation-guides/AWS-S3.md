---
title: "Django AWS-S3本機的settings.py如何做設定"
date: 2024-12-24
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

#### 注意需要安裝 django-storages 以及 boto3

##### django-storages 安裝

```
poetry add django-storages
```

##### boto3 安裝

```
poetry add boto3
```

這兩者的作用，在最後做說明

#### settings.py 設定方法

```python
# 設定默認的文件存儲後端為 AWS S3
DEFAULT_FILE_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"

# 設定媒體文件的 URL 前綴
MEDIA_URL = os.getenv("MEDIA_URL")

# Django Storages 配置
STORAGES = {
    # 默認存儲配置（用於媒體文件）
    "default": {
        # 使用 AWS S3 作為存儲後端
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "access_key": os.getenv("AWS_ACCESS_KEY_ID"),  # AWS 訪問密鑰 ID
            "secret_key": os.getenv("AWS_SECRET_ACCESS_KEY"),  # AWS 訪問密鑰
            "bucket_name": os.getenv("AWS_STORAGE_BUCKET_NAME"),  # S3 儲存桶名稱
            "region_name": os.getenv("AWS_S3_REGION_NAME"),  # AWS 區域名稱
        },
    },
    # 靜態文件存儲配置
    "staticfiles": {
        # 使用 Django 默認的靜態文件存儲
        "BACKEND": "django.contrib.staticfiles.storage.StaticFilesStorage",
    },
}
```

##### django-storages 的作用

django-storages 是一個 Django 的第三方應用，用來支持各種雲端存儲後端（包括 AWS S3、Google Cloud Storage、Azure 等），它幫助開發者更輕鬆地在 Django 中使用這些存儲服務。

1. 主要功能

- 擴展 Django 的文件存儲後端：
- 替代 Django 默認的文件存儲後端，將媒體文件（MEDIA）或靜態文件（STATIC）直接存儲到 S3。
- 通過 DEFAULT_FILE_STORAGE 和 STATICFILES_STORAGE 配置使用不同的存儲後端。
- 與 Django 無縫集成：
- 支持使用 Django 提供的文件存儲 API，無需直接處理 S3 的低層代碼。
- 例如，django-storages 允許你在管理面板中上傳圖片，並自動存儲到 S3。

2. 簡化的配置與使用

- 使用 django-storages，只需在 settings.py 中配置幾個參數（如 AWS_ACCESS_KEY_ID 和 AWS_STORAGE_BUCKET_NAME），即可輕鬆將文件存儲到 S3，而不需要自己處理複雜的邏輯。

##### boto3 的作用

boto3 是 AWS 提供的官方 Python SDK，用來與 AWS 服務（如 S3、DynamoDB、EC2 等）進行交互。django-storages 的 AWS S3 支持就是基於 boto3 實現的。

1. 主要功能

- 直接訪問 AWS S3 API：
- 允許開發者直接執行低層次的操作，例如創建 S3 存儲桶、上傳文件、生成預簽名 URL、刪除文件等。
- 全面支持 AWS 服務：
- 除了 S3，boto3 還支持 EC2、CloudWatch、DynamoDB 等多個 AWS 服務。
- 細粒度控制：
  - 允許開發者細緻地控制 S3 文件的訪問權限、對象標籤、加密選項等。

2. 適合的場景

- 當你需要直接與 AWS API 交互時，例如：
- 自動化文件管理（批量上傳/刪除文件）。
- 配置自定義的文件生命周期。
- 自動生成預簽名 URL，用於臨時授權訪問。
