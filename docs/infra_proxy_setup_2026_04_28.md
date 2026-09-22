# WinePool — Инфраструктура прокси (28.04.2026)

## Проблема

Supabase работает на серверах AWS (us-east-1). Часть российских провайдеров блокирует IP-диапазоны AWS через DPI. Приложение не открывается без VPN у части пользователей на мобильном интернете.

## Решение: Yandex API Gateway → Cloud Function → Supabase

Все запросы из Flutter-приложения идут через `api.winepool.ru` — кастомный домен на Yandex API Gateway. IP-адреса Yandex не блокируются российскими провайдерами.

### Схема

```
Flutter app
    ↓ HTTPS
api.winepool.ru  (Yandex API Gateway, custom domain)
    ↓ invoke
supabase-proxy  (Yandex Cloud Function, Node.js 22)
    ↓ HTTPS
wnzewejxrhjnjvyrshzp.supabase.co  (Supabase / AWS)
```

### Почему Cloud Function, а не прямой HTTP-прокси в API Gateway

Yandex API Gateway (тип интеграции `http`) не форвардит заголовок `Content-Type` — ни автоматически, ни через явное объявление в OpenAPI спеке (Content-Type является запрещённым параметром в `in: header` по стандарту OpenAPI 3.0, и Yandex строго следует этому). Без Content-Type Supabase GoTrue не может распарсить JSON-тело auth-запросов и возвращает `unsupported_grant_type`.

Cloud Function получает полный HTTP-запрос включая все заголовки и тело, форвардит их к Supabase без изменений.

---

## Компоненты

### 1. Yandex Cloud Function `supabase-proxy`

- **ID функции:** `d4e4u9ggkorardrbl3gu`
- **Среда:** Node.js 22
- **Точка входа:** `index.handler`
- **Таймаут:** 10 секунд
- **Память:** 128 МБ
- **Публичная:** да (unauthenticated invocations allowed)

**Код функции (`index.js`):**

```javascript
const https = require('https');

const SUPABASE_HOST = 'wnzewejxrhjnjvyrshzp.supabase.co';
const SKIP_HEADERS = new Set(['host', 'x-forwarded-for', 'x-real-ip', 'x-forwarded-proto', 'x-request-id']);

module.exports.handler = async (event) => {
  // event.url = фактический путь + query string ("/auth/v1/token?grant_type=password")
  // event.path = шаблон маршрута ("/{path+}") — НЕ использовать!
  const targetPath = event.url || '/';

  const headers = {};
  for (const [k, v] of Object.entries(event.headers || {})) {
    if (!SKIP_HEADERS.has(k.toLowerCase())) headers[k] = v;
  }
  headers['host'] = SUPABASE_HOST;

  let body;
  if (event.body) {
    body = event.isBase64Encoded ? Buffer.from(event.body, 'base64') : Buffer.from(event.body, 'utf-8');
    headers['content-length'] = body.length.toString();
  }

  return new Promise((resolve) => {
    const req = https.request(
      { hostname: SUPABASE_HOST, port: 443, path: targetPath, method: event.httpMethod || 'GET', headers },
      (res) => {
        const chunks = [];
        res.on('data', (c) => chunks.push(c));
        res.on('end', () => {
          const buf = Buffer.concat(chunks);
          const respHeaders = {};
          for (const [k, v] of Object.entries(res.headers)) {
            if (k.toLowerCase() !== 'transfer-encoding')
              respHeaders[k] = Array.isArray(v) ? v.join(', ') : v;
          }
          respHeaders['access-control-allow-origin'] = '*';
          respHeaders['access-control-allow-headers'] = 'apikey, Authorization, Content-Type, Prefer, X-Client-Info';
          respHeaders['access-control-allow-methods'] = 'GET, POST, PUT, PATCH, DELETE, OPTIONS';
          resolve({ statusCode: res.statusCode, headers: respHeaders, body: buf.toString('base64'), isBase64Encoded: true });
        });
      }
    );
    req.on('error', (e) => {
      resolve({ statusCode: 502, headers: { 'content-type': 'application/json' }, body: JSON.stringify({ error: e.message }), isBase64Encoded: false });
    });
    if (body) req.write(body);
    req.end();
  });
};
```

**Важный нюанс:** В Yandex Cloud Functions при вызове через API Gateway:
- `event.path` = шаблон маршрута (`/{path+}`), **не** реальный путь
- `event.url` = реальный путь + query string → использовать это

### 2. Yandex API Gateway

- **ID шлюза:** `d5dkn2q6908e4j1arc46`
- **URL шлюза:** `https://d5dkn2q6908e4j1arc46.y3q8o1jq.apigw.yandexcloud.net`
- **Кастомный домен:** `api.winepool.ru`
- **SSL-сертификат:** Let's Encrypt через Yandex Certificate Manager (auto-renew)

**OpenAPI спецификация:**

```yaml
openapi: "3.0.0"
info:
  title: Supabase Proxy
  version: 1.0.0
paths:
  /{path+}:
    parameters:
      - name: path
        in: path
        required: true
        schema:
          type: string
    get:
      operationId: proxyGet
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
    post:
      operationId: proxyPost
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
    put:
      operationId: proxyPut
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
    patch:
      operationId: proxyPatch
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
    delete:
      operationId: proxyDelete
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
    options:
      operationId: proxyOptions
      responses: {'200': {description: OK}}
      x-yc-apigateway-integration:
        type: cloud_functions
        function_id: d4e4u9ggkorardrbl3gu
```

### 3. DNS (Cloudflare)

| Тип | Имя | Значение |
|-----|-----|---------|
| CNAME | `api` | `d5dkn2q6908e4j1arc46.y3q8o1jq.apigw.yandexcloud.net` |

SSL-сертификат выпущен через DNS-валидацию Let's Encrypt:
- CNAME `_acme-challenge.api` → `fpqaufr885etot290p7t.cm.yandexcloud.net` (Yandex Certificate Manager)

### 4. Flutter-приложение (`lib/main.dart`)

```dart
await Supabase.initialize(
  url: 'https://api.winepool.ru',  // прокси, не напрямую к Supabase
  anonKey: '...',
);
```

Все изображения из Supabase Storage также проксируются через `api.winepool.ru` функцией `proxyStorageUrl()` в `lib/core/utils/storage_url.dart`.

---

## Текущие ограничения и post-MVP план

**Ограничения:**
- Лишний hop (API Gateway → Cloud Function → Supabase) добавляет ~50–100ms latency
- Cold start Cloud Function: ~25ms на первый запрос после паузы

**Почему приемлемо для MVP:** задержка незаметна пользователю, решение надёжно работает.

> Историческая пометка 13.08.2026: упоминание Timeweb ниже было планом до выбора инфраструктуры. Текущий сайт и VPS находятся у **Host-Food**; актуальный источник истины — `docs/hosting_host_food_runbook_2026_08_13.md`.

**Post-MVP (рефакторинг):** заменить Cloud Function на nginx reverse proxy на Host-Food VPS. Nginx форвардит все заголовки нативно, убирает лишний hop, упрощает архитектуру до 2 уровней вместо 3.

```nginx
# post-MVP: nginx на Host-Food VPS, субдомен api.winepool.ru
location / {
    proxy_pass https://wnzewejxrhjnjvyrshzp.supabase.co;
    proxy_set_header Host wnzewejxrhjnjvyrshzp.supabase.co;
    proxy_pass_header *;
}
```
