# LINE 收單系統

LINE 內下單 + 綠界金流 + 訂單後台，自架版的 CatchOpz。

## 🏗 系統架構

```
顧客在 LINE 聊天 → 點選單按鈕 → 開啟 LIFF 下單頁
  ↓
選商品、填地址 → 跳轉綠界付款
  ↓
付款成功 → 綠界 webhook 通知後端 → 推 LINE 訊息給顧客
  ↓
你在後台看訂單、標記出貨 → 自動推送出貨通知
```

## 📁 檔案結構

```
line-order-system/
├── package.json          # 套件清單
├── .env.example          # 環境變數範本
├── .gitignore
├── src/
│   ├── server.js         # 主伺服器
│   ├── db.js             # SQLite 資料庫
│   └── ecpay.js          # 綠界金流處理
└── public/
    ├── index.html        # LIFF 下單頁
    ├── result.html       # 付款結果頁
    └── admin.html        # 訂單後台
```

---

## 🚀 部署 SOP（共 6 步驟）

### Step 1：把專案推上 GitHub

在你電腦上開終端機，cd 到專案資料夾，跑：

```bash
git init
git add .
git commit -m "init: LINE order system"
git branch -M main
git remote add origin https://github.com/809540023-lgtm/line-order-system.git
git push -u origin main
```

> 推送前，先去 GitHub 開一個叫 `line-order-system` 的空 repo。

---

### Step 2：申請 LINE Developers（Agent 可協助）

1. 前往 https://developers.line.biz/console/
2. 用 LINE 登入 → 建立 Provider（隨便取名，例：「我的小店」）
3. 在 Provider 內建立 **Messaging API Channel**（這是 OA）
   - 記下：**Channel Secret**、**Channel Access Token**（要按「Issue」產生）
4. 再建立 **LIFF App**：
   - Size: `Full`
   - Endpoint URL: `https://你的render網址.onrender.com/`（先用佔位的，等 Render 部署完再改）
   - Scope: 勾 `profile`、`openid`
   - 記下：**LIFF ID**

---

### Step 3：申請綠界（Agent 可協助前段，身分證上傳要你自己）

> **測試階段可以先跳過這步**，直接用 `.env.example` 裡的測試帳號（綠界官方提供）

正式上線時：
1. 前往 https://www.ecpay.com.tw/
2. 申請特約商店 → 上傳身分證、存摺
3. 審核通過後拿到：**MerchantID**、**HashKey**、**HashIV**

---

### Step 4：部署到 Render

1. 登入 https://dashboard.render.com/（你的帳號 `809540023-lgtm`）
2. **New** → **Web Service** → 連結 `line-order-system` repo
3. 填寫：
   - **Name**: `line-order-system`（之後網址會是 `line-order-system.onrender.com`）
   - **Region**: Singapore（離台灣近）
   - **Branch**: `main`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Instance Type**: Free（先用免費的試）
4. **Environment Variables** 加入：
   ```
   LINE_CHANNEL_ACCESS_TOKEN = (Step 2 取得)
   LINE_CHANNEL_SECRET       = (Step 2 取得)
   LIFF_ID                   = (Step 2 取得)
   ECPAY_MERCHANT_ID         = 2000132
   ECPAY_HASH_KEY            = 5294y06JbISpM5x9
   ECPAY_HASH_IV             = v77hoKGq4kWxNNIS
   ECPAY_API_URL             = https://payment-stage.ecpay.com.tw/Cashier/AioCheckOut/V5
   BASE_URL                  = https://line-order-system.onrender.com
   ```
5. **Create Web Service**，等 3~5 分鐘部署完成

---

### Step 5：回 LINE Console 設定 Webhook 和 LIFF URL

部署完成後，拿到 Render 網址（例 `https://line-order-system.onrender.com`），回去更新：

1. **LIFF App** 的 Endpoint URL → 改成 `https://line-order-system.onrender.com/`
2. **Messaging API 設定**：
   - Webhook URL → `https://line-order-system.onrender.com/webhook`
   - Use webhook → **開啟**
   - Auto-reply messages → **關閉**（不然會跟你的 bot 衝突）
   - Greeting messages → 看你要不要

---

### Step 6：測試流程

1. 用手機加你自己的 LINE OA 為好友
2. 傳「下單」給它 → 應該收到帶按鈕的訊息
3. 點按鈕 → 跳出 LIFF 下單頁
4. 選商品、填收件資料 → 跳綠界
5. 用綠界測試卡號付款：
   ```
   卡號: 4311-9522-2222-2222
   有效期限: 任何未來日期
   CVV: 222
   ```
6. 付完款應該收到 LINE 通知，並且後台 `/admin.html` 看得到訂單

---

## 🛠 本機開發（選用）

```bash
npm install
cp .env.example .env
# 編輯 .env 填入你的設定
npm start
```

> 本機要測 LIFF 比較麻煩，建議直接在 Render 部署測。

---

## 📋 重要網址

部署完成後你會有：

| 用途 | 網址 |
|---|---|
| LIFF 下單頁 | `https://你的網址.onrender.com/` |
| 付款結果頁 | `https://你的網址.onrender.com/payment-result` |
| 訂單後台 | `https://你的網址.onrender.com/admin.html` |
| LINE Webhook | `https://你的網址.onrender.com/webhook` |
| 綠界 Webhook | `https://你的網址.onrender.com/api/ecpay/notify` |

---

## ⚠️ 上線前檢查清單

- [ ] 換成正式綠界帳號（不是測試的 2000132）
- [ ] `ECPAY_API_URL` 改為正式環境（去掉 `-stage`）
- [ ] 把範例商品刪掉、放真實商品（直接改 SQLite，或寫個簡單後台）
- [ ] Render 免費方案閒置會睡著，正式營運建議升 Starter（$7/月）
- [ ] 訂單後台 `/admin.html` 加上密碼保護（目前沒鎖，誰知道網址都能進）

---

## 💡 進階功能（之後可加）

- 後台商品管理（新增/編輯/刪除商品）
- 庫存自動扣減
- 折扣碼
- 紅利點數
- 7-11 / 全家超商取貨
- 多店家支援（改成 SaaS）

有需要就告訴 Claude。
