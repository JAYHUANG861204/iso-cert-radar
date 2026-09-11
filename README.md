# 資安證照課程雷達

ISO 27001／27701／27017・27018／42001 與個資管理的**稽核員培訓開課清單**，四個人共同維護。

每天打開看一眼，別讓早鳥價和報名截止日溜掉。

**線上版：** https://jayhuang861204.github.io/iso-cert-radar/

---

## 這個清單想解決什麼

開課資訊散在十幾個主辦單位的官網，沒有人整理。更麻煩的是，**很多課名都叫「內部稽核員」，但發的證書等級完全不同** — 有的是 IRCA 國際登錄，有的只是結業證書，拿去應付驗證是無效的。

所以這份清單除了彙整梯次，還做了兩層分類：

- 🏅 **國際登錄證照** — CQI & IRCA 登錄，可註冊為第三方稽核員
- 📜 **ISO 稽核員證書** — 可作為驗證時的內稽訓練佐證
- 📘 **補知識・不發證照** — 結業證書，不構成稽核員資格

加上 🏢 **平日（要請假）** 與 🌅 **假日（不影響上班）** 的標示。

---

## 我想…

| | |
| --- | --- |
| **看課程** | 直接開 [線上版](https://jayhuang861204.github.io/iso-cert-radar/) |
| **改資料 / 補一梯新的** | 讀 [CONTRIBUTING.md](CONTRIBUTING.md) |
| **回報但不想自己改** | 開一個 [Issue](../../issues/new/choose) |
| **看權限怎麼設定的 / 加減人** | 讀 [docs/rbac.md](docs/rbac.md) |

---

## 這個 repo 長什麼樣

```
iso-cert-radar/
├── index.html              網頁本身（版面與邏輯）
├── data/courses.json       ⭐ 課程資料 — 平常只會動到這個檔
├── firestore.rules         Firestore 權限規則（見 docs/rbac.md）
├── scripts/validate.mjs    資料檢查器，CI 會跑
└── .github/workflows/
    ├── validate.yml        CI：每個 PR 自動檢查資料格式
    └── deploy.yml          CD：合併進 main 就自動上線
```

**內容與程式是分開的。** 要改課程資訊，動 `data/courses.json` 就好，不需要碰 HTML。

---

## 本機跑起來

不需要安裝任何套件。

```bash
npm start                    # 或 python3 -m http.server 8000
node scripts/validate.mjs    # 推上去之前先自己檢查一次
```

> ⚠️ 不要用滑鼠雙擊 `index.html`，瀏覽器會擋掉本機讀取 JSON，畫面會是空的。

---

## 資料來源

各主辦單位官網／報名頁：[領導力 ISO 管理學院](https://isokm.com.tw/event/category/68)、[全智網](https://ainetwork-training.com/courses/)、[中國生產力中心](https://edu.cpc.org.tw/class/content/303)、[電腦稽核協會](https://www.caa.org.tw/coursedetail-37002.html)、[亞瑞仕](https://www.ares-registration.com/)、[TPIPAS](https://www.tpipas.org.tw/course_list.aspx?no=118)、[臺北市中小企業知識學苑](https://www.startup.taipei/index.php?action=course&cid=83)、[勞動部產業人才投資方案](https://ojt.wda.gov.tw/ClassSearch)。

價格一律標早鳥／優惠價，**報名前請務必到來源頁再確認一次** — 早鳥條件（開課前 10 天、一個月前、三人同行）各家不同。
