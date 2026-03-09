# U03｜API 回傳假資料

> 能用 REST Controller 回傳固定 JSON 並在 Swagger 測試 ｜ 60 min ｜ 前置依賴：U01

---

## 為什麼先教這個？

U02 做了給人看的頁面，這堂課做給機器用的 API。同樣用假資料，讓學員先理解「REST Controller 怎麼回傳 JSON」。有了 API，就能用 Swagger UI 互動式測試，也為後面串接真實 DB 打下基礎。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.api.controller.ts` | REST API 端點（回傳 JSON），此刻先用假資料 |
| `main.ts` | Swagger / OpenAPI 文件設定 |

---

## 核心觀念

### REST Controller — 回傳 JSON

與 U02 的 `@Render()` 對比，REST Controller 不使用模板，方法回傳值直接序列化為 JSON：

```typescript
import { Controller, Get, Post, Put, Delete, Param, Body, HttpCode, HttpStatus } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';

@ApiTags('books')  // Swagger 分組標籤
@Controller('api/books')
export class BookApiController {

  @Get()
  @ApiOperation({ summary: '取得所有書籍' })
  @ApiResponse({ status: 200, description: '成功取得書籍清單' })
  findAll() {
    // 假資料——後面 U06 會換成真實 DB 資料
    return [
      { id: 1, title: 'Clean Code', author: 'Robert C. Martin', isbn: '978-0132350884', stock: 5 },
      { id: 2, title: 'Effective TypeScript', author: 'Dan Vanderkam', isbn: '978-1492053743', stock: 3 },
      { id: 3, title: 'NestJS in Action', author: 'Jay Bell', isbn: '978-1617298035', stock: 8 },
    ];
  }

  @Get(':id')
  @ApiOperation({ summary: '取得單本書籍' })
  @ApiResponse({ status: 200, description: '成功取得書籍' })
  @ApiResponse({ status: 404, description: '書籍不存在' })
  findById(@Param('id') id: string) {
    // 固定回傳同一本書（假資料）
    return { id: Number(id), title: 'Clean Code', author: 'Robert C. Martin', isbn: '978-0132350884', stock: 5 };
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)  // 新增成功回 201
  @ApiOperation({ summary: '新增書籍' })
  @ApiResponse({ status: 201, description: '書籍已建立' })
  create(@Body() createBookDto: any) {
    console.log('收到的資料：', createBookDto);
    return { id: 4, ...createBookDto };  // 模擬新增後回傳
  }

  @Put(':id')
  @ApiOperation({ summary: '更新書籍' })
  @ApiResponse({ status: 200, description: '書籍已更新' })
  update(@Param('id') id: string, @Body() updateBookDto: any) {
    return { id: Number(id), ...updateBookDto };
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)  // 刪除成功回 204
  @ApiOperation({ summary: '刪除書籍' })
  @ApiResponse({ status: 204, description: '書籍已刪除' })
  delete(@Param('id') id: string) {
    // 不回傳內容
  }
}
```

### RESTful 動詞對應

| HTTP 動詞 | 裝飾器 | 用途 | 典型回應碼 |
|-----------|--------|------|-----------|
| GET | `@Get()` | 查詢 | 200 OK |
| POST | `@Post()` | 新增 | 201 Created |
| PUT | `@Put()` | 更新（全量） | 200 OK |
| PATCH | `@Patch()` | 更新（部分） | 200 OK |
| DELETE | `@Delete()` | 刪除 | 204 No Content |

### `@Controller('api/books')` — 路徑前綴

統一路徑前綴，REST API 慣例以 `/api/` 開頭，與 MVC 頁面（`/books`）區隔：

- `/api/books` — REST API（JSON）
- `/books` — MVC 頁面（HTML）

### Swagger UI 設定

在 `main.ts` 設定 Swagger：

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Swagger 設定
  const config = new DocumentBuilder()
    .setTitle('書籍管理系統 API')
    .setDescription('書籍 CRUD API 文件')
    .setVersion('1.0')
    .addTag('books')
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);  // 訪問 /api 看 Swagger UI

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

訪問 `http://localhost:3000/api` 就能在瀏覽器中互動式測試 API，不需要安裝 Postman 等工具。

### Swagger 常用裝飾器

| 裝飾器 | 用途 | 範例 |
|--------|------|------|
| `@ApiTags()` | 分組標籤 | `@ApiTags('books')` |
| `@ApiOperation()` | 描述操作 | `@ApiOperation({ summary: '取得所有書籍' })` |
| `@ApiResponse()` | 定義回應 | `@ApiResponse({ status: 200, description: '成功' })` |
| `@ApiProperty()` | DTO 欄位描述 | 在 DTO 類別的屬性上使用 |
| `@ApiBody()` | 請求 Body 描述 | `@ApiBody({ type: CreateBookDto })` |

### 此刻不需要 DB

Controller 直接回傳假的陣列或物件，先專注學 REST 概念：

```typescript
@Get()
findAll() {
  return [
    { id: 1, title: 'Clean Code', author: 'Robert C. Martin' },
    { id: 2, title: 'Effective TypeScript', author: 'Dan Vanderkam' },
  ];
}
```

---

## 動手做

### 必做

**1. 寫一個回傳假書的 API**

建立 `GET /api/books` 端點，回傳包含 3 本假書的 JSON 陣列：

```bash
curl http://localhost:3000/api/books
# 預期：[{"id":1,"title":"Clean Code",...}, ...]
```

**2. 寫一個帶路徑參數的 API**

建立 `GET /api/books/:id` 端點，依 ID 回傳單筆假書（固定回傳同一本即可）：

```bash
curl http://localhost:3000/api/books/1
# 預期：{"id":1,"title":"Clean Code",...}
```

**3. 用 Swagger UI 測試**

打開 `http://localhost:3000/api`，找到剛才寫的 API，點 "Try it out" 執行測試。

### 延伸挑戰

1. 加入 `POST /api/books` 端點，接收 JSON body 並用 `console.log` 印出收到的資料（還沒有 DB，先確認能收到請求）：

```typescript
@Post()
@HttpCode(HttpStatus.CREATED)
create(@Body() createBookDto: any) {
  console.log('收到的資料：', createBookDto);
  return { id: 4, ...createBookDto };
}
```

用 curl 測試：

```bash
curl -X POST http://localhost:3000/api/books \
  -H "Content-Type: application/json" \
  -d '{"title":"New Book","author":"Author Name"}'
```

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| API 回傳 404 | Controller 沒有被 import 到 Module | 確認 `BookModule` 的 `controllers` 陣列包含 `BookApiController` |
| Swagger UI 看不到新加的 API | Controller 沒有註冊到 Module | 確認 Module 的 controllers 陣列；確認 Module 已被 import 到 AppModule |
| `@Body()` 收到的是 undefined | 沒有設定 body parser 或 Content-Type 不對 | NestJS 預設已含 body parser；確認請求的 Content-Type 是 `application/json` |
| Swagger UI 無法訪問 | SwaggerModule 設定錯誤 | 確認 `main.ts` 中的 Swagger 設定正確 |

---

## 驗收標準（DoD）

- [ ] 能用 Swagger UI 或 `curl http://localhost:3000/api/books` 看到 JSON 回應
- [ ] 能用 `GET /api/books/:id` 查詢單筆資料
- [ ] 能說出 `@Controller('api/books')` 的作用——定義路徑前綴
- [ ] 能說出 REST Controller 與 MVC Controller（`@Render()`）的差異
- [ ] 能說出 RESTful 四個 HTTP 動詞對應的操作（GET 查、POST 建、PUT 改、DELETE 刪）
