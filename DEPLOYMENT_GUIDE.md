# 🚀 РУКОВОДСТВО ПО РАЗВЕРТЫВАНИЮ SUBCLOUDY

## 📋 СОДЕРЖАНИЕ
1. [Загрузка кода в GitHub](#загрузка-в-github)
2. [Деплой Frontend (GitHub Pages)](#деплой-frontend)
3. [Деплой Backend (Railway/Render)](#деплой-backend)
4. [Настройка домена](#настройка-домена)

---

## 1️⃣ ЗАГРУЗКА В GITHUB

### ✅ ШАГ 1: Push кода

```bash
cd D:\project\Subcloudy

# Код уже закоммичен, теперь push
git push -u origin main
```

**Важно:** При первом push GitHub может запросить авторизацию.

---

## 2️⃣ ДЕПЛОЙ FRONTEND (GitHub Pages) 

### Вариант A: GitHub Pages (БЕСПЛАТНО)

#### ШАГ 1: Создать production build

```bash
cd frontend

# Настроить базовый URL для GitHub Pages
# В vite.config.js добавить:
base: '/market/'

# Собрать проект
npm run build
```

#### ШАГ 2: Настроить GitHub Pages

1. Перейти на https://github.com/Ivan14044/market/settings/pages
2. Source: **GitHub Actions**
3. Создать файл `.github/workflows/deploy-frontend.yml`:

```yaml
name: Deploy Frontend to GitHub Pages

on:
  push:
    branches: [ main ]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: cd frontend && npm install
        
      - name: Build
        run: cd frontend && npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
        
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: './frontend/dist'
          
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v2
```

**Сайт будет доступен:** `https://ivan14044.github.io/market/`

---

### Вариант B: Vercel (БЕСПЛАТНО, РЕКОМЕНДУЕТСЯ)

**Проще и быстрее чем GitHub Pages!**

#### ШАГ 1: Установить Vercel CLI

```bash
npm i -g vercel
```

#### ШАГ 2: Деплой

```bash
cd frontend
vercel
```

Следуйте инструкциям в терминале.

**Сайт будет доступен:** `https://your-project.vercel.app`

**Преимущества Vercel:**
- ✅ Автоматический HTTPS
- ✅ Быстрый CDN
- ✅ Автодеплой при push
- ✅ Custom domain support
- ✅ Бесплатный tier

---

### Вариант C: Netlify (БЕСПЛАТНО)

#### ШАГ 1: Установить Netlify CLI

```bash
npm i -g netlify-cli
```

#### ШАГ 2: Деплой

```bash
cd frontend
netlify deploy --prod
```

**Сайт будет доступен:** `https://your-project.netlify.app`

---

## 3️⃣ ДЕПЛОЙ BACKEND (Laravel API)

⚠️ **Важно:** GitHub Pages НЕ поддерживает PHP/Laravel!
Backend нужно разместить на отдельном хостинге.

### Вариант A: Railway (БЕСПЛАТНО, РЕКОМЕНДУЕТСЯ)

#### ШАГ 1: Зарегистрироваться на Railway
https://railway.app/

#### ШАГ 2: Создать проект

1. New Project → Deploy from GitHub repo
2. Выбрать репозиторий `Ivan14044/market`
3. Root Directory: `backend`

#### ШАГ 3: Настроить переменные окружения

В Railway Dashboard → Variables:

```env
APP_NAME=SubCloudy
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:YOUR_APP_KEY_HERE
APP_URL=https://your-backend.up.railway.app

DB_CONNECTION=mysql
# Railway автоматически предоставит MySQL

FRONTEND_URL=https://your-frontend.vercel.app

SESSION_DRIVER=database
CACHE_DRIVER=database

# Добавить все ваши API ключи
GOOGLE_CLIENT_ID=...
TELEGRAM_BOT_TOKEN=...
```

#### ШАГ 4: Генерация APP_KEY

Локально выполнить:
```bash
cd backend
php artisan key:generate --show
```

Скопировать значение в Railway.

**Backend будет доступен:** `https://your-backend.up.railway.app`

---

### Вариант B: Render (БЕСПЛАТНО)

https://render.com/

1. New → Web Service
2. Connect GitHub repo `Ivan14044/market`
3. Root Directory: `backend`
4. Build Command: `composer install --no-dev`
5. Start Command: `php artisan serve --host=0.0.0.0 --port=$PORT`

---

### Вариант C: Heroku (Платный, $5/месяц)

```bash
# Установить Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

cd backend
heroku create subcloudy-api
git push heroku main
heroku run php artisan migrate
```

---

## 4️⃣ СВЯЗАТЬ FRONTEND И BACKEND

### В Frontend (.env.production):

```env
VITE_API_URL=https://your-backend.up.railway.app/api
```

### В Backend (.env):

```env
FRONTEND_URL=https://your-frontend.vercel.app
```

Обновить CORS в `backend/config/cors.php`:

```php
'allowed_origins' => [
    env('FRONTEND_URL'),
    'https://your-frontend.vercel.app',
],
```

---

## 5️⃣ НАСТРОЙКА ДОМЕНА (ОПЦИОНАЛЬНО)

### Для Frontend (Vercel):
1. Settings → Domains
2. Добавить свой домен (например, subcloudy.com)
3. Настроить DNS записи

### Для Backend (Railway):
1. Settings → Networking
2. Generate Domain или добавить свой

---

## 🎯 РЕКОМЕНДУЕМАЯ КОНФИГУРАЦИЯ

**Лучшая комбинация (100% бесплатно):**

1. **Frontend:** Vercel
   - URL: `https://subcloudy.vercel.app`
   - Автодеплой из GitHub
   
2. **Backend:** Railway
   - URL: `https://subcloudy-api.up.railway.app`
   - Бесплатный tier: 500 часов/месяц
   
3. **База данных:** Railway MySQL (входит в бесплатный tier)

---

## 📝 CHECKLIST ПЕРЕД ДЕПЛОЕМ

### Backend:
- [ ] Создать `.env.production` с правильными настройками
- [ ] APP_DEBUG=false
- [ ] APP_ENV=production
- [ ] SESSION_SECURE_COOKIE=true
- [ ] Настроить CORS
- [ ] Запустить миграции
- [ ] Создать админа

### Frontend:
- [ ] Обновить VITE_API_URL на production backend
- [ ] Собрать production build
- [ ] Проверить роуты
- [ ] Проверить API calls

---

## 🔐 БЕЗОПАСНОСТЬ

### Важные настройки для production:

```env
# backend/.env
APP_DEBUG=false
APP_ENV=production
SESSION_SECURE_COOKIE=true
SANCTUM_STATEFUL_DOMAINS=your-frontend.vercel.app
```

---

## 🚀 БЫСТРЫЙ СТАРТ (Рекомендуемый путь)

### 1. Push в GitHub (УЖЕ ГОТОВ)
```bash
git push -u origin main
```

### 2. Деплой Frontend на Vercel
```bash
cd frontend
npm i -g vercel
vercel --prod
```

### 3. Деплой Backend на Railway
1. Зайти на railway.app
2. New Project → Deploy from GitHub
3. Выбрать репозиторий
4. Root directory: `backend`
5. Добавить переменные окружения

### 4. Обновить API URL
```bash
# В frontend создать .env.production
echo "VITE_API_URL=https://your-backend.up.railway.app/api" > .env.production

# Пересобрать
npm run build
vercel --prod
```

---

## 📞 ПОДДЕРЖКА

### Документация:
- [Vercel Docs](https://vercel.com/docs)
- [Railway Docs](https://docs.railway.app)
- [Laravel Deployment](https://laravel.com/docs/10.x/deployment)

### Если возникли проблемы:
1. Проверьте логи в Railway/Vercel
2. Проверьте CORS настройки
3. Проверьте переменные окружения
4. Проверьте миграции БД

---

## ✅ ИТОГОВАЯ СХЕМА

```
GitHub Repository (Ivan14044/market)
    ↓
    ├── Frontend → Vercel → https://subcloudy.vercel.app
    └── Backend → Railway → https://subcloudy-api.up.railway.app
```

**Пользователи будут заходить на:** `https://subcloudy.vercel.app` 🌐

---

**Создано для SubCloudy** ❤️

