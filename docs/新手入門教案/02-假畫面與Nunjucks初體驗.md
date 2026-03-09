# U02｜假畫面 — Nunjucks 初體驗

> 能用 Nunjucks 做出一個書籍清單頁面（假資料） ｜ 60 min ｜ 前置依賴：U01

---

## 為什麼先教這個？

U01 讓程式跑起來了，但只有文字回應。這堂課讓學員第一次「看到畫面」——用 Nunjucks 做出一個有書籍清單的 HTML 頁面。資料先用 Controller 中的假資料（hardcoded），不需要 DB，**重點是讓學員體驗「改了模板，刷新就看到結果」的即時回饋感**。

---

## 對應檔案

| 檔案 | 角色 |
|------|------|
| `book/book.controller.ts` | MVC 端點（回傳 HTML 頁面），此刻先用假資料 |
| `views/book/list.njk` | 書籍清單頁 |
| `views/book/detail.njk` | 書籍詳情頁 |
| `views/book/form.njk` | 新增 / 編輯表單頁 |
| `views/layouts/main.njk` | 共用版型 |

---

## 核心觀念

### 伺服器端渲染（SSR）

Nunjucks 在 server 端組好完整 HTML 再送出，不需要前端框架（React / Vue）。瀏覽器收到的就是一般的 HTML，任何裝置都能直接看。

### `@Render()` 裝飾器 — 回傳模板

與 U01 的 REST 端點不同，使用 `@Render()` 會將回傳的物件注入模板上下文：

```typescript
import { Controller, Get, Render } from '@nestjs/common';

@Controller('books')
export class BookController {

  /** 書籍清單頁 */
  @Get()
  @Render('book/list')  // 渲染 views/book/list.njk
  list() {
    // 假資料——後面 U06 會換成真實 DB 資料
    const books = [
      { id: 1, title: 'Clean Code', author: 'Robert C. Martin', isbn: '978-0132350884', stock: 5 },
      { id: 2, title: 'Effective TypeScript', author: 'Dan Vanderkam', isbn: '978-1492053743', stock: 3 },
      { id: 3, title: 'NestJS in Action', author: 'Jay Bell', isbn: '978-1617298035', stock: 8 },
    ];
    return { books };  // 資料丟給模板
  }

  /** 書籍詳情頁 */
  @Get(':id')
  @Render('book/detail')
  detail(@Param('id') id: string) {
    const book = { id: 1, title: 'Clean Code', author: 'Robert C. Martin', isbn: '978-0132350884', stock: 5 };
    return { book };
  }
}
```

> **`return { books }`** 把資料從 Controller 丟給模板引擎。模板中用 `{{ books }}` 或 `{% for book in books %}` 就能取到這個資料。

### Nunjucks 關鍵語法 — `list.njk`

```html
{% extends 'layouts/main.njk' %}

{% block content %}
<h1>書籍清單</h1>
<p><a href="/books/new">新增書籍</a></p>

<table border="1" cellpadding="6">
  <thead>
    <tr><th>ID</th><th>書名</th><th>作者</th><th>ISBN</th><th>庫存</th><th>操作</th></tr>
  </thead>
  <tbody>
    <!-- {% for %} — 迴圈渲染，類似 for-each -->
    {% for book in books %}
    <tr>
      <td>{{ book.id }}</td>
      <!-- {{ variable }} — 輸出變數，自動 XSS 防護 -->
      <td><a href="/books/{{ book.id }}">{{ book.title }}</a></td>
      <td>{{ book.author }}</td>
      <td>{{ book.isbn }}</td>
      <td>{{ book.stock }}</td>
      <td>
        <a href="/books/{{ book.id }}/edit">編輯</a>
        <form action="/books/{{ book.id }}/delete" method="post" style="display:inline"
              onsubmit="return confirm('確定要刪除這本書嗎？')">
          <button type="submit">刪除</button>
        </form>
      </td>
    </tr>
    {% else %}
    <!-- {% else %} 在迴圈為空時顯示 -->
    <tr>
      <td colspan="6">目前無書籍資料</td>
    </tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

### Nunjucks 語法速查

| 語法 | 說明 | 範例 |
|------|------|------|
| `{{ variable }}` | 輸出變數（自動 XSS 防護） | `{{ book.title }}` |
| `{% for item in items %}` | 迴圈渲染 | `{% for book in books %}` |
| `{% if condition %}` | 條件判斷 | `{% if book.stock == 0 %}` |
| `{% extends 'path' %}` | 繼承版型 | `{% extends 'layouts/main.njk' %}` |
| `{% block name %}` | 定義/覆寫區塊 | `{% block content %}` |
| `{% include 'path' %}` | 引入子模板 | `{% include 'partials/nav.njk' %}` |

### 詳情頁 — `detail.njk`

```html
{% extends 'layouts/main.njk' %}

{% block content %}
<h1>{{ book.title }}</h1>
<dl>
  <dt>作者</dt>   <dd>{{ book.author }}</dd>
  <dt>ISBN</dt>   <dd>{{ book.isbn }}</dd>
  <dt>庫存</dt>   <dd>{{ book.stock }}</dd>
  <dt>建立時間</dt><dd>{{ book.createdAt | date('YYYY-MM-DD HH:mm') }}</dd>
</dl>
<a href="/books">返回清單</a>
{% endblock %}
```

### 共用版型 — `layouts/main.njk`

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>{% block title %}書籍管理系統{% endblock %}</title>
</head>
<body>
  <nav>
    <a href="/">首頁</a> |
    <a href="/books">書籍清單</a>
  </nav>
  <main>
    {% block content %}{% endblock %}
  </main>
</body>
</html>
```

### main.ts 設定 Nunjucks

```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';
import * as nunjucks from 'nunjucks';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);

  // 設定 Nunjucks 作為模板引擎
  const express = app.getHttpAdapter().getInstance();
  const viewsPath = join(__dirname, '..', 'views');
  nunjucks.configure(viewsPath, { express, autoescape: true });
  app.setViewEngine('njk');
  app.setBaseViewsDir(viewsPath);

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### 此刻不需要 DB

Controller 直接建立假的書籍清單（`const books = [...]`）丟給模板，先專注學 Nunjucks 語法。

> **講師提示**：先 demo 假資料版本，讓學員專心學 Nunjucks 語法。後面 U06 串接 DB 時，只要改 Controller 的資料來源，頁面不用動——這就是模板引擎的價值。

---

## 動手做

### 必做

**1. 觀察 Controller → 模板的資料傳遞**

閱讀 `BookController.list()` 如何回傳資料，對照 `list.njk` 中的 `{% for %}` 如何渲染每一行。

**2. 寫一個假資料版本的書籍清單**

在 Controller 方法中回傳包含 3 本假書的清單，瀏覽 `http://localhost:3000/books` 看到表格：

```typescript
@Get()
@Render('book/list')
list() {
  const books = [
    { id: 1, title: 'Clean Code', author: 'Robert C. Martin', isbn: '978-0132350884', stock: 5 },
    { id: 2, title: 'Effective TypeScript', author: 'Dan Vanderkam', isbn: '978-1492053743', stock: 3 },
    { id: 3, title: 'NestJS in Action', author: 'Jay Bell', isbn: '978-1617298035', stock: 8 },
  ];
  return { books };
}
```

**3. 點進詳情頁**

點進某本書的連結，觀察詳情頁如何用 `{{ book.title }}` 顯示各欄位。

### 延伸挑戰

1. 修改 `list.njk`，在表格加一欄「狀態」，當庫存 = 0 時顯示紅字「缺貨」：

```html
<td>
  {% if book.stock == 0 %}
    <span style="color: red;">缺貨</span>
  {% else %}
    <span style="color: green;">有庫存</span>
  {% endif %}
</td>
```

---

## 踩坑提示

| 現象 | 原因 | 解法 |
|------|------|------|
| 500 找不到模板 | `@Render()` 指定的模板路徑錯誤（如 `"books/list"` 而非 `"book/list"`） | 確認 `views/` 下的資料夾名稱與回傳字串完全一致 |
| 頁面顯示 `{{ books }}` 原始文字 | 檔案副檔名不是 `.njk` 或 Nunjucks 未正確設定 | 確認 `main.ts` 中的 view engine 設定；確認副檔名為 `.njk` |
| 模板變更後頁面沒更新 | Nunjucks 有內部快取 | 開發環境設定 `noCache: true`；或重啟 `npm run start:dev` |

---

## 驗收標準（DoD）

- [ ] 能在瀏覽器看到一個有書籍清單的 HTML 頁面（即使資料是假的）
- [ ] 能說出 `{% for %}` 的用途——迴圈渲染多筆資料
- [ ] 能說出 `{{ variable }}` 的用途——輸出變數值（自動 XSS 防護）
- [ ] 能說出 `@Render()` 裝飾器的作用——指定要渲染的模板
- [ ] 能解釋 `@Render()` 與普通 Controller 方法（回傳 JSON）的差異
