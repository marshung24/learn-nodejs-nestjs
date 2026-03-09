# U06｜TypeORM 串接資料庫

> 能用 TypeORM 完成真實 CRUD，頁面與 API 顯示 DB 真實資料 ｜ 120 min ｜ 前置依賴：U02, U03, U05

---

## 為什麼先教這個？

U05 設計好了資料表，這堂課要把假資料換成真的——用 TypeORM 串接 MySQL，讓 Controller 直接從 DB 讀寫資料。**完成後學員會第一次看到「頁面上的資料來自真實資料庫」的完整 CRUD 循環。** 此刻先不抽 Service 層，Controller 直接操作 Repository——保持最短路徑讓 CRUD 跑起來。

> **講師提示**：這是課程的第一個「啊哈時刻」——學員親眼看到頁面從假資料切換到真實 DB 資料。建議 live demo 時先秀假資料頁面，再切換到 Repository 呼叫，重新整理瀏覽器的瞬間最有衝擊力。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/entities/book.entity.ts` | DB 實體（對應 books 表） |
| `book/book.repository.ts` | Repository 擴充（自訂查詢方法） |
| `book/book.module.ts` | 模組設定（注入 Repository） |

---

## 核心觀念

### `book.entity.ts` — DB 實體

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity('books')  // 對應資料表名稱
export class Book {

  @PrimaryGeneratedColumn('increment', { type: 'bigint' })
  id: number;

  @Column({ type: 'varchar', length: 200 })
  title: string;

  @Column({ type: 'varchar', length: 100 })
  author: string;

  @Column({ type: 'varchar', length: 20, unique: true })
  isbn: string;

  @Column({ type: 'int', default: 0 })
  stock: number;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}
```

### Entity 裝飾器說明

| 裝飾器 | 用途 | 範例 |
|--------|------|------|
| `@Entity('table_name')` | 標記為資料表對應類別 | `@Entity('books')` |
| `@PrimaryGeneratedColumn()` | 自增主鍵 | `@PrimaryGeneratedColumn('increment')` |
| `@Column()` | 一般欄位 | `@Column({ type: 'varchar', length: 200 })` |
| `@CreateDateColumn()` | 自動填入建立時間 | `@CreateDateColumn({ name: 'created_at' })` |
| `@UpdateDateColumn()` | 自動填入更新時間 | `@UpdateDateColumn({ name: 'updated_at' })` |

### `snake_case` 與 `camelCase` 轉換

TypeORM 可透過 `@Column({ name: 'created_at' })` 指定 DB 欄位名稱，讓 TypeScript 使用 `camelCase`，DB 使用 `snake_case`：

```typescript
@CreateDateColumn({ name: 'created_at' })  // DB: created_at
createdAt: Date;                            // TS: createdAt
```

### Repository Pattern — 內建 CRUD 方法

使用 `@InjectRepository()` 注入 Repository，獲得內建 CRUD 方法：

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Book } from './entities/book.entity';

@Injectable()
export class BookRepository {
  constructor(
    @InjectRepository(Book)
    private readonly repository: Repository<Book>,
  ) {}

  // 查詢所有
  async findAll(): Promise<Book[]> {
    return this.repository.find({ order: { id: 'DESC' } });
  }

  // 依 ID 查詢
  async findById(id: number): Promise<Book | null> {
    return this.repository.findOne({ where: { id } });
  }

  // 依 ISBN 查詢
  async findByIsbn(isbn: string): Promise<Book | null> {
    return this.repository.findOne({ where: { isbn } });
  }

  // 新增
  async create(book: Partial<Book>): Promise<Book> {
    const entity = this.repository.create(book);
    return this.repository.save(entity);
  }

  // 更新
  async update(id: number, book: Partial<Book>): Promise<Book | null> {
    await this.repository.update(id, book);
    return this.findById(id);
  }

  // 刪除
  async delete(id: number): Promise<void> {
    await this.repository.delete(id);
  }
}
```

### Module 設定 — 注入 Repository

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Book } from './entities/book.entity';
import { BookRepository } from './book.repository';
import { BookController } from './book.controller';
import { BookApiController } from './book.api.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([Book]),  // 註冊 Entity
  ],
  controllers: [BookController, BookApiController],
  providers: [BookRepository],
  exports: [BookRepository],
})
export class BookModule {}
```

### 替換假資料

修改 U02/U03 的 Controller，把 hardcoded list 改成呼叫 `repository.findAll()`：

```typescript
// 修改前（U02 假資料）
@Get()
@Render('book/list')
list() {
  const books = [
    { id: 1, title: 'Clean Code', author: 'Robert C. Martin', ... },
  ];
  return { books };
}

// 修改後（真實 DB）
@Get()
@Render('book/list')
async list() {
  const books = await this.bookRepository.findAll();
  return { books };
}
```

頁面立刻顯示真實 DB 資料，**模板完全不用改**——這就是分層的價值。

### 完整 CRUD Controller 範例

```typescript
import { Controller, Get, Post, Put, Delete, Param, Body, Render, Redirect } from '@nestjs/common';
import { BookRepository } from './book.repository';

@Controller('books')
export class BookController {
  constructor(private readonly bookRepository: BookRepository) {}

  @Get()
  @Render('book/list')
  async list() {
    const books = await this.bookRepository.findAll();
    return { books };
  }

  @Get('new')
  @Render('book/form')
  createForm() {
    return { book: null };  // 新增時沒有既有資料
  }

  @Post()
  @Redirect('/books')
  async create(@Body() body: any) {
    await this.bookRepository.create(body);
    // PRG 模式：Post 後 Redirect，避免重複提交
  }

  @Get(':id')
  @Render('book/detail')
  async detail(@Param('id') id: string) {
    const book = await this.bookRepository.findById(Number(id));
    return { book };
  }

  @Get(':id/edit')
  @Render('book/form')
  async editForm(@Param('id') id: string) {
    const book = await this.bookRepository.findById(Number(id));
    return { book };
  }

  @Post(':id')
  @Redirect('/books')
  async update(@Param('id') id: string, @Body() body: any) {
    await this.bookRepository.update(Number(id), body);
  }

  @Post(':id/delete')
  @Redirect('/books')
  async delete(@Param('id') id: string) {
    await this.bookRepository.delete(Number(id));
  }
}
```

---

## 動手做

### 必做

**1. 閱讀 Entity 與資料表的對應**

閱讀 `book.entity.ts`，對照每個 `@Column()` 與資料表欄位的對應關係。回答：

- `@Entity('books')` 的 `'books'` 代表什麼？
- `@Column({ name: 'created_at' })` 的 `name` 選項作用是什麼？

**2. 替換假資料**

修改 `BookController` 的 `list()` 方法：

```typescript
// 從
const books = [{ id: 1, title: 'Clean Code', ... }];

// 改成
const books = await this.bookRepository.findAll();
```

重新整理頁面 `http://localhost:3000/books`，確認顯示 DB 資料。

**3. 新增自訂查詢方法**

在 `BookRepository` 新增 `findByIsbn(isbn: string)` 方法：

```typescript
async findByIsbn(isbn: string): Promise<Book | null> {
  return this.repository.findOne({ where: { isbn } });
}
```

執行 `npm run start:dev` 確認編譯通過。

### 延伸挑戰

1. 新增一個多條件查詢方法（如依作者 + 庫存範圍），體驗 TypeORM QueryBuilder 的用法：

```typescript
async findByAuthorAndStock(author: string, minStock: number): Promise<Book[]> {
  return this.repository
    .createQueryBuilder('book')
    .where('book.author = :author', { author })
    .andWhere('book.stock >= :minStock', { minStock })
    .getMany();
}
```

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Repository 注入時拋 `Nest could not find...` 錯誤 | Module 沒有 import `TypeOrmModule.forFeature([Book])` | 在 `book.module.ts` 的 imports 加入 |
| Entity 欄位與 DB 不一致 | Entity 定義與 migration 建立的表結構不符 | 對照 migration 的欄位定義修正 Entity |
| `findOne` 回傳 null | 查詢條件不對或資料不存在 | 確認 `where` 條件正確；用 `find()` 先確認有資料 |
| 日期欄位顯示異常 | 時區設定不一致 | 確認 DB 連線字串加上 `?timezone=+08:00` |

---

## 驗收標準（DoD）

- [ ] 能在瀏覽器看到來自 DB 的書籍清單（不再是假資料）
- [ ] 能完成完整的 CRUD 操作（新增、查詢、更新、刪除）
- [ ] 能新增一個 Repository 查詢方法（如 `findByIsbn`）
- [ ] 能說出 Entity 與資料表的對應關係
- [ ] 能解釋 `@InjectRepository()` 的作用
