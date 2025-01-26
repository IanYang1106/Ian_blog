---
title: "Accounts 功能解析"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# Accounts 功能解說

## 核心功能實現

### 1. 用戶註冊 (CustomUserCreationForm)
```python
class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)
    username = forms.CharField(required=True)
    password1 = forms.CharField(required=True, widget=forms.PasswordInput)
    password2 = forms.CharField(required=True, widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ("username", "email", "password1", "password2")
```

### 2. 用戶個人資料管理
```python
@login_required
def index(request):
    account = request.user
    profile = Profile.objects.get_or_create(
        account=account,
        defaults={
            'name': account.username,
            'location': "",
            'bio': "",
            'birthday': None,
            'website': "",
        }
    )[0]
```

### 3. 登入/登出功能
```python
def login(request):
    user = authenticate(
        username=request.POST.get("username"),
        password=request.POST.get("password"),
    )
    if user is not None:
        login_user(request, user)
```

## 特色功能

1. **整合郵件通知**
   - 註冊歡迎郵件
   - 使用 Anymail 發送模板郵件

2. **聊天室整合**
   - 顯示用戶參與的聊天室
   - 區分專案擁有者和訪客聊天

3. **贊助記錄管理**
   - 顯示用戶的贊助歷史
   - 包含贊助狀態和獎勵信息

4. **個人資料更新**
   - 支持頭像上傳
   - 自動記錄更新時間

## 安全性實現

1. **密碼驗證**
   - 密碼匹配檢查
   - Django 內建密碼加密

2. **訪問控制**
   - 使用 `@login_required` 裝飾器
   - POST 請求保護 (`@require_POST`)

## URL 配置
```python
urlpatterns = [
    path("login/", views.login, name="login"),
    path("logout/", views.logout, name="logout"),
    path("register/", views.register, name="register"),
    path("", views.index, name="index"),
    path("terms/", views.terms, name="terms"),
    path("privacy/", views.privacy, name="privacy"),
]
```

## 使用注意事項

1. **資料完整性**
   - 用戶註冊需要有效的電子郵件
   - 密碼需符合安全要求

2. **關聯處理**
   - 自動創建用戶個人資料
   - 維護用戶與贊助、聊天室的關聯

3. **郵件發送**
   - 需配置 Anymail 設置
   - 確保郵件模板 ID 正確 