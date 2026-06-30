---
title: Аварийное восстановление Hermes Agent
summary: Инструкция по восстановлению Hermes Agent при краше или потере конфигурации — откат config.yaml, восстановление из бэкапов, перезапуск gateway, профили агентов.
type: concept
created: 2026-05-30
updated: 2026-05-30
tags: [hermes, disaster-recovery, backup]
---

# Аварийное восстановление Hermes Agent

> **Для кого:** Владимир. Если Шиза (и все агенты) легли, а ты сам всё поднимаешь.
> **Дата актуальности:** 30 мая 2026, версия 0.13.0, апдейт на 0.15.2

---

## 1. Общая архитектура (что у нас стоит)

| Компонент | Где | Статус |
|-----------|-----|--------|
| Hermes Agent (0.13.0) | `/usr/local/lib/hermes-agent/` | Работает |
| Шиза (основная) | systemd: `hermes-gateway` | ✅ |
| Шнырь (разведчик) | systemd: `hermes-gateway-shnyr` | ✅ |
| Шы (заместитель) | systemd: `hermes-gateway-shy` | ✅ |
| Max Messenger (все 3) | Встроен в gateway-процессы | В каждом профиле |
| Telegram (Шиза) | Встроен в `hermes-gateway` | ✅ |
| Дашборд (финансы+заказы) | Port 8080, systemd | FastAPI |
| MariaDB (orders+finance) | localhost:3306 | ✅ |
| Obsidian vault | GitHub: chukcha1977/obsidian-shyza-vault | ✅ |
| Google OAuth | `/root/.google-credentials/` | Gmail, Calendar, Drive |
| LM Studio Proxy | systemd: `hermes-lm-proxy` | Port 1337 (ноутбук) |
| Whisper Proxy | systemd: `hermes-whisper-proxy` | Ноутбук |

---

## 2. Что делать, если Шиза не отвечает (полная авария)

### 2.1. Проверить статус сервисов

```bash
systemctl status hermes-gateway
systemctl status hermes-gateway-shnyr
systemctl status hermes-gateway-shy
```

Если все dead — идти дальше. Если живы, но не отвечают в Max — проверить логи:

```bash
grep -i "error\|failed\|traceback" ~/.hermes/logs/gateway.log | tail -30
```

### 2.2. Рестарт gateway-сервисов

```bash
# По одному
systemctl restart hermes-gateway
systemctl restart hermes-gateway-shnyr
systemctl restart hermes-gateway-shy

# Или все сразу
for s in hermes-gateway hermes-gateway-shnyr hermes-gateway-shy; do
  echo "=== $s ==="
  systemctl restart "$s"
  sleep 3
  systemctl is-active "$s"
done
```

### 2.3. После рестарта

Gateway пишет новый marker=0 в Max adapter — теряются сообщения за последнюю минуту. Напиши что-нибудь в чат, чтобы проверить.

---

## 3. Полное восстановление из бэкапа (Шиза не поднялась после апдейта)

### 3.1. Остановить ВСЕ сервисы

```bash
systemctl stop hermes-gateway
systemctl stop hermes-gateway-shnyr
systemctl stop hermes-gateway-shy
systemctl stop hermes-lm-proxy
systemctl stop hermes-whisper-proxy
```

### 3.2. Выбрать бэкап

Последний полный бэкап: `/root/backups/full_20260527_144112/`
Если его нет — есть бэкапы в `/root/.hermes/backups/` (только конфиги).

Что содержит `full_20260527_144112/`:
- `config.yaml` — конфиг Hermes (актуальный на 27.05)
- `dashboard/` — дашборд целиком
- `skills/` — все навыки (~32 шт)
- `scripts/` — все скрипты
- `alldbs.sql` — все БД (chuchm_orders + chuchm_finance)
- `config.py` — конфиг дашборда

**Дополнительные бэкапы на сервере:**
- `/root/.hermes/backups/config.yaml.bak.20260524.v3` — самый свежий конфиг в бэкапах (до 24 мая)
- `/root/.hermes/backups/daily/` — ежедневные бэкапы (если есть)
- `/root/backups/dashboard/` — дашборд с бэкапами app.py

### 3.3. Восстановление Hermes Agent (код)

**Вариант A: PyPI (простая установка):**
```bash
pip3 install hermes-agent==0.15.2
hermes setup  # пройтись по визарду
```

**Вариант B: Git (исходники, рекомендуем — сохраняет плагины):**
```bash
cd /usr/local/lib/hermes-agent
git reset --hard HEAD
git fetch origin
git checkout main
git pull --ff-only origin main
source venv/bin/activate
pip install -e . --break-system-packages
```

### 3.4. Восстановление конфига

```bash
# Если config.yaml испорчен:
cp /root/backups/full_20260527_144112/config.yaml /root/.hermes/config.yaml

# Или из бэкапа .hermes:
cp ~/.hermes/backups/config.yaml.bak.20260524.v3 /root/.hermes/config.yaml
```

**ВАЖНО:** После восстановления из бэкапа от 27.05 нужно:
1. Добавить POLZA_KEY_3 в `.env` (создана позже, её может не быть в старом бэкапе)
2. Проверить auxiliary.compression.context_length: 1000000
3. Проверить `prefill_messages_file: /root/.hermes/prefill_interaction_laws.json`

### 3.5. Восстановление .env

Если `.env` пустой или повреждён:
1. Открыть `/root/.hermes/.env`
2. Проверить наличие переменных:
   - `POLZA_KEY_1`, `POLZA_KEY_2`, `POLZA_KEY_3` — три ключа Polza.ai
   - `MAX_BOT_TOKEN` — токен бота Max (для Шизы). Длинная строка — есть в .env
   - `TELEGRAM_BOT_TOKEN` — токен бота Telegram
3. Если .env нет совсем — восстановить из любого бэкапа `.hermes`

### 3.6. Восстановление SOUL.md и personality

```bash
# Копия в бэкапе:
cp /root/.hermes/backups/SOUL.md.bak.20260524.v3 /root/.hermes/SOUL.md
```

Также проверить `display.personality: shiza` в config.yaml.

### 3.7. Восстановление дашборда

```bash
systemctl stop dashboard  # если есть
cp -r /root/backups/full_20260527_144112/dashboard/* /root/dashboard/
systemctl start dashboard
```

### 3.8. Восстановление БД

```bash
mysql -uadmin -p'ChuchmDb!Admin2025' < /root/backups/full_20260527_144112/alldbs.sql
```

**Внимание:** пароль в одинарных кавычках! `!` в bash интерпретируется, если без кавычек.

### 3.9. Восстановление навыков

```bash
cp -r /root/backups/full_20260527_144112/skills/* /root/.hermes/skills/
```

### 3.10. Восстановление профилей (Шнырь + Шы)

Если профили пропали (встречается при `hermes model` — он убивает кастомные настройки):

```bash
# Навыки для Шныря:
cp -r /root/backups/full_20260527_144112/skills/* /root/.hermes/profiles/shnyr/skills/
# SOUL.md Шныря — вручную из vault или из session_search
# SOUL.md Шы — вручную из vault
```

**Содержимое SOUL.md Шныря:** см. vault `concepts/multi-agent-architecture.md`
**Содержимое SOUL.md Шы:** там же.

---

## 4. Проверка работоспособности после восстановления

### 4.1. Базовые проверки

```bash
# Hermes жив?
hermes --version
# Должно показать: hermes-agent 0.15.2 (после апдейта)

# Config валиден?
python3 -c "import yaml; yaml.safe_load(open('/root/.hermes/config.yaml'))" && echo "OK"

# Gateway жив?
systemctl is-active hermes-gateway
```

### 4.2. Проверка Max Messenger

Напиши в чат Шизы что-нибудь. Если нет ответа в течение 30 секунд:

```bash
grep "Max" ~/.hermes/logs/gateway.log | tail -20
```

Ищи типичные ошибки:
- `MAX_BOT_TOKEN not set` — нет токена в .env
- `HTTP 40x` — истёк токен Max
- `error` — смотри стёк

### 4.3. Проверка Telegram

Напиши в Telegram боту. Если молчит:

```bash
grep "Telegram\|error" ~/.hermes/logs/gateway.log | tail -20
```

### 4.4. Проверка дашборда

```bash
curl -s http://localhost:8080/ | head -5
```

Должен вернуть HTML. Если нет:
```bash
systemctl status dashboard 2>/dev/null || echo "Нет systemd-сервиса"
fuser -k 8080/tcp 2>/dev/null
cd /root/dashboard && python3 app.py &
```

### 4.5. Проверка БД

```bash
mysql -uadmin -p'ChuchmDb!Admin2025' -e "SELECT COUNT(*) FROM chuchm_orders.orders;"
mysql -uadmin -p'ChuchmDb!Admin2025' -e "SELECT COUNT(*) FROM chuchm_finance.transactions;"
```

---

## 5. Типичные проблемы и решения

| Проблема | Причина | Решение |
|----------|---------|---------|
| Gateway не стартует после апдейта | Изменения в API | Проверить `grep error ~/.hermes/logs/gateway.log \| tail -20` |
| Max не видит сообщения | Marker=0 после рестарта | Игнорировать, через 2-3 сообщения восстановится |
| «Primary provider auth failed» | Нет Polza ключа в .env | Проверить POLZA_KEY_3 |
| «Failed to process config.yaml» | YAML сломан | `python3 -c "import yaml; yaml.safe_load(open('config.yaml'))"` — покажет ошибку |
| Max adapter не регистрируется | Max plugin лежит вне стандартного пути | Он в `/root/.hermes/hermes-agent/plugins/platforms/max/` |
| Шнырь/Шы не отвечают | У них свой профиль | Проверить `systemctl status hermes-gateway-shnyr` |
| Telegram работает, Max нет | Telegram — built-in, Max — plugin | Разные механизмы загрузки |

---

## 6. Крайний случай: ручной запуск Шизы

Если systemd не работает, можно запустить вручную:

```bash
# Для проверки в консоли (не gateway, просто чат):
cd /usr/local/lib/hermes-agent
source venv/bin/activate
hermes chat

# Для gateway (в фоне):
nohup hermes gateway run --replace > /root/gateway-manual.log 2>&1 &
```

---

## 7. Важные файлы и пути (шпаргалка)

| Файл/путь | Назначение |
|-----------|------------|
| `~/.hermes/config.yaml` | Главный конфиг |
| `~/.hermes/.env` | Все ключи API |
| `~/.hermes/SOUL.md` | Личность Шизы |
| `~/.hermes/prefill_interaction_laws.json` | Prefill для законов |
| `~/.hermes/profiles/shnyr/` | Профиль Шныря |
| `~/.hermes/profiles/shy/` | Профиль Шы |
| `~/.hermes/skills/` | Все навыки |
| `~/.hermes/hermes-agent/plugins/platforms/max/` | Max adapter (plugin) |
| `/usr/local/lib/hermes-agent/` | Код Hermes |
| `/root/dashboard/` | Дашборд (FastAPI) |
| `/root/.google-credentials/` | Google OAuth токен |

---

## 8. Если совсем всё плохо — написать Владимиру в Telegram

> Токен Telegram бота есть в .env. Если у тебя есть доступ к Telegram — напиши боту напрямую (не через Max). Если и Telegram не отвечает — проверь `TELEGRAM_BOT_TOKEN` в `.env` и `/etc/systemd/system/hermes-gateway.service`.

---

**Связанные концепции:** [[concepts/hermes-architecture-reference|Архитектура Hermes]] | [[concepts/hermes-slash-commands|Slash-команды]] | [[concepts/restore|RESTORE — инструкция]] | [[concepts/restore-guide|Восстановление из бэкапа (для Шы)]]