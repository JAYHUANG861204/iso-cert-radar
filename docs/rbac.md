# 權限設計（RBAC）

這份紀錄說明「同步資料到資料庫」這個功能背後的權限機制是怎麼運作的、怎麼加減人。

---

## 為什麼要改

原本的白名單機制（`emails` 陣列）只在**前端 JavaScript** 檢查：登入後拿 email 去比對，比對不到就顯示「沒有權限」的畫面，但沒有把人擋在資料庫外面。也就是說，只要有 Google 帳號登入過，理論上都能繞過前端畫面，直接用 Firebase SDK 對 Firestore 讀寫，白名單擋不住。

現在改成兩層：

1. **前端**：登入後查自己的角色，決定要不要顯示「同步資料到資料庫」按鈕（體驗層，好不好用）
2. **後端（Firestore Security Rules）**：不管前端顯示什麼，資料庫真正的讀寫权限都由 `firestore.rules` 強制檢查（安全層，擋不擋得住）

---

## 角色

| 角色 | 可以做什麼 |
| --- | --- |
| `editor` | 讀課程資料 + 按「同步資料到資料庫」寫入 `iso_courses`／`iso_categories`／`iso_cert_levels`／`iso_meta` |
| `viewer` | 只能讀課程資料，看不到同步按鈕，就算手動呼叫 SDK 寫入也會被 Rules 擋掉 |
| （不在名單內） | 登入後顯示「沒有這個頁面的權限」，什麼都讀不到 |

目前名單（2026-09-11 建立）：

| Email | 角色 |
| --- | --- |
| jayhuang5974993@gmail.com | editor |
| workto51k@gmail.com | viewer |
| kuoyihsuan7924@gmail.com | viewer |
| robin.lexus@gmail.com | viewer |

---

## 資料存在哪裡

Firestore 的 **`userid`** collection，一人一筆文件：

```
userid/{email}
  ├── email: "xxx@gmail.com"   （跟文件 ID 一樣，方便直接看）
  └── role:  "editor" | "viewer"
```

文件 ID 直接用 email（全小寫）。

---

## 怎麼加人 / 改角色

**只能在 Firebase Console 手動改，App 本身沒有寫入這個 collection 的入口。**

1. 打開 [Firestore 資料頁](https://console.firebase.google.com/project/iso-cert--radar/firestore/databases/-default-/data/~2Fuserid)
2. 新增文件，文件 ID 填新人的 email（全小寫）
3. 加兩個欄位：`email`（string，跟文件 ID 一樣）、`role`（string，`editor` 或 `viewer`）

要移除某人權限，把該文件刪掉，或把 `role` 改成非 `editor`/`viewer` 的值都可以。

### 為什麼刻意不讓 App 自己改名單

`firestore.rules` 裡 `userid/{userId}` 這個路徑的 `allow write` 寫死是 `false`——任何角色，包括 `editor`，都不能透過網站幫自己或別人加名單、升級角色。這是刻意設計，避免任何一個帳號被盜用後，攻擊者可以直接把自己加進白名單、甚至把自己設成 editor 亂寫資料。要改名單一定要有 Firebase Console 的專案存取權限，多一層保護。

---

## 相關檔案

```
firestore.rules   ← 實際的權限規則（唯一真正擋權限的地方）
firebase.json     ← 指定 firestore.rules 要部署去哪
index.html        ← getRole()：登入後查自己角色、決定畫面顯示什麼
```

## 部署規則

改了 `firestore.rules` 之後要記得部署，不然 Console 上看到的規則不會變：

```bash
firebase deploy --only firestore:rules --project iso-cert--radar
```

---

## 這次還做了什麼（2026-09-11 ~ 2026-09-12 變更紀錄）

1. Firebase CLI 登入本機環境（`firebase login`）
2. 清空 Firestore 內所有舊資料（原本只有 `config` collection 有東西，`iso_*` 系列都還是空的）
3. 暫時下架 `iso-cert--radar.web.app`（`firebase hosting:disable`），確認清空範圍後才重新部署
4. 把白名單機制從「前端判斷、`config/access` 的 `emails` 陣列」改成「後端 Firestore Rules 強制、`userid/{email}` 一人一筆文件」
5. 部署新版 `index.html` + `firestore.rules` 到 Firebase Hosting，並 push 到 [GitHub fork](https://github.com/JAYHUANG861204/iso-cert-radar)（GitHub Pages 也會自動同步更新）
6. 在 Console 手動建立 4 筆 `userid` 文件，抓出一次 collection 命名（`userid` vs 程式碼原本寫的 `users`）打錯導致全員被擋在外面的問題，修正並重新部署
