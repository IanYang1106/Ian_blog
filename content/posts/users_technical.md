---
title: "Users 功能解說"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# Users 功能解說

## 核心模型結構

### Profile 模型（用戶檔案）
```python
class Profile(models.Model):
    GENDER_CHOICES = [
        ("M", "男"),
        ("F", "女"),
        ("O", "其他"),
    ]

    name = models.CharField(max_length=50)
    birthday = models.DateField(null=True, blank=True, default="")
    location = models.CharField(max_length=100, null=True, blank=True, default="")
    website = models.URLField(null=True, blank=True, default="")
    bio = models.TextField(null=True, blank=True, default="")
    phone = models.CharField(max_length=10, null=True, blank=True, default="")
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(null=True, auto_now=True)
    account = models.OneToOneField(User, on_delete=models.CASCADE)
    profile_picture = models.ImageField(upload_to="profile_pictures/", null=True, blank=True)
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES, null=True, blank=True)
```

主要欄位說明：
- `name`: 用戶名稱
- `birthday`: 生日
- `location`: 所在地
- `website`: 個人網站
- `bio`: 個人簡介
- `phone`: 手機號碼
- `profile_picture`: 個人頭像
- `gender`: 性別（男/女/其他）

## 表單驗證

### ProfileForm
```python
class ProfileForm(ModelForm):
    class Meta:
        model = Profile
        fields = [
            "name", "phone", "birthday", "location",
            "website", "bio", "profile_picture", "gender",
        ]

    def clean_phone(self):
        phone = self.cleaned_data.get("phone")
        if phone:
            # 台灣手機號碼格式檢查
            if not re.match(r'^09\d{8}$', phone):
                raise forms.ValidationError("手機號碼必須是有效的台灣格式，如：09xxxxxxxx。")
        return phone
    
    def clean_birthday(self):
        birthday = self.cleaned_data.get("birthday")
        if birthday and birthday > now().date():
            raise forms.ValidationError("生日不能是未來的日期。")
        return birthday
```

## 自動化處理

### 用戶檔案自動創建
```python
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(
            name=instance.username,
            account=instance,
            location="",
            bio="",
            birthday=None,
            website=""
        )
```

## 視圖功能

### 1. 新建檔案
```python
@login_required
def new(request):
    account = request.user
    profile = get_object_or_404(Profile, account=account)
    form = ProfileForm(request.FILES, instance=profile)
    today = timezone.now().date()
    return render(request, "profiles/new.html", {
        "profile": profile,
        "form": form,
        "account": account,
        "today": today
    })
```

### 2. 編輯檔案
```python
@login_required
def edit(request):
    account = request.user
    profile = get_object_or_404(Profile, account=account)
    if profile.birthday:
        format_time = profile.birthday.strftime("%Y-%m-%d")
    else:
        format_time = ""
```

## 特色功能

1. **自動檔案創建**
   - 用戶註冊時自動創建檔案
   - 基本資料預設值設定
   - 一對一關聯確保

2. **資料驗證**
   - 手機號碼格式驗證（台灣格式）
   - 生日日期驗證
   - 必要欄位檢查

3. **檔案管理**
   - 個人頭像上傳
   - 個人資料編輯
   - 時間戳記追蹤

4. **權限控制**
   - 登入驗證
   - 個人資料保護

## 使用注意事項

1. **資料格式**
   - 手機號碼必須符合台灣格式
   - 生日不能是未來日期
   - 圖片上傳限制

2. **必要欄位**
   - 用戶名稱為必填
   - 其他欄位可選
   - 性別提供三種選項

3. **資料關聯**
   - 與 Django User 模型一對一關聯
   - 刪除用戶時級聯刪除檔案 