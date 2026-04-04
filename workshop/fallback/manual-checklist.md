# Ручной чеклист — если Claude Code не справился

> Если автоматическая установка не сработала — пройдите по этому списку вручную.
> Каждый шаг — одна команда в терминале.

## Как открыть терминал

- **macOS:** нажмите `Cmd + Пробел`, наберите "Terminal", нажмите Enter
- **Windows:** установите WSL2 (`wsl --install` в PowerShell от администратора), затем откройте Ubuntu из меню Пуск
- **Linux:** `Ctrl + Alt + T`

## Базовая установка (macOS)

Копируйте каждую команду и вставляйте в терминал (`Cmd + V`). После каждой — нажмите Enter и дождитесь завершения.

### 1. Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Первый запуск и авторизация:
```bash
claude
```

Следуйте инструкциям на экране — Claude попросит войти в аккаунт.

Проверка: `claude --version`

### 2. Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Если у вас Mac на Apple Silicon (M1/M2/M3/M4) — после установки выполните:
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Проверка: `brew --version`

### 3. Node.js

```bash
brew install node
```

Проверка: `node -v` (должно быть v22 или выше), `npm -v` (должно быть 11.x.x)

### 4. Git

```bash
brew install git
```

Настройка (подставьте свои данные):
```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your@email.com"
```

Проверка: `git --version`

### 5. GitHub CLI

```bash
brew install gh
```

Авторизация (откроется браузер):
```bash
gh auth login
```
Выберите: GitHub.com → HTTPS → Login with a web browser

Проверка: `gh auth status`

### 6. Vercel CLI

```bash
npm install -g vercel
```

Проверка: `vercel --version`

### 7. Supabase CLI

```bash
brew install supabase/tap/supabase
```

Проверка: `supabase --version`

### 8. Безопасность — ~/.npmrc

Добавьте недостающие строки (если файл уже существует — он НЕ будет перезаписан):
```bash
grep -q "^audit=true" ~/.npmrc 2>/dev/null || echo "audit=true" >> ~/.npmrc
grep -q "^ignore-scripts=true" ~/.npmrc 2>/dev/null || echo "ignore-scripts=true" >> ~/.npmrc
grep -q "^engine-strict=true" ~/.npmrc 2>/dev/null || echo "engine-strict=true" >> ~/.npmrc
grep -q "^fund=false" ~/.npmrc 2>/dev/null || echo "fund=false" >> ~/.npmrc
```

Проверка: `cat ~/.npmrc`

### 9. GitGuardian (опционально)

```bash
brew install gitguardian/tap/ggshield
ggshield auth login
```

## Базовая установка (Linux / WSL)

### 1. Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude
```

### 2. Обновить пакеты

```bash
sudo apt update && sudo apt upgrade -y
```

### 3. Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

### 4. Git

```bash
sudo apt install -y git
git config --global user.name "Ваше Имя"
git config --global user.email "your@email.com"
```

### 5-9: те же команды что для macOS

(gh, vercel, supabase CLI, npmrc — команды `npm install -g` и `gh auth login` одинаковы на всех платформах)

Для GitHub CLI на Linux:
```bash
(type -p wget >/dev/null || sudo apt install wget -y) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && out=$(mktemp) && wget -nv -O "$out" https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

## Дополнительно (по типу проекта)

### Telegram-бот / Автоматизации
```bash
brew install python      # macOS
# или
sudo apt install -y python3 python3-pip python3-venv   # Linux
```

### Мобильное приложение
```bash
npm install -g expo-cli eas-cli
```

### Деплой на VPS (Docker + hcloud)

**Docker Desktop (macOS):** скачайте с [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)

**Docker (Linux):**
```bash
curl -fsSL https://get.docker.com | sh
```

**hcloud (Hetzner Cloud CLI):**
```bash
brew install hcloud
```

Проверка: `docker --version` и `hcloud version`

## Финальная проверка

Запустите все команды — каждая должна показать версию:

```bash
node -v
npm -v
claude --version
git --version
gh auth status
vercel --version
supabase --version
cat ~/.npmrc
```

Если всё работает — вы готовы к воркшопу!

Если что-то не получается — напишите ведущему с описанием ошибки (скопируйте текст из терминала).
