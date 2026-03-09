# U07｜正規化架構 — Service 與 DTO 分層

> 能從 Controller 抽出 Service 層，引入 DTO 做資料驗證與格式轉換 ｜ 90 min ｜ 前置依賴：U06

---

## 為什麼先教這個？

U06 完成了 CRUD，但 Controller 直接操作 Repository——所有邏輯擠在一起，改一個地方可能影響全部。這堂課把程式碼「整理乾淨」：抽出 Service 層集中業務邏輯，引入 DTO 分離輸入/輸出格式。**重構前後功能完全一樣，但程式碼更好維護、更好測試。**

> **講師提示**：先展示重構前（Controller 直接操作 Repository）和重構後（Controller → Service → Repository）的對比，讓學員感受到「程式碼可以一樣能跑，但結構差很多」。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.service.ts` | Service 層（業務邏輯，此刻先不含快取） |
| `book/dto/create-book.dto.ts` | 輸入驗證 DTO（新增/更新用） |
| `book/dto/book-response.dto.ts` | 輸出格式 DTO（API 回傳用） |

---

## 核心觀念

### 重構前後對比

```
重構前（U06）：                    重構後（U07）：

  Controller                        Controller
      │                                │
      ▼                                ▼
  BookRepository                   BookService
      │                                │
      ▼                                ▼
     DB                         BookRepository
                                       │
                                       ▼
                                      DB
```

### `book.service.ts` — Service 實作

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { BookRepository } from './book.repository';
import { Book } from './entities/book.entity';
import { CreateBookDto } from './dto/create-book.dto';
import { BookResponseDto } from './dto/book-response.dto';

@Injectable()
export class BookService {
  constructor(private readonly bookRepository: BookRepository) {}

  async findAll(): Promise<BookResponseDto[]> {
    const books = await this.bookRepository.findAll();
    return books.map(book => BookResponseDto.from(book));
  }

  async findById(id: number): Promise<BookResponseDto> {
    const book = await this.bookRepository.findById(id);
    if (!book) {
      throw new NotFoundException(`Book with ID ${id} not found`);
    }
    return BookResponseDto.from(book);
  }

  async create(dto: CreateBookDto): Promise<BookResponseDto> {
    const book = await this.bookRepository.create({
      title: dto.title,
      author: dto.author,
      isbn: dto.isbn,
      stock: dto.stock,
    });
    return BookResponseDto.from(book);
  }

  async update(id: number, dto: CreateBookDto): Promise<BookResponseDto> {
    const existing = await this.bookRepository.findById(id);
    if (!existing) {
      throw new NotFoundException(`Book with ID ${id} not found`);
    }
    const book = await this.bookRepository.update(id, {
      title: dto.title,
      author: dto.author,
      isbn: dto.isbn,
      stock: dto.stock,
    });
    return BookResponseDto.from(book!);
  }

  async delete(id: number): Promise<void> {
    const existing = await this.bookRepository.findById(id);
    if (!existing) {
      throw new NotFoundException(`Book with ID ${id} not found`);
    }
    await this.bookRepository.delete(id);
  }
}
```

### `@Injectable()` — 可被 DI 管理

`@Injectable()` 標記類別可被 NestJS 依賴注入容器管理，Controller 可透過 constructor 注入使用：

```typescript
@Injectable()
export class BookService { ... }

// Controller 中使用
@Controller('books')
export class BookController {
  constructor(private readonly bookService: BookService) {}
}
```

### `create-book.dto.ts` — 輸入驗證 DTO

```typescript
import { IsNotEmpty, IsString, IsInt, Min, MaxLength, IsOptional } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateBookDto {

  @ApiProperty({ description: '書名', example: 'Clean Code' })
  @IsNotEmpty({ message: '書名不可為空' })
  @IsString()
  @MaxLength(200)
  title: string;

  @ApiProperty({ description: '作者', example: 'Robert C. Martin' })
  @IsNotEmpty({ message: '作者不可為空' })
  @IsString()
  @MaxLength(100)
  author: string;

  @ApiProperty({ description: 'ISBN', example: '9780132350884' })
  @IsNotEmpty({ message: 'ISBN 不可為空' })
  @IsString()
  @MaxLength(20)
  isbn: string;

  @ApiProperty({ description: '庫存數量', example: 5, minimum: 0 })
  @IsInt({ message: '庫存必須是整數' })
  @Min(0, { message: '庫存不可為負數' })
  stock: number;
}
```

### class-validator 常用裝飾器

| 裝飾器 | 用途 | 範例 |
|--------|------|------|
| `@IsNotEmpty()` | 不可為空 | `@IsNotEmpty({ message: '書名不可為空' })` |
| `@IsString()` | 必須是字串 | `@IsString()` |
| `@IsInt()` | 必須是整數 | `@IsInt()` |
| `@Min(n)` | 最小值 | `@Min(0)` |
| `@Max(n)` | 最大值 | `@Max(100)` |
| `@MaxLength(n)` | 最大長度 | `@MaxLength(200)` |
| `@IsOptional()` | 可選欄位 | `@IsOptional()` |
| `@IsEmail()` | Email 格式 | `@IsEmail()` |

### `book-response.dto.ts` — 輸出格式 DTO

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { Book } from '../entities/book.entity';

export class BookResponseDto {

  @ApiProperty({ description: '書籍 ID' })
  id: number;

  @ApiProperty({ description: '書名' })
  title: string;

  @ApiProperty({ description: '作者' })
  author: string;

  @ApiProperty({ description: 'ISBN' })
  isbn: string;

  @ApiProperty({ description: '庫存數量' })
  stock: number;

  @ApiProperty({ description: '建立時間' })
  createdAt: Date;

  @ApiProperty({ description: '更新時間' })
  updatedAt: Date;

  /**
   * Entity → DTO 轉換的靜態工廠方法
   */
  static from(book: Book): BookResponseDto {
    const dto = new BookResponseDto();
    dto.id = book.id;
    dto.title = book.title;
    dto.author = book.author;
    dto.isbn = book.isbn;
    dto.stock = book.stock;
    dto.createdAt = book.createdAt;
    dto.updatedAt = book.updatedAt;
    return dto;
  }
}
```

### Entity 與 DTO 分離

```
資料流向：

  HTTP Request
      │
      ▼
  CreateBookDto ──→ Book（Entity）──→ DB
  （收什麼）         （存什麼）

  DB ──→ Book（Entity）──→ BookResponseDto ──→ HTTP Response
         （存什麼）         （給什麼）
```

- `Book` 對應 DB 表結構
- `CreateBookDto` 對應「API 收什麼」（含驗證規則）
- `BookResponseDto` 對應「API 給什麼」（含 DB 產生的時間戳）

修改其中一邊不會牽動另一邊。

### 啟用全域 ValidationPipe

在 `main.ts` 啟用全域驗證：

```typescript
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 啟用全域驗證
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,       // 自動移除 DTO 中未定義的屬性
    forbidNonWhitelisted: true,  // 存在未定義屬性時拋錯
    transform: true,       // 自動轉換類型
  }));

  await app.listen(process.env.PORT ?? 3000);
}
```

### PRG 模式（Post-Redirect-Get）

MVC 表單送出後用 redirect 避免使用者按 F5 重複提交：

```typescript
@Post()
@Redirect('/books')  // PRG：Post 後 Redirect 到 GET
async create(@Body() dto: CreateBookDto) {
  await this.bookService.create(dto);
  // 不回傳內容，直接重導向到清單頁
}
```

---

## 動手做

### 必做

**1. 閱讀 Service**

閱讀 `book.service.ts`，列出所有方法並說明每個方法的用途：

| 方法 | 用途 |
|------|------|
| `findAll()` | ? |
| `findById()` | ? |
| `create()` | ? |
| `update()` | ? |
| `delete()` | ? |

**2. 追蹤 create 流程**

追蹤 `BookService.create()` 的完整流程：

```
CreateBookDto → Book（Entity）→ save → BookResponseDto
     ↑              ↑                       ↑
   輸入驗證      存入 DB            回傳給 Client
```

**3. 重構 Controller**

把 `BookController` 和 `BookApiController` 中直接操作 `BookRepository` 的程式碼改為呼叫 `BookService`：

```typescript
// 修改前
constructor(private readonly bookRepository: BookRepository) {}

// 修改後
constructor(private readonly bookService: BookService) {}
```

確認頁面和 API 行為與重構前完全一致。

### 延伸挑戰

1. 在 Service 加入 `findByIsbn()` 方法（`⟵ 需完成 U06 必做 3`），串接 Repository 的新方法
2. 畫出 CreateDto → Entity → ResponseDto 的資料流向圖

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Service 注入時噴 `Nest could not find...` 錯誤 | Service 沒有在 Module 的 providers 註冊 | 在 `book.module.ts` 的 providers 加入 `BookService` |
| DTO 驗證沒生效 | 沒有全域啟用 ValidationPipe | 在 `main.ts` 加入 `app.useGlobalPipes(new ValidationPipe())` |
| 驗證錯誤訊息是英文 | 沒有自訂 message | 在裝飾器加入 `{ message: '自訂訊息' }` |
| `whitelist: true` 導致欄位消失 | DTO 沒有定義該欄位 | 確認 DTO 包含所有需要的欄位 |

---

## 驗收標準（DoD）

- [ ] 能說明 Service 層存在的理由——集中業務邏輯，Controller 只負責 HTTP
- [ ] 能解釋 Entity / CreateDto / ResponseDto 三者的職責差異
- [ ] 頁面和 API 行為與重構前完全一致
- [ ] 能說出 `@Injectable()` 的作用
- [ ] 能解釋 PRG 模式的目的——避免重複提交
