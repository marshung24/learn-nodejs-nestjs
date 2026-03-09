# U07｜正規化架構 — Service 與 DTO 分層

> 建議時數：90 min
> 前置依賴：U06

---

## ① 為什麼先教這個？

U06 完成了 CRUD，但 Controller 直接操作 Repository——所有邏輯擠在一起，改一個地方可能影響全部。這堂課把程式碼「整理乾淨」：抽出 Service 層集中業務邏輯，引入 DTO 分離輸入/輸出格式。**重構前後功能完全一樣，但程式碼更好維護、更好測試。**

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/book.service.ts` | Service 層（業務邏輯） |
| `src/book/dto/create-book.dto.ts` | 輸入驗證 DTO |
| `src/book/dto/book-response.dto.ts` | 輸出格式 DTO |

---

## ③ 核心觀念

### Service 層職責

- 集中業務規則（如檢查重複 ISBN）
- Controller 只負責 HTTP 協議

### `@Injectable()`

- 標記類別可被 DI 容器管理

### Entity 與 DTO 分離

| 類別 | 職責 |
|------|------|
| `Book` (Entity) | 對應 DB |
| `CreateBookDto` | 對應「收什麼」（輸入） |
| `BookResponseDto` | 對應「給什麼」（輸出） |

### class-validator

| 裝飾器 | 用途 |
|--------|------|
| `@IsNotEmpty()` | 不可為空 |
| `@IsString()` | 必須是字串 |
| `@Min(0)` | 最小值 |
| `@IsOptional()` | 可選欄位 |

### class-transformer

- `plainToInstance()` / `instanceToPlain()` 做 Entity ↔ DTO 轉換

### PRG 模式（Post-Redirect-Get）

- MVC 表單送出後用 redirect
- 避免使用者按 F5 重複提交

---

## ④ 動手做

### 必做

1. **閱讀 Service**
   - 閱讀 `book.service.ts`
   - 列出所有方法並說明用途

2. **追蹤資料流**
   - 追蹤 `create()` 的完整流程：
   - DTO → Entity → save → ResponseDto

3. **重構 Controller**
   - 把直接操作 Repository 的程式碼改為呼叫 Service
   - 確認頁面和 API 行為不變

### 延伸挑戰

1. 在 Service 加入 `findByIsbn()` 方法（`⟵ 需完成 U06 必做 3`）
2. 畫出 CreateDto → Entity → ResponseDto 的資料流向圖

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Service 注入時噴錯 | Service 沒有在 Module 的 providers 註冊 | 在 `book.module.ts` 的 providers 加入 |
| DTO 驗證沒生效 | 沒有全域啟用 ValidationPipe | 在 `main.ts` 加入 `app.useGlobalPipes(new ValidationPipe())` |

---

## ⑥ 驗收標準（DoD）

- [ ] 能說明 Service 層存在的理由
- [ ] 能解釋 Entity/CreateDto/ResponseDto 三者的職責差異
- [ ] 頁面和 API 行為與重構前完全一致
