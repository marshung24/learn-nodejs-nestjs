# U09｜Redis 快取整合

> 能用 redis-cli 驗證快取行為並說明 Cache-Aside 流程 ｜ 90 min ｜ 前置依賴：U07

---

## 為什麼先教這個？

資料庫查詢是 Web 應用最慢的環節。把「熱資料」放在 Redis 記憶體中，回應速度可以從毫秒降到微秒等級。本單元在 U07 建立的 Service 層上加入快取邏輯。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.service.ts` | Cache-Aside 讀寫邏輯的實際所在 |
| `config/redis.config.ts` | Redis 連線與設定 |
| `common/cache/cache.service.ts` | Redis 快取操作封裝 |

---

## 核心觀念

### Cache-Aside 模式

最常見的快取模式。核心原則：**讀時填充，寫時驅逐**。

```
讀取流程（findById）：

    Request
      │
      ▼
  ┌──────────┐     hit     ┌───────┐
  │  查 Redis │ ──────────→│ 回傳  │
  └──────────┘             └───────┘
      │ miss
      ▼
  ┌──────────┐
  │  查 DB   │
  └──────────┘
      │
      ▼
  ┌──────────────┐
  │ 寫回 Redis   │ ← TTL 10 分鐘
  └──────────────┘
      │
      ▼
  ┌──────────┐
  │  回傳    │
  └──────────┘


寫入/刪除流程（create / update / delete）：

    Request
      │
      ▼
  ┌──────────┐
  │ 操作 DB  │
  └──────────┘
      │
      ▼
  ┌──────────────┐
  │ 驅逐 Redis   │ ← 刪除對應 key
  └──────────────┘
```

> **為什麼 update / delete 要清快取？** 避免讀到過期資料。如果不清，使用者更新書名後查詢還會看到舊名稱。

### `cache.service.ts` — Redis 快取封裝

```typescript
import { Injectable, Logger } from '@nestjs/common';
import Redis from 'ioredis';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class CacheService {
  private readonly logger = new Logger(CacheService.name);
  private readonly redis: Redis;
  private readonly defaultTtl = 600; // 10 分鐘

  constructor(private configService: ConfigService) {
    this.redis = new Redis({
      host: this.configService.get('REDIS_HOST', 'localhost'),
      port: this.configService.get('REDIS_PORT', 6379),
    });
  }

  /**
   * 取得快取值
   */
  async get<T>(key: string): Promise<T | null> {
    const cached = await this.redis.get(key);
    if (cached) {
      this.logger.debug(`Cache hit: ${key}`);
      return JSON.parse(cached) as T;
    }
    this.logger.debug(`Cache miss: ${key}`);
    return null;
  }

  /**
   * 設定快取值
   */
  async set(key: string, value: any, ttlSeconds?: number): Promise<void> {
    const ttl = ttlSeconds ?? this.defaultTtl;
    await this.redis.set(key, JSON.stringify(value), 'EX', ttl);
    this.logger.debug(`Cache set: ${key} (TTL: ${ttl}s)`);
  }

  /**
   * 刪除快取
   */
  async del(key: string): Promise<void> {
    await this.redis.del(key);
    this.logger.debug(`Cache deleted: ${key}`);
  }

  /**
   * 刪除符合 pattern 的所有 key
   */
  async delByPattern(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
      this.logger.debug(`Cache deleted by pattern: ${pattern} (${keys.length} keys)`);
    }
  }
}
```

### `book.service.ts` — 加入快取邏輯

```typescript
import { Injectable, NotFoundException, Logger } from '@nestjs/common';
import { BookRepository } from './book.repository';
import { CacheService } from '../common/cache/cache.service';
import { CreateBookDto } from './dto/create-book.dto';
import { BookResponseDto } from './dto/book-response.dto';

@Injectable()
export class BookService {
  private readonly logger = new Logger(BookService.name);
  private readonly KEY_PREFIX = 'book:';
  private readonly CACHE_TTL = 600; // 10 分鐘

  constructor(
    private readonly bookRepository: BookRepository,
    private readonly cacheService: CacheService,
  ) {}

  async findById(id: number): Promise<BookResponseDto> {
    const cacheKey = `${this.KEY_PREFIX}${id}`;

    // Step 1：查快取
    const cached = await this.cacheService.get<BookResponseDto>(cacheKey);
    if (cached) {
      return cached;  // 命中 → 直接回傳，不查 DB
    }

    // Step 2：Cache miss → 查 DB
    const book = await this.bookRepository.findById(id);
    if (!book) {
      throw new NotFoundException(`Book with ID ${id} not found`);
    }

    // Step 3：回寫快取
    const dto = BookResponseDto.from(book);
    await this.cacheService.set(cacheKey, dto, this.CACHE_TTL);

    return dto;
  }

  async create(createDto: CreateBookDto): Promise<BookResponseDto> {
    const book = await this.bookRepository.create(createDto);
    // 新增不需要清快取（因為是新 key）
    return BookResponseDto.from(book);
  }

  async update(id: number, updateDto: CreateBookDto): Promise<BookResponseDto> {
    const book = await this.bookRepository.update(id, updateDto);
    if (!book) {
      throw new NotFoundException(`Book with ID ${id} not found`);
    }

    // 更新後驅逐快取
    await this.cacheService.del(`${this.KEY_PREFIX}${id}`);

    return BookResponseDto.from(book);
  }

  async delete(id: number): Promise<void> {
    await this.bookRepository.delete(id);

    // 刪除後驅逐快取
    await this.cacheService.del(`${this.KEY_PREFIX}${id}`);
  }
}
```

### JSON 序列化

用 `JSON.stringify()` / `JSON.parse()` 存取物件：

```typescript
// 存入
await redis.set(key, JSON.stringify(data), 'EX', 600); // TTL 600 秒

// 讀取
const cached = await redis.get(key);
const data = cached ? JSON.parse(cached) : null;
```

### 快取 key 設計

| 項目 | 值 |
|------|-----:|
| key 格式 | `book:{id}`（如 `book:1`） |
| TTL | 10 分鐘（600 秒） |
| 序列化 | JSON |
| 驅逐時機 | update / delete 操作後立即刪除對應 key |

### 快取預熱（Warmup）

冷啟動時所有 key 都不存在，第一波請求全部 cache miss 會瞬間打到 DB（稱為「快取雪崩」）。

```typescript
import { Injectable, OnModuleInit, Logger } from '@nestjs/common';
import { BookRepository } from '../book/book.repository';
import { CacheService } from './cache.service';
import { BookResponseDto } from '../book/dto/book-response.dto';

@Injectable()
export class CacheWarmupService implements OnModuleInit {
  private readonly logger = new Logger(CacheWarmupService.name);

  constructor(
    private readonly bookRepository: BookRepository,
    private readonly cacheService: CacheService,
  ) {}

  async onModuleInit() {
    const books = await this.bookRepository.findAll();
    for (const book of books) {
      await this.cacheService.set(`book:${book.id}`, BookResponseDto.from(book));
    }
    this.logger.log(`Cache warmup complete. ${books.length} books loaded.`);
  }
}
```

---

## 動手做

### 必做

**1. 追蹤 `findById()` 完整流程**

閱讀 `BookService.findById()` 程式碼，回答：
- Step 1 做了什麼？（查 Redis）
- Cache hit 時走哪條路？（直接 return）
- Cache miss 時走哪條路？（查 DB → 寫 Redis → return）

**2. 用 redis-cli 觀察快取 key**

```bash
# 進入 Redis 容器
docker compose exec redis redis-cli

# 查看所有 book 相關的 key
KEYS book:*
# 預期：1) "book:1"  2) "book:2"  3) "book:3"

# 查看某個 key 的內容
GET book:1
# 預期：JSON 格式的書籍資料

# 查看剩餘 TTL（秒）
TTL book:1
```

**3. 手動驅逐快取並觀察回源**

```bash
# 在 redis-cli 中刪除一個 key
DEL book:1
exit
```

然後呼叫 API：

```bash
curl http://localhost:3000/api/books/1
```

觀察 console log，應該看到 `Cache miss: book:1`（因為剛刪了），代表走了 cache miss → 查 DB → 回寫 Redis 的流程。

再用 redis-cli 確認 key 已被回寫：

```bash
docker compose exec redis redis-cli GET book:1
# 預期：有 JSON 資料
```

### 延伸挑戰

1. 呼叫 `GET /api/books/1` 兩次，比較回應時間差異（第一次 miss 較慢，第二次 hit 較快）
2. 將 TTL 從 10 分鐘改為 1 分鐘，觀察快取命中率變化，思考 TTL 設太短與太長的取捨

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `KEYS` 指令在正式環境不能用 | `KEYS` 會阻塞 Redis（全表掃描） | 教學環境直觀好用；正式環境改用 `SCAN 0 MATCH book:*` |
| Redis 連線失敗 | Redis 容器沒啟動或連線設定錯誤 | 確認 `docker compose ps` Redis 在 running，檢查 `.env` 設定 |
| JSON 序列化錯誤 | Date 物件無法直接序列化 | 確保 Entity 中的日期欄位在轉換成 DTO 時是正確的 Date 或 string |
| 更新後讀到舊資料 | 忘記在 update 後清快取 | 確認 `update()` 方法中有 `cacheService.del()` |

---

## 驗收標準（DoD）

- [ ] 能用 redis-cli 驗證快取行為（確認 key 存在、查看 TTL、手動刪除後觀察回源）
- [ ] 能說明 Cache-Aside 的讀取流程（查快取 → miss → 查 DB → 寫快取）
- [ ] 能說明為什麼 update / delete 要驅逐快取
- [ ] 能解釋快取預熱（warmup）的目的——避免快取雪崩
