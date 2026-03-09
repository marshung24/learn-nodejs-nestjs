# NestJS 新手入門教案綱要

> 以本 Repo「書籍管理系統」為實作主軸，從零帶出 NestJS 全棧開發核心技能。
>
> **給教案作者的提醒**：本綱要為「骨架級」指導文件，內容刻意詳盡以確保教案撰寫時不遺漏關鍵要素。依此綱要寫出的**教案本身應力求精練**——每個單元的講義以「此刻實作所需的最小可行知識」為上限，避免資訊過載。
>
> **單元排列原則**：遵循「對應專案流程的漸進式推演」——教案對應專案推進流程（需求分析 → 討論 → 拆解 → 規劃 → 設計 → 開發 → 測試），在開發階段以「可視性」為脈絡，先讓學員看到畫面（假資料），再串接真實資料庫完成 CRUD 循環，最後才抽出 Service/DTO 分層與進階功能。每一步都有可見的產出，避免「寫了半天什麼都看不到」的挫折。

---

## §1 教案資訊

| 項目 | 說明 |
|------|------|
| 對象 | 有基礎程式概念（變數、迴圈、函式）與基本命令列操作經驗，但**尚未做過 Node.js Web 專案**的初學者 |
| 目標 | 獨立完成一個「能跑、能測、能部署」的 CRUD Web 應用（書籍管理系統）；所有單元的成果最終匯聚為同一個可動可用的範例網站 |
| 技術棧 | Node.js 20、NestJS 11、TypeScript 5、TypeORM、MySQL 8、Redis 7、Nunjucks、@nestjs/swagger（Swagger UI）、Docker Compose、Jest |
| 時數 | 課前準備（自學）+ 10 單元（各 60–120 分鐘，詳見 §5 地圖層） |
| 教學方式 | 觀念講解 → Repo 實作示範 → 學員動手做 → Code Review → 驗收 |
| 先備知識 | 基礎 SQL（SELECT / INSERT）、HTTP 概念（GET / POST）、Git 基本操作（clone / commit）、JavaScript 基礎語法 |
| 不含範圍 | 前端框架（React / Vue / Angular）、Passport.js 進階認證、雲端進階架構（K8s / Terraform）、微服務拆分、GraphQL |

---

## §2 學習成果（結訓時能做到）

1. **能說明** NestJS 分層架構（Controller → Service → Repository / Provider → Entity / DTO）各層的職責與呼叫方向，理解 Module 與 DI 的作用
2. **能使用** npm 建置專案、用 Docker Compose 一鍵啟動整套開發環境（App + MySQL + Redis）
3. **能撰寫** TypeORM migration 腳本管理資料庫 schema 演進，理解 migration 版本化與回滾機制
4. **能實作** TypeORM Repository 查詢層，新增自訂查詢方法
5. **能設計** RESTful API（正確使用 HTTP 動詞與狀態碼），用 Exception Filter / Validation Pipe 統一錯誤回應與驗證，並產出 Swagger 互動式文件
6. **能製作** NestJS + Nunjucks 伺服器端渲染頁面（列表、表單、詳情），理解 PRG 模式
7. **能實作** Redis Cache-Aside 快取模式，並用 redis-cli 驗證快取命中與失效機制
8. **能撰寫** Jest 單元測試與 e2e 測試，驗證 Controller、Service 與 API 行為

---

## §3 閱讀指引

### 練習標記說明

| 標記 | 意義 | 使用場景 |
|------|------|----------|
| **必做** | 課堂內所有學員都應完成的核心練習 | 達成該單元最低驗收門檻 |
| **延伸挑戰** | 給進度快的學員或課後作業 | 加深理解、跨單元整合 |

### 依賴標記說明

- 標有 `⟵ 需完成 UXX` 的項目，表示需先完成指定單元的對應練習才能進行
- 標有 `⟵ 需完成 UXX 延伸` 的項目，特指依賴某單元的「延伸挑戰」成果

### 命名規範

| 項目 | 格式 | 範例 |
|------|------|------|
| 單元編號 | U00–U10 | U03 |
| 檔案路徑 | 省略共同前綴 `src/`，保持可讀性 | `book/book.controller.ts` |
| 視圖路徑 | 省略 `views/` | `book/list.njk` |
| 測試路徑 | 省略 `src/` 或 `test/` | `book/book.service.spec.ts` |

### 單元內部結構

每個單元依序包含六個子區塊：① 為什麼先教這個？ → ② 對應檔案 → ③ 核心觀念 → ④ 動手做 → ⑤ 踩坑提示 → ⑥ 驗收標準（DoD）

---

## §4 課前準備（U00）

> 本單元為課前自修，不佔正課時間。目標：確保第一堂課不卡在環境問題上。

### 安裝清單

| 工具 | 版本 | 推薦安裝方式（macOS） | Windows WSL 注意事項 |
|------|------|----------------------|---------------------|
| Node.js 20 | 20.x LTS | [nvm](https://github.com/nvm-sh/nvm)：`nvm install 20` | WSL2 中同樣使用 nvm 安裝；參考 [Microsoft WSL 安裝指南](https://learn.microsoft.com/zh-tw/windows/wsl/install) |
| Docker Desktop | 最新版 | [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/) | 安裝 [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/) 並啟用 WSL2 後端整合 |
| Git | 最新版 | `brew install git` 或 Xcode Command Line Tools | WSL2 內建或 `sudo apt install git` |
| IDE | — | **VS Code** + [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) + [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) | 同 macOS；可搭配 [Remote - WSL 擴充](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) |

> **作業系統說明**：本教案以 **macOS** 為主要示範環境。Windows 使用者建議透過 **WSL2**（Windows Subsystem for Linux）進行開發，可獲得與 macOS/Linux 一致的命令列體驗。

### 驗證指令

```bash
node -v                          # 確認顯示 v20.x
npm -v                           # 確認 npm 可用
docker compose version           # 確認 Docker Compose 可用
git clone <本 Repo URL> && cd nodejs-nestjs-sample
npm install                      # 確認能下載依賴
```

### 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `docker: command not found` | Docker Desktop 未啟動或未安裝 | macOS / Windows 需先啟動 Docker Desktop 應用程式 |
| npm install 卡住或逾時 | 公司網路可能封鎖 npm registry | 設定 npm proxy 或切換至個人網路；或使用 `npm config set registry https://registry.npmmirror.com` |
| `node -v` 顯示非 20 | 系統有多版本 Node.js | 用 nvm `nvm use 20` 或設定 `.nvmrc` |

---

## §5 課程模組

### 地圖層（單元總覽）

| 單元 | 名稱 | 時數 | 一句話目標 | 前置依賴 |
|------|------|------|-----------|----------|
| U01 | 專案啟動與第一支 API | 90 min | 能啟動專案並用瀏覽器呼叫自訂端點 | — |
| U02 | 假畫面 — Nunjucks 初體驗 | 60 min | 能用 Nunjucks 做出一個書籍清單頁面（假資料） | U01 |
| U03 | API 回傳假資料 | 60 min | 能用 REST Controller 回傳固定 JSON 並在 Swagger 測試 | U01 |
| U04 | 設定檔與多環境切換 | 60 min | 能解釋 ConfigModule 機制並用 Docker Compose 啟動完整環境 | U01 |
| U05 | TypeORM Migration 與資料庫設計 | 90 min | 能撰寫 migration 並在啟動時自動建表與塞入資料 | U04 |
| U06 | TypeORM 串接資料庫 | 120 min | 能用 TypeORM 完成真實 CRUD，頁面與 API 顯示 DB 真實資料 | U02, U03, U05 |
| U07 | 正規化架構 — Service 與 DTO 分層 | 90 min | 能從 Controller 抽出 Service 層，引入 DTO 做資料驗證與格式轉換 | U06 |
| U08 | REST API 完善與錯誤處理 | 90 min | 能用 Exception Filter 統一錯誤回應，正確使用 HTTP 狀態碼 | U07 |
| U09 | Redis 快取整合 | 90 min | 能用 redis-cli 驗證快取行為並說明 Cache-Aside 流程 | U07 |
| U10 | 測試策略與實戰 | 120 min | 能為新方法撰寫單元測試與 e2e 測試 | U08 |

> **設計說明**：單元順序遵循「對應專案流程的漸進式推演」——對應開發階段的可視性脈絡：環境確認（U01）→ 假畫面 Level-1 UI（U02）→ 假資料 Level-2 程式（U03）→ 設定環境（U04）→ 資料庫設計 Level-3 資料（U05）→ 串接真實 DB（U06）→ 架構正規化（U07）→ 完善 API（U08）→ 效能優化（U09）→ 測試驗證（U10）。學員在 U02 就能在瀏覽器看到書籍清單頁面，U06 結束時已有一個從 DB 讀寫的完整 CRUD 網站，之後才逐步重構為正規架構。

---

### U01｜專案啟動與第一支 API（90 min）

#### ① 為什麼先教這個？

讓學員在第一堂課就看到「程式跑起來」的成就感。沒有什麼比親手讓一個 Web 應用回應你的請求更能建立學習信心。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `package.json` | 依賴管理與腳本設定（定義專案用到的所有套件） |
| `main.ts` | NestJS 應用程式進入點 |
| `app.module.ts` | 根模組（AppModule），組織所有功能模組 |
| `app.controller.ts` | 最簡易的 REST 端點，示範如何回應 HTTP 請求 |

#### ③ 核心觀念

- **專案標準目錄架構** — `src/`（程式碼）、`views/`（模板）、`test/`（測試），三區各司其職
- **npm 做了什麼** — 下載依賴、執行腳本；`npm run start:dev` 是最常用的開發指令
- **NestJS 核心概念** — Module（模組）、Controller（控制器）、Provider/Service（服務）三者構成基本架構
- **依賴注入（DI）** — NestJS 的核心機制，在 constructor 宣告依賴，框架自動注入實例
- **`@Controller()` 與 `@Get()`** — 裝飾器定義路由；回傳物件自動序列化為 JSON

#### ④ 動手做

**必做**
1. 執行 `docker compose up -d` 啟動 MySQL 與 Redis，再執行 `npm run start:dev`，瀏覽器打開 `http://localhost:3000/hello`，確認看到回應
2. 在 AppController 新增一個 `/whoami` 端點，回傳自己的名字（JSON 格式 `{ name: 'xxx' }`）

**延伸挑戰**
1. 觀察 `package.json` 的 dependencies，寫下每個套件的用途（如 `@nestjs/core` 提供什麼功能）

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `start:dev` 後卡住或報 port 衝突 | Port 3000 被其他程式占用 | macOS：`lsof -i :3000` 找出占用程式並關閉；或在 `.env` 改 `PORT` |
| 修改 TypeScript 後瀏覽器沒變化 | 需等待 hot reload 重新編譯 | 注意看 console log 的 restart 訊息，等重啟完成再刷新 |

#### ⑥ 驗收標準（DoD）

能用瀏覽器或 `curl http://localhost:3000/whoami` 呼叫自訂端點並看到正確的 JSON 回應。

---

### U02｜假畫面 — Nunjucks 初體驗（60 min）

#### ① 為什麼先教這個？

U01 讓程式跑起來了，但只有文字回應。這堂課讓學員第一次「看到畫面」——用 Nunjucks 做出一個有書籍清單的 HTML 頁面。資料先用 Controller 中的假資料（hardcoded），不需要 DB，**重點是讓學員體驗「改了模板，刷新就看到結果」的即時回饋感**。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.controller.ts` | MVC 端點（回傳 HTML 頁面），此刻先用假資料 |
| `views/book/list.njk` | 書籍清單頁 |
| `views/book/detail.njk` | 書籍詳情頁 |
| `views/book/form.njk` | 新增 / 編輯表單頁 |
| `views/layouts/main.njk` | 共用版型 |

#### ③ 核心觀念

- **伺服器端渲染（SSR）** — Nunjucks 在 server 端組好完整 HTML 再送出，不需要前端框架
- **`@Render()` 裝飾器** — 指定要渲染的模板名稱
- **傳遞資料給模板** — 回傳物件會自動注入模板上下文
- **Nunjucks 關鍵語法**：
  - `{% for item in items %}` — 迴圈渲染
  - `{{ variable }}` — 輸出變數（自動 XSS 防護）
  - `{% extends 'layouts/main.njk' %}` — 繼承版型
  - `{% block content %}` — 區塊覆寫
- **此刻不需要 DB** — Controller 直接建立假的書籍清單丟給模板，先專注學 Nunjucks 語法

> **講師提示**：先 demo 假資料版本，讓學員專心學 Nunjucks 語法。後面 U06 串接 DB 時，只要改 Controller 的資料來源，頁面不用動——這就是模板引擎的價值。

#### ④ 動手做

**必做**
1. 觀察 `BookController.list()` 如何回傳資料，對照 `list.njk` 中的 `{% for %}` 如何渲染每一行
2. 寫一個簡易版 Controller 方法，回傳一個包含 3 本假書的清單，瀏覽 `http://localhost:3000/books` 看到表格
3. 點進某本書的連結，觀察詳情頁如何用 `{{ book.title }}` 顯示各欄位

**延伸挑戰**
1. 修改 `list.njk`，在表格加一欄「狀態」，當庫存 = 0 時顯示紅字「缺貨」（提示：`{% if %}` + style）

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Nunjucks 回 500 找不到模板 | Controller 的 `@Render()` 指定的模板路徑錯誤 | 確認 `views/` 下的資料夾名稱與回傳字串完全一致 |
| 頁面顯示 `{{ books }}` 原始文字 | 檔案副檔名不是 `.njk` 或 Nunjucks 未正確設定 | 確認 `main.ts` 中的 view engine 設定 |

#### ⑥ 驗收標準（DoD）

能在瀏覽器看到一個有書籍清單的 HTML 頁面（即使資料是假的），並能說出 `{% for %}` 和 `{{ }}` 的用途。

---

### U03｜API 回傳假資料（60 min）

#### ① 為什麼先教這個？

U02 做了給人看的頁面，這堂課做給機器用的 API。同樣用假資料，讓學員先理解「REST Controller 怎麼回傳 JSON」。有了 API，就能用 Swagger UI 互動式測試，也為後面串接真實 DB 打下基礎。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.api.controller.ts` | REST API 端點（回傳 JSON），此刻先用假資料 |
| `main.ts` | Swagger / OpenAPI 文件設定 |

#### ③ 核心觀念

- **REST Controller** — 不使用 `@Render()`，方法回傳值直接序列化為 JSON，與 U02 的 MVC Controller 對比
- **路徑前綴 `@Controller('api/books')`** — 統一路徑前綴，REST API 慣例以 `/api/` 開頭
- **RESTful 動詞對應** — `@Get()`（查）、`@Post()`（建）、`@Put()`（改）、`@Delete()`（刪）
- **Swagger UI** — 訪問 `/api` 可互動式測試所有 API，無需額外工具
- **`@ApiTags()` 與 `@ApiOperation()`** — 為 Swagger 文件添加描述

#### ④ 動手做

**必做**
1. 寫一個 `GET /api/books` 端點，回傳包含 3 本假書的 JSON 陣列
2. 寫一個 `GET /api/books/:id` 端點，依 ID 回傳單筆假書（固定回傳同一本即可）
3. 打開 Swagger UI（`http://localhost:3000/api`），測試剛才寫的 API

**延伸挑戰**
1. 加入 `POST /api/books` 端點，接收 JSON body 並用 `console.log` 印出收到的資料（還沒有 DB，先確認能收到請求）

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Swagger UI 看不到新加的 API | Controller 沒有被 import 到 Module | 確認 `BookModule` 的 `controllers` 陣列包含該 Controller |
| `@Body()` 收到的是 undefined | 沒有設定 body parser 或 Content-Type 不對 | NestJS 預設已含 body parser；確認請求的 Content-Type 是 `application/json` |

#### ⑥ 驗收標準（DoD）

能用 Swagger UI 或 `curl http://localhost:3000/api/books` 看到 JSON 回應，並能說出 REST Controller 與 MVC Controller 的差異。

---

### U04｜設定檔與多環境切換（60 min）

#### ① 為什麼先教這個？

接下來要串接真實 DB（U05-U06），得先搞懂「DB 連線設定在哪、怎麼改、怎麼切環境」。不懂設定管理，後面 MySQL 連不上就會卡住。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `.env` / `.env.example` | 環境變數檔 |
| `config/database.config.ts` | TypeORM 連線設定 |
| `config/redis.config.ts` | Redis 連線設定 |
| `app.module.ts` | ConfigModule 載入設定 |
| `docker-compose.yml` | 多容器編排（App + MySQL + Redis） |

#### ③ 核心觀念

- **`.env` 檔與環境變數** — 用 `@nestjs/config` 的 `ConfigModule` 載入
- **`ConfigService`** — 依賴注入取得設定值，避免直接讀取 `process.env`
- **設定優先順序** — 環境變數 > `.env` 檔 > 預設值
- **Docker Compose 一鍵啟動** — 一個 `docker compose up -d` 拉起 App + MySQL + Redis 三個容器

> **開發方式補充**：本教案以 **Docker Compose** 為主要開發環境方案。其他常見方式包括：
> - 本機直接安裝 MySQL + Redis
> - [DevContainer](https://code.visualstudio.com/docs/devcontainers/containers)（VS Code 整合容器開發環境）

#### ④ 動手做

**必做**
1. 執行 `docker compose up -d`，用 `docker compose ps` 確認三個容器都在 running 狀態
2. 閱讀 `database.config.ts`，說明如何從 `ConfigService` 取得設定值

**延伸挑戰**
1. 新增一個自訂環境變數 `APP_NAME`，在 Controller 中用 `ConfigService` 讀取並回傳

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `ConfigService.get()` 回傳 undefined | `.env` 檔放錯位置或變數名稱打錯 | 確認 `.env` 在專案根目錄，變數名稱完全一致 |
| `docker compose up` 後 App 啟動失敗 | MySQL 容器尚未 ready | 等 healthcheck 通過後重試，或查看 `docker compose logs app` |

#### ⑥ 驗收標準（DoD）

能解釋 ConfigModule 的設定載入機制，並能成功用 Docker Compose 啟動完整環境。

---

### U05｜TypeORM Migration 與資料庫設計（90 min）

#### ① 為什麼先教這個？

U02-U03 用的是假資料，接下來要換成真的。第一步是「設計資料庫結構」——用 TypeORM Migration 管理 schema 版本，確保多人協作時每個人的 DB 長一樣。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `migrations/1700000000000-CreateBooksTable.ts` | 建表（初始 schema） |
| `migrations/1700000000001-InsertSampleData.ts` | 塞入範例資料（seed data） |
| `data-source.ts` | TypeORM CLI 使用的資料來源設定 |

#### ③ 核心觀念

- **Migration 機制** — 用 TypeScript 撰寫 `up()` 和 `down()` 方法，分別執行升級與回滾
- **時間戳命名** — `{timestamp}-{描述}.ts`，確保執行順序
- **`migrations` 表** — TypeORM 在 DB 中自動建立的紀錄表，記錄每次 migration 的執行狀態
- **不可修改已執行的 migration** — 已執行的 migration 不應修改，需用新 migration 修補
- **欄位設計思考** — 為什麼用 `BIGINT` 不用 `INT`？為什麼 `isbn` 加 `UNIQUE`？

#### ④ 動手做

**必做**
1. 閱讀現有 migration 檔案，說出每個欄位的設計理由
2. 執行 `npm run migration:run`，確認 migration 成功執行
3. 新增 `{timestamp}-AddPublisherColumn.ts`，為 books 表加一個 `publisher VARCHAR(100)` 欄位
4. 再次執行 migration，連線 MySQL 查詢 `SELECT * FROM migrations` 確認紀錄

**延伸挑戰**
1. 執行 `npm run migration:revert` 回滾最後一次 migration，觀察 `down()` 方法的效果

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Migration 指令報錯找不到 data-source | `data-source.ts` 路徑設定錯誤 | 確認 `package.json` 中的 migration 腳本指向正確路徑 |
| Migration 執行後 DB 沒變化 | 該 migration 已執行過 | 檢查 `migrations` 表，已執行的不會重跑 |

#### ⑥ 驗收標準（DoD）

能撰寫一支新的 migration 並在執行後看到資料表結構變更，且能在 `migrations` 表確認紀錄。

---

### U06｜TypeORM 串接資料庫（120 min）

#### ① 為什麼先教這個？

U05 設計好了資料表，這堂課要把假資料換成真的——用 TypeORM 串接 MySQL，讓 Controller 直接從 DB 讀寫資料。**完成後學員會第一次看到「頁面上的資料來自真實資料庫」的完整 CRUD 循環。** 此刻先不抽 Service 層，Controller 直接操作 Repository——保持最短路徑讓 CRUD 跑起來。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/entities/book.entity.ts` | DB 實體（對應 books 表） |
| `book/book.repository.ts` | Repository 擴充（自訂查詢方法） |
| `book/book.module.ts` | 模組設定（注入 Repository） |

#### ③ 核心觀念

- **Entity 裝飾器** — `@Entity()`、`@Column()`、`@PrimaryGeneratedColumn()` 定義表結構
- **Repository Pattern** — `@InjectRepository()` 注入 Repository，使用內建 CRUD 方法
- **自訂 Repository 方法** — 繼承 `Repository<Entity>` 或使用 `createQueryBuilder()`
- **`snake_case` 與 `camelCase`** — TypeORM 可設定自動轉換命名風格
- **替換假資料** — 修改 U02/U03 的 Controller，把 hardcoded list 改成呼叫 `repository.find()`，頁面立刻顯示真實 DB 資料

> **講師提示**：這是課程的第一個「啊哈時刻」——學員親眼看到頁面從假資料切換到真實 DB 資料。建議 live demo 時先秀假資料頁面，再切換到 Repository 呼叫，重新整理瀏覽器的瞬間最有衝擊力。

#### ④ 動手做

**必做**
1. 閱讀 `book.entity.ts`，對照每個 `@Column()` 與資料表欄位的對應關係
2. 修改 `BookController`，把 `list()` 方法中的假資料替換為 `this.bookRepository.find()`，重新整理頁面確認顯示 DB 資料
3. 在 Repository 新增 `findByIsbn(isbn: string)` 方法，執行 `npm run start:dev` 確認編譯通過

**延伸挑戰**
1. 新增一個多條件查詢方法（如依作者 + 庫存範圍），體驗 QueryBuilder 的用法

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Repository 注入時拋 `Nest could not find...` 錯誤 | Module 沒有 import `TypeOrmModule.forFeature([Book])` | 在 `book.module.ts` 的 imports 加入 |
| Entity 欄位與 DB 不一致 | Entity 定義與 migration 建立的表結構不符 | 對照 migration 的欄位定義修正 Entity |

#### ⑥ 驗收標準（DoD）

能在瀏覽器看到來自 DB 的書籍清單（不再是假資料），並能新增一個 Repository 方法。

---

### U07｜正規化架構 — Service 與 DTO 分層（90 min）

#### ① 為什麼先教這個？

U06 完成了 CRUD，但 Controller 直接操作 Repository——所有邏輯擠在一起，改一個地方可能影響全部。這堂課把程式碼「整理乾淨」：抽出 Service 層集中業務邏輯，引入 DTO 分離輸入/輸出格式。**重構前後功能完全一樣，但程式碼更好維護、更好測試。**

> **講師提示**：先展示重構前（Controller 直接操作 Repository）和重構後（Controller → Service → Repository）的對比，讓學員感受到「程式碼可以一樣能跑，但結構差很多」。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.service.ts` | Service 層（業務邏輯，此刻先不含快取） |
| `book/dto/create-book.dto.ts` | 輸入驗證 DTO（新增/更新用） |
| `book/dto/book-response.dto.ts` | 輸出格式 DTO（API 回傳用） |

#### ③ 核心觀念

- **Service 層職責** — 集中業務規則（如檢查重複 ISBN），Controller 只負責 HTTP 協議
- **`@Injectable()`** — 標記類別可被 DI 容器管理
- **Entity 與 DTO 分離** — `Book` 對應 DB，`CreateBookDto` 對應「收什麼」，`BookResponseDto` 對應「給什麼」
- **class-validator** — `@IsNotEmpty()`、`@IsString()`、`@Min()` — 在 DTO 欄位上宣告驗證規則
- **class-transformer** — `plainToInstance()` / `instanceToPlain()` 做 Entity ↔ DTO 轉換
- **PRG 模式（Post-Redirect-Get）** — MVC 表單送出後用 redirect 避免使用者按 F5 重複提交

#### ④ 動手做

**必做**
1. 閱讀 `book.service.ts`，列出所有方法並說明每個方法的用途
2. 追蹤 `create()` 的完整流程：DTO → Entity → save → 回傳 ResponseDto
3. 重構 Controller：把直接操作 Repository 的程式碼改為呼叫 Service，確認頁面和 API 行為不變

**延伸挑戰**
1. 在 Service 加入 `findByIsbn()` 方法（`⟵ 需完成 U06 必做 3`），串接 Repository 的新方法
2. 畫出 CreateDto → Entity → ResponseDto 的資料流向圖

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Service 注入時噴 `Nest could not find...` 錯誤 | Service 沒有在 Module 的 providers 註冊 | 在 `book.module.ts` 的 providers 加入 `BookService` |
| DTO 驗證沒生效 | 沒有全域啟用 ValidationPipe | 在 `main.ts` 加入 `app.useGlobalPipes(new ValidationPipe())` |

#### ⑥ 驗收標準（DoD）

能說明 Service 層存在的理由，能解釋 Entity/CreateDto/ResponseDto 三者的職責差異，頁面和 API 行為與重構前完全一致。

---

### U08｜REST API 完善與錯誤處理（90 min）

#### ① 為什麼先教這個？

U07 完成了分層，但 API 還缺少正確的 HTTP 狀態碼和統一的錯誤回應。使用者送了錯誤資料、查了不存在的 ID，應該要拿到清楚的錯誤訊息而非一堆 stack trace。這堂課把 API 做到「生產品質」。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.api.controller.ts` | REST API 端點（加入正確的狀態碼） |
| `common/filters/http-exception.filter.ts` | 全域錯誤處理（統一所有例外的回應格式） |
| `common/pipes/validation.pipe.ts` | 全域驗證 Pipe |

#### ③ 核心觀念

- **HTTP 狀態碼語意** — 200 OK、201 Created、204 No Content、400 Bad Request、404 Not Found、500 Internal Server Error
- **`@HttpCode(HttpStatus.CREATED)`** — 新增成功回 201 而非預設的 200
- **`ValidationPipe`** — 自動觸發 DTO 上的 class-validator，驗證失敗回 400
- **`ExceptionFilter`** — 攔截所有 Controller 拋出的例外，統一回應格式
- **`@ApiResponse()`** — 為 Swagger 文件標示可能的回應狀態碼

#### ④ 動手做

**必做**
1. 在 API Controller 加上 `@HttpCode(HttpStatus.CREATED)` 和 `@HttpCode(HttpStatus.NO_CONTENT)`，確認 POST 回 201、DELETE 回 204
2. 故意送缺欄位的 JSON（如缺少 `title`），觀察 400 錯誤回應格式
3. 查一筆不存在的 ID（如 `GET /api/books/99999`），觀察 404 錯誤回應
4. 在新增表單留空送出，觀察 Nunjucks 驗證錯誤如何顯示在頁面上

**延伸挑戰**
1. 自訂一個業務例外（如 `DuplicateIsbnException`），在 ExceptionFilter 加入對應處理，回傳 409 Conflict

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| DTO 驗證沒生效，錯誤資料直接進 Service | Controller 沒有套用 ValidationPipe | 確認全域或該 Controller 有 `@UsePipes(ValidationPipe)` |
| ExceptionFilter 沒攔截到錯誤 | Filter 沒有全域註冊 | 在 `main.ts` 加入 `app.useGlobalFilters(new HttpExceptionFilter())` |

#### ⑥ 驗收標準（DoD）

能用 Swagger UI 完成完整 CRUD 操作（POST 201、DELETE 204），並能觸發 400 和 404 錯誤觀察統一的錯誤回應格式。

---

### U09｜Redis 快取整合（90 min）

#### ① 為什麼先教這個？

資料庫查詢是 Web 應用最慢的環節。把「熱資料」放在 Redis 記憶體中，回應速度可以從毫秒降到微秒等級。這堂課在 U07 建立的 Service 層上加入快取邏輯。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.service.ts` | Cache-Aside 讀寫邏輯的實際所在 |
| `config/redis.config.ts` | Redis 連線與設定 |
| `common/cache/cache.service.ts` | Redis 快取操作封裝 |

#### ③ 核心觀念

- **Cache-Aside 模式** — 讀：查快取 → miss → 查 DB → 寫快取；寫/刪：操作 DB → 驅逐快取
- **`ioredis` 套件** — Node.js 最常用的 Redis client
- **JSON 序列化** — 用 `JSON.stringify()` / `JSON.parse()` 存取物件
- **快取 key 設計** — `book:{id}`，TTL 10 分鐘
- **為什麼 update / delete 要清快取？** — 避免讀到過期資料（快取一致性）

#### ④ 動手做

**必做**
1. 追蹤 `BookService.findById()` 完整流程：Redis hit → 直接回傳 / Redis miss → 查 DB → 寫回 Redis
2. 用 `redis-cli` 觀察快取 key：`KEYS book:*`（教學環境限定指令）
3. 手動刪除一個 key（`DEL book:1`），再呼叫 API 觀察 console log 中的 cache miss 與回寫

**延伸挑戰**
1. 呼叫 `GET /api/books/1` 兩次，比較回應時間差異
2. 將 TTL 從 10 分鐘改為 1 分鐘，觀察快取命中率變化

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `KEYS` 指令在正式環境不能用 | `KEYS` 會阻塞 Redis（全表掃描） | 教學環境直觀好用；正式環境改用 `SCAN 0 MATCH book:*` |
| Redis 連線失敗 | Redis 容器沒啟動或連線設定錯誤 | 確認 `docker compose ps` Redis 在 running，檢查 `.env` 設定 |

#### ⑥ 驗收標準（DoD）

能用 `redis-cli` 驗證快取行為（確認 key 存在、手動刪除後觀察回源），並能說明 Cache-Aside 的讀寫流程與快取失效策略。

---

### U10｜測試策略與實戰（120 min）

#### ① 為什麼先教這個？

不寫測試的程式碼是「可能會動」，有測試的程式碼是「確定會動」。測試是修改程式碼後不會造成回歸錯誤的安全網——這是從「學習專案」走向「生產專案」的關鍵一步。

#### ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.service.spec.ts` | 單元測試（用 Jest mock 隔離外部依賴） |
| `book/book.api.controller.spec.ts` | Controller 單元測試 |
| `test/app.e2e-spec.ts` | e2e 測試（完整 HTTP 請求測試） |

#### ③ 核心觀念

- **測試金字塔** — 單元測試（多且快）→ 整合測試（少且慢），先追求單元測試覆蓋
- **Jest 核心概念**：
  - `jest.mock()` — 模擬模組
  - `jest.fn()` — 建立 mock function
  - `mockResolvedValue()` — 定義 async mock 的回傳值
  - `expect().toHaveBeenCalledWith()` — 驗證呼叫參數
- **`@nestjs/testing`** — NestJS 提供的測試模組，可建立隔離的測試模組
- **e2e 測試** — 用 `supertest` 模擬完整 HTTP 請求
- **AAA 模式** — Arrange（準備測試資料）→ Act（執行被測方法）→ Assert（驗證結果）

#### ④ 動手做

**必做**
1. 執行 `npm run test`，閱讀測試報告
2. 在 `book.service.spec.ts` 新增一個測試方法：`findById when not found throws exception`

**延伸挑戰**
1. 在 `book.api.controller.spec.ts` 新增測試：`update returns 200`
2. 執行 `npm run test:e2e` 觀察 e2e 測試的執行過程
3. 觀察測試速度差異：單元測試 vs e2e 測試，記錄並解釋差異原因

#### ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Mock 物件全是 undefined，測試直接報錯 | 忘記設定 mock 的回傳值 | 確保所有被呼叫的 mock 方法都有設定 `mockResolvedValue()` 或 `mockReturnValue()` |
| e2e 測試連不上資料庫 | 測試環境沒有啟動 DB 容器 | e2e 測試前先 `docker compose up -d` 或使用 in-memory DB |

#### ⑥ 驗收標準（DoD）

能為新方法撰寫單元測試與 e2e 測試，能解釋單元測試 mock vs e2e 測試真實 DB 的差異。建議門檻：行覆蓋率 ≥ 60%，Service 核心流程覆蓋率 ≥ 80%。

---

## §6 里程碑檢核

| 里程碑 | 涵蓋單元 | 達成標準 | 未達標補救 |
|--------|----------|----------|-----------|
| **M1：能跑能看** | U01–U03 | 學員可啟動專案、做出假資料頁面與 API、在瀏覽器看到書籍清單 | 補做 U01–U03 的必做練習；講師一對一確認環境問題 |
| **M2：真實 CRUD 循環** | U04–U06 | 學員可串接真實 DB，頁面與 API 顯示 DB 資料，完成新增/查詢/更新/刪除 | 回頭確認 DB 連線設定，重做 U06 的 Repository 串接 |
| **M3：正規架構與完整功能** | U07–U10 | 學員有 Service/DTO 分層、統一錯誤處理、Redis 快取、自動化測試，可展示完整成品 | 優先完成 U07 分層重構；U09、U10 可簡化為閱讀理解 + 跑既有測試 |

---

## §7 自學流程建議

### 每單元學習節奏

| 步驟 | 時間佔比 | 說明 |
|------|----------|------|
| 1. 閱讀核心觀念 | 20% | 先理解「為什麼」和「是什麼」 |
| 2. 執行範例程式 | 10% | 把 Repo 程式碼跑起來，確認能動 |
| 3. 必做練習 | 50% | 動手實作，這是學習的核心 |
| 4. 自我驗收 | 20% | 對照驗收標準檢查成果 |

### 自學時數建議

| 單元 | 教學時數 | 自學時數 | 說明 |
|------|----------|----------|------|
| U01–U03 | 210 min | 4–5 hr | 環境設定可能需要額外時間排錯 |
| U04–U06 | 270 min | 5–6 hr | 資料庫相關概念較多，建議多做延伸挑戰 |
| U07–U10 | 390 min | 6–8 hr | 架構概念需要反覆實作才能內化 |

---

## §8 評量機制

### 課程練習評量

| 項目 | 比重 | 具體說明 |
|------|------|----------|
| 功能完成度 | 40% | 必做練習 100% 完成；能正確執行 CRUD、快取、驗證等核心功能 |
| 程式碼品質 | 20% | 命名規範、分層職責明確、無重複程式碼 |
| 測試覆蓋 | 20% | 具備正確的 Mock 運用與測試斷言；行覆蓋率 ≥ 60%，Service 核心流程 ≥ 80% |
| 問題分析 | 20% | 能判讀錯誤 log 並說明修復思路；遇到踩坑問題能自行排查 |

### 結訓驗收專案

> **設計理念**：驗收專案採用與教案**不同的業務主題**，驗證學員是否真正理解分層架構與開發流程，而非只是照抄書籍管理系統的程式碼。

**驗收方式**：限時實作（建議 3–4 小時）

**建議主題**：「員工通訊錄管理系統」（或其他具備 CRUD 性質的主題，如待辦事項、商品庫存）

| 驗收項目 | 達成條件 |
|----------|----------|
| TypeORM Migration | 至少一支建表 migration + 一支 seed data migration |
| Entity + DTO | Entity 對應 DB 表、CreateDto 含 class-validator、ResponseDto 含轉換方法 |
| Repository | 完成基本 CRUD 查詢 |
| Service 層 | 業務邏輯集中在 Service |
| REST API | 至少 4 支 API（GET list / GET by ID / POST / DELETE），狀態碼正確 |
| Nunjucks 頁面 | 至少一個列表頁 + 一個新增表單頁，能正常操作 |
| 測試 | 至少 1 支 Service 單元測試 + 1 支 e2e 測試 |
| 全部可運行 | `npm run start:dev` 能啟動、頁面能操作、API 能呼叫、`npm run test` 全綠 |

**評分標準**：同上方課程練習評量比重。

### 標準化評語範例

| 等級 | 評語範例 |
|------|----------|
| 優秀 | 功能完整且程式碼結構清晰，測試覆蓋充分，能獨立排查問題並提出合理的設計理由 |
| 達標 | 核心功能完成，分層架構正確，有基本測試，能在提示下排查常見問題 |
| 待加強 | 部分功能缺失或分層不明確，測試不足，排查問題時需要較多協助 |
| 未達標 | 核心 CRUD 流程無法跑通，建議補做指定練習後重新驗收 |

---

## §9 參考文件

| 文件 | 用途 |
|------|------|
| `README.md` | 環境啟動與操作手冊 |
| `/api` | 互動式 API 測試介面（Swagger UI） |
| `/api-json` | OpenAPI 3.0 JSON 規格 |

### 外部參考資源

| 主題 | 資源 |
|------|------|
| NestJS 官方文件 | [docs.nestjs.com](https://docs.nestjs.com/) |
| TypeORM 官方文件 | [typeorm.io](https://typeorm.io/) |
| Nunjucks 官方文件 | [mozilla.github.io/nunjucks](https://mozilla.github.io/nunjucks/) |
| class-validator 文件 | [github.com/typestack/class-validator](https://github.com/typestack/class-validator) |
| Jest 官方文件 | [jestjs.io](https://jestjs.io/) |
| Docker Compose 文件 | [docs.docker.com/compose](https://docs.docker.com/compose/) |

---

## §10 延伸學習方向

- **認證授權**：加入 Passport.js / JWT 認證（登入、角色權限）
- **進階查詢**：分頁、排序、模糊搜尋（TypeORM QueryBuilder）
- **CI/CD**：GitHub Actions 自動測試與建置
- **雲端部署**：用 Docker 部署到 AWS / GCP / Azure
- **監控與可觀測性**：NestJS Terminus 健康檢查、Prometheus Metrics
- **GraphQL**：NestJS GraphQL 模組整合

---

## §11 維護與版本管理

### 教案版本資訊

| 項目 | 值 |
|------|-----|
| 教案版本 | v1.0 |
| 最後更新日期 | 2026-03-09 |
| 對應 Repo 分支 | `main` |
| Node.js 版本 | 20 |
| NestJS 版本 | 11 |

### 與程式碼同步原則

- 當 Repo 的依賴版本升級（如 NestJS 版本）時，教案需同步更新 §1 技術棧與 §4 驗證指令
- 當檔案路徑異動（重構、改名）時，教案 §5 各單元的「對應檔案」表格需同步更新
- 當 API 行為變更時，§5 U08 的動手做與踩坑提示需重新驗證
- 負責人：教案維護者應在每次 Repo 重大更新後檢查上述項目

### 開課前檢查清單

- [ ] §4 的驗證指令是否仍可正常執行？（`node -v`、`docker compose version`、`npm install`）
- [ ] `npm run test` 是否全部通過？
- [ ] 所有單元列出的檔案路徑是否仍存在且正確？
- [ ] Swagger UI 是否可正常訪問？
- [ ] Docker Compose 是否能一鍵啟動所有服務？
- [ ] 驗收專案的模板或 starter 是否已準備好？
