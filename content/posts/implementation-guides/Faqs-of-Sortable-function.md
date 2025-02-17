---
title: "Django 使用 Sortable.js 實現FAQ's拖曳功能"
date: 2025-01-03
draft: false
categories: ["程式開發"]
tags: ["學習筆記"]
---

# FAQ 拖曳排序功能

## 功能說明

1. 程式碼會在頁面加載完成後初始化
2. 使用 Sortable.js 實現拖曳排序功能
3. 當用戶完成拖曳排序後，會自動向後端發送更新請求
4. 包含完整的錯誤處理和 CSRF 保護
5. 適用於 Django 後端框架的前端實現

## 使用須知

- 需要引入 Sortable.js (由打包器處理)
- 需要在 HTML 中有 id 為 "faq-list" 的元素
- 需要設置正確的後端 API 路徑
- 需要確保 Django CSRF 保護正確配置

## 程式碼解析

```javascript
import Sortable from "sortablejs";

// 等待 DOM 完全載入後才執行代碼
document.addEventListener("DOMContentLoaded", function () {
  // 通過 ID 獲取 FAQ 列表元素
  var element = document.getElementById("faq-list");

  // 確認 FAQ 列表元素存在
  if (element) {
    // 初始化 Sortable，使列表可拖曳排序
    var sortable = Sortable.create(element, {
      // 設定拖曳動畫時間為 150 毫秒
      animation: 150,

      // 定義拖曳結束後的處理函數
      onEnd: function () {
        // 獲取排序後的項目順序
        var order = sortable.toArray();

        // 向後端發送 POST 請求，更新排序順序
        fetch("/faqs/updated-faq-position/", {
          method: "POST", // 使用 POST 方法
          headers: {
            "Content-Type": "application/json", // 設定內容類型為 JSON
            "X-CSRFToken": getCookie("csrftoken"), // 添加 CSRF Token
          },
          // 將排序數據轉換為 JSON 字符串
          body: JSON.stringify({ position: order }),
        })
          // 將伺服器回應轉換為 JSON
          .then((response) => response.json())
          // 處理成功情況
          .then((data) => {
            console.log("Success:", data);
          })
          // 處理錯誤情況
          .catch((error) => {
            console.error("Error:", error);
          });
      },
    });
  }
});

/**
 * 獲取指定名稱的 Cookie 值
 * @param {string} name - Cookie 名稱
 * @returns {string|null} Cookie 值或 null
 */
function getCookie(name) {
  let cookieValue = null;

  // 檢查 document.cookie 是否存在且不為空
  if (document.cookie && document.cookie !== "") {
    // 將 cookie 字符串分割成陣列
    const cookies = document.cookie.split(";");

    // 遍歷所有 cookies 尋找目標 cookie
    for (let i = 0; i < cookies.length; i++) {
      const cookie = cookies[i].trim();
      // 檢查當前 cookie 是否為目標 cookie
      if (cookie.substring(0, name.length + 1) === name + "=") {
        // 解碼並返回 cookie 值
        cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
        break;
      }
    }
  }
  return cookieValue;
}
```

HTML 需要有 faq-list

```html
<div class="container mx-auto py-8 px-4">
  <div class="grid grid-cols-1 gap-6" id="faq-list">
    {% for faq in faqs %}
    <a
      href="{% url 'faqs:show' faq.slug %}"
      class="faq-item block bg-white border border-gray-200 shadow-sm rounded-lg p-6 hover:shadow-md transition-shadow"
      data-id="{{ faq.id }}"
    >
      <div class="text-lg font-semibold mb-2 text-gray-800">
        {{ faq.question|linebreaks }}
      </div>
      <span class="text-sm text-gray-500"
        >建立時間: {{ faq.created_at|date:"Y-m-d H:i:s" }}</span
      >
    </a>
    {% endfor %}
  </div>
</div>
```
