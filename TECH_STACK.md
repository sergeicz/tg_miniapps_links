# 🛠️ Tech Stack

## Frontend

- **HTML5** - структура
- **CSS3** - стили с glassmorphism эффектами
- **Vanilla JavaScript** - логика без фреймворков
- **Telegram WebApp API** - интеграция с Telegram

## Backend

### Cloudflare Workers
- **Runtime:** V8 isolates (edge computing)
- **Size:** ~197 KiB (compressed: 39 KiB)
- **Cold start:** ~18ms
- **Memory:** Efficient, serverless

### Библиотеки

#### grammY (`grammy`)
- **Версия:** Latest
- **Назначение:** Telegram Bot framework
- **Преимущества:**
  - Оптимизирован для edge runtime (Deno, Cloudflare Workers)
  - Современный TypeScript с отличной типизацией
  - Middleware паттерн из коробки
  - Удобные хелперы (InlineKeyboard, Context API)
  - Минимальный размер (~8 packages)

#### Google Sheets API
- `google-auth-library` - JWT аутентификация
- Direct REST API calls - без тяжелых SDK
- Service Account authentication

### Хранилище

#### Cloudflare Workers KV
- **Назначение:** Хранение состояния создания рассылок
- **TTL:** 3600 секунд (1 час)
- **API:** Simple key-value interface

#### Google Sheets
- **Назначение:** Основная база данных
- **Листы:**
  - `users` - пользователи приложения
  - `partners` - партнерские ссылки
  - `admins` - список администраторов
  - `clicks` - статистика кликов
- **Преимущества:**
  - Бесплатно
  - Визуальное управление
  - API доступ
  - Легко редактировать

## Deployment

### Frontend
- **Платформа:** GitHub Pages
- **URL:** `https://<username>.github.io/<repo>/frontend/`
- **Deploy:** Automatic on push to main

### Backend
- **Платформа:** Cloudflare Workers
- **URL:** `https://<worker-name>.workers.dev`
- **Deploy:** `npx wrangler deploy`
- **Webhook:** `https://<worker-name>.workers.dev/bot<TOKEN>`

## API

### REST Endpoints
```
GET  /api/health        - Health check
GET  /api/partners      - Список партнеров
POST /api/click         - Регистрация клика
POST /api/user          - Регистрация пользователя
POST /api/me            - Проверка админа
GET  /api/subscribers   - Статистика подписчиков
```

### Bot Webhook
```
POST /bot<BOT_TOKEN>    - Telegram webhook (grammY handler)
```

## Development Tools

- **Node.js** - runtime для разработки
- **npm** - package manager
- **Wrangler** - Cloudflare Workers CLI
- **Git** - version control

## Code Quality

- **No TypeScript** - Pure JavaScript для простоты
- **No build step** - Прямой деплой без компиляции
- **ESM modules** - Современные ES6+ модули
- **Async/await** - Чистая асинхронность

## Performance

### Bundle Size Comparison

| Version | Size | Gzip | Холодный старт |
|---------|------|------|----------------|
| Vanilla (old) | 771 KiB | 144 KiB | 35ms |
| **grammY (new)** | **197 KiB** | **39 KiB** | **18ms** |
| **Improvement** | **-74%** | **-73%** | **-49%** |

### Why grammY?

1. **Специально для edge runtime** - работает в Cloudflare Workers из коробки
2. **Легкий** - только необходимый функционал, без bloat
3. **Современный** - TypeScript, ESM, async/await
4. **Удобный API** - чистый код, легко читать и поддерживать

### Code Comparison

**Before (Vanilla):**
```javascript
if (update.message?.text === '/start') {
  await fetch(`https://api.telegram.org/bot${TOKEN}/sendMessage`, {
    method: 'POST',
    body: JSON.stringify({
      chat_id: chatId,
      text: 'Hello',
      reply_markup: { inline_keyboard: [[...]] }
    })
  });
}
```

**After (grammY):**
```javascript
bot.command('start', async (ctx) => {
  const keyboard = new InlineKeyboard().text('Button', 'data');
  await ctx.reply('Hello', { reply_markup: keyboard });
});
```

## Security

- **Environment Variables:** Все секреты в Cloudflare Secrets
- **CORS:** Настроено для безопасного доступа
- **Service Account:** Google Sheets доступ через JWT
- **Webhook:** Telegram webhook на защищенном URL

## Scalability

- **Serverless:** Автоматическое масштабирование
- **Edge Network:** Cloudflare CDN (194+ города)
- **No Database:** Нет узких мест с БД
- **KV Storage:** Глобально распределенное хранилище

## Cost

- **Frontend:** Free (GitHub Pages)
- **Backend:** Free tier (100,000 requests/day)
- **Database:** Free (Google Sheets)
- **KV Storage:** Free tier (100,000 reads/day)

**Total:** $0/month для большинства проектов 🎉
