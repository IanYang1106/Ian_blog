---
title: "CH1 React + Vite 啟動方法"
date: 2025-01-28
draft: false
categories: ["React"]
tags: ["學習筆記"]
---

# React + Vite 啟動方法

## 第一步：建立新專案

### 1. 使用指令建立專案
打開終端機（Terminal），輸入以下指令：
```bash
npm create vite@latest my-react-app -- --template react
```
這個指令會建立一個名為 `my-react-app` 的新專案

### 2. 安裝需要的套件
```bash
cd my-react-app    # 進入專案資料夾
npm install        # 安裝所需套件
```

## 專案資料夾結構說明
```
my-react-app/
├── node_modules/      # 存放所有安裝的套件
├── public/           # 存放靜態檔案（如圖片、字型）
├── src/              # 主要的程式碼資料夾
│   ├── assets/      # 專案用到的資源檔案
│   ├── App.css      # App 元件的樣式
│   ├── App.jsx      # 主要的 App 元件
│   ├── index.css    # 全域樣式
│   └── main.jsx     # 程式進入點
├── index.html        # 網頁主檔案
├── package.json      # 專案設定和套件清單
└── vite.config.js    # Vite 設定檔
```

## 常用指令說明

### 1. 啟動開發伺服器
```bash
npm run dev
```
執行後會開啟網頁，網址通常是 http://localhost:5173
- 當你修改程式碼時，網頁會自動更新
- 可以即時看到修改的結果

### 2. 打包專案
```bash
npm run build
```
- 會產生一個 `dist` 資料夾
- 裡面是可以部署到網路上的檔案

### 3. 預覽打包結果
```bash
npm run preview
```
可以在本機預覽打包後的網站

## 常用套件安裝教學

### 1. 路由套件（頁面切換用）
```bash
npm install react-router-dom
```
用來處理網頁之間的切換

### 2. UI 套件（漂亮的介面元件）
```bash
# 安裝 Material-UI（推薦新手使用）
npm install @mui/material @emotion/react @emotion/styled

# 或是安裝 Ant Design
npm install antd
```

### 3. 串接 API 用的套件
```bash
npm install axios
```
用來和後端伺服器溝通



## 建議的資料夾結構
```
src/
├── components/     # 可重複使用的元件
│   ├── Button/    # 按鈕元件
│   └── Header/    # 頁首元件
├── pages/         # 網頁元件
│   ├── Home/      # 首頁
│   └── About/     # 關於頁面
├── assets/        # 圖片等資源
└── styles/        # 樣式檔案
```


## 實用連結
- React 官方文件：https://react.dev/
- Vite 官方文件：https://vitejs.dev/
- Material-UI 元件庫：https://mui.com/
- React Router 教學：https://reactrouter.com/ 