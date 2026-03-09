# U01｜專案啟動與第一支 API

> 建議時數：90 min
> 前置依賴：—

---

## ① 為什麼先教這個？

讓學員在第一堂課就看到「程式跑起來」的成就感。沒有什麼比親手讓一個 Web 應用回應你的請求更能建立學習信心。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `package.json` | 依賴管理與腳本設定 |
| `src/main.ts` | NestJS 應用程式進入點 |
| `src/app.module.ts` | 根模組（AppModule） |
| `src/app.controller.ts` | 最簡易的 REST 端點 |

---

## ③ 核心觀念

### 專案標準目錄架構

```
src/           # 程式碼
views/         # 模板（Nunjucks）
test/          # 測試
migrations/    # 資料庫 migration
```

### npm 做了什麼

- 下載依賴、執行腳本
- `npm run start:dev` 是最常用的開發指令（hot reload）

### NestJS 核心概念

- **Module**：組織功能的容器（`@Module()`）
- **Controller**：處理 HTTP 請求（`@Controller()`）
- **Provider/Service**：業務邏輯（`@Injectable()`）

### 依賴注入（DI）

- NestJS 的核心機制
- 在 constructor 宣告依賴，框架自動注入實例

### 裝飾器與路由

- `@Controller()` 定義路由前綴
- `@Get()`, `@Post()` 等定義 HTTP 方法
- 回傳物件自動序列化為 JSON

---

## ④ 動手做

### 必做

1. **啟動專案**
   - 執行 `docker compose up -d` 啟動 MySQL 與 Redis
   - 執行 `npm run start:dev`
   - 瀏覽器打開 `http://localhost:3000/hello`，確認看到回應

2. **新增自訂端點**
   - 在 AppController 新增一個 `/whoami` 端點
   - 回傳 `{ name: '你的名字' }`

### 延伸挑戰

1. 觀察 `package.json` 的 dependencies，寫下每個套件的用途

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `start:dev` 後報 port 衝突 | Port 3000 被占用 | `lsof -i :3000` 找出占用程式並關閉；或在 `.env` 改 `PORT` |
| 修改後瀏覽器沒變化 | 等待 hot reload 重新編譯 | 注意看 console 的 restart 訊息 |

---

## ⑥ 驗收標準（DoD）

- [ ] 能用瀏覽器訪問 `http://localhost:3000/hello` 看到回應
- [ ] 能用 `curl http://localhost:3000/whoami` 看到自訂的 JSON 回應
- [ ] 能說出 Module、Controller、Provider 的關係
