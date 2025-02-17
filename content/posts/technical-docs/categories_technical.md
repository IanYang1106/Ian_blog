---
title: "Categories 功能解說"
date: 2025-01-25
draft: false
categories: ["專題功能解說"]
tags: ["學習筆記"]
---

# Categories 功能解說

## 核心模型結構

### Category Model
```python
class Category(models.Model):
    title = models.CharField(max_length=100, unique=True)
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='children')

    def __str__(self):
        return self.title
```

主要欄位說明：
- `title`: 分類名稱，唯一值
- `parent`: 父分類，自引用外鍵，允許為空（用於頂層分類）
- `children`: 反向關聯名稱，用於獲取子分類

## 視圖實現

### 主視圖 (index)
```python
def index(request):
    categories = Category.objects.filter(parent__isnull=True)
    return render(request, "categories/index.html", {"categories": categories})
```

功能說明：
- 獲取所有頂層分類（沒有父分類的類別）
- 渲染到 index.html 模板

## 種子資料

### 預設分類結構
專案提供了完整的種子資料命令 `seed_categories`，包含 15 個主要分類，每個分類下有 5 個子分類：

1. **藝術類**
   - 概念藝術、數位藝術、裝置藝術、表演藝術、公共藝術

2. **漫畫類**
   - 作品選集、漫畫書、活動、圖畫小說、網路漫畫

3. **工藝類**
   - 玻璃、編織、陶藝、拼布、木工

4. **舞蹈類**
   - 演出、現代舞、街舞、芭雷舞、踢踏舞

5. **設計類**
   - 建築、城市設計、平面設計、產品設計、互動式設計

其他類別包括：時尚、影劇、食品、遊戲、新聞、音樂、攝影、出版、科技、劇院。

### 種子資料使用方式
```python
python manage.py seed_categories
```

種子資料特點：
- 使用 `get_or_create` 避免重複創建
- 支持批量導入
- 維護父子關係
- 保證資料一致性

## URL 配置
```python
app_name = "categories"

urlpatterns = [
    path('', index, name='index'),
]
```

## 特色功能

1. **層級分類結構**
   - 使用自引用外鍵實現無限層級分類
   - 通過 `related_name='children'` 實現父子關係查詢
   - 預設兩層分類結構，可擴展性強

2. **唯一性約束**
   - 分類名稱（title）設置為唯一
   - 避免重複分類
   - 支持中文分類名稱

3. **資料初始化**
   - 提供完整的種子資料
   - 涵蓋常見專案類別
   - 易於擴展和修改

## 使用注意事項

1. **資料完整性**
   - 刪除父分類時會級聯刪除所有子分類
   - 分類名稱不能重複
   - 建議使用種子資料確保基礎分類的完整性

2. **查詢優化**
   - 使用 `parent__isnull=True` 查詢頂層分類
   - 使用 `children` 反向關聯查詢子分類
   - 可以使用 `select_related` 優化父類查詢

3. **模型關係**
   - 支持多層級分類結構
   - 可以靈活建立分類樹狀結構
   - 注意避免創建循環引用

4. **擴展建議**
   - 可以添加排序欄位
   - 可以添加分類描述
   - 可以添加分類圖標
   - 考慮添加分類狀態（啟用/禁用） 