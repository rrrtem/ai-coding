# Этап 1: Базовая установка

> Инструкции для Claude Code. Выполняй шаги последовательно.

## 1.1 Определи операционную систему

Определи OS участника (`uname -s` или аналог). Дальнейшие команды зависят от OS:
- **macOS** — основной путь (Homebrew)
- **Linux** — apt/dnf/pacman в зависимости от дистрибутива
- **Windows (WSL)** — apt внутри WSL

Сообщи участнику какую OS ты определил.

## 1.2 Менеджер пакетов

### macOS: Homebrew

Проверь: `brew --version`

Если не установлен — скажи участнику:

> Сейчас я установлю Homebrew — это менеджер пакетов для macOS. Как App Store, но для инструментов разработчика. Установка может занять пару минут.

Установи:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Важно:** после установки Homebrew на Apple Silicon (M1/M2/M3/M4) может потребоваться добавить его в PATH:
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Проверь что работает: `brew --version`

### Linux: apt (Ubuntu/Debian)

Проверь что `apt` доступен. Обнови индекс:
```bash
sudo apt update
```

## 1.3 Node.js 20+

Проверь: `node -v`

Если не установлен или версия ниже 20:

> Node.js — среда для запуска JavaScript. Большинство современных веб-инструментов работают на нём.

**macOS:**
```bash
brew install node
```

**Linux:**
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Проверь: `node -v` (должно быть v20+) и `npm -v`

## 1.4 Git

Проверь: `git --version`

Если не установлен:

**macOS:**
```bash
brew install git
```

**Linux:**
```bash
sudo apt install -y git
```

### Настройка git

Спроси у участника его имя и email для Git:

> Git — система контроля версий, она отслеживает кто какие изменения внёс. Мне нужно указать твоё имя и email. Они будут видны в истории изменений. Какое имя и email использовать?

После получения ответа:
```bash
git config --global user.name "Имя Участника"
git config --global user.email "email@example.com"
```

## 1.5 GitHub CLI

Проверь: `gh --version`

Если не установлен:

> GitHub CLI — позволяет работать с GitHub прямо из терминала: создавать репозитории, Pull Request-ы и другое, не открывая браузер.

**macOS:**
```bash
brew install gh
```

**Linux:**
```bash
(type -p wget >/dev/null || sudo apt install wget -y) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

### Авторизация GitHub CLI

**Предупреди участника ПЕРЕД запуском:**

> Сейчас я запущу авторизацию GitHub. Откроется браузер с GitHub — нужно будет нажать "Authorize". Готов?

Дождись подтверждения, затем:

Попроси участника выполнить эту команду самостоятельно (она интерактивная):
```
gh auth login
```

Подскажи: выбрать GitHub.com → HTTPS → Login with a web browser.

Проверь: `gh auth status` — должно показать "Logged in to github.com"

## 1.6 Vercel CLI

Проверь: `vercel --version`

Если не установлен:

> Vercel — платформа для публикации сайтов. Один push в GitHub — и сайт автоматически обновляется. CLI нужен для управления из терминала.

```bash
npm install -g vercel
```

Проверь: `vercel --version`

## 1.7 Supabase CLI

Проверь: `supabase --version`

Если не установлен:

> Supabase — готовая база данных с панелью управления. CLI нужен для работы с базой из терминала и создания миграций.

**macOS:**
```bash
brew install supabase/tap/supabase
```

**Linux:**
```bash
brew install supabase/tap/supabase
```
(Если Homebrew нет на Linux — используй npm: `npm install -g supabase`)

Проверь: `supabase --version`

## 1.8 Глобальная безопасность: ~/.npmrc

> Настрою базовую безопасность для npm. Это защитит от вредоносных пакетов которые иногда маскируются под популярные.

Создай файл `~/.npmrc` (если существует — добавь строки которых нет):

```ini
audit=true
ignore-scripts=true
engine-strict=true
fund=false
```

Объясни участнику:
> `ignore-scripts=true` — самая важная настройка. Она блокирует автоматическое выполнение скриптов при `npm install`. Это главный способ, которым вредоносные пакеты запускают свой код.

## 1.9 Скрипт npm-safe-install

Создай файл `~/bin/npm-safe-install`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# npm-safe-install — проверяет пакет перед установкой
# Использование: npm-safe-install <package-name> [--force]

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

if [ $# -lt 1 ]; then
  echo "Usage: npm-safe-install <package-name> [--force]"
  exit 1
fi

PACKAGE="$1"
FORCE="${2:-}"

echo "Checking package: $PACKAGE"
echo "---"

# Fetch package info from registry
REGISTRY_DATA=$(curl -s "https://registry.npmjs.org/$PACKAGE" 2>/dev/null)

if echo "$REGISTRY_DATA" | grep -q '"error"'; then
  echo -e "${RED}ERROR: Package '$PACKAGE' not found on npmjs.com${NC}"
  exit 1
fi

# Get creation date
CREATED=$(echo "$REGISTRY_DATA" | python3 -c "import sys,json; print(json.load(sys.stdin)['time']['created'])" 2>/dev/null || echo "unknown")

# Get latest version
LATEST=$(echo "$REGISTRY_DATA" | python3 -c "import sys,json; print(json.load(sys.stdin)['dist-tags']['latest'])" 2>/dev/null || echo "unknown")

# Get latest version publish date
LATEST_DATE=$(echo "$REGISTRY_DATA" | python3 -c "import sys,json; print(json.load(sys.stdin)['time'][json.load(sys.stdin)['dist-tags']['latest']])" 2>/dev/null || echo "unknown")

# Get weekly downloads
DOWNLOADS=$(curl -s "https://api.npmjs.org/downloads/point/last-week/$PACKAGE" 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin).get('downloads', 0))" 2>/dev/null || echo "0")

WARNINGS=0

# Check age > 365 days
if [ "$CREATED" != "unknown" ]; then
  CREATED_TS=$(date -j -f "%Y-%m-%dT%H:%M:%S" "${CREATED%%.*}" "+%s" 2>/dev/null || date -d "${CREATED%%.*}" "+%s" 2>/dev/null || echo "0")
  NOW_TS=$(date "+%s")
  AGE_DAYS=$(( (NOW_TS - CREATED_TS) / 86400 ))
  if [ "$AGE_DAYS" -lt 365 ]; then
    echo -e "${YELLOW}WARNING: Package is only $AGE_DAYS days old (threshold: 365)${NC}"
    WARNINGS=$((WARNINGS + 1))
  else
    echo -e "${GREEN}OK: Package age: $AGE_DAYS days${NC}"
  fi
fi

# Check latest version age > 7 days
if [ "$LATEST_DATE" != "unknown" ]; then
  LATEST_TS=$(date -j -f "%Y-%m-%dT%H:%M:%S" "${LATEST_DATE%%.*}" "+%s" 2>/dev/null || date -d "${LATEST_DATE%%.*}" "+%s" 2>/dev/null || echo "0")
  NOW_TS=$(date "+%s")
  VER_AGE_DAYS=$(( (NOW_TS - LATEST_TS) / 86400 ))
  if [ "$VER_AGE_DAYS" -lt 7 ]; then
    echo -e "${YELLOW}WARNING: Latest version ($LATEST) is only $VER_AGE_DAYS days old (threshold: 7)${NC}"
    WARNINGS=$((WARNINGS + 1))
  else
    echo -e "${GREEN}OK: Version $LATEST age: $VER_AGE_DAYS days${NC}"
  fi
fi

# Check downloads > 500
if [ "$DOWNLOADS" -lt 500 ]; then
  echo -e "${YELLOW}WARNING: Only $DOWNLOADS weekly downloads (threshold: 500)${NC}"
  WARNINGS=$((WARNINGS + 1))
else
  echo -e "${GREEN}OK: Weekly downloads: $DOWNLOADS${NC}"
fi

echo "---"

if [ "$WARNINGS" -gt 0 ] && [ "$FORCE" != "--force" ]; then
  echo -e "${YELLOW}$WARNINGS warning(s) found. Install anyway? (y/N)${NC}"
  read -r REPLY
  if [ "$REPLY" != "y" ] && [ "$REPLY" != "Y" ]; then
    echo "Cancelled."
    exit 1
  fi
fi

echo "Installing $PACKAGE..."
npm install "$PACKAGE"
echo -e "${GREEN}Done!${NC}"
```

Сделай исполняемым:
```bash
mkdir -p ~/bin
chmod +x ~/bin/npm-safe-install
```

Добавь `~/bin` в PATH (если ещё нет):
```bash
# Проверь какой shell
if [ -f ~/.zshrc ]; then
  grep -q 'export PATH="$HOME/bin:$PATH"' ~/.zshrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
elif [ -f ~/.bashrc ]; then
  grep -q 'export PATH="$HOME/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
fi
```

## 1.10 GitGuardian (ggshield)

> GitGuardian сканирует код на случайно закоммиченные секреты: API-ключи, пароли, токены. Это как антивирус, но для секретов в коде.

Установи:

**macOS:**
```bash
brew install gitguardian/tap/ggshield
```

**Или через pip:**
```bash
pip install ggshield
```

### Авторизация ggshield

**Предупреди участника ПЕРЕД запуском:**

> Сейчас нужно авторизовать GitGuardian. Откроется браузер — нужно создать бесплатный аккаунт (или войти) и разрешить доступ. Готов?

Дождись подтверждения, затем попроси участника выполнить:
```
ggshield auth login
```

Если участник не хочет создавать ещё один аккаунт — это опционально. Отметь в отчёте что ggshield не авторизован.

## 1.11 Итог этапа

Перед тем как перейти к этапу 2, убедись что всё из списка ниже работает:

- [ ] Менеджер пакетов (brew / apt)
- [ ] Node.js 20+ и npm
- [ ] Git настроен (имя + email)
- [ ] GitHub CLI авторизован
- [ ] Vercel CLI
- [ ] Supabase CLI
- [ ] ~/.npmrc создан
- [ ] npm-safe-install скрипт создан
- [ ] ggshield установлен (авторизация опциональна)

Сообщи участнику результат и переходи к этапу 2.
