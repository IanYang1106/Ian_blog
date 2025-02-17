---
title: "Rewards 功能解說"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# Rewards 功能解說

## 核心模型結構

### 1. Reward 模型（回饋方案）
```python
class Reward(models.Model):
    title = models.CharField(max_length=300)
    description = models.TextField(blank=True)
    price = models.DecimalField(default=0, max_digits=10, decimal_places=0)
    estimated_delivery = models.DateField(default=default_delivery_date)
    quantity = models.PositiveIntegerField(default=1)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(null=True)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
```

主要欄位說明：
- `title`: 回饋方案標題
- `description`: 回饋方案描述
- `price`: 回饋方案價格
- `estimated_delivery`: 預計出貨日期（預設為建立日期 +7 天）
- `quantity`: 回饋數量
- `project`: 關聯的專案

### 2. RewardProduct 模型（回饋商品）
```python
class RewardProduct(models.Model):
    name = models.CharField(max_length=300)
    rewards = models.ManyToManyField(Reward, blank=True)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
```

主要欄位說明：
- `name`: 商品名稱
- `rewards`: 關聯的回饋方案（多對多關係）
- `project`: 關聯的專案

### 3. OptionalAdd 模型（加購選項）
```python
class OptionalAdd(models.Model):
    name = models.CharField(max_length=300)
    price = models.DecimalField(default=0, max_digits=10, decimal_places=0)
    rewards = models.ManyToManyField(Reward, blank=True)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
```

主要欄位說明：
- `name`: 加購選項名稱
- `price`: 加購選項價格
- `rewards`: 關聯的回饋方案（多對多關係）
- `project`: 關聯的專案

## 核心功能實現

### 1. 回饋方案管理

#### 新增回饋方案
```python
@login_required
def index(request, slug):
    if request.method == "POST" and "reward" in request.POST:
        reward = reward_form.save(commit=False)
        reward.project = project
        reward.save()
        
        # 關聯商品
        product_ids = request.POST.getlist("products")
        products = RewardProduct.objects.filter(id__in=product_ids)
        for product in products:
            product.rewards.add(reward)
            
        # 關聯加購選項
        option_ids = request.POST.getlist("options")
        options = OptionalAdd.objects.filter(id__in=option_ids)
        for option in options:
            option.rewards.add(reward)
```

#### 商品與選項管理
```python
@login_required
def reward_items(request, slug):
    # 批量創建商品
    if "save_products" in request.POST:
        products = [name for key, name in request.POST.items() 
                   if key.startswith("products[") and name.strip()]
        RewardProduct.objects.bulk_create([
            RewardProduct(name=name, project_id=project.id) 
            for name in products
        ])
        
    # 批量創建加購選項
    if "save_options" in request.POST:
        options = []
        for key, name in request.POST.items():
            if key.startswith("options[") and key.endswith("][name]"):
                index = key.split("[")[1].split("]")[0]
                price = request.POST.get(f"options[{index}][price]")
                options.append({"name": name, "price": price})
```

### 2. 贊助功能實現

#### 自由贊助
```python
@login_required
def free_sponsor(request, slug):
    if request.POST:
        amount = request.POST.get("amount")
        Sponsor.objects.create(
            account=request.user,
            project_id=project.id,
            reward=None,
            amount=amount,
            status="pending"
        )
```

#### 回饋方案贊助
```python
@login_required
def reward_sponsor(request, slug):
    reward = get_object_or_404(Reward, id=reward_id)
    # 建立贊助記錄
    sponsor = Sponsor.objects.create(
        account=request.user,
        project=project,
        reward=reward,
        amount=reward.price,
        status="pending"
    )
```

### 3. 加購選項處理
```python
@login_required
def reward_sponsor_options(request, slug):
    # 處理加購選項
    selected_options = request.POST.getlist("selected_options")
    total_amount = reward.price
    
    for option_id in selected_options:
        option = OptionalAdd.objects.get(id=option_id)
        total_amount += option.price
```

## 整合功能

### 1. 與支付系統整合
```python
def reward_sponsor_confirm(request, slug):
    # 建立訂單
    order = Order.objects.create(
        order_id=f"R{timezone.now().strftime('%Y%m%d%H%M%S')}",
        amount=total_amount,
        description=f"{project.title} - {reward.title}",
        reward=reward,
        user=request.user
    )
    
    # 建立 ECPay 支付
    ecpay = ECPayPayment()
    order_params = ecpay.create_order(
        order_id=order.order_id,
        amount=order.amount,
        description=order.description
    )
```

### 2. 與專案系統整合
```python
def update_project_status(sponsor):
    project = sponsor.project
    if sponsor.status == "paid":
        project.update_raised_amount()
        project.save()
```

## 特色功能

1. **預設出貨日期計算**
```python
def default_delivery_date():
    return datetime.date.today() + datetime.timedelta(days=7)
```
- 自動設定預計出貨日期為建立日期後 7 天

2. **多層次回饋系統**
- 基礎回饋方案（Reward）
- 回饋商品組合（RewardProduct）
- 加購選項（OptionalAdd）

3. **關聯結構**
- 回饋方案與專案的一對多關係
- 商品與回饋方案的多對多關係
- 加購選項與回饋方案的多對多關係

## 資料關聯

1. **專案關聯**
- 所有模型都與專案（Project）關聯
- 使用 CASCADE 刪除，確保專案刪除時相關資料一併刪除

2. **回饋方案關聯**
- RewardProduct 和 OptionalAdd 都可以關聯多個回饋方案
- 使用 blank=True 允許空關聯

## 使用注意事項

1. **資料完整性**
- 回饋方案必須關聯到專案
- 商品和加購選項可以不關聯回饋方案

2. **價格設定**
- 回饋方案和加購選項的價格欄位最多支援 10 位數
- 價格不含小數點

## 使用流程

1. **回饋方案設定**
   - 創建基礎回饋
   - 設定商品組合
   - 添加加購選項

2. **贊助流程**
   - 選擇回饋方案
   - 選擇加購選項
   - 確認贊助金額
   - 完成支付程序

3. **管理功能**
   - 回饋方案管理
   - 商品管理
   - 選項管理
   - 贊助記錄查看 