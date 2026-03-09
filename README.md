# 書籍借閱管理系統

> Node.js 20 / NestJS 11 / TypeScript 5 / TypeORM / Nunjucks / MySQL 8 / Redis 7 / Docker Compose

以「書籍借閱管理」為案例的 NestJS 練習專案，涵蓋常見分層架構：
書籍 CRUD（MySQL 持久化 + Redis 快取）、MVC 頁面與 REST API 並存、TypeORM Migration 資料庫版本管理、啟動預熱與 Redis 過期事件監聽。

## 前置條件

本專案搭配容器化開發環境使用（例如 GitHub Codespaces 或 Dev Container），
容器啟動後 MySQL 與 Redis 服務已就緒，不需另行安裝。

若要在本機直接執行，請自行準備以下服務：

- Node.js 20（建議使用 nvm 管理版本）
- MySQL 8.0（DB: `app`, User: `app`, Password: `app`）
  - JDBC URL: `mysql://localhost:3306/app`
- Redis 7
  - Host: `localhost`, Port: `6379`

## 快速開始

### 使用 Dev Container（推薦）

1. 安裝 VS Code 擴充套件 **Dev Containers**（`ms-vscode-remote.remote-containers`）
2. Clone 專案並用 VS Code 開啟
3. 左下角點選 `><` 圖示 → 選擇 **Reopen in Container**
4. 等待容器建置完成（首次需較長時間），MySQL 與 Redis 會自動啟動並通過 healthcheck
5. 在容器內 Terminal 執行：

```bash
# 安裝依賴
npm install

# 執行 migration（自動建表）
npm run migration:run

# 啟動開發伺服器
npm run start:dev
```

> GitHub Codespaces 使用者：直接在 Codespaces 開啟專案即可，會自動套用 Dev Container 設定。

### 本機直接執行

請先確認[前置條件](#前置條件)中的服務已就緒，再執行：

```bash
git clone <repo-url> /workspaces/nodejs-nestjs-sample
cd /workspaces/nodejs-nestjs-sample

# 安裝依賴
npm install

# 執行 migration（自動建表與塞入範例資料）
npm run migration:run

# 啟動開發伺服器
npm run start:dev
```

啟動成功後驗證：

```bash
curl http://localhost:3000/hello          # Hello, NestJS!
curl http://localhost:3000/api/books      # JSON 陣列，3 筆範例書籍
```

瀏覽器開啟 http://localhost:3000/books 可查看 Nunjucks 書籍清單頁。

## 可用網址

| 網址 | 說明 |
|------|------|
| http://localhost:3000/books | 書籍清單頁（Nunjucks） |
| http://localhost:3000/books/:id | 書籍詳情頁（Nunjucks） |
| http://localhost:3000/api | Swagger UI — 互動式 API 文件與測試 |
| http://localhost:3000/api-json | OpenAPI 3.0 JSON spec |

> Swagger UI 僅顯示 `/api/books` 相關端點，MVC 頁面不會出現。

## API 端點

### REST API

| Method | Path | 說明 |
|--------|------|------|
| GET | `/api/books` | 取得所有書籍 |
| GET | `/api/books/:id` | 取得單本書籍 |
| POST | `/api/books` | 新增書籍 |
| PUT | `/api/books/:id` | 更新書籍 |
| DELETE | `/api/books/:id` | 刪除書籍 |

### MVC 頁面

| Path | 說明 |
|------|------|
| `/books` | 書籍清單頁 |
| `/books/:id` | 書籍詳情頁 |
| `/books/new` | 新增書籍表單頁 |
| `/books/:id/edit` | 編輯書籍表單頁 |

## 專案結構

```
.devcontainer/
└── devcontainer.json              # Dev Container 設定（Codespaces / VS Code 遠端開發）
docker/
└── node/
    └── Dockerfile                 # Node.js 20 開發容器映像
docker-compose.yml                 # 三服務編排：app(bind mount) / db(MySQL 8) / redis(Redis 7)
.env.example                       # 環境變數範本（PORT、DATABASE_URL）

src/
├── main.ts                        # NestJS 應用程式進入點
├── app.module.ts                  # 根模組（AppModule）
├── config/
│   ├── database.config.ts         # TypeORM 連線設定
│   └── redis.config.ts            # Redis 連線與 JSON 序列化設定
├── common/
│   ├── filters/                   # Exception Filter（統一錯誤處理）
│   ├── pipes/                     # Validation Pipe
│   └── interceptors/              # 攔截器
├── book/
│   ├── book.module.ts             # Book 功能模組
│   ├── book.controller.ts         # @Controller — Nunjucks 頁面
│   ├── book.api.controller.ts     # @Controller — JSON API
│   ├── book.service.ts            # Service（Cache-Aside，TTL 10m）
│   ├── book.repository.ts         # TypeORM Repository 擴充
│   ├── entities/
│   │   └── book.entity.ts         # Entity
│   └── dto/
│       ├── create-book.dto.ts     # 請求 DTO（class-validator）
│       └── book-response.dto.ts   # 回應 DTO

views/
├── layouts/
│   └── main.njk                   # 共用版型
└── book/
    ├── list.njk                   # 書籍清單頁
    ├── detail.njk                 # 書籍詳情頁
    └── form.njk                   # 新增/編輯表單頁

migrations/
├── 1700000000000-CreateBooksTable.ts
└── 1700000000001-InsertSampleData.ts
```

## 設定檔與環境變數

| 環境變數 | 預設值 | 說明 |
|----------|--------|------|
| `PORT` | 3000 | 應用程式監聽 Port |
| `DATABASE_HOST` | localhost | MySQL 主機位址 |
| `DATABASE_PORT` | 3306 | MySQL Port |
| `DATABASE_USER` | app | MySQL 使用者 |
| `DATABASE_PASSWORD` | app | MySQL 密碼 |
| `DATABASE_NAME` | app | MySQL 資料庫名稱 |
| `REDIS_HOST` | localhost | Redis 主機位址 |
| `REDIS_PORT` | 6379 | Redis Port |

容器環境已透過 `.env` 或 `docker-compose.yml` 設定，不需手動調整。
本機開發時複製 `.env.example` 為 `.env` 後修改即可。

### 容器服務與 Port 對應

| 服務 | 映像 | 容器內 Port | 預設對外 Port |
|------|------|-------------|--------------|
| app | `docker/node/Dockerfile` | 3000 | 3000 |
| db | `mysql:8.0` | 3306 | 3306 |
| redis | `redis:7-alpine` | 6379 | 6379 |

## 開發

### Hot Reload 模式

```bash
npm run start:dev
```

NestJS 會自動監聽檔案變更並重新編譯。

### 測試

```bash
npm run test                       # 執行單元測試
npm run test:e2e                   # 執行 e2e 測試
npm run test:cov                   # 產生覆蓋率報告
```

| 測試類 | 類型 | 說明 |
|--------|------|------|
| **book.service.spec.ts** | Jest 單元測試 | 使用 Mock 隔離外部依賴 |
| **book.api.controller.spec.ts** | Jest 單元測試 | Controller 單元測試 |
| **app.e2e-spec.ts** | e2e 測試 | 完整 HTTP 請求測試 |

### Migration 指令

```bash
npm run migration:generate -- -n MigrationName  # 產生新 migration
npm run migration:run                            # 執行所有 pending migration
npm run migration:revert                         # 回滾最後一次 migration
```

## 教案文件

- [NestJS 新手入門教案（導覽）](docs/新手入門教案/README.md)
- [NestJS 新手入門教案綱要](docs/新手入門教案/NestJS新手入門教案綱要.md)
- [U01 — 專案啟動與第一支 API](docs/新手入門教案/01-專案啟動與第一支API.md)
- [U02 — 假畫面與 Nunjucks 初體驗](docs/新手入門教案/02-假畫面與Nunjucks初體驗.md)
- [U03 — API 回傳假資料](docs/新手入門教案/03-API回傳假資料.md)
- [U04 — 設定檔與多環境切換](docs/新手入門教案/04-設定檔與多環境切換.md)
- [U05 — TypeORM Migration 與資料庫設計](docs/新手入門教案/05-TypeORM-Migration與資料庫設計.md)
- [U06 — TypeORM 串接資料庫](docs/新手入門教案/06-TypeORM串接資料庫.md)
- [U07 — 正規化架構與 Service 分層](docs/新手入門教案/07-正規化架構與Service分層.md)
- [U08 — REST API 完善與錯誤處理](docs/新手入門教案/08-REST-API完善與錯誤處理.md)
- [U09 — Redis 快取整合](docs/新手入門教案/09-Redis快取整合.md)
- [U10 — 測試策略與實戰](docs/新手入門教案/10-測試策略與實戰.md)
