# U09｜Redis 快取整合

> 建議時數：90 min
> 前置依賴：U07

---

## ① 為什麼先教這個？

資料庫查詢是 Web 應用最慢的環節。把「熱資料」放在 Redis 記憶體中，回應速度可以從毫秒降到微秒等級。這堂課在 U07 建立的 Service 層上加入快取邏輯。

---

## ② 對應檔案

| 檔案 | 角色 |
|------|------|
| `src/book/book.service.ts` | Cache-Aside 讀寫邏輯 |
| `src/config/redis.config.ts` | Redis 連線設定 |
| `src/common/cache/cache.service.ts` | Redis 快取操作封裝 |

---

## ③ 核心觀念

### Cache-Aside 模式

**讀取流程：**
1. 查快取
2. 快取 hit → 直接回傳
3. 快取 miss → 查 DB → 寫回快取 → 回傳

**寫入/刪除流程：**
1. 操作 DB
2. 驅逐（刪除）相關快取

### `ioredis` 套件

- Node.js 最常用的 Redis client
- 支援 Promise API

### JSON 序列化

```typescript
// 存入
await redis.set(key, JSON.stringify(data), 'EX', 600); // TTL 600 秒

// 讀取
const cached = await redis.get(key);
const data = cached ? JSON.parse(cached) : null;
```

### 快取 key 設計

- 格式：`book:{id}`
- TTL：10 分鐘

### 為什麼要清快取？

- update / delete 後要清快取
- 避免讀到過期資料（快取一致性）

---

## ④ 動手做

### 必做

1. **追蹤快取流程**
   - 追蹤 `BookService.findById()` 完整流程：
   - Redis hit → 直接回傳 / Redis miss → 查 DB → 寫回 Redis

2. **觀察快取 key**
   - 用 `redis-cli` 觀察快取 key：`KEYS book:*`

3. **手動清除快取**
   - 手動刪除一個 key：`DEL book:1`
   - 再呼叫 API 觀察 console log 中的 cache miss 與回寫

### 延伸挑戰

1. 呼叫 `GET /api/books/1` 兩次，比較回應時間差異
2. 將 TTL 從 10 分鐘改為 1 分鐘，觀察快取命中率變化

---

## ⑤ 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| `KEYS` 指令在正式環境不能用 | `KEYS` 會阻塞 Redis | 正式環境改用 `SCAN` |
| Redis 連線失敗 | Redis 容器沒啟動 | 確認 `docker compose ps` Redis 在 running |

---

## ⑥ 驗收標準（DoD）

- [ ] 能用 `redis-cli` 驗證快取行為
- [ ] 能說明 Cache-Aside 的讀寫流程
- [ ] 能解釋為什麼 update / delete 要清快取
