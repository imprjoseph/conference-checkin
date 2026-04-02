[README.md](https://github.com/user-attachments/files/26437275/README.md)
# imPR 會展報到系統 — 部署說明

## 系統架構

```
GitHub Pages (靜態前端)
    ↕ HTTP/JSON
Google Apps Script (後端 API)
    ↕ Sheets API
Google Sheet (資料庫)
```

---

## STEP 1 — 建立 Google Sheet

1. 前往 [Google Sheets](https://sheets.google.com) → 建立新試算表
2. 將試算表命名為「imPR 會展報到系統 2025」
3. 複製網址列中的 **Sheet ID**（格式如：`1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms`）

---

## STEP 2 — 部署 Google Apps Script

1. 在 Google Sheet 中點選選單：**擴充功能 → Apps Script**
2. 刪除預設的 `myFunction()` 程式碼
3. 將 `Code.gs` 的內容全部貼入
4. **修改第 4-5 行：**
   ```javascript
   const SHEET_ID = "貼上您的 Sheet ID";
   const SECRET   = "設定一個複雜的秘密金鑰，例如：impr2025!@#secret";
   ```
5. 點選選單列 **執行 → 執行函式 → `initSheets`**（首次執行需授權）
6. 點選左側 **部署 → 新增部署作業**
   - 類型選：**網頁應用程式**
   - 執行身分：**我（您的 Google 帳號）**
   - 存取權限：**任何人**
   - 點選「部署」
7. 複製顯示的 **網頁應用程式 URL**（格式如 `https://script.google.com/macros/s/AKfy.../exec`）

---

## STEP 3 — 設定前端

編輯 `public/index.html` 的第 11-14 行：

```javascript
window.APP_CONFIG = {
  GAS_URL: "貼上 Step 2 的網頁應用程式 URL",
  SECRET:  "與 Code.gs 相同的秘密金鑰",
  VERSION: "4.0.0"
};
```

---

## STEP 4 — 部署到 GitHub Pages

### 4a. 建立 GitHub Repository

1. 前往 [github.com](https://github.com) → **New repository**
2. Repository name: `conference-checkin`（或任意名稱）
3. 設定為 **Public**
4. 點選 **Create repository**

### 4b. 上傳檔案

**方法 A — GitHub 網頁介面（最簡單）：**
1. 在 repository 頁面點選 **Add file → Upload files**
2. 上傳 `public/index.html` 和 `public/app.jsx`
3. Commit message: `Initial deployment`

**方法 B — Git 指令：**
```bash
git clone https://github.com/您的帳號/conference-checkin.git
cd conference-checkin
# 複製 public/ 資料夾中的檔案到此目錄
cp public/* .
git add .
git commit -m "Initial deployment"
git push
```

### 4c. 啟用 GitHub Pages

1. 在 repository 點選 **Settings**
2. 左側選單點 **Pages**
3. Source 選 **Deploy from a branch**
4. Branch 選 **main**，資料夾選 **/ (root)**
5. 點選 **Save**
6. 等待 1-3 分鐘，網址將顯示為：
   `https://您的帳號.github.io/conference-checkin/`

---

## STEP 5 — 新增與會者帳號

在 Google Sheet 的 `users` 工作表中，每位與會者需要一列：

| id | role | name | nameEn | email | password | org |
|----|------|------|--------|-------|----------|-----|
| u10 | attendee | 陳大明 | Chen Da-Ming | chen@example.com | 自設密碼 | 公司名稱 |

在 `attendees` 工作表中對應新增一列（type 填 `vip` 或 `regular`）：

| id | userId | type | checkInNo | name | nameEn | email | ... |
|----|--------|------|-----------|------|--------|-------|-----|
| a10 | u10 | vip | VIP-1007 | 陳大明 | Chen Da-Ming | chen@example.com | ... |

---

## 測試帳號（由 initSheets 自動建立）

| 身份 | Email | 密碼 |
|------|-------|------|
| 🔑 管理員 | admin@impr.com.tw | admin123 |
| 📋 工作同仁 | staff1@impr.com.tw | staff1 |
| 👑 貴賓 VIP | chen@ex.com | vip1001 |
| 🎫 一般與會者 | lin@ex.com | reg1002 |

---

## Google Sheet 欄位說明

### users 工作表
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼（如 u1, u2...）|
| role | admin / staff / attendee |
| name | 中文姓名 |
| nameEn | 英文姓名 |
| email | 登入信箱 |
| password | 登入密碼 |
| org | 單位名稱 |

### attendees 工作表
| 欄位 | 說明 |
|------|------|
| id | 唯一識別碼（如 a1, a2...）|
| userId | 對應 users.id（與會者登入後識別用）|
| type | vip / regular |
| checkInNo | 報到序號（VIP-1001 或 REG-1002）|
| name | 中文姓名 |
| nameEn | 英文姓名 |
| email | 信箱 |
| org | 單位 |
| title | 職稱 |
| phone | 電話 |
| diet | normal / veggie / halal |
| session | full / morning / afternoon |
| needManual | TRUE / FALSE |
| day1 | JSON：`{"ci":false,"ciT":null,"co":false,"coT":null}` |
| day2 | 同上 |
| acts | JSON：`{"booth_d1":false,"lunch_d1":false,...}` |
| manualCollected | TRUE / FALSE |
| reminderSent | TRUE / FALSE |

---

## 常見問題

**Q: 登入後資料沒有載入？**
A: 檢查 `index.html` 中的 `GAS_URL` 是否正確，GAS 部署是否設定「任何人」可存取。

**Q: 掃描 QR Code 後找不到與會者？**
A: QR Code 格式為 `CONF2025-{序號}-{信箱}`，掃描槍讀取後請確認輸入欄位接收到正確內容。

**Q: Apps Script 執行超時？**
A: Google Apps Script 免費版每次執行限制 6 分鐘，批次匯入時每次建議不超過 100 筆。

**Q: 如何讓工作人員從手機連入？**
A: 直接用手機瀏覽器開啟 GitHub Pages 網址即可，已針對手機介面最佳化。

---

## 檔案結構

```
conference-checkin/
├── index.html    ← 主頁面，包含設定和載入邏輯
├── app.jsx       ← 完整 React 應用（含 imPR Logo）
└── README.md     ← 本說明文件
```

Google Apps Script（獨立部署）：
```
Code.gs           ← 後端 API（部署為 Web App）
```
