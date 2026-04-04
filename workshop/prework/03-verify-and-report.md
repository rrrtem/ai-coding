# Этап 3: Проверка и отчёт

> Инструкции для Claude Code.

## 3.1 Прогони все проверки

Запусти каждую команду и запиши результат:

```bash
# OS
uname -srm

# Claude Code
claude --version 2>/dev/null || echo "claude: не установлен"

# Менеджер пакетов
brew --version 2>/dev/null || echo "brew: не установлен"

# Node.js
node -v 2>/dev/null || echo "node: не установлен"
npm -v 2>/dev/null || echo "npm: не установлен"

# Git
git --version 2>/dev/null || echo "git: не установлен"
git config --global user.name 2>/dev/null || echo "git user.name: не задан"
git config --global user.email 2>/dev/null || echo "git user.email: не задан"

# GitHub CLI
gh --version 2>/dev/null || echo "gh: не установлен"
gh auth status 2>&1 || echo "gh: не авторизован"

# Vercel CLI
vercel --version 2>/dev/null || echo "vercel: не установлен"

# Supabase CLI
supabase --version 2>/dev/null || echo "supabase: не установлен"

# Security
cat ~/.npmrc 2>/dev/null || echo "~/.npmrc: не найден"
which npm-safe-install 2>/dev/null || echo "npm-safe-install: не найден"
ggshield --version 2>/dev/null || echo "ggshield: не установлен"
ggshield auth status 2>&1 || echo "ggshield: не авторизован"

# Python (если устанавливали)
python3 --version 2>/dev/null || echo "python3: не установлен"

# Expo (если устанавливали)
expo --version 2>/dev/null || echo "expo: не установлен (не нужен если нет мобильных проектов)"
eas --version 2>/dev/null || echo "eas: не установлен (не нужен если нет мобильных проектов)"

# Docker (если устанавливали)
docker --version 2>/dev/null || echo "docker: не установлен (не нужен если нет VPS деплоя)"
docker compose version 2>/dev/null || echo "docker compose: не установлен"

# hcloud (если устанавливали)
hcloud version 2>/dev/null || echo "hcloud: не установлен (не нужен если нет VPS деплоя)"
```

## 3.2 Создай отчёт

Создай файл `setup-report.md` в **корне проекта** (не в workshop/).

Формат отчёта:

```markdown
# Setup Report

> Сгенерировано автоматически Claude Code
> Дата: [текущая дата и время]

## Система

| Параметр | Значение |
|----------|----------|
| OS | [результат uname] |
| Менеджер пакетов | [brew версия или apt] |

## Базовые инструменты

| Инструмент | Версия | Статус |
|------------|--------|--------|
| Claude Code | [версия] | [OK / ОШИБКА] |
| Node.js | [версия] | [OK / ОШИБКА] |
| npm | [версия] | [OK / ОШИБКА] |
| Git | [версия] | [OK / ОШИБКА] |
| Git user.name | [значение] | [OK / не задан] |
| Git user.email | [значение] | [OK / не задан] |
| GitHub CLI | [версия] | [OK / ОШИБКА] |
| GitHub Auth | — | [OK / не авторизован] |
| Vercel CLI | [версия] | [OK / ОШИБКА] |
| Supabase CLI | [версия] | [OK / ОШИБКА] |

## Безопасность

| Компонент | Статус |
|-----------|--------|
| ~/.npmrc | [OK / не создан] |
| npm-safe-install | [OK / не создан] |
| ggshield | [OK / не установлен] |
| ggshield auth | [OK / не авторизован / пропущено] |

## Дополнительные инструменты

| Инструмент | Версия | Статус | Зачем |
|------------|--------|--------|-------|
| Python | [версия или —] | [OK / не нужен] | [тип проекта] |
| Expo CLI | [версия или —] | [OK / не нужен] | [тип проекта] |
| EAS CLI | [версия или —] | [OK / не нужен] | [тип проекта] |
| Docker | [версия или —] | [OK / не нужен] | [тип проекта] |
| Docker Compose | [версия или —] | [OK / не нужен] | [тип проекта] |
| hcloud | [версия или —] | [OK / не нужен] | [тип проекта] |

## Выбранные типы проектов

- [x/—] Сайт / лендинг
- [x/—] Веб-приложение с личным кабинетом
- [x/—] Telegram-бот
- [x/—] Мобильное приложение
- [x/—] Автоматизация / скрипты
- [x/—] Свой вариант: [описание если было]

## Итог

**Статус: [ГОТОВ К ВОРКШОПУ / ЕСТЬ ПРОБЛЕМЫ]**

[Если есть проблемы — перечислить что не установилось и почему]
```

## 3.3 Сообщи результат

Покажи участнику краткий итог:

- Сколько инструментов установлено из скольки
- Есть ли проблемы
- Где лежит отчёт (`setup-report.md`)
- Напомни: "Отправь этот файл ведущему воркшопа"

Если всё ОК:
> Всё готово! Отчёт сохранён в `setup-report.md`. Отправь его ведущему воркшопа. Этот проект можешь использовать как шпаргалку — здесь лежит гайдбук и все инструкции.

Если есть проблемы:
> Почти всё готово, но есть [N] проблем(а). Они записаны в `setup-report.md`. Попробуй решить их с помощью файла `workshop/fallback/manual-checklist.md` или отправь отчёт ведущему — поможем разобраться.
