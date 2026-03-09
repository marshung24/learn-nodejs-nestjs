# U08｜REST API 完善與錯誤處理

> 能用 Exception Filter 統一錯誤回應，正確使用 HTTP 狀態碼 ｜ 90 min ｜ 前置依賴：U07

---

## 為什麼先教這個？

U07 完成了分層，但 API 還缺少正確的 HTTP 狀態碼和統一的錯誤回應。使用者送了錯誤資料、查了不存在的 ID，應該要拿到清楚的錯誤訊息而非一堆 stack trace。這堂課把 API 做到「生產品質」。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.api.controller.ts` | REST API 端點（加入正確的狀態碼） |
| `common/filters/http-exception.filter.ts` | 全域錯誤處理（統一所有例外的回應格式） |
| `common/pipes/validation.pipe.ts` | 全域驗證 Pipe |

---

## 核心觀念

### HTTP 狀態碼語意

| 狀態碼 | 含義 | 對應場景 |
|--------|------|----------|
| 200 OK | 成功 | GET 查詢、PUT 更新 |
| 201 Created | 建立成功 | POST 新增 |
| 204 No Content | 成功但無回傳內容 | DELETE 刪除 |
| 400 Bad Request | 請求格式錯誤 | DTO 驗證失敗 |
| 404 Not Found | 找不到資源 | ID 不存在 |
| 409 Conflict | 資源衝突 | ISBN 重複 |
| 500 Internal Server Error | 伺服器內部錯誤 | 未預期的例外 |

### `book.api.controller.ts` — 完整 REST API

```typescript
import {
  Controller, Get, Post, Put, Delete,
  Param, Body, HttpCode, HttpStatus,
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';
import { BookService } from './book.service';
import { CreateBookDto } from './dto/create-book.dto';
import { BookResponseDto } from './dto/book-response.dto';

@ApiTags('books')
@Controller('api/books')
export class BookApiController {
  constructor(private readonly bookService: BookService) {}

  @Get()
  @ApiOperation({ summary: '取得所有書籍' })
  @ApiResponse({ status: 200, description: '成功取得書籍清單', type: [BookResponseDto] })
  async findAll(): Promise<BookResponseDto[]> {
    return this.bookService.findAll();
  }

  @Get(':id')
  @ApiOperation({ summary: '取得單本書籍' })
  @ApiResponse({ status: 200, description: '成功取得書籍', type: BookResponseDto })
  @ApiResponse({ status: 404, description: '書籍不存在' })
  async findById(@Param('id') id: string): Promise<BookResponseDto> {
    return this.bookService.findById(Number(id));
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)  // 新增成功回 201
  @ApiOperation({ summary: '新增書籍' })
  @ApiResponse({ status: 201, description: '書籍已建立', type: BookResponseDto })
  @ApiResponse({ status: 400, description: '驗證失敗' })
  async create(@Body() dto: CreateBookDto): Promise<BookResponseDto> {
    return this.bookService.create(dto);
  }

  @Put(':id')
  @ApiOperation({ summary: '更新書籍' })
  @ApiResponse({ status: 200, description: '書籍已更新', type: BookResponseDto })
  @ApiResponse({ status: 404, description: '書籍不存在' })
  async update(
    @Param('id') id: string,
    @Body() dto: CreateBookDto,
  ): Promise<BookResponseDto> {
    return this.bookService.update(Number(id), dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)  // 刪除成功回 204
  @ApiOperation({ summary: '刪除書籍' })
  @ApiResponse({ status: 204, description: '書籍已刪除' })
  @ApiResponse({ status: 404, description: '書籍不存在' })
  async delete(@Param('id') id: string): Promise<void> {
    return this.bookService.delete(Number(id));
  }
}
```

### `@HttpCode()` — 指定回應狀態碼

NestJS 預設回應 200，需手動指定其他狀態碼：

```typescript
@Post()
@HttpCode(HttpStatus.CREATED)  // POST 成功應回 201
create() { ... }

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)  // DELETE 成功應回 204
delete() { ... }
```

### `http-exception.filter.ts` — 統一錯誤回應

```typescript
import {
  ExceptionFilter, Catch, ArgumentsHost,
  HttpException, HttpStatus, Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()  // 攔截所有例外
export class HttpExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(HttpExceptionFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal Server Error';

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      message = typeof exceptionResponse === 'string'
        ? exceptionResponse
        : (exceptionResponse as any).message || exception.message;
    } else if (exception instanceof Error) {
      // 未預期的例外，只記 log，不對外暴露細節
      this.logger.error(`Unexpected error on ${request.url}`, exception.stack);
    }

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: Array.isArray(message) ? message : [message],
    });
  }
}
```

### 在 `main.ts` 全域註冊 Filter

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { HttpExceptionFilter } from './common/filters/http-exception.filter';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 全域驗證 Pipe
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));

  // 全域錯誤處理 Filter
  app.useGlobalFilters(new HttpExceptionFilter());

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### NestJS 內建例外類別

| 例外類別 | HTTP 狀態碼 | 使用場景 |
|----------|-------------|----------|
| `BadRequestException` | 400 | 請求格式錯誤 |
| `UnauthorizedException` | 401 | 未授權 |
| `ForbiddenException` | 403 | 禁止存取 |
| `NotFoundException` | 404 | 找不到資源 |
| `ConflictException` | 409 | 資源衝突 |
| `InternalServerErrorException` | 500 | 伺服器內部錯誤 |

使用方式：

```typescript
import { NotFoundException, ConflictException } from '@nestjs/common';

async findById(id: number) {
  const book = await this.bookRepository.findById(id);
  if (!book) {
    throw new NotFoundException(`Book with ID ${id} not found`);
  }
  return book;
}

async create(dto: CreateBookDto) {
  const existing = await this.bookRepository.findByIsbn(dto.isbn);
  if (existing) {
    throw new ConflictException(`ISBN ${dto.isbn} already exists`);
  }
  // ...
}
```

### Swagger 回應標示

使用 `@ApiResponse()` 標示所有可能的回應狀態碼，讓 API 文件更完整：

```typescript
@Get(':id')
@ApiResponse({ status: 200, description: '成功取得書籍', type: BookResponseDto })
@ApiResponse({ status: 404, description: '書籍不存在' })
async findById(@Param('id') id: string) { ... }
```

---

## 動手做

### 必做

**1. 確認狀態碼**

在 `BookApiController` 加上正確的 `@HttpCode()`：

```bash
# POST 新增 → 預期 201
curl -X POST http://localhost:3000/api/books \
     -H "Content-Type: application/json" \
     -d '{"title":"Test","author":"Author","isbn":"999","stock":1}' \
     -w "\nHTTP Status: %{http_code}\n"

# DELETE 刪除 → 預期 204
curl -X DELETE http://localhost:3000/api/books/1 -w "\nHTTP Status: %{http_code}\n"
```

**2. 觸發 400 錯誤**

故意送缺欄位的 JSON（如缺少 `title`），觀察 400 錯誤回應格式：

```bash
curl -X POST http://localhost:3000/api/books \
     -H "Content-Type: application/json" \
     -d '{"author":"Author"}' \
     -w "\nHTTP Status: %{http_code}\n"

# 預期：
# {
#   "statusCode": 400,
#   "timestamp": "...",
#   "path": "/api/books",
#   "message": ["title should not be empty"]
# }
```

**3. 觸發 404 錯誤**

查一筆不存在的 ID：

```bash
curl http://localhost:3000/api/books/99999 -w "\nHTTP Status: %{http_code}\n"

# 預期：
# {
#   "statusCode": 404,
#   "timestamp": "...",
#   "path": "/api/books/99999",
#   "message": ["Book with ID 99999 not found"]
# }
```

**4. 測試表單驗證**

在新增表單（`http://localhost:3000/books/new`）留空所有欄位直接送出，觀察錯誤如何顯示在頁面上。

### 延伸挑戰

1. 自訂一個業務例外 `DuplicateIsbnException`，在 Service 中使用：

```typescript
// common/exceptions/duplicate-isbn.exception.ts
import { ConflictException } from '@nestjs/common';

export class DuplicateIsbnException extends ConflictException {
  constructor(isbn: string) {
    super(`ISBN ${isbn} already exists`);
  }
}

// book.service.ts
async create(dto: CreateBookDto) {
  const existing = await this.bookRepository.findByIsbn(dto.isbn);
  if (existing) {
    throw new DuplicateIsbnException(dto.isbn);
  }
  // ...
}
```

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| DTO 驗證沒生效，錯誤資料直接進 Service | 沒有全域啟用 ValidationPipe | 在 `main.ts` 加入 `app.useGlobalPipes(new ValidationPipe())` |
| ExceptionFilter 沒攔截到錯誤 | Filter 沒有全域註冊 | 在 `main.ts` 加入 `app.useGlobalFilters(new HttpExceptionFilter())` |
| 錯誤訊息不統一 | 不同例外回傳不同格式 | 確保所有例外都經過 ExceptionFilter 統一處理 |
| `@HttpCode()` 沒生效 | 放在錯誤的位置 | 確保 `@HttpCode()` 放在方法上，不是放在類別上 |

---

## 驗收標準（DoD）

- [ ] 能用 Swagger UI 完成完整 CRUD 操作（POST 201、DELETE 204）
- [ ] 能觸發 400 錯誤並看到統一的錯誤回應格式
- [ ] 能觸發 404 錯誤並看到統一的錯誤回應格式
- [ ] 所有錯誤回應有一致的結構（`statusCode`、`timestamp`、`path`、`message`）
- [ ] 能解釋 `@Catch()` 和 `ExceptionFilter` 的作用
