---
title: "FAQs 功能解說"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# FAQs 功能解說

## 核心模型結構

### FAQ Model
```python
class Faq(models.Model):
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    question = models.CharField(max_length=100)
    answer = models.TextField(null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    position = models.PositiveIntegerField(default=0)
    deleted_at = models.DateTimeField(null=True, db_index=True)
    created_by = models.ForeignKey(User, on_delete=models.CASCADE, related_name="faqs")
    slug = AutoSlugField(populate_from=generate_random_slug, unique=True)

    class Meta:
        ordering = ["position"]
```

主要欄位說明：
- `project`: 關聯的專案
- `question`: FAQ 問題
- `answer`: FAQ 答案
- `position`: 排序位置
- `deleted_at`: 軟刪除時間戳
- `created_by`: FAQ 創建者
- `slug`: 唯一標識符

## 表單實現

### FAQ Form
```python
class FaqForm(ModelForm):
    class Meta:
        model = Faq
        fields = ["question", "answer"]
```

## 視圖實現

### 1. FAQ 列表與創建 (index)
```python
def index(request, slug):
    project = get_object_or_404(Project, slug=slug)
    faqs = Faq.objects.filter(project=project, deleted_at__isnull=True)
```
- 支持 HTMX 請求
- 權限驗證
- 軟刪除過濾

### 2. FAQ 管理功能
```python
@login_required
def show(request, slug):
    faq = get_object_or_404(Faq, slug=slug, created_by=request.user)
```
- 查看詳情
- 編輯更新
- 軟刪除

### 3. 排序功能
```python
@csrf_exempt
@require_POST
def updated_faq_position(request):
    data = json.loads(request.body)
    position = data.get("position", [])
    for index, faq_id in enumerate(position):
        Faq.objects.filter(id=faq_id).update(position=index)
```

## URL 配置
```python
urlpatterns = [
    path("<slug:slug>", views.show, name="show"),
    path("<slug:slug>/edit", views.edit, name="edit"),
    path("<slug:slug>/delete", views.delete, name="delete"),
    path("updated-faq-position/", views.updated_faq_position, name="updated_faq_position"),
]
```

## 特色功能

1. **排序管理**
   - 自定義排序位置
   - AJAX 拖曳排序
   - 即時更新順序

2. **軟刪除**
   - 使用時間戳標記刪除
   - 支持資料恢復
   - 保留歷史記錄

3. **HTMX 整合**
   - 無刷新更新
   - 部分頁面刷新
   - 優化用戶體驗

4. **權限控制**
   - 創建者權限驗證
   - 專案擁有者驗證
   - 訪問控制

## 使用注意事項

1. **資料完整性**
   - FAQ 必須關聯專案
   - 問題不能為空
   - 排序位置唯一

2. **查詢優化**
   - 軟刪除索引
   - 排序欄位索引
   - 關聯查詢優化

3. **前端整合**
   - 需要配置 HTMX
   - 需要配置拖曳排序 JS
   - CSRF 設置

4. **擴展建議**
   - 可添加 FAQ 分類
   - 可添加搜索功能
   - 可添加熱門問題標記
   - 可添加問題有用度評分 