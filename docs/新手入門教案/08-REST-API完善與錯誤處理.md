# U08｜REST API 完善與錯誤處理

> 建議時數：90 min
> 前置依賴：U07

---

## ① 為什麼先教這個？

U07 完成了分層，但 API 還缺少正確的 HTTP 狀態碼和統一的錯誤回應。使用者送了錯誤資料、查了不存在的 ID，應該要拿到清楚的錯誤訊息而非一堆 stack trace。這堂課把 API 做到「生產品質」。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/book.api.controller.ts` | REST API 端點 |
| `src/common/filters/http-exception.filter.ts` | 全域錯誤處理 |
| `src/common/pipes/validation.pipe.ts` | 全域驗證 Pipe |

---

## ③ 核心觀念

### HTTP 狀態碼語意

| 狀態碼 | 意義 | 使用場景 |
|--------|------|----------|
| 200 OK | 成功 | GET、PUT 成功 |
| 201 Created | 已建立 | POST 成功 |
| 204 No Content | 無內容 | DELETE 成功 |
| 400 Bad Request | 請求錯誤 | 驗證失敗 |
| 404 Not Found | 找不到 | 資源不存在 |
| 500 Internal Server Error | 伺服器錯誤 | 未預期的錯誤 |

### `@HttpCode()`

- 指定回應狀態碼
- 預設是 200，POST 成功應改為 201

### ValidationPipe

- 自動觸發 DTO 上的 class-validator
- 驗證失敗回 400

### ExceptionFilter

- 攔截所有 Controller 拋出的例外
- 統一回應格式

### Swagger 回應標示

- `@ApiResponse()` 標示可能的回應狀態碼

---

## ④ 動手做

### 必做

1. **加入正確狀態碼**
   - POST 加上 `@HttpCode(HttpStatus.CREATED)`
   - DELETE 加上 `@HttpCode(HttpStatus.NO_CONTENT)`
   - 確認 POST 回 201、DELETE 回 204

2. **測試 400 錯誤**
   - 故意送缺欄位的 JSON（如缺少 `title`）
   - 觀察 400 錯誤回應格式

3. **測試 404 錯誤**
   - 查一筆不存在的 ID（如 `GET /api/books/99999`）
   - 觀察 404 錯誤回應

4. **測試表單驗證**
   - 在新增表單留空送出
   - 觀察 Nunjucks 驗證錯誤如何顯示

### 延伸挑戰

1. 自訂一個業務例外 `DuplicateIsbnException`
   - 在 ExceptionFilter 加入對應處理
   - 回傳 409 Conflict

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| DTO 驗證沒生效 | Controller 沒有套用 ValidationPipe | 確認全域或該 Controller 有 `@UsePipes(ValidationPipe)` |
| ExceptionFilter 沒攔截到錯誤 | Filter 沒有全域註冊 | 在 `main.ts` 加入 `app.useGlobalFilters()` |

---

## ⑥ 驗收標準（DoD）

- [ ] POST 回 201、DELETE 回 204
- [ ] 能觸發 400 和 404 錯誤
- [ ] 所有錯誤回應有統一格式
