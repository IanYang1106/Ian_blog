---
title: "Projects 功能解說"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# Projects 功能解說

## 核心模型結構

### 1. Project 模型
```python
class Project(models.Model):
    STATUS_CHOICES = [
        ("pending", "待上架"),
        ("live", "已上架"),
        ("ended", "已下架"),
    ]
    title = models.CharField(max_length=100)
    subtitle = models.CharField(max_length=100, null=True)
    cover_image = models.ImageField(upload_to="cover_img/")
    raised_amount = models.DecimalField(default=0, decimal_places=0, max_digits=10)
    goal_amount = models.DecimalField(decimal_places=0, max_digits=10)
    start_at = models.DateTimeField(null=True)
    end_at = models.DateTimeField()
    story = models.TextField()
    categories = models.ManyToManyField(Category, through="ProjectCategory")
    location = models.CharField(null=True, max_length=100)
    latitude = models.FloatField(null=True)
    longitude = models.FloatField(null=True)
```

主要功能：
1. **專案狀態管理**
   - 待上架、已上架、已下架三種狀態
   - 自動根據時間更新狀態

2. **金額追蹤**
   - 目標金額設定
   - 已籌金額自動計算
   - 支援自由贊助和回饋方案

3. **地理位置功能**
   - 位置資訊儲存
   - 經緯度座標支援

### 2. 相關模型

#### Sponsor 模型（贊助記錄）
```python
class Sponsor(models.Model):
    account = models.ForeignKey(User, on_delete=models.CASCADE)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    reward = models.ForeignKey(Reward, null=True)
    amount = models.DecimalField(max_digits=10, decimal_places=0)
    status = models.CharField(choices=[
        ("pending", "Pending"),
        ("paid", "Paid"),
        ("failed", "Failed"),
    ])
```

#### CollectProject 模型（收藏功能）
```python
class CollectProject(models.Model):
    account = models.ForeignKey(User, on_delete=models.CASCADE)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
```

## 核心功能實現

### 1. 專案管理功能

#### 自動狀態更新
```python
def update_status(self):
    current_time = now()
    if self.status == "pending" and self.start_at and current_time >= self.start_at:
        self.status = "live"
    elif self.status == "live" and self.end_at and current_time >= self.end_at:
        self.status = "ended"
```

#### 募資金額計算
```python
def update_raised_amount(self):
    # 計算回饋方案訂單
    reward_orders_amount = Order.objects.filter(
        reward__project=self, 
        paid=True
    ).aggregate(total=Sum("amount"))["total"] or 0
    
    # 計算自由贊助訂單
    free_orders_amount = Order.objects.filter(
        description__startswith=f"{self.title} - 自由贊助",
        paid=True
    ).aggregate(total=Sum("amount"))["total"] or 0
    
    self.raised_amount = reward_orders_amount + free_orders_amount
```

### 2. 搜尋與過濾功能

```python
class ProjectSearchForm(Form):
    query = CharField(max_length=255)
    status = ChoiceField(choices=Project.STATUS_CHOICES)
    location = CharField(max_length=100)
    categories = ModelMultipleChoiceField(queryset=Category.objects.all())
```

### 3. 數據分析功能

#### 贊助者統計
```python
def get_backers_count(self):
    return Sponsor.objects.filter(
        project=self, 
        status="paid"
    ).values("account").distinct().count()
```

#### 互動數據追蹤
```python
def get_collect_count(self):
    return self.collect_account.count()

def get_like_count(self):
    return self.favorite_users.count()
```

## URL 路由配置

### 1. 基礎功能路由
```python
urlpatterns = [
    path("", index, name="index"),
    path("new/", new, name="new"),
    path("<slug:slug>", show, name="show"),
    path("<slug:slug>/edit", edit, name="edit"),
    path("<slug:slug>/delete", delete, name="delete"),
]
```

### 2. 互動功能路由
```python
urlpatterns += [
    path("<slug:slug>/collect", collect_projects, name="collect"),
    path("<slug:slug>/like", like_projects, name="like"),
    path("<slug:slug>/comment", comment, name="comment"),
]
```

### 3. 數據分析路由
```python
urlpatterns += [
    path("<slug:slug>/chart_page/", chart_page, name="chart_page"),
    path("<slug:slug>/gender_proportion/", gender_proportion),
    path("<slug:slug>/daily_sponsorship_amount/", daily_sponsorship_amount),
    path("<slug:slug>/gender_amount_scatter/", gender_amount_scatter),
]
```

## 整合功能

### 1. 與其他應用的整合

1. **回饋系統整合**
   - 與 Rewards 應用連接
   - 支援回饋方案管理
   - 處理贊助訂單

2. **支付系統整合**
   - 與 Payments 應用連接
   - 處理交易狀態更新
   - 自動更新募資金額

3. **評論系統整合**
   - 與 Comments 應用連接
   - 專案討論功能
   - 互動記錄追蹤

4. **FAQ 系統整合**
   - 與 FAQs 應用連接
   - 專案常見問題管理
   - 支援動態更新

### 2. 數據匯出功能

```python
urlpatterns += [
    path("<slug:slug>/gender_amount_scatter_excel/", gender_amount_scatter_excel),
    path("<slug:slug>/gender_proportion_excel/", gender_proportion_excel),
    path("<slug:slug>/reward_grouped_bar_chart_excel/", reward_grouped_bar_chart_excel),
    path("<slug:slug>/daily_sponsorship_amount_excel/", daily_sponsorship_amount_excel),
]
```

## 特色功能

1. **多維度專案管理**
   - 狀態自動更新
   - 募資進度追蹤
   - 地理位置支援

2. **完整的互動系統**
   - 收藏功能
   - 按讚功能
   - 評論系統
   - FAQ 管理

3. **豐富的數據分析**
   - 性別比例分析
   - 贊助金額分析
   - 時間趨勢分析
   - 回饋方案分析

4. **資料匯出功能**
   - Excel 格式匯出
   - 多種數據圖表
   - 客製化報表 