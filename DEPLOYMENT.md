# 🚀 Deployment Guide

## 📋 Architecture Overview

```
Frontend (GitHub Pages) → Cloudflare Worker (API + Bot with grammY) → Google Sheets
```

**One Worker handles everything:**
- REST API for Mini App
- Telegram Bot with grammY
- Google Sheets integration
- Broadcast state (KV storage)

---

## 🛠️ Prerequisites

- GitHub account (for frontend hosting)
- Cloudflare account (for backend + bot)
- Google Cloud account (for Sheets API)
- Telegram Bot Token (from @BotFather)
- Node.js installed locally

---

## 📝 Step 1: Google Sheets Setup

### 1.1 Create Spreadsheet

Create a new Google Sheet with **4 tabs**:

**Tab 1: `users`**
```
telegram_id | username | first_name | date_added | bot_started
```

**Tab 2: `partners`**
```
title | url | category
```
Example data:
```
Amazon | https://amazon.com/ref=123 | Shopping
Udemy | https://udemy.com/ref=789 | Education
```

**Tab 3: `admins`**
```
username | telegram_id
```
Add your username WITHOUT @ symbol:
```
your_username | 123456789
```

**Tab 4: `clicks`**
```
telegram_id | url | timestamp
```

### 1.2 Create Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create new project or select existing
3. Enable **Google Sheets API**
4. Create **Service Account**:
   - IAM & Admin → Service Accounts → Create
   - Name: `telegram-miniapp`
   - Role: None needed
5. Create **JSON Key**:
   - Click on service account → Keys → Add Key → JSON
   - Download `credentials.json`

### 1.3 Share Spreadsheet

1. Copy the `client_email` from `credentials.json`
2. Share your Google Sheet with this email (Editor access)
3. Copy the Spreadsheet ID from URL

---

## 🤖 Step 2: Telegram Bot Setup

### 2.1 Create Bot

1. Message [@BotFather](https://t.me/BotFather)
2. Send `/newbot`
3. Follow instructions
4. Copy the Bot Token

### 2.2 Create Mini App

1. Message @BotFather again
2. Send `/newapp`
3. Select your bot
4. Follow instructions
5. URL will be set later

---

## ☁️ Step 3: Cloudflare Worker Deployment

### 3.1 Install Dependencies

```bash
cd worker
npm install
```

This installs:
- `grammy` - Telegram bot framework
- `google-auth-library` - Google API auth
- `wrangler` - Cloudflare CLI

### 3.2 Login to Cloudflare

```bash
npx wrangler login
```

### 3.3 Create KV Namespace

```bash
npx wrangler kv:namespace create BROADCAST_STATE
```

Copy the ID and update `wrangler.toml`:
```toml
[[kv_namespaces]]
binding = "BROADCAST_STATE"
id = "your-kv-id-here"
```

### 3.4 Add Secrets

```bash
# Bot token from BotFather
npx wrangler secret put BOT_TOKEN

# Spreadsheet ID from Google Sheets URL
npx wrangler secret put SHEET_ID

# Content of credentials.json (paste entire JSON)
npx wrangler secret put CREDENTIALS_JSON

# Frontend URL (will be set later, use placeholder)
npx wrangler secret put WEBAPP_URL
```

### 3.5 Deploy Worker

```bash
npx wrangler deploy
```

Copy the Worker URL (e.g., `https://telegram-miniapp-api.workers.dev`)

### 3.6 Set Telegram Webhook

```bash
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=https://your-worker.workers.dev/bot<YOUR_BOT_TOKEN>"
```

Verify:
```bash
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getWebhookInfo"
```

---

## 🌐 Step 4: Frontend Deployment

### 4.1 Update Config

Edit `frontend/index.html`:

```javascript
const CONFIG = {
  API_URL: 'https://your-worker.workers.dev'  // Your Worker URL
};
```

### 4.2 Push to GitHub

```bash
git add .
git commit -m "Update API URL"
git push origin main
```

### 4.3 Enable GitHub Pages

1. Go to repo → Settings → Pages
2. Source: Deploy from branch
3. Branch: `main`, folder: `/` (root)
4. Save

Wait 1-2 minutes, then access:
```
https://<username>.github.io/<repo>/frontend/
```

### 4.4 Update WEBAPP_URL Secret

```bash
npx wrangler secret put WEBAPP_URL
# Enter: https://<username>.github.io/<repo>/frontend/
```

### 4.5 Update Mini App URL in BotFather

1. Message @BotFather
2. Send `/myapps`
3. Select your app
4. Edit → Edit Web App URL
5. Enter your GitHub Pages URL

---

## ✅ Step 5: Test Everything

### 5.1 Test Worker

```bash
curl https://your-worker.workers.dev/api/health
```

Expected:
```json
{
  "status": "ok",
  "version": "3.0.0-grammy",
  "mode": "production_with_grammy"
}
```

### 5.2 Test Bot

1. Open your bot in Telegram
2. Send `/start`
3. Should see:
   - Welcome message
   - "🚀 Открыть Mini App" button
   - "⚙️ Админ-панель" button (if you're admin)

### 5.3 Test Mini App

1. Click "🚀 Открыть Mini App"
2. Should see partner links
3. Click any link
4. Should open and register click

### 5.4 Test Admin Panel

1. Click "⚙️ Админ-панель"
2. Try "📊 Статистика"
3. Try "📢 Рассылка":
   - Enter title
   - Enter subtitle
   - Upload image or URL
   - Add button
   - Preview should show with image
   - Send to all users

---

## 🐛 Troubleshooting

### Bot doesn't respond

**Check webhook:**
```bash
curl "https://api.telegram.org/bot<TOKEN>/getWebhookInfo"
```

Should show:
- `url`: Your worker URL
- `pending_update_count`: 0

**Check logs:**
```bash
cd worker
npx wrangler tail
```

### Admin panel not showing

**Check:**
1. Your username is in `admins` sheet (WITHOUT @)
2. OR your telegram_id is in `admins` sheet
3. Sheet is shared with service account email

**Debug:**
```bash
# Check logs
npx wrangler tail

# Look for "Admin check" messages
```

### Images not showing in broadcast

**Make sure:**
1. You uploaded image as file (not URL)
2. OR URL is direct image link (not webpage)
3. Check Worker logs for errors

### Frontend not loading data

**Check:**
1. CORS is enabled in Worker (it is by default)
2. API_URL in frontend/index.html is correct
3. Worker is deployed and accessible

---

## 📊 Monitoring

### Check Worker Logs

```bash
cd worker
npx wrangler tail
```

### Check Analytics

1. Cloudflare Dashboard
2. Workers & Pages → your-worker
3. Metrics tab

### Check Google Sheets

Open your spreadsheet to see:
- New users
- Clicks
- Real-time data

---

## 🔄 Updates

### Update Worker

```bash
cd worker
# Edit index.js
npx wrangler deploy
```

### Update Frontend

```bash
# Edit frontend/index.html
git add .
git commit -m "Update frontend"
git push
# GitHub Pages updates automatically in 1-2 minutes
```

### Update Bot Commands

1. Edit `worker/index.js`
2. Find bot handlers (e.g., `bot.command('start')`)
3. Make changes
4. Deploy: `npx wrangler deploy`

---

## 🎉 Success!

Your Telegram Mini App is now live with:

✅ Beautiful frontend on GitHub Pages  
✅ Powerful backend with grammY on Cloudflare Workers  
✅ Admin panel in Telegram bot  
✅ Broadcast system with image support  
✅ Google Sheets as free database  
✅ All for $0/month!  

**Next steps:**
- Add more partner links in Google Sheets
- Test broadcast system
- Share your Mini App with users!

---

## 📚 More Resources

- [grammY Documentation](https://grammy.dev/)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [Project GitHub](https://github.com/your-repo)

---

**Need help?** Check [TECH_STACK.md](TECH_STACK.md) for technical details.
