# U03｜API 回傳假資料

> 建議時數：60 min
> 前置依賴：U01

---

## ① 為什麼先教這個？

U02 做了給人看的頁面，這堂課做給機器用的 API。同樣用假資料，讓學員先理解「REST Controller 怎麼回傳 JSON」。有了 API，就能用 Swagger UI 互動式測試，也為後面串接真實 DB 打下基礎。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/book.api.controller.ts` | REST API 端點（回傳 JSON） |
| `src/main.ts` | Swagger / OpenAPI 文件設定 |

---

## ③ 核心觀念

### REST Controller

- 不使用 `@Render()`
- 方法回傳值直接序列化為 JSON
- 與 U02 的 MVC Controller 對比

### 路徑前綴

- `@Controller('api/books')`
- REST API 慣例以 `/api/` 開頭

### RESTful 動詞對應

| 裝飾器 | HTTP 動詞 | 用途 |
|--------|----------|------|
| `@Get()` | GET | 查詢 |
| `@Post()` | POST | 新增 |
| `@Put()` | PUT | 更新 |
| `@Delete()` | DELETE | 刪除 |

### Swagger UI

- 訪問 `/api` 可互動式測試所有 API
- 無需額外工具（如 Postman）

### Swagger 裝飾器

| 裝飾器 | 用途 |
|--------|------|
| `@ApiTags()` | 分組標籤 |
| `@ApiOperation()` | 描述操作 |
| `@ApiResponse()` | 回應狀態碼 |

---

## ④ 動手做

### 必做

1. **建立 GET 列表端點**
   - 寫一個 `GET /api/books` 端點
   - 回傳包含 3 本假書的 JSON 陣列

2. **建立 GET 單筆端點**
   - 寫一個 `GET /api/books/:id` 端點
   - 依 ID 回傳單筆假書（固定回傳同一本即可）

3. **用 Swagger 測試**
   - 打開 `http://localhost:3000/api`
   - 測試剛才寫的 API

### 延伸挑戰

1. 加入 `POST /api/books` 端點
   - 接收 JSON body
   - 用 `console.log` 印出收到的資料

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Swagger 看不到新加的 API | Controller 沒有被 import 到 Module | 確認 `BookModule` 的 `controllers` 陣列包含該 Controller |
| `@Body()` 收到 undefined | Content-Type 不對 | 確認請求的 Content-Type 是 `application/json` |

---

## ⑥ 驗收標準（DoD）

- [ ] 能用 Swagger UI 測試 `GET /api/books`
- [ ] 能用 `curl http://localhost:3000/api/books` 看到 JSON 回應
- [ ] 能說出 REST Controller 與 MVC Controller 的差異
