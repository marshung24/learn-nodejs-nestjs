# U01｜專案啟動與第一支 API

> 能啟動專案並用瀏覽器呼叫自訂端點 ｜ 90 min ｜ 前置依賴：無

---

## 為什麼先教這個？

讓學員在第一堂課就看到「程式跑起來」的成就感。沒有什麼比親手讓一個 Web 應用回應你的請求更能建立學習信心。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `package.json` | 依賴管理與腳本設定（定義專案用到的所有套件） |
| `src/main.ts` | NestJS 應用程式進入點 |
| `src/app.module.ts` | 根模組（AppModule），組織所有功能模組 |
| `src/app.controller.ts` | 最簡易的 REST 端點，示範如何回應 HTTP 請求 |

---

## 核心觀念

### 專案標準目錄架構

```
nodejs-nestjs-sample/
├── package.json                          ← 依賴管理
├── docker-compose.yml                    ← 容器編排
├── src/                                  ← 程式碼
│   ├── main.ts                           ← 進入點
│   ├── app.module.ts                     ← 根模組
│   ├── app.controller.ts                 ← Hello Controller
│   ├── config/                           ← 設定類
│   ├── common/                           ← 共用元件（Filter、Pipe、Interceptor）
│   └── book/                             ← Book 功能模組
│       ├── book.module.ts
│       ├── book.controller.ts            ← MVC Controller
│       ├── book.api.controller.ts        ← REST API Controller
│       ├── book.service.ts               ← 業務邏輯
│       ├── book.repository.ts            ← 資料存取
│       ├── entities/                     ← Entity
│       └── dto/                          ← DTO
├── views/                                ← Nunjucks 模板
│   ├── layouts/
│   │   └── main.njk                      ← 共用版型
│   └── book/
│       ├── list.njk                      ← 書籍清單頁
│       ├── detail.njk                    ← 書籍詳情頁
│       └── form.njk                      ← 新增/編輯表單頁
├── migrations/                           ← TypeORM migration
└── test/                                 ← 測試
```

### `main.ts` — 應用程式進入點

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  // 建立 NestJS 應用程式實例
  const app = await NestFactory.create(AppModule);

  // 啟動 HTTP 伺服器，監聽指定 Port
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

`NestFactory.create()` 初始化整個應用程式容器，載入所有模組、控制器、服務。

### `@Module()` — 根模組

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],        // 匯入其他模組
  controllers: [AppController],  // 註冊控制器
  providers: [AppService],       // 註冊服務（Provider）
})
export class AppModule {}
```

`@Module()` 是組織功能的容器，將相關的 Controller、Service 組織在一起。

### `@Controller()` — 最簡易的端點

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('hello')
  getHello(): string {
    return 'Hello, NestJS!';
  }

  @Get('health')
  health(): string {
    return 'OK';
  }
}
```

- `@Controller()` 定義路由前綴（此處為空，表示根路徑）
- `@Get('hello')` 定義 GET `/hello` 端點
- 回傳字串直接作為 HTTP 回應內容

### NestJS 三大核心概念

| 概念 | 裝飾器 | 職責 |
|------|--------|------|
| Module | `@Module()` | 組織功能的容器，匯入其他模組、註冊 Controller 與 Provider |
| Controller | `@Controller()` | 處理 HTTP 請求，定義路由與回應 |
| Provider/Service | `@Injectable()` | 業務邏輯、可被依賴注入 |

### 依賴注入（DI）

NestJS 的核心機制。在 constructor 宣告依賴，框架自動注入實例：

```typescript
@Controller()
export class AppController {
  // NestJS 自動注入 AppService 實例
  constructor(private readonly appService: AppService) {}
}
```

好處：鬆散耦合、易於測試（可替換 mock）、統一管理實例生命週期。

### npm 做了什麼

`package.json` 的 scripts 區塊定義了常用指令：

```json
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:prod": "node dist/main",
    "build": "nest build",
    "test": "jest",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

`npm run start:dev` 是最常用的開發指令——編譯 TypeScript、啟動伺服器、監聽檔案變更（hot reload）。

---

## 動手做

### 必做

**1. 啟動專案**

```bash
docker compose up -d         # 啟動 MySQL 與 Redis
npm install                  # 安裝依賴（首次需要）
npm run start:dev            # 啟動 NestJS 應用
```

瀏覽器打開 `http://localhost:3000/hello`，確認看到 `Hello, NestJS!`。

**2. 新增自訂端點**

在 `AppController` 新增一個 `/whoami` 端點，回傳自己的名字：

```typescript
@Get('whoami')
whoami(): { name: string } {
  return { name: '你的名字' };
}
```

用瀏覽器或 curl 測試：

```bash
curl http://localhost:3000/whoami
# 預期：{"name":"你的名字"}
```

### 延伸挑戰

1. 觀察 `package.json` 的 dependencies 區塊，寫下每個套件的用途：
   - `@nestjs/core` — 提供什麼功能？
   - `@nestjs/platform-express` — 為什麼需要 Express？
   - `typeorm` — 用來做什麼？

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `start:dev` 後卡住或報 port 衝突 | Port 3000 被其他程式占用 | `lsof -i :3000` 找出占用程式並關閉；或在 `.env` 改 `PORT` |
| 修改 TypeScript 後瀏覽器沒變化 | 需等待 hot reload 重新編譯 | 注意看 console log 的 restart 訊息，等重啟完成再刷新 |
| `Cannot find module` 錯誤 | 依賴未安裝或路徑錯誤 | 執行 `npm install`；確認 import 路徑正確 |

---

## 驗收標準（DoD）

- [ ] 能用瀏覽器訪問 `http://localhost:3000/hello` 看到回應
- [ ] 能用 `curl http://localhost:3000/whoami` 看到自訂的 JSON 回應
- [ ] 能說出 `@Module()` 做了什麼——組織 Controller 與 Provider
- [ ] 能說出 Module、Controller、Provider 三者的關係
- [ ] 能說出 `npm run start:dev` 的作用
