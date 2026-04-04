# VPS Hardening Runbook

> Инструкции для Claude Code. Этот файл — runbook для настройки безопасности VPS.
> Используется когда участник деплоит проект на собственный сервер (Hetzner, DigitalOcean и т.д.).
> Скрипт идемпотентный — безопасно запускать повторно.

## Когда использовать

Этот runbook запускается **на сервере** (через SSH), не на локальной машине.
Участник говорит: "настрой безопасность моего VPS" или "захардень сервер".

## Предварительные требования

- VPS с Ubuntu 22.04/24.04 LTS
- SSH-доступ к серверу (root или с sudo)
- Участник знает IP-адрес сервера

## Порядок выполнения

### Phase 1: Создание deploy-пользователя

**Никогда не работай от root в production.** Создай отдельного пользователя.

Проверь существует ли уже:
```bash
id deploy 2>/dev/null && echo "ПОЛЬЗОВАТЕЛЬ СУЩЕСТВУЕТ" || echo "НЕ СУЩЕСТВУЕТ"
```

Если не существует:
```bash
adduser --disabled-password --gecos "" deploy
mkdir -p /home/deploy/.ssh
cp /root/.ssh/authorized_keys /home/deploy/.ssh/
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys
```

Дай ограниченный sudo:
```bash
echo 'deploy ALL=(ALL) NOPASSWD: /usr/bin/docker, /usr/bin/docker-compose, /usr/bin/docker compose' > /etc/sudoers.d/deploy
chmod 440 /etc/sudoers.d/deploy
```

### Phase 2: SSH Hardening

**Предупреди участника:**
> Сейчас я настрою SSH — отключу вход по паролю и вход как root. После этого доступ будет только по SSH-ключу через пользователя deploy. Убедись что ты можешь подключиться как deploy прежде чем мы продолжим.

Создай конфиг:
```bash
mkdir -p /etc/ssh/sshd_config.d

tee /etc/ssh/sshd_config.d/99-hardening.conf > /dev/null << 'EOF'
PasswordAuthentication no
PermitRootLogin no
X11Forwarding no
MaxAuthTries 3
LoginGraceTime 30
AllowTcpForwarding no
AllowAgentForwarding no
PermitEmptyPasswords no
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers deploy
EOF
```

**Проверь конфиг ПЕРЕД перезапуском:**
```bash
sshd -t
```

Если валиден — перезапусти:
```bash
systemctl restart ssh 2>/dev/null || systemctl restart sshd
```

Если НЕ валиден — удали конфиг и сообщи об ошибке:
```bash
rm -f /etc/ssh/sshd_config.d/99-hardening.conf
```

### Phase 3: Firewall (UFW)

```bash
# Проверь текущий статус
ufw status

# Если не активен — настрой:
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment "SSH"
ufw allow 80/tcp comment "HTTP"
ufw allow 443/tcp comment "HTTPS"
ufw --force enable
```

Если уже активен — проверь что правила 22, 80, 443 присутствуют.

### Phase 4: Fail2ban

Проверь: `fail2ban-server --version 2>/dev/null`

Если не установлен:
```bash
apt install -y fail2ban
```

Настрой:
```bash
tee /etc/fail2ban/jail.local > /dev/null << 'EOF'
[sshd]
enabled = true
port = ssh
backend = systemd
maxretry = 3
findtime = 1h
bantime = 24h
bantime.increment = true
bantime.maxtime = 1w
EOF

systemctl enable fail2ban
systemctl restart fail2ban
```

### Phase 5: Docker

Проверь: `docker --version 2>/dev/null`

Если не установлен:
```bash
curl -fsSL https://get.docker.com | sh
```

Добавь deploy в группу docker:
```bash
groups deploy | grep -q docker || usermod -aG docker deploy
```

**Важно про порты Docker:**
> Docker по умолчанию обходит UFW и открывает порты напрямую. В `docker-compose.yml` всегда привязывай порты к 127.0.0.1:
> ```yaml
> ports:
>   - "127.0.0.1:5432:5432"  # НЕ "5432:5432"
> ```

### Phase 6: SSL с Let's Encrypt

```bash
apt install -y certbot python3-certbot-nginx

# Или если nginx ещё нет:
apt install -y nginx certbot python3-certbot-nginx
```

Получи сертификат (участник должен подставить свой домен):
```bash
certbot --nginx -d your-domain.com
```

Автообновление уже настроено (certbot ставит cron/timer автоматически).

### Phase 7: Kernel Hardening (sysctl)

```bash
**Важно:** Если Docker установлен, `ip_forward` ДОЛЖЕН быть `1`, иначе контейнеры не смогут общаться друг с другом и с интернетом.

```bash
# Проверь, установлен ли Docker
DOCKER_INSTALLED=$(systemctl is-active docker 2>/dev/null && echo "yes" || echo "no")

if [ "$DOCKER_INSTALLED" = "yes" ]; then
  IP_FORWARD_VALUE=1
else
  IP_FORWARD_VALUE=0
fi

tee /etc/sysctl.d/99-hardening.conf > /dev/null << EOF
net.ipv4.ip_forward = $IP_FORWARD_VALUE
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.log_martians = 1
fs.suid_dumpable = 0
kernel.randomize_va_space = 2
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
vm.swappiness = 10
EOF

sysctl --system
```

**Если Docker установлен** — перезапусти его после применения sysctl:
```bash
systemctl is-active docker &>/dev/null && systemctl restart docker
```

### Phase 8: Auto-updates

```bash
apt install -y unattended-upgrades

tee /etc/apt/apt.conf.d/20auto-upgrades > /dev/null << 'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
EOF

systemctl enable unattended-upgrades
```

### Phase 9: Swap (если нет)

Проверь: `swapon --show`

Если нет swap:
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
grep -q "/swapfile" /etc/fstab || echo "/swapfile none swap sw 0 0" >> /etc/fstab
```

### Phase 10: .env безопасность

```bash
# Убедись что .env файлы имеют правильные права
find /home/deploy -name ".env*" -exec chmod 600 {} \;
```

## Итоговый чеклист

После выполнения сообщи участнику:

```
Что настроено:
• deploy user (не root) с ограниченным sudo
• SSH: key-only, no root, AllowUsers deploy
• UFW: deny incoming, allow 22/80/443
• Fail2ban: 3 попытки → бан 24ч → до 1 недели
• sysctl: ASLR, SYN cookies, no redirects
• unattended-upgrades: security patches
• Swap: 2GB
• Docker: установлен, deploy в docker group

Важно помнить:
• Docker порты привязывать к 127.0.0.1
• .env файлы с правами 600
• Подключаться как deploy, не root
```
