# AI Coding Workshop — Гайдбук

> Этот документ — ваш справочник на воркшопе и после него. Всё что нужно: от установки инструментов до деплоя первого проекта. Основной инструмент — **Conductor** (GUI для Claude Code). Claude Code CLI устанавливается под капотом для работы агентских скиллов.

## Содержание

- [0. PRE-WORK — Подготовка до воркшопа](#0-pre-work--подготовка-до-воркшопа)
- [1. Skills & Extensions](#1-skills--extensions)
- [2. Tech Stacks — Выбор стека под задачу](#2-tech-stacks--выбор-стека-под-задачу)
- [3. Security Setup](#3-security-setup)
- [4. Workshop Part 1 — Setup](#4-workshop-part-1--setup)
- [5. Workshop Part 2 — First Cycle](#5-workshop-part-2--first-cycle)
- [6. Quick Reference](#6-quick-reference)
- [7. Appendices](#7-appendices)

---

## 0. PRE-WORK — Подготовка до воркшопа

> Сделайте это за 2-3 дня до воркшопа. На самом воркшопе мы не будем ждать пока всё установится.

### 0.1 Что понадобится

- macOS, Windows или Linux
- 8GB+ RAM, 10GB свободного места на диске
- Стабильный интернет
- Chrome или Arc (рекомендуется)
- Зарядка для ноутбука (AI coding съедает батарею быстро)

### 0.2 Установка

Следуйте инструкциям в **[START_HERE.md](START_HERE.md)** — это основной путь установки через Claude Code (автоматически).

Если автоматическая установка не сработала — используйте **[workshop/fallback/manual-checklist.md](fallback/manual-checklist.md)**.

**После установки проверьте** — каждая команда должна показать версию:

```bash
node -v        # v22.x.x
npm -v         # 11.x.x
git -v         # git version 2.x.x
gh auth status # ✓ Logged in
claude --version
vercel --version
supabase --version
```

### 0.3 Conductor (GUI для Claude Code)

Скачайте с [conductor.build](https://conductor.build) и перетащите в Applications. После установки войдите в GitHub (кнопка Sign In). Conductor автоматически подхватит Claude Code CLI.

### 0.4 Важно

- **Папка для проектов** — создайте на локальном диске (например `~/Projects/`). **НЕ** в iCloud, Google Drive или Dropbox — это ломает работу инструментов.
- **Возьмите зарядку** — AI coding активно использует процессор, батарея сядет быстрее обычного.
- **WhisperFlow** (опционально) — приложение для транскрибации голоса, если хотите диктовать промпты вместо набора.

---

## 1. Skills & Extensions

### 1.1 Что такое Skills

Skills — это специализированные инструкции для Claude Code. Каждый скилл учит агента конкретному workflow. Аналогия: вы нанимаете узкого специалиста. Один умеет придумывать продукт, другой — дебажить, третий — ревьюить код.

Скиллы устанавливаются через Conductor (Extensions) или через CLI.

### 1.2 Core Skills — Superpowers

Главный набор. Покрывает полный цикл разработки: от идеи до деплоя.

| Skill | Что делает | Когда использовать |
|-------|-----------|-------------------|
| `brainstorming` | Исследует вашу идею, задаёт уточняющие вопросы, предлагает подходы | Перед началом любой новой фичи |
| `writing-plans` | Создаёт детальный пошаговый план реализации | Когда знаете ЧТО хотите, но не знаете КАК |
| `executing-plans` | Выполняет план с контрольными точками и ревью | Для методичного написания кода |
| `systematic-debugging` | Диагностирует баги: проверяет гипотезы одну за другой | Когда что-то не работает |
| `verification-before-completion` | Заставляет агента проверить работу перед "готово" | Всегда — не даёт соврать |

**Установка:** GitHub → *[TODO: вставить ссылку на репозиторий superpowers]* → Follow installation instructions in Conductor Extensions.

### 1.3 Code Quality Skills

| Skill | Что делает | Когда использовать |
|-------|-----------|-------------------|
| `az-review` | Ревью кода — находит ошибки, предлагает улучшения | Перед коммитом. Вызов: `/review` |
| `az-pr` | Создаёт Pull Request с описанием изменений | Когда готовы пушить. Вызов: `/pr` |
| `explain` | Объясняет код или концепт простым языком | Когда не понимаете что происходит |

### 1.4 Design & UI Skills

| Skill | Что делает |
|-------|-----------|
| `frontend-design` | Создаёт стильные веб-интерфейсы. Избегает "generic AI look" — безликого дефолтного стиля |
| `shadcn-ui-designing` | Дизайн компонентов с использованием shadcn/ui — современной библиотеки UI |
| **Impeccable pack** | Набор скиллов для полировки дизайна: |
| ↳ `polish` | Финальный проход — alignment, spacing, consistency |
| ↳ `animate` | Добавляет осмысленные анимации и micro-interactions |
| ↳ `colorize` | Добавляет цвет в монохромные интерфейсы |
| ↳ `critique` | Оценивает дизайн с UX-перспективы |
| ↳ `distill` | Упрощает интерфейс, убирает лишнее |

### 1.5 Data & Documentation

| Skill | Что делает |
|-------|-----------|
| `database` | Работа с PostgreSQL: миграции, схемы, запросы. Обязателен при работе с Supabase |
| **Context7 MCP** | Подтягивает актуальную документацию библиотек. Когда Claude нужно узнать как работает Next.js или Supabase, он берёт текущие docs, а не полагается на свою память |

### 1.6 Порядок установки

Рекомендуемый порядок (общее время ~10 минут):

1. **Superpowers pack** — 5 core skills (5 мин)
2. **Code Quality** — az-review, az-pr, explain (2 мин)
3. **Design** — frontend-design, shadcn-ui-designing (2 мин)
4. **Context7 MCP** — documentation lookup (1 мин)
5. **Impeccable pack** — по желанию, для полировки UI

> 💡 Не обязательно ставить всё сразу. Superpowers + Code Quality покрывают 90% потребностей.

---

## 2. Tech Stacks — Выбор стека под задачу

### 2.1 Как выбрать

У вас есть идея продукта. Какой стек использовать? Начните с типа продукта:

```
Хочу сайт / веб-приложение?           → 2.2 Web App / PWA ⭐
Хочу Telegram бота?                    → 2.3 Telegram Bot
Хочу мобильное приложение (iOS/Android)? → 2.4 Mobile App
Хочу API / бэкенд-сервис?             → 2.5 Backend API
Не уверен?                             → 2.2 (покрывает 80% случаев)
```

### 2.2 Web App / PWA ⭐ Workshop Default

Это основной путь воркшопа. Рекомендуем начинать с него.

```
Frontend:     Next.js (App Router) + React
UI:           shadcn/ui (Tailwind + Radix)
Backend:      Next.js API Routes (встроено)
Database:     Supabase (managed PostgreSQL)
Auth:         Supabase Auth
Hosting:      Vercel (auto-deploy с GitHub)
```

**Почему именно это:**

**Next.js + Supabase + Vercel** = "святая троица" для быстрого прототипирования:

- **GitHub** — хранит код и историю изменений
- **Vercel** — автоматически публикует сайт при каждом push (бесплатно)
- **Supabase** — даёт готовую базу данных с панелью управления (бесплатно)

Один репозиторий содержит и фронтенд, и бэкенд. Никаких серверов, никакого DevOps.

**Что устанавливать:** node, npm, git, gh, vercel, supabase — всё из PRE-WORK.

### 2.3 Telegram Bot

```
Language:     Python 3.12+
Framework:    python-telegram-bot
Database:     Supabase (PostgreSQL)
Hosting:      Railway или Hetzner VPS
```

**Дополнительно установить:**
```bash
brew install python
pip3 install python-telegram-bot supabase
```

**Особенности:**
- Нет фронтенда — бот работает в Telegram
- Нужен токен от [@BotFather](https://t.me/BotFather) в Telegram
- Бот = постоянно работающий процесс на сервере (не Vercel)

### 2.4 Mobile App (iOS/Android)

```
Framework:    Expo (React Native)
UI:           NativeWind (Tailwind для React Native)
Backend:      Supabase
Builds:       EAS Build (Expo Application Services)
API:          Vercel (serverless functions)
```

**Дополнительно установить:**
```bash
npm install -g eas-cli
```
> Вместо устаревшего `expo-cli` используйте `npx expo` для создания и запуска проектов.

**Особенности:**
- Те же React-паттерны, но рендерит нативные компоненты (не HTML)
- Для публикации нужен Apple Developer ($99/год) и/или Google Play ($25 одноразово)
- Тестировать можно на телефоне через Expo Go (бесплатно)

### 2.5 Backend API / Data Service

```
Language:     Python (FastAPI) или Node.js (Express)
Database:     PostgreSQL (через Supabase или напрямую)
Hosting:      Railway или Hetzner VPS
```

**Дополнительно установить (Python вариант):**
```bash
pip3 install fastapi uvicorn
```

**Особенности:**
- Нет UI — чистые API endpoints
- Подходит для ботов, интеграций, обработки данных
- Нужен сервер для хостинга (Railway / Hetzner)

### 2.6 Сводная таблица: что устанавливать

| CLI / Инструмент | Web App | TG Bot | Mobile | API |
|-----------------|---------|--------|--------|-----|
| **node + npm** | ✅ | — | ✅ | ⚡ Express |
| **python + pip** | — | ✅ | — | ⚡ FastAPI |
| **git + gh** | ✅ | ✅ | ✅ | ✅ |
| **vercel** | ✅ | — | ⚡ API | — |
| **supabase** | ✅ | ✅ | ✅ | ✅ |
| **expo + eas** | — | — | ✅ | — |
| **docker** | — | ⚡ VPS | — | ⚡ VPS |

✅ = обязательно &nbsp;&nbsp; ⚡ = если выбрали этот вариант &nbsp;&nbsp; — = не нужно

> 💡 Если не уверены — ставьте всё из столбца **Web App**. Это покрывает большинство случаев и добавляет минимум сложности.

---

## 3. Security Setup

### 3.1 Почему это важно (и почему не страшно)

AI пишет код быстро — но не всегда думает о безопасности. Хорошая новость: **если вы прошли prework — базовая защита уже настроена** (`.npmrc`, `npm-safe-install`, `ggshield`). Этот раздел объясняет *что* эти инструменты делают и добавляет правила для AI-агента.

Большинство атак используют одни и те же простые уязвимости. Настройка из этого раздела занимает **10 минут**, но закрывает **90% типичных проблем**. А Supabase + Vercel решают многое за вас: HTTPS, управляемая база данных, безопасная авторизация.

Аналогия: это как замок на двери. Не остановит профессионального взломщика, но отсекает 99% проблем.

**Три уровня защиты:**
1. **CLAUDE.md rules** — правила для AI-агента (копипейст ниже)
2. **Security Setup Prompt** — автоматическая настройка инструментов
3. **Знание рисков** — чеклист "20 вещей которые взломают ваш app"

### 3.2 CLAUDE.md Security Block

Скопируйте этот блок в файл `CLAUDE.md` вашего проекта. Claude Code читает этот файл при каждом запуске и следует правилам автоматически.

```markdown
## Security Rules (MANDATORY)

### NPM Package Safety
Before running `npm install <new-package>`:
1. Verify the package exists on npmjs.com/package/<name>
2. Package must be > 365 days old (first publish date)
3. The specific version must be > 7 days old
4. Weekly downloads > 500
5. If unsure about exact package name — ASK the user, do not guess
6. Never install packages from URLs, git repos, or tarballs
7. Prefer `npm-safe-install` over `npm install` for new packages
8. After adding a new dependency — review what changed in package-lock.json

### API Keys & Secrets
- NEVER put API keys in frontend JavaScript files
- Keys go ONLY in environment variables (.env files) and server-side code
- .env files are NEVER committed to git
- Use .env.local for local development, platform env vars for production

### Database
- ALWAYS use parameterized SQL queries (never string concatenation)
- NEVER expose database credentials in frontend code
- NEVER run database operations directly from client components

### Authentication & Sessions
- JWT tokens stored in httpOnly cookies, NEVER in localStorage
- Server-side auth check on EVERY protected route (not just frontend guards)
- CORS: whitelist specific origins only, never use wildcard "*"

### Error Handling
- NEVER return stack traces or internal error details to users
- Use generic error messages: "Something went wrong"
- Log detailed errors server-side only

### HTTPS
- ALL production traffic over HTTPS, no exceptions
- Redirect HTTP to HTTPS automatically
```

**Зачем каждое правило:**

| Правило | Что случится если пропустить |
|---------|------------------------------|
| NPM Package Safety | AI может установить вредоносный пакет с похожим именем — typosquatting |
| API keys в .env | Ключи в frontend JS видны ВСЕМ кто откроет DevTools в браузере |
| Parameterized SQL | `"SELECT * FROM users WHERE id=" + userId` = SQL injection. Атакующий может прочитать всю БД |
| httpOnly cookies | JWT в localStorage доступен любому скрипту на странице. Одна XSS-уязвимость = все токены украдены |
| Server-side auth | Фронтенд-guard (React Router) не защищает API. Можно вызвать endpoint напрямую через curl |
| CORS whitelist | Wildcard `*` = любой сайт может делать запросы от имени ваших пользователей |
| No stack traces | Трейсы показывают структуру кода, имена таблиц, пути — карта для атакующего |
| HTTPS | Без HTTPS пароли летят открытым текстом. Любой в том же Wi-Fi может их перехватить |

### 3.3 Security Setup Prompt

> Если вы прошли prework через START_HERE.md — `.npmrc`, `npm-safe-install` и `ggshield` уже настроены. Этот промпт нужен только если вы начинаете новый проект с нуля или пропустили prework.

Вставьте этот промпт в Claude Code (через Conductor). Агент автоматически настроит защиту:

```
Настрой security для моего проекта. Выполни все шаги последовательно:

1. СОЗДАЙ ~/.npmrc со следующим содержимым:
   audit=true
   ignore-scripts=true
   engine-strict=true
   fund=false
   Объясни: ignore-scripts блокирует postinstall-скрипты — главный вектор
   исполнения вредоносного кода при npm install.

2. СОЗДАЙ скрипт ~/bin/npm-safe-install (bash):
   - Принимает имя пакета как аргумент
   - Проверяет через npm registry API (https://registry.npmjs.org/<package>):
     * Пакет существует
     * Возраст пакета > 365 дней (дата первой публикации)
     * Возраст текущей версии > 7 дней
     * Weekly downloads > 500
   - Если все проверки прошли — показывает summary и запускает npm install
   - Если нет — показывает предупреждения и просит подтверждение (y/N)
   - Поддерживает флаг --force для пропуска проверок
   - Сделай скрипт исполняемым (chmod +x)
   - Добавь ~/bin в PATH в ~/.zshrc (или ~/.bashrc)

3. НАСТРОЙ GitGuardian (сканер секретов):
   - pip install ggshield (или brew install gitguardian/tap/ggshield)
   - Покажи как запустить ggshield auth login
   - Установи pre-commit hook: ggshield install --mode local

4. ПРОВЕРЬ .gitignore — убедись что содержит:
   .env
   .env.local
   .env.*.local
   .env.production
   *.pem
   *.key
   node_modules/

5. ПОКАЖИ итоговый статус: что было настроено, что проверить.
```

**Что вы должны увидеть после выполнения:**
- `~/.npmrc` создан — `cat ~/.npmrc` показывает настройки
- `npm-safe-install` работает — `npm-safe-install zod` показывает info о пакете
- `ggshield` установлен — `ggshield --version` показывает версию
- `.gitignore` содержит правила для .env файлов

### 3.4 "20 Things That Will Get Your Vibe-Coded App Hacked"

Чеклист рисков. Для каждого — что плохого может случиться и как это починить одним действием.

---

**1. API ключи в frontend JavaScript**
- Проблема: любой кто откроет DevTools в браузере увидит ваши ключи
- Fix: все ключи — только в `.env` и серверном коде. Для фронта используйте `NEXT_PUBLIC_` только для публичных ключей (Supabase anon key — ок, OpenAI key — нет)

**2. .env закоммичен в git (даже один раз)**
- Проблема: файл остаётся в истории git даже если вы его удалили. `git log --all -- .env` найдёт его
- Fix: `.gitignore` + `ggshield` pre-commit hook. Если уже закоммитили — ротируйте ВСЕ ключи из этого файла

**3. SQL injection через конкатенацию строк**
- Проблема: `"SELECT * FROM users WHERE id=" + userId` — атакующий подставит SQL-команду вместо id
- Fix: параметризованные запросы. Supabase делает это автоматически через свой клиент

**4. CORS установлен в wildcard (`*`)**
- Проблема: любой сайт в интернете может делать запросы к вашему API от имени ваших пользователей
- Fix: указать конкретные origins: `['https://your-app.vercel.app', 'http://localhost:3000']`

**5. JWT токены в localStorage**
- Проблема: одна XSS-уязвимость = все токены украдены. localStorage читается любым скриптом на странице
- Fix: `httpOnly` cookies. Supabase Auth делает это правильно по умолчанию

**6. No HTTPS**
- Проблема: пароли и данные летят открытым текстом. Любой в том же Wi-Fi видит их
- Fix: Vercel включает HTTPS автоматически. VPS: `certbot --nginx`

**7. Нет rate limiting на /login**
- Проблема: боты могут перебирать 10,000 комбинаций паролей пока вы спите
- Fix: rate limiting + блокировка после 5 неудачных попыток

**8. Защита роутов только на фронтенде**
- Проблема: React Router guard не защищает API. `curl https://your-app.com/api/admin/users` обходит все фронтенд-проверки
- Fix: middleware с проверкой auth на КАЖДОМ серверном route

**9. Stack traces в ответах API**
- Проблема: вы отдаёте атакующему карту: имена таблиц, структуру кода, пути файлов
- Fix: generic сообщения для клиента, детальные логи — только на сервере

**10. npm install без проверки пакета**
- Проблема: AI может "придумать" название пакета (hallucination), а оно окажется зарегистрировано злоумышленником
- Fix: `npm-safe-install` из §3.3 — проверяет возраст, скачивания, существование

**11. Пароли захешированы MD5 или SHA1**
- Проблема: rainbow tables взламывают MD5 за секунды
- Fix: `bcrypt` или `argon2`. Supabase Auth использует bcrypt по умолчанию

**12. Токены без срока действия**
- Проблема: украденный токен = постоянный доступ навсегда
- Fix: set expiry (7 дней максимум) + refresh token rotation

**13. Сервер работает от root**
- Проблема: один эксплойт = полный доступ ко всей системе
- Fix: создайте отдельного пользователя `deploy` с ограниченными правами

**14. Порт базы данных открыт в интернет**
- Проблема: ваш PostgreSQL на порту 5432 доступен всему миру
- Fix: firewall (UFW) или private network. Supabase делает это за вас

**15. IDOR — доступ к чужим данным через ID**
- Проблема: меняем ID в URL `/api/users/123` → `/api/users/456` и видим чужие данные
- Fix: проверка ownership на сервере: `WHERE id = $1 AND user_id = $currentUser`

**16. File upload без валидации**
- Проблема: загружают .exe или .php вместо картинки → выполнение кода на сервере
- Fix: whitelist MIME types + переименование в UUID

**17. Нет сканирования секретов**
- Проблема: API ключ случайно попал в коммит → утечка
- Fix: GitGuardian pre-commit hook (настроен в §3.3)

**18. Sessions не инвалидируются при logout**
- Проблема: после нажатия "Выйти" старый токен всё ещё работает
- Fix: инвалидация сессии на сервере, не только удаление cookie на клиенте

**19. Open redirects**
- Проблема: `your-app.com/login?redirect=evil.com` — фишинг через ваш доверенный домен
- Fix: whitelist разрешённых redirect URLs

**20. npm пакеты не проверяются после установки**
- Проблема: известные уязвимости в зависимостях
- Fix: `npm audit` в CI + регулярно локально. Настроено в §3.3 (audit=true в .npmrc)

---

> 💡 **Если вы используете Supabase + Vercel**, многие из этих проблем решены за вас: HTTPS (Vercel), auth и RLS (Supabase), managed database (Supabase). Но правила из §3.2 всё равно нужны — они защищают от ошибок в ВАШЕМ коде.

---

## 4. Workshop Part 1 — Setup

> ⏱ **~60 мин.** Цель: от пустого экрана до проекта с Vision Doc, Techstack Doc и установленным стеком.

### 4.1 Создание проекта в Conductor (10 мин)

**Ведущий показывает, участники повторяют:**

1. Откройте **Conductor**
2. **Quick Start** → выберите папку (например `~/Projects/my-app`)
3. Дайте проекту имя (короткое, без пробелов, латиницей)
4. Conductor создаст Git-репозиторий автоматически

**Что происходит:** Conductor создаёт папку, инициализирует Git (систему контроля версий) и открывает рабочее пространство.

> **Checkpoint:** У всех есть проект в Conductor?

### 4.2 Vision Doc через Brainstorming (20 мин)

1. В Conductor откройте терминал (или новый таб)
2. Используйте команду `/brainstorm`
3. Опишите свою идею продукта

**Пример промпта:**
```
Я хочу сделать приложение для [описание].
Целевая аудитория: [кто].
Основная задача: [что пользователь должен сделать].
```

Claude задаст уточняющие вопросы. Отвечайте как можете. Лучшая фраза: **"я не знаю, объясни trade-offs"** — агент предложит варианты с плюсами и минусами.

4. Когда идея сформирована, попросите: **"сохрани результат в docs/vision.md"**

> **Checkpoint:** У всех есть Vision Doc в файлах?

### 4.3 Tech Stack Doc (15 мин)

1. Откройте **новый таб** в той же ветке
2. Промпт:

```
Вот мой vision (файл docs/vision.md). Помоги выбрать techstack.
Рекомендуемый стек: Next.js + Supabase + Vercel.
Сохрани результат в docs/tech-stack.md.
```

Claude сгенерирует документ с обоснованием каждого выбора.

> **Checkpoint:** У всех есть Techstack Doc?

### 4.4 CLAUDE.md + Установка стека (15 мин)

**CLAUDE.md — самый важный файл проекта.** Claude Code читает его при каждом запуске. Это "должностная инструкция" для агента.

1. Инициализация:
```bash
claude init
```
Claude создаст базовый `CLAUDE.md` с описанием проекта.

2. **Скопируйте Security Block** из [§3.2](#32-claudemd-security-block) и вставьте в `CLAUDE.md`

3. Попросите агента установить стек:
```
Установи всё что нужно для стека из docs/tech-stack.md.
Создай базовую структуру проекта.
```

4. Настройте credentials:
   - Откройте Supabase Dashboard → Project Settings → API
   - Скопируйте **Project URL** и **anon key**
   - Создайте файл `.env.local`:
   ```
   NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
   ```

> ⚠️ **API ключи НИКОГДА не коммитятся в git!** Только в `.env` файлах и в Environment Variables на Vercel.
>
> ⚠️ **Префикс `NEXT_PUBLIC_`** означает что переменная будет видна в браузере (через JavaScript). Это безопасно для Supabase `anon key` (он публичный, ограничен RLS-политиками). Но **НИКОГДА** не добавляйте `NEXT_PUBLIC_` к `service_role_key`, OpenAI ключам или другим секретным ключам — они должны быть только серверными (без префикса).

> **Checkpoint:** `npm install` прошёл без ошибок?

### 4.5 Итоговый чеклист Part 1

- [ ] Проект создан в Conductor
- [ ] Vision Doc (`docs/vision.md`) написан
- [ ] Techstack Doc (`docs/tech-stack.md`) написан
- [ ] `CLAUDE.md` создан с Security Rules
- [ ] `npm install` прошёл успешно
- [ ] `.env.local` с Supabase credentials создан
- [ ] `.gitignore` содержит `.env.local`

---

## 5. Workshop Part 2 — First Cycle

> ⏱ **~60 мин.** Цель: пройти полный цикл spec → code → local → debug → deploy на одной фиче.

### 5.1 Написать спек первой фичи (15 мин)

**Важно: маленькие фичи!** "Покажи landing page" — хорошо. "Сделай весь app" — плохо. Двигайтесь маленькими шагами.

1. Откройте **новый таб** (чистый контекст, но видит файлы проекта)
2. Используйте `/write-plan`:

```
Я хочу сделать [маленькую конкретную фичу].
Пользователь должен видеть [что].
Создай plan для реализации.
```

3. Claude создаст план с шагами. Проверьте — всё ли понятно.
4. Запустите `/execute-plan` — агент начнёт кодить по плану.

> **Checkpoint:** Агент закончил? Файлы появились?

### 5.2 Запуск локально (10 мин)

В терминале Conductor:
```bash
npm run dev
```

Откройте в браузере: **http://localhost:3000**

Что-то отображается? 🎉 → переходите к §5.4

Что-то сломалось? → следующий шаг.

> **Checkpoint:** У всех что-то видно в браузере?

### 5.3 Debugging — когда что-то не работает (10 мин)

**Три способа дебажить:**

**Способ 1: Скопировать ошибку**
- Видите красный текст в терминале? Скопируйте его и вставьте Claude
- Агент прочитает ошибку и предложит fix

**Способ 2: Скриншот**
- Видите что-то странное в браузере? Сделайте скриншот
- Перетащите картинку в чат с Claude — он умеет "видеть" изображения

**Способ 3: Systematic debugging**
- Используйте `/debug`
- Агент будет методично проверять гипотезы одну за другой

**Частые ошибки** — см. таблицу в [§6.4](#64-common-errors).

> **Checkpoint:** У всех работает локально?

### 5.4 Review + Deploy (15 мин)

**Code Review:**
```
/review
```
Агент проверит код и предложит улучшения. Исправьте что нужно.

**Create Pull Request:**
```
/pr
```
Агент создаст PR с описанием → merge в main.

**Deploy на Vercel:**
1. Зайдите на [vercel.com](https://vercel.com)
2. **Add New Project** → выберите ваш GitHub-репозиторий
3. Framework: **Next.js** (определится автоматически)
4. **Environment Variables** — добавьте те же ключи что в `.env.local`:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
5. Нажмите **Deploy**
6. Через 1-2 минуты — ваш сайт в интернете! 🎉

> **Checkpoint:** Все видят свой сайт по URL `xxx.vercel.app`?

### 5.5 Второй цикл (если есть время, ~10 мин)

1. Создайте новую ветку в Conductor
2. Напишите маленький spec на вторую фичу
3. `/execute-plan` → Local → `/review` → `/pr` → Merge
4. Vercel автоматически редеплоит после merge в main — проверьте!

> ⚠️ Environment Variables нужно добавить в Vercel один раз — они применяются ко всем деплоям.

### 5.6 Полный цикл

```
Идея → /brainstorm → /write-plan → /execute-plan → npm run dev → /debug → /review → /pr → merge → Live! 🚀
```

Повторяйте этот цикл для каждой фичи. Каждый круг занимает 15-30 минут.

---

## 6. Quick Reference

### 6.1 Команды Claude Code

| Команда | Что делает | Когда использовать |
|---------|-----------|-------------------|
| `/brainstorm` | Исследовать идею, задать вопросы | В начале новой фичи |
| `/write-plan` | Создать план реализации | Когда идея ясна |
| `/execute-plan` | Выполнить план по шагам | Кодить по плану |
| `/debug` | Systematic debugging | Что-то не работает |
| `/review` | Code review | Перед коммитом |
| `/pr` | Создать Pull Request | Готовы пушить |
| `/explain` | Объяснить код/концепт | Не понятно |
| `Esc` | Отменить текущую генерацию | Агент пошёл не туда |

### 6.2 Conductor Workflow

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. Open Project                                     │
│     ↓                                                │
│  2. Quick Start → описать идею                       │
│     ↓                                                │
│  3. /brainstorm → уточнить                           │
│     ↓                                                │
│  4. /write-plan → создать план                       │
│     ↓                                                │
│  5. /execute-plan → агент кодит                      │
│     ↓                                                │
│  6. npm run dev → проверить локально                 │
│     ↓                                                │
│  7. /debug → если сломалось                          │
│     ↓                                                │
│  8. /review → code review                            │
│     ↓                                                │
│  9. /pr → merge → auto-deploy на Vercel              │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 6.3 Terminal Commands

**Проект:**
```bash
npm run dev          # запустить локально (http://localhost:3000)
npm run build        # собрать для production
npm run lint         # проверить код на ошибки
npm install          # установить зависимости
```

**Git:**
```bash
git status           # что изменилось?
git add .            # добавить все изменения (⚠️ сначала проверь git status!)
git commit -m "описание"  # сохранить изменения
git push             # отправить на GitHub
git pull             # скачать изменения с GitHub
git log --oneline -5 # последние 5 коммитов
```

**Vercel:**
```bash
vercel               # деплой preview (тестовый URL)
vercel --prod        # деплой production
vercel env pull      # скачать env vars с Vercel
vercel logs          # посмотреть логи деплоя
```

**Supabase:**
```bash
supabase start       # запустить локальный Supabase
supabase db push     # применить миграции
supabase gen types typescript --local  # сгенерировать TypeScript типы
supabase status      # статус локального инстанса
```

### 6.4 Common Errors

| Ошибка | Причина | Как починить |
|--------|---------|-------------|
| `EADDRINUSE :3000` | Порт уже занят другим процессом | `lsof -i :3000` → `kill -9 <PID>` |
| `Module not found` | Пакет не установлен | `npm install` |
| `Cannot find module '@/...'` | Неверный путь импорта | Проверить путь, `@/` = корень проекта |
| `NEXT_PUBLIC_ not working` | Env var не обновилась | Перезапустить `npm run dev` после изменения `.env.local` |
| `Supabase connection refused` | Неверный URL или key | Проверить `.env.local` — скопировать заново из Supabase Dashboard |
| `git push rejected` | На GitHub есть новые изменения | `git pull --rebase` затем `git push` |
| `Permission denied (publickey)` | SSH ключ не настроен | `gh auth login` (проще) или добавить SSH key |
| `Hydration mismatch` | HTML на сервере ≠ HTML на клиенте | Обернуть динамический контент в `useEffect` |
| `TypeError: Cannot read properties of undefined` | Обращение к null/undefined | Использовать optional chaining: `obj?.property` |
| `npm ERR! ERESOLVE` | Конфликт версий зависимостей | `npm install --legacy-peer-deps` |
| `Vercel build failed` | Ошибка видна только в production | Смотреть Build Logs на vercel.com → Deployments |
| `ENOENT: no such file or directory` | Файл не найден по указанному пути | Проверить путь и имя файла |
| `429 Too Many Requests` | Превышен лимит API-запросов | Подождать минуту или проверить rate limits |
| `Supabase RLS policy` | Row Level Security блокирует запрос | Настроить policies в Supabase Dashboard → Authentication → Policies |
| `TypeScript error` | Несовпадение типов | Прочитать сообщение об ошибке, исправить тип переменной |

> 💡 **Универсальный способ:** скопируйте текст ошибки и вставьте в Claude Code. В 90% случаев агент починит сам.

### 6.5 Ready-Made Prompts

Готовые промпты — просто скопируйте и подставьте свои значения:

**Создание:**
```
Создай landing page для [продукт] с hero section, описанием features и CTA-кнопкой.
```

```
Добавь авторизацию через Supabase Auth (email + password).
Создай страницы /login и /signup.
```

```
Создай таблицу в Supabase для [сущность] с полями: [список полей].
Добавь Row Level Security policies.
```

**Улучшение:**
```
Сделай эту страницу responsive — должна хорошо выглядеть на мобильном.
```

```
Добавь dark mode toggle. Используй Tailwind dark: классы.
```

```
Добавь форму которая сохраняет данные в Supabase таблицу [название].
С валидацией полей и обработкой ошибок.
```

**Диагностика:**
```
Исправь все TypeScript ошибки в проекте.
```

```
Проверь все API routes на наличие проблем с безопасностью.
```

```
Объясни что делает этот файл: [путь к файлу]
```

### 6.6 Git для не-разработчиков

Git — это система контроля версий. Думайте о ней как об **истории изменений** вашего проекта.

**Основные понятия:**

| Термин | Аналогия | Что это |
|--------|----------|---------|
| **Repository (repo)** | Папка с историей | Ваш проект + вся его история изменений |
| **Commit** | Сохранение в игре | Снимок состояния проекта в конкретный момент |
| **Branch** | Параллельная вселенная | Копия проекта где можно экспериментировать |
| **Pull Request (PR)** | Запрос на проверку | "Посмотри мои изменения и одобри слияние" |
| **Push** | Загрузка | Отправить коммиты с компьютера на GitHub |
| **Pull** | Скачивание | Скачать изменения с GitHub на компьютер |
| **Merge** | Объединение | Слить ветку с изменениями обратно в основную |
| **Main** | Основная версия | Ветка которая всегда рабочая и идёт в production |

**Как это работает:**

```
Ваш компьютер           GitHub             Vercel
   (код)         ──push──→  (репозиторий)  ──auto──→  (сайт)
                 ←──pull──
```

При push в `main` → Vercel автоматически публикует новую версию сайта.

**Минимальный набор команд:**
```bash
git add .               # подготовить все изменения
git commit -m "что сделал"  # сохранить
git push                # отправить на GitHub
```

Или проще — используйте `/pr` в Claude Code, он сделает всё за вас.

### 6.7 Vercel — Подробная настройка

1. Зайдите на [vercel.com](https://vercel.com) → **Sign Up with GitHub**
2. Нажмите **Add New → Project**
3. Выберите ваш GitHub-репозиторий из списка
4. **Framework Preset:** Next.js (определяется автоматически)
5. **Root Directory:** оставьте как есть (или укажите если Next.js в подпапке)
6. **Environment Variables** — добавьте каждую переменную из `.env.local`:
   - `NEXT_PUBLIC_SUPABASE_URL` → значение
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY` → значение
   - (и другие ключи если есть)
7. Нажмите **Deploy**
8. Через 1-2 минуты получите URL: `your-project.vercel.app`

**Автоматический деплой:** после настройки каждый push в `main` автоматически обновляет сайт.

**Custom domain (опционально):**
- Settings → Domains → Add Domain
- Укажите ваш домен
- Настройте DNS у регистратора (Vercel покажет инструкции)

### 6.8 Supabase — Подробная настройка

1. Зайдите на [supabase.com](https://supabase.com) → **Sign Up with GitHub**
2. **New Project** → выберите регион ближайший к вашим пользователям
3. Задайте пароль для базы данных (сохраните его!)
4. После создания проекта → **Settings → API**:
   - Скопируйте **Project URL** (начинается с `https://`)
   - Скопируйте **anon public key** (начинается с `eyJ`)
5. Добавьте в `.env.local` вашего проекта:
   ```
   NEXT_PUBLIC_SUPABASE_URL=https://abcdefgh.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOi...
   ```
6. Установите клиент:
   ```bash
   npm install @supabase/supabase-js
   ```
7. Создайте файл `lib/supabase.ts`:
   ```typescript
   import { createClient } from '@supabase/supabase-js'

   export const supabase = createClient(
     process.env.NEXT_PUBLIC_SUPABASE_URL!,
     process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
   )
   ```
8. Проверьте подключение — создайте тестовую таблицу в Supabase Dashboard и попробуйте прочитать данные.

> ⚠️ **Обязательно включите Row Level Security (RLS)** на каждой таблице: Table Editor → таблица → RLS Enabled. Без RLS `anon key` даёт полный доступ к чтению и записи всех данных. RLS — это правила "кто может видеть/менять какие строки".

**Полезные фичи Supabase:**
- **Table Editor** — создавайте таблицы через UI, без SQL
- **Auth** — готовая авторизация (email, Google, GitHub...)
- **Storage** — хранение файлов (картинки, документы)
- **Edge Functions** — serverless функции
- **Realtime** — подписка на изменения в БД в реальном времени

---

## 7. Appendices

### 7.1 Глоссарий

| Термин | Определение |
|--------|------------|
| **API** | Application Programming Interface — способ общения программ друг с другом |
| **API Route** | URL на сервере который обрабатывает запросы (например `/api/users`) |
| **Auth** | Аутентификация — проверка "кто вы?" (логин/пароль) |
| **Backend** | Серверная часть — код который работает на сервере, не видим пользователю |
| **Build** | Процесс подготовки кода к production (оптимизация, минификация) |
| **CDN** | Content Delivery Network — сеть серверов по миру для быстрой доставки контента |
| **CI/CD** | Continuous Integration / Deployment — автоматическая проверка и деплой кода |
| **CLI** | Command Line Interface — программа которая работает в терминале |
| **Component** | Переиспользуемый блок UI (кнопка, форма, карточка) |
| **Container** | Изолированная среда для запуска приложения (Docker) |
| **CORS** | Cross-Origin Resource Sharing — правила какие сайты могут обращаться к вашему API |
| **Deploy** | Публикация кода на сервер, делая его доступным в интернете |
| **DNS** | Domain Name System — связывает домен (example.com) с IP-адресом сервера |
| **Docker** | Платформа для запуска приложений в изолированных контейнерах |
| **Endpoint** | Конкретный URL API который обрабатывает запрос определённого типа |
| **Environment Variable** | Переменная окружения — настройка хранящаяся вне кода (API ключи, URLs) |
| **Frontend** | Клиентская часть — то что пользователь видит в браузере |
| **Hook** | В React — функция для добавления логики в компонент (`useState`, `useEffect`) |
| **HTTPS** | HTTP с шифрованием — безопасная передача данных |
| **JWT** | JSON Web Token — зашифрованный токен для аутентификации |
| **Middleware** | Код который выполняется между запросом и ответом (проверка auth, логирование) |
| **Migration** | Скрипт изменения структуры базы данных (добавить таблицу, колонку) |
| **npm** | Node Package Manager — менеджер пакетов для JavaScript |
| **ORM** | Object-Relational Mapping — работа с БД через объекты вместо SQL |
| **Props** | Данные которые передаются компоненту (как аргументы функции) |
| **PWA** | Progressive Web App — сайт который работает как мобильное приложение |
| **Query** | Запрос к базе данных для получения или изменения данных |
| **Responsive** | Дизайн который адаптируется под размер экрана (десктоп, планшет, телефон) |
| **Route** | Путь URL который соответствует определённой странице или API-обработчику |
| **Schema** | Структура базы данных — какие таблицы, колонки и типы данных |
| **Serverless** | Код который запускается по запросу, без постоянного сервера |
| **SSR** | Server-Side Rendering — генерация HTML на сервере |
| **State** | Состояние компонента — данные которые меняются и влияют на отображение |
| **Token** | Строка-идентификатор для аутентификации или доступа к API |
| **TypeScript** | JavaScript с типами — помогает находить ошибки до запуска кода |
| **Webhook** | URL который вызывается автоматически при определённом событии |

### 7.2 Полезные ссылки

**Документация:**
- [Next.js Docs](https://nextjs.org/docs) — фреймворк для React
- [Supabase Docs](https://supabase.com/docs) — база данных и бэкенд
- [Vercel Docs](https://vercel.com/docs) — деплой и хостинг
- [shadcn/ui](https://ui.shadcn.com) — компоненты UI
- [Tailwind CSS](https://tailwindcss.com/docs) — utility-first CSS фреймворк

**Claude Code:**
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code) — официальная документация
- [Conductor](https://conductor.build) — GUI для Claude Code

**Supabase + Next.js:**
- [Supabase + Next.js Quickstart](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs) — официальный гайд

### 7.3 Что делать после воркшопа

**Неделя 1: Доделать MVP**
1. Закончите 3-5 основных фич вашего продукта
2. Используйте цикл: `/brainstorm` → `/write-plan` → `/execute-plan` → deploy
3. Двигайтесь маленькими шагами — одна фича за раз

**Неделя 2: Первые пользователи**
4. Покажите MVP 5-10 людям из целевой аудитории
5. Соберите фидбек — что работает, что нет
6. Добавьте авторизацию (Supabase Auth) если ещё не добавили

**Неделя 3-4: Полировка**
7. Добавьте аналитику (Vercel Analytics — бесплатно, или PostHog)
8. Купите custom domain (Namecheap, Cloudflare)
9. Настройте error tracking (Sentry — бесплатный план)
10. Используйте `/review` для code review перед каждым деплоем

**Дальше:**
- Изучите Supabase Auth, Storage, Realtime
- Попробуйте Impeccable skills для полировки UI
- Присоединитесь к сообществу AI-билдеров

### 7.4 VPS Hardening (для продвинутых)

> Полный автоматизированный runbook для Claude Code: [`workshop/prework/vps-hardening-runbook.md`](prework/vps-hardening-runbook.md). Скажите Claude Code: "захардень мой сервер по runbook из workshop/prework/vps-hardening-runbook.md".

Если вы деплоите на Hetzner, Railway или другой VPS (вместо Vercel), используйте runbook выше. Ключевые принципы:

- Отдельный deploy user (не root) с ограниченным sudo
- SSH только по ключам, пароль отключён
- UFW firewall: только 22/80/443
- Docker порты привязаны к 127.0.0.1 (не открыты в интернет)
- Nginx reverse proxy + Let's Encrypt SSL
- `.env` файлы с правами 600

---

> 📖 **Этот гайдбук — живой документ.** Сохраните его и возвращайтесь когда нужно. Удачи с вашим проектом!
