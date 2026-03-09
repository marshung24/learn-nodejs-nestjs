# U05｜TypeORM Migration 與資料庫設計

> 能撰寫 migration 並在啟動時自動建表與塞入資料 ｜ 90 min ｜ 前置依賴：U04

---

## 為什麼先教這個？

U02-U03 用的是假資料，接下來要換成真的。第一步是「設計資料庫結構」——用 TypeORM Migration 管理 schema 版本，確保多人協作時每個人的 DB 長一樣。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `migrations/1700000000000-CreateBooksTable.ts` | 建表（初始 schema） |
| `migrations/1700000000001-InsertSampleData.ts` | 塞入範例資料（seed data） |
| `data-source.ts` | TypeORM CLI 使用的資料來源設定 |

---

## 核心觀念

### Migration 機制

用 TypeScript 撰寫 `up()` 和 `down()` 方法，分別執行升級與回滾：

- **`up()`**：升級（建表、加欄位）
- **`down()`**：回滾（刪表、移除欄位）

### 時間戳命名慣例

`{timestamp}-{描述}.ts`，確保執行順序：

```
migrations/
├── 1700000000000-CreateBooksTable.ts     ← 第一個執行
├── 1700000000001-InsertSampleData.ts     ← 第二個執行
└── 1700000000002-AddPublisherColumn.ts   ← 第三個執行
```

### `1700000000000-CreateBooksTable.ts` 全文

```typescript
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateBooksTable1700000000000 implements MigrationInterface {

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'books',
        columns: [
          {
            name: 'id',
            type: 'bigint',
            isPrimary: true,
            isGenerated: true,
            generationStrategy: 'increment',
          },
          {
            name: 'title',
            type: 'varchar',
            length: '200',
            isNullable: false,
          },
          {
            name: 'author',
            type: 'varchar',
            length: '100',
            isNullable: false,
          },
          {
            name: 'isbn',
            type: 'varchar',
            length: '20',
            isNullable: false,
            isUnique: true,
          },
          {
            name: 'stock',
            type: 'int',
            default: 0,
          },
          {
            name: 'created_at',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'updated_at',
            type: 'datetime',
            default: 'CURRENT_TIMESTAMP',
            onUpdate: 'CURRENT_TIMESTAMP',
          },
        ],
      }),
      true,  // ifNotExists
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('books');
  }
}
```

### 欄位設計思考

| 決策 | 理由 |
|------|------|
| **`BIGINT`** 不用 `INT` | 避免未來資料量大時溢位（INT 最大約 21 億） |
| **`isbn UNIQUE`** | 同一本書不會重複匯入 |
| **`stock DEFAULT 0`** | 代表「尚未進貨」 |
| **`created_at DEFAULT CURRENT_TIMESTAMP`** | 自動記錄建立時間 |
| **`updated_at ON UPDATE CURRENT_TIMESTAMP`** | 每次 UPDATE 自動更新時間 |

### `1700000000001-InsertSampleData.ts` 全文

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class InsertSampleData1700000000001 implements MigrationInterface {

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      INSERT INTO books (title, author, isbn, stock) VALUES
      ('Clean Code',              'Robert C. Martin', '9780132350884', 5),
      ('Effective TypeScript',    'Dan Vanderkam',    '9781492053743', 3),
      ('NestJS in Action',        'Jay Bell',         '9781617298035', 8)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      DELETE FROM books WHERE isbn IN ('9780132350884', '9781492053743', '9781617298035')
    `);
  }
}
```

### `migrations` 表

TypeORM 在 DB 中自動建立的紀錄表，記錄每次 migration 的執行狀態：

| id | timestamp | name |
|----|-----------|------|
| 1 | 1700000000000 | CreateBooksTable1700000000000 |
| 2 | 1700000000001 | InsertSampleData1700000000001 |

### 不可修改已執行的 migration

已執行的 migration 不應修改——TypeORM 不會重新執行已存在於 `migrations` 表的項目。需要修正就用新 migration（如 `AddPublisherColumn`）。

### data-source.ts — CLI 設定檔

```typescript
import { DataSource } from 'typeorm';
import { ConfigService } from '@nestjs/config';
import { config } from 'dotenv';

config();  // 載入 .env

const configService = new ConfigService();

export default new DataSource({
  type: 'mysql',
  host: configService.get('DATABASE_HOST', 'localhost'),
  port: configService.get('DATABASE_PORT', 3306),
  username: configService.get('DATABASE_USER', 'app'),
  password: configService.get('DATABASE_PASSWORD', 'app'),
  database: configService.get('DATABASE_NAME', 'app'),
  entities: ['src/**/*.entity.ts'],
  migrations: ['migrations/*.ts'],
});
```

### package.json migration 腳本

```json
{
  "scripts": {
    "migration:generate": "typeorm migration:generate -d data-source.ts",
    "migration:run": "typeorm migration:run -d data-source.ts",
    "migration:revert": "typeorm migration:revert -d data-source.ts"
  }
}
```

---

## 動手做

### 必做

**1. 閱讀現有 migration 檔案**

閱讀 `CreateBooksTable` migration，說出每個欄位的設計理由：

- 為什麼用 `BIGINT` 不用 `INT`？
- 為什麼 `isbn` 加 `UNIQUE`？
- `created_at` 和 `updated_at` 有什麼作用？

**2. 執行 migration**

```bash
npm run migration:run
```

確認 migration 成功執行，沒有錯誤訊息。

**3. 新增一支 migration**

建立 `migrations/1700000000002-AddPublisherColumn.ts`：

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddPublisherColumn1700000000002 implements MigrationInterface {

  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn('books', new TableColumn({
      name: 'publisher',
      type: 'varchar',
      length: '100',
      isNullable: true,  // 新欄位允許 NULL（已有資料無法給值）
    }));
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('books', 'publisher');
  }
}
```

**4. 執行並驗證**

```bash
# 執行 migration
npm run migration:run

# 進入 MySQL 容器驗證
docker compose exec db mysql -u app -papp app

# 查看 migrations 紀錄
SELECT * FROM migrations;

# 確認新欄位存在
DESCRIBE books;
```

### 延伸挑戰

1. 執行 `npm run migration:revert` 回滾最後一次 migration，觀察 `down()` 方法的效果：

```bash
npm run migration:revert

# 再次檢查 books 表，publisher 欄位應該消失
docker compose exec db mysql -u app -papp app -e "DESCRIBE books"
```

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| Migration 指令報錯找不到 data-source | `data-source.ts` 路徑設定錯誤 | 確認 `package.json` 中的 migration 腳本指向正確路徑 `-d data-source.ts` |
| Migration 執行後 DB 沒變化 | 該 migration 已執行過 | 檢查 `migrations` 表，已執行的不會重跑 |
| 新 migration 沒被執行 | 類別名稱與檔名不一致 | 確保類別名稱包含時間戳，如 `CreateBooksTable1700000000000` |
| `ECONNREFUSED` 連線失敗 | DB 容器未啟動或 `.env` 設定錯誤 | 執行 `docker compose up -d`；確認 `.env` 中的 `DATABASE_HOST` |

---

## 驗收標準（DoD）

- [ ] 能撰寫一支新的 migration 並在執行後看到資料表結構變更
- [ ] 能在 `migrations` 表確認執行紀錄
- [ ] 能說出 `up()` 和 `down()` 的用途
- [ ] 能說出時間戳命名的用途——確保執行順序
- [ ] 能解釋為什麼不能修改已執行過的 migration
