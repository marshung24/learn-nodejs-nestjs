# U05｜TypeORM Migration 與資料庫設計

> 建議時數：90 min
> 前置依賴：U04

---

## ① 為什麼先教這個？

U02-U03 用的是假資料，接下來要換成真的。第一步是「設計資料庫結構」——用 TypeORM Migration 管理 schema 版本，確保多人協作時每個人的 DB 長一樣。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `migrations/1700000000000-CreateBooksTable.ts` | 建表（初始 schema） |
| `migrations/1700000000001-InsertSampleData.ts` | 塞入範例資料 |
| `data-source.ts` | TypeORM CLI 使用的資料來源設定 |

---

## ③ 核心觀念

### Migration 機制

- 用 TypeScript 撰寫 `up()` 和 `down()` 方法
- `up()`：升級（建表、加欄位）
- `down()`：回滾（刪表、移除欄位）

### 時間戳命名

- `{timestamp}-{描述}.ts`
- 確保執行順序

### `migrations` 表

- TypeORM 在 DB 中自動建立的紀錄表
- 記錄每次 migration 的執行狀態

### 不可修改已執行的 migration

- 已執行的 migration 不應修改
- 需用新 migration 修補

### 欄位設計思考

| 決策 | 理由 |
|------|------|
| 用 `BIGINT` 不用 `INT` | 避免未來 ID 溢位 |
| `isbn` 加 `UNIQUE` | 確保唯一性 |
| `created_at` / `updated_at` | 審計追蹤 |

---

## ④ 動手做

### 必做

1. **閱讀現有 migration**
   - 閱讀 `CreateBooksTable` migration
   - 說出每個欄位的設計理由

2. **執行 migration**
   - 執行 `npm run migration:run`
   - 確認 migration 成功執行

3. **新增 migration**
   - 新增 `{timestamp}-AddPublisherColumn.ts`
   - 為 books 表加一個 `publisher VARCHAR(100)` 欄位
   - 執行 migration 並確認生效

4. **確認紀錄**
   - 連線 MySQL 查詢 `SELECT * FROM migrations`

### 延伸挑戰

1. 執行 `npm run migration:revert` 回滾最後一次 migration
   - 觀察 `down()` 方法的效果

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Migration 指令報錯找不到 data-source | `data-source.ts` 路徑設定錯誤 | 確認 `package.json` 中的 migration 腳本指向正確路徑 |
| Migration 執行後 DB 沒變化 | 該 migration 已執行過 | 檢查 `migrations` 表，已執行的不會重跑 |

---

## ⑥ 驗收標準（DoD）

- [ ] 能撰寫一支新的 migration 並在執行後看到資料表結構變更
- [ ] 能在 `migrations` 表確認執行紀錄
- [ ] 能說出 `up()` 和 `down()` 的用途
