# 日序 Daybook

以「今天」為中心的個人行程與任務管理網站。設計思考與行動 APP 程式設計應用：Project A。

- **前端**：HTML / CSS / JavaScript 單頁，部署在 GitHub Pages
- **登入**：Firebase Authentication（Google）
- **資料庫**：Cloud Firestore（每位使用者只能讀寫自己的資料）
- **AI**：Gemini，透過 Firebase AI Logic 呼叫，API Key 由 Firebase 代管，不會出現在前端

## 功能

| 類別 | 功能 |
|---|---|
| 帳號 | Google 登入／登出、訪客模式（資料存在本機）、第一次登入時自動把訪客資料搬到雲端 |
| 任務 | 新增、查看、編輯、刪除（可復原）、標記完成（可復原）、截止日、優先度（高／中／低）、分類 |
| 行程 | 有時段或全天行程，並檢查結束時間是否晚於開始時間 |
| 檢視 | 今日時間軸（含「現在」指示線）、七日日期列、側邊小月曆、整月行事曆（雙擊日期可新增） |
| 查詢與統計 | 全文搜尋（標題、備註、分類）；依狀態篩選（未完成／今天前到期／7 天內／已逾期／全部）與分類篩選；依截止日、優先度、建立時間、標題排序；完成率與逾期數 |
| 提醒 | 鈴鐺列出逾期、今天到期與兩小時內開始的行程 |
| AI 小幫手 | 用一句話新增（例如「明天下午三點跟教授開會」，AI 解析後先填好表單讓使用者確認）、規劃今天、建議優先順序、回顧本週 |
| 同步 | Firestore 即時同步與離線快取，並顯示同步狀態（已同步／同步中／離線／失敗） |
| 資料 | 匯出／匯入 JSON、一鍵清除已完成任務 |
| 易用性 | 桌面與手機自適應版面（390px 測試）；鍵盤快捷鍵 N 新增、Ctrl/⌘K 搜尋、A 開啟 AI、T 回到今天、Esc 關閉 |

## 系統架構

```
瀏覽器 ── GitHub Pages（index.html）
   │
   ├─ Firebase Authentication（Google 登入，取得 UID）
   ├─ Cloud Firestore   users/{uid}/items/{itemId}
   └─ Firebase AI Logic → Gemini（gemini-2.5-flash）
```

### 資料模型 `users/{uid}/items/{itemId}`

| 欄位 | 說明 |
|---|---|
| `type` | `task` 或 `event` |
| `title`、`note` | 標題（最多 80 字）與備註（最多 500 字） |
| `date` | `YYYY-MM-DD`：行程日期，或任務安排在哪天做 |
| `due`、`done`、`doneAt` | 只有任務才有：截止日、是否完成、完成時間 |
| `allDay`、`start`、`end` | 只有行程才有 |
| `category`、`priority` | 分類（學習、工作、個人、生活、健康、社團）與優先度 |
| `createdAt`、`updatedAt` | 時間戳記（毫秒） |

## Firebase 設定

1. 在 [Firebase Console](https://console.firebase.google.com) 建立專案（Spark 免費方案），並新增 **Web App**（不用勾 Hosting）。
2. **Authentication → Sign-in method**：啟用 **Google**。
3. **Authentication → Settings → Authorized domains**：加入 `happylifecpy.github.io`。本機測試時另外加 `localhost`。
4. **Firestore Database → Create database**：選 `asia-east1`，並用 Production mode。
5. **Firestore → Rules**：貼上本專案的 `firestore.rules`，再按 **Publish**。
6. **AI Logic → Get started**：選 **Gemini Developer API**。
7. 把 Project settings 裡的 `firebaseConfig` 填入 `index.html` 最上方的 `firebaseConfig`。

> Firebase Web 設定是公開的用戶端識別值，可以放在前端。服務帳戶金鑰、Gemini API Key 等秘密**不可以**放進前端或 repo。正式上線時，建議再到 Firebase 開啟 **App Check**，避免 AI 額度被他人濫用。

## 測試清單

- [ ] 在正式網址按 Google 登入，能顯示名稱和頭像
- [ ] 重新整理後仍是登入狀態，資料也還在
- [ ] 取消登入視窗不會出現假的成功訊息
- [ ] 登出後回到訪客畫面，看不到前一位使用者的資料
- [ ] 換一台裝置登入同一帳號，看得到同一份資料
- [ ] 新增、編輯、刪除、復原、完成都正常
- [ ] 篩選、排序、搜尋結果正確
- [ ] AI 一句話新增能正確填好表單；AI 規劃會參考實際行程
- [ ] 手機 390px 寬度沒有水平捲動，也沒有元素重疊
- [ ] 未登入時直接呼叫 Firestore 會被安全規則拒絕

## 部署

把檔案推到 `main` 分支，GitHub Pages（main / root）就會自動重新部署。請勿用 `file://` 測試登入，要用 HTTPS 或 `localhost`。
