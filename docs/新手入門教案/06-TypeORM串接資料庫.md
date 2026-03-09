# U06｜TypeORM 串接資料庫

> 建議時數：120 min
> 前置依賴：U02, U03, U05

---

## ① 為什麼先教這個？

U05 設計好了資料表，這堂課要把假資料換成真的——用 TypeORM 串接 MySQL，讓 Controller 直接從 DB 讀寫資料。**完成後學員會第一次看到「頁面上的資料來自真實資料庫」的完整 CRUD 循環。** 此刻先不抽 Service 層，Controller 直接操作 Repository——保持最短路徑讓 CRUD 跑起來。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/entities/book.entity.ts` | DB 實體（對應 books 表） |
| `src/book/book.repository.ts` | Repository 擴充（自訂查詢方法） |
| `src/book/book.module.ts` | 模組設定（注入 Repository） |

---

## ③ 核心觀念

### Entity 裝飾器

| 裝飾器 | 用途 |
|--------|------|
| `@Entity()` | 標記為資料表對應類別 |
| `@PrimaryGeneratedColumn()` | 自增主鍵 |
| `@Column()` | 一般欄位 |
| `@CreateDateColumn()` | 自動填入建立時間 |
| `@UpdateDateColumn()` | 自動填入更新時間 |

### Repository Pattern

- `@InjectRepository()` 注入 Repository
- 使用內建 CRUD 方法：
  - `find()` / `findOne()`
  - `save()`
  - `delete()`

### 自訂 Repository 方法

```typescript
async findByIsbn(isbn: string): Promise<Book | null> {
  return this.repository.findOne({ where: { isbn } });
}
```

### 替換假資料

- 修改 U02/U03 的 Controller
- 把 hardcoded list 改成呼叫 `repository.find()`
- 頁面立刻顯示真實 DB 資料

---

## ④ 動手做

### 必做

1. **閱讀 Entity**
   - 閱讀 `book.entity.ts`
   - 對照每個 `@Column()` 與資料表欄位的對應關係

2. **替換假資料**
   - 修改 `BookController`
   - 把 `list()` 方法中的假資料替換為 `this.bookRepository.find()`
   - 重新整理頁面確認顯示 DB 資料

3. **新增查詢方法**
   - 在 Repository 新增 `findByIsbn(isbn: string)` 方法
   - 確認編譯通過

### 延伸挑戰

1. 新增多條件查詢方法（依作者 + 庫存範圍）
   - 體驗 QueryBuilder 的用法

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Repository 注入時拋錯 | Module 沒有 import `TypeOrmModule.forFeature([Book])` | 在 `book.module.ts` 的 imports 加入 |
| Entity 欄位與 DB 不一致 | Entity 定義與 migration 建立的表結構不符 | 對照 migration 的欄位定義修正 Entity |

---

## ⑥ 驗收標準（DoD）

- [ ] 能在瀏覽器看到來自 DB 的書籍清單（不再是假資料）
- [ ] 能新增一個 Repository 方法
- [ ] 能說出 Entity 與資料表的對應關係
