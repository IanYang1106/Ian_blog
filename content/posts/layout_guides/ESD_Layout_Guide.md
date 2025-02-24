---
title: "IC 設計的 ESD Layout"
date: 2025-02-24T16:10:00+08:00
draft: false
categories: ["Layout"]
tags: ["學習筆記", "IC Layout"]
---

# IC 設計的 ESD Layout 

## ESD 量測組合

### ESD 測試模式
IC 產品在不同使用環境中可能遭遇不同類型的 ESD 事件，因此需要進行多種測試模式的驗證。以下是主要的 ESD 測試模式：

#### 1. Human Body Model (HBM)
![HBM 測試模式](/posts/layout_guides/ESD_Layout_Guide/G3.1-1.jpe)
*圖 1: Human Body Model (HBM) 測試模式*

HBM 模擬人體觸摸 IC 時產生的放電現象：
- 測試電壓範圍：通常為 2kV ~ 4kV
- 放電時間：約 150ns
- 主要用途：模擬人體操作時的靜電放電
- 等效電路：包含人體電容(100pF)和人體電阻(1500Ω)

#### 2. Machine Model (MM)
![MM 測試模式](/posts/layout_guides/ESD_Layout_Guide/G3.1-2.jpe)
*圖 2: Machine Model (MM) 測試模式*

MM 模擬機器設備對 IC 造成的放電：
- 測試電壓：通常為 100V ~ 200V
- 放電特性：較 HBM 電流大但時間短
- 主要用途：模擬自動化設備的金屬接觸放電
- 等效電路：包含較大電容(200pF)和較小電阻(0Ω)

#### 3. Charged Device Model (CDM)
![CDM 測試模式](/posts/layout_guides/ESD_Layout_Guide/G3.1-3.jpe)
*圖 3: Charged Device Model (CDM) 測試模式*

CDM 模擬 IC 本身帶電後的快速放電：
- 測試電壓：500V ~ 1kV
- 放電特性：極短時間（<1ns）的高電流脈衝
- 主要用途：模擬元件在製程和運送過程中的自身放電
- 特點：最快速的放電模式，對氧化層破壞風險最高

#### 4. System Level ESD
![系統層級 ESD 測試](/posts/layout_guides/ESD_Layout_Guide/G3.1-4.jpe)
*圖 4: System-Level ESD 測試配置*

系統層級 ESD 測試模擬終端產品使用環境：
- 測試標準：IEC 61000-4-2
- 測試電壓：可高達 8kV（接觸放電）或 15kV（空氣放電）
- 主要用途：驗證產品在實際應用環境中的 ESD 防護能力
- 測試點：產品外部可接觸的金屬部分

#### 5. 整合性保護策略
![完整 ESD 保護策略](/posts/layout_guides/ESD_Layout_Guide/G3.1-5-new.jpe)
*圖 5: 完整的 ESD 保護策略與測試方法*

完整的 ESD 保護必須考慮：
- 多層次保護：包含元件層級到系統層級
- 不同測試模式的要求
- 保護電路的正確配置
- 佈局設計的最佳化
- 製程與應用環境的限制

## ESD Layout 設計重點

### 1. Guard Ring 的設計原則

#### 基本概念
- Guard Ring 是用來防止 Latch-up 的重要結構
- 包含 N+ Guard Ring 和 P+ Guard Ring
- 需要確保完整包圍需保護的元件

#### 設計要點
1. **尺寸規範**：
   - 最小寬度：通常 > 0.6μm（依製程規格而定）
   - 接觸數量：每隔 1~2μm 放置一個接點
   - 環形完整性：避免斷裂或間隔

2. **放置位置**：
   - NMOS：使用 P+ Guard Ring
   - PMOS：使用 N+ Guard Ring
   - 類比電路：雙層 Guard Ring 保護

3. **接地方式**：
   - P+ Guard Ring 接 VSS
   - N+ Guard Ring 接 VDD
   - 確保低阻抗連接

### 2. ESD 保護元件的 Layout

#### 1. 二極體保護
- **尺寸考量**：
  - 面積要足夠大以承受 ESD 電流
  - 通常是 PAD 面積的 1/3 到 1/2
  
- **Layout 重點**：
  - 採用多指結構（Multi-finger）
  - 確保電流路徑均勻
  - 金屬層要足夠寬

#### 2. GGNMOS 設計
- **結構特點**：
  - Gate 和 Source 接地
  - Drain 接到需保護的 PAD
  - 通常使用多指結構

- **Layout 考量**：
  - 每個 finger 寬度控制
  - Silicide Block 的正確放置
  - 接觸和金屬走線的最佳化

### 3. PAD 設計原則

#### 1. PAD 結構
- **基本配置**：
  - 金屬層疊最佳化
  - 避免應力集中
  - 確保足夠的接觸數量

#### 2. 保護元件擺放
- **位置考量**：
  - 靠近 PAD 放置
  - 保持對稱性
  - 縮短保護路徑

### 4. 走線設計重點

#### 1. 電源線規劃
- **寬度規範**：
  - 主電源線 > 10μm
  - 分支電源線依電流大小決定
  - 轉彎處避免直角

#### 2. 訊號線注意事項
- **保護策略**：
  - 避免平行長距離走線
  - 關鍵訊號要加保護
  - 考慮雜訊耦合效應

### 5. 特殊考量

#### 1. 高速電路
- 降低寄生電容影響
- 考慮傳輸線效應
- 阻抗匹配設計

#### 2. 類比電路
- 隔離數位雜訊
- 獨立的保護策略
- 考慮對稱性設計

#### 3. 混合訊號電路
- 數位類比分區
- 獨立的電源系統
- 適當的隔離設計

## Layout 驗證重點

### 1. 基本檢查項目
- Guard Ring 完整性
- 保護元件尺寸
- 金屬層連接

### 2. 特殊規則檢查
- 天線效應規則
- ESD 相關 DRC
- 客製化規則

## 常見問題與解決方案

### 1. 面積限制
- 最佳化元件擺放
- 共用保護元件
- 權衡保護等級

### 2. 效能影響
- 最小化寄生效應
- 優化保護電路位置
- 平衡保護與效能

### 3. 成本考量
- 選擇適當保護策略
- 最佳化元件數量
- 考慮製程限制

## 設計建議

1. **規劃階段**：
   - 預留足夠保護空間
   - 考慮測試需求
   - 參考製程設計規則

2. **實作階段**：
   - 遵循設計指南
   - 定期執行驗證
   - 保持彈性調整

3. **驗證階段**：
   - 完整的 DRC/LVS 檢查
   - ESD 相關規則驗證
   - 必要時進行模擬

## 總結

良好的 ESD Layout 設計需要：
1. 完整的保護策略
2. 謹慎的版圖規劃
3. 嚴格的規則遵循
4. 全面的驗證測試

只有通過完整的設計流程，才能確保 IC 產品具有足夠的 ESD 防護能力。 

---

## 圖片來源
本文件中的 ESD 測試模式相關圖片均來自交通大學 Advanced Logic and Memory Design Laboratory：
[https://alab.iee.nycu.edu.tw/~mdker/ESD](https://alab.iee.nycu.edu.tw/~mdker/ESD) 