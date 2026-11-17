# clericalwork：報到暨班員統計

這是「年度課程暨勤務管理系統」的獨立報到系統。它和年度課程系統用同一個 Firebase 專案（`annual-calendar-c4a37`），但不需要修改年度課程系統的任何程式。

- 網址：`https://<帳號>.github.io/clericalwork/`
- 只有管理者可以使用，管理密碼和行事曆相同。每台裝置第一次使用時要輸入一次，之後會記住。
- 登入身分和行事曆分開：在行事曆登出，不會影響這裡。

## 部署

1. 在 GitHub 新增 repo `clericalwork`，放入 `index.html`（以及這份 README）。
2. 到 Settings → Pages，Source 選 `main` 分支的根目錄。
3. 若想在頁首顯示「行事曆」按鈕，把 `index.html` 開頭的 `CAL_URL` 改成行事曆網址，例如 `"../你的行事曆repo/"`。
4. 在 Firebase 主控台加入下方的 Firestore 規則。
5. 想先試用、不碰正式資料，可以在網址後面加 `?demo`。資料只會存在瀏覽器裡。

## 資料存放

| 集合 | 內容 | 誰會寫入 |
|---|---|---|
| `courses` | 課程（只讀） | 只有現場補登時會更新名額計數 `counts` |
| `signups` | 報名（只讀選定那一場） | 只有現場補登時會新增一筆（帶 `walkin: true`） |
| `checkins/{課程ID}` | 每個課程一份報到文件：`d.{日期}.{報名ID} = {t: 時間, by: 報到人員}` | 報到系統 |
| `ciStats/{課程ID}` | 每個課程一份統計摘要，給統計頁使用 | 報到系統（點名後約 15 秒自動更新，或按「全年重算」） |

## Firestore 規則（需要新增）

請加在現有規則的 `match /databases/{database}/documents { ... }` 裡。如果你的規則裡已經有判斷管理者的函式，直接用那個函式，取代下面的 `isManager()`。

```
function isManager() {
  let c = get(/databases/$(database)/documents/claims/$(request.auth.uid)).data;
  let s = get(/databases/$(database)/documents/config/secrets).data;
  return (c.level == 'super' && c.k == s.superKey) || (c.level == 'manager' && c.k == s.managerKey);
}

match /checkins/{cid} {
  allow read, write: if request.auth != null && isManager();
}
match /ciStats/{cid} {
  allow read, write: if request.auth != null && isManager();
}
```

現場補登會新增 `signups` 文件，帶 `walkin` 欄位，並更新 `courses` 的 `counts`。如果現有規則限制了新建報名時可寫的欄位，請把 `walkin` 加進去。
