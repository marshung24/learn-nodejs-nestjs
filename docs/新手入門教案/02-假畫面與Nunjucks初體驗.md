# U02｜假畫面 — Nunjucks 初體驗

> 建議時數：60 min
> 前置依賴：U01

---

## ① 為什麼先教這個？

U01 讓程式跑起來了，但只有文字回應。這堂課讓學員第一次「看到畫面」——用 Nunjucks 做出一個有書籍清單的 HTML 頁面。資料先用 Controller 中的假資料，不需要 DB，**重點是體驗「改了模板，刷新就看到結果」的即時回饋感**。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/book.controller.ts` | MVC 端點（回傳 HTML 頁面） |
| `views/book/list.njk` | 書籍清單頁 |
| `views/book/detail.njk` | 書籍詳情頁 |
| `views/book/form.njk` | 新增 / 編輯表單頁 |
| `views/layouts/main.njk` | 共用版型 |

---

## ③ 核心觀念

### 伺服器端渲染（SSR）

- Nunjucks 在 server 端組好完整 HTML 再送出
- 不需要前端框架

### `@Render()` 裝飾器

- 指定要渲染的模板名稱
- 回傳物件會自動注入模板上下文

### Nunjucks 關鍵語法

| 語法 | 說明 |
|------|------|
| `{% for item in items %}` | 迴圈渲染 |
| `{{ variable }}` | 輸出變數（自動 XSS 防護） |
| `{% extends 'layouts/main.njk' %}` | 繼承版型 |
| `{% block content %}` | 區塊覆寫 |
| `{% if condition %}` | 條件判斷 |

### 此刻不需要 DB

- Controller 直接建立假的書籍清單
- 先專注學 Nunjucks 語法

---

## ④ 動手做

### 必做

1. **觀察模板渲染流程**
   - 閱讀 `BookController.list()` 如何回傳資料
   - 對照 `list.njk` 中的 `{% for %}` 如何渲染每一行

2. **建立假資料清單**
   - 寫一個 Controller 方法，回傳包含 3 本假書的清單
   - 瀏覽 `http://localhost:3000/books` 看到表格

3. **觀察詳情頁**
   - 點進某本書的連結
   - 觀察 `{{ book.title }}` 如何顯示各欄位

### 延伸挑戰

1. 修改 `list.njk`，在表格加一欄「狀態」
   - 當庫存 = 0 時顯示紅字「缺貨」
   - 提示：`{% if book.stock == 0 %}`

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| 500 找不到模板 | `@Render()` 指定的路徑錯誤 | 確認 `views/` 下的資料夾名稱一致 |
| 頁面顯示 `{{ books }}` 原始文字 | 檔案副檔名不是 `.njk` | 確認副檔名與 view engine 設定 |

---

## ⑥ 驗收標準（DoD）

- [ ] 能在瀏覽器看到書籍清單的 HTML 頁面（假資料）
- [ ] 能說出 `{% for %}` 和 `{{ }}` 的用途
- [ ] 能解釋 `@Render()` 與普通 Controller 方法的差異
