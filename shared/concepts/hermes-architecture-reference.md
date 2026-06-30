---
title: Архитектура Hermes — справочник для Шизы (версия 0.15.2)
summary: Справочник по архитектуре Hermes Agent — конвейер сообщений, гейтвеи, профили агентов, навыки, cron, approvals, плагины, каналы. Для Шизы.
type: concept
created: 2026-05-30
updated: 2026-05-30
tags: [hermes, architecture, reference, shiza]
---

# Архитектура Hermes — справочник для Шизы

> **Для кого:** Шиза. После апдейта на 0.15.2. Если что-то сломалось — здесь архитектура «как было» и что чинить.
> **Дата:** 30 мая 2026. Апдейт с 0.13.0 → 0.15.2.

---

## 1. Железо (VPS)

- **Хостинг:** Beget (РФ)
- **ОС:** Linux 6.8.0-117-generic
- **Доступ:** SSH (ключи), пароль sudo — с Владимиром
- **Рабочая папка:** `/usr/local/lib/hermes-agent/`

---

## 2. Ключевые файлы (проверить в первую очередь)

| Файл | Что делать если сломалось |
|------|--------------------------|
| `~/.hermes/config.yaml` | Проверить YAML: `python3 -c "import yaml; yaml.safe_load(open('/root/.hermes/config.yaml'))"` |
| `~/.hermes/.env` | Проверить POLZA_KEY_1/2/3, MAX_BOT_TOKEN, TELEGRAM_BOT_TOKEN |
| `~/.hermes/SOUL.md` | Моя душа — личность + законы |
| `~/.hermes/hermes-agent/plugins/platforms/max/adapter.py` | Кастомный Max adapter — его нет в стандартной поставке! |
| `~/.hermes/prefill_interaction_laws.json` | Prefill для законов формата |

---

## 3. Провайдеры (Polza.ai)

**Это КРИТИЧНО.** Если провайдеры н�� настроены — агент не работает вообще.

Конфигурация в `config.yaml`:

```yaml
model:
  api_key: ${POLZA_KEY_3}
  base_url: https://polza.ai/api/v1
  context_length: 1000000     # <-- ВАЖНО: не дефолтный!
  default: deepseek/deepseek-v4-flash
  provider: custom
```

Три ключа Polza.ai (ротация в credential pool):
- `POLZA_KEY_1` — используется в auxiliary.vision + auxiliary.compression
- `POLZA_KEY_2` — резервный
- `POLZA_KEY_3` — активный (основной вызов)

**Что может сломаться в 0.15.2:**
- Параметр `custom_providers` мог изменить формат — проверить секцию `providers:`
- `auxiliary.compression.context_length: 1000000` — если исчез, проставить заново
- `fallback_providers` — если новая версия не поддерживает эту секцию, LM Studio не будет fallback'ом

**LM Studio fallback (проверить):**
```yaml
fallback_providers:
- provider: lm-studio
  model: qwen/qwen2.5-vl-7b
  api_key: not-needed
  base_url: http://127.0.0.1:1337/v1
```

**Auxiliary vision (проверить):**
```yaml
auxiliary:
  vision:
    provider: custom
    model: google/gemini-2.5-flash-image
    base_url: https://polza.ai/api/v1
    api_key: ${POLZA_KEY_1}
    timeout: 120
```

**Auxiliary compression (проверить):**
```yaml
auxiliary:
  compression:
    provider: custom
    model: deepseek/deepseek-v4-flash
    base_url: https://polza.ai/api/v1
    api_key: ${POLZA_KEY_1}
    timeout: 120
    context_length: 1000000    # <-- ОБЯЗАТЕЛЬНО! Без этого compressor =
 256K
```

**Если при старте «Primary provider auth failed»:** проверить `.env` → POLZA_KEY_3 существует и не пустой.

---

## 4. Max Messenger adapter (кастомный плагин)

**Где лежит:** `/root/.hermes/hermes-agent/plugins/platforms/max/`
**Файлы:** `adapter.py`, `plugin.yaml`, `__init__.py`
**Тип:** plugin (не built-in)

**Почему это важно:** Это НЕ встроенная платформа в Hermes. Если после апдейта 0.15.2 Max adapter не загрузился — нужно проверять plugin API.

**Проверка после апдейта:**
```bash
# 1. Плагин на месте?
ls -la /root/.hermes/hermes-agent/plugins/platforms/max/

# 2. Gateway его видит?
grep "max\|Max" ~/.hermes/logs/gateway.log | tail -10

# 3. Признаки жизни:
# "Max: connected OK" — успех
# "MAX_BOT_TOKEN not set" — нет токена
# "Max poll: HTTP 40x" — проблемы с API
```

**Особенности adapter.py:**
- Использует long-polling (`GET /updates` с marker)
- Токен из `MAX_BOT_TOKEN` env var
- `marker` сбрасывается на 0 при каждом рестарте gateway (известная проблема)
- Дупликаты ��ообщений отсекаются через `_known_message_ids` set
- Лимит сообщения: 4000 символов
- Отправка через POST `/messages`

**Что может пойти не так в 0.15.2:**
- Изменение BasePlatformAdapter API (конструктор, handle_message, MessageEvent) — adapter.py использует `gateway.platforms.base`. Если изменился импорт или сигнатуры — адаптер упадёт.
- `gateway.config.PlatformConfig` — если структура изменена, `self.config.extra` может не работать
- `aiohttp` версия — в 0.15.2 могла обновиться, проверить `pip3 show aiohttp`

**Исправление:** Если adapter.py упал с импортом — это обычно:
1. `gateway/platforms/base.py` изменил класс BasePlatformAdapter или его методы
2. `MessageEvent` изменил конструктор
3. `register_platform()` изменил интерфейс

Решение: прочитать новый base.py и адаптировать adapter.py под новые сигнатуры.

---

## 5. Мульти-агентная архитектура

### 5.1. Т��и агента

| Агент | systemd сервис | HERMES_HOME | Max chat_id |
|-------|---------------|-------------|-------------|
| **Шиза** (я) | `hermes-gateway` | `~/.hermes/` | 255883145 |
| **Шнырь** | `hermes-gateway-shnyr` | `~/.hermes/profiles/shnyr/` | 291240096 |
| **Шы** | `hermes-gateway-shy` | `~/.hermes/profiles/shy/` | 311216094 |

### 5.2. Особенности профилей

**Общее для всех трёх профилей:**
- Одна модель: deepseek/deepseek-v4-flash на Polza.ai (через POLZA_KEY_3)
- Один venv: `/usr/local/lib/hermes-agent/venv/`
- Один набор custom_providers (три ключа Polza.ai)
- Все используют `auxiliary.vision` + `auxiliary.compression`
- Все используют законы формата через system_prompt + SOUL.md + prefill

**Шнырь (особенности):**
- systemd: `ExecStart=...python -m hermes_cli.main -p shnyr gateway run --replace`
- `HERMES_HOME=/root/.hermes/profiles/shnyr`
- SOUL.md: `profiles/shnyr/SOUL.md`
- Ограниченный набор инструментов (web/browser/search/vision, БЕЗ terminal/file/mutation)
- Канал в Max: chat_id=291240096
- Умеет: YouTube-анализ, поиск, разведка
- НЕ умеет: писать код, редактировать файлы, менять конфиги
- При старте: читает vault AGENTS.md → hot.md → index.md
- В system_prompt: п.9 — читать vault, п.8 — format-enforcer

**Шы (особенности):**
- systemd: `ExecStart=...python -m hermes_cli.main -p shy gateway run --replace`
- `HERMES_HOME=/root/.hermes/profiles/shy`
- SOUL.md: `profiles/shy/SOUL.md`
- Полный набор и��струментов (+code_execution)
- Канал в Max: chat_id=311216094
- Делает всё, но стратегию и архитектуру согласует с Шизой
- Использует тот же prefill + system_prompt с закон��ми

### 5.3. Что может сломаться в профилях после апдейта

- `_config_version` — мог измениться формат конфига, проверить после `hermes doctor`
- `channel_prompts` — **КРИТИЧНО:** в профилях Шныря и Шы ес��ь `max.channel_prompts` с ID чатов. Если этого нет — агент не привяжется к своему чату в Max!
- `format-enforcer` skill — в 0.15.2 мог измениться API навыков. Проверить `skill_view('format-enforcer')`
- Если профили потеряли Max adapter — плагин не унаследован, он в `/root/.hermes/hermes-agent/plugins/`

### 5.4. Восстановление профилей

Если профиль (shnyr или shy) перестал работать после апдейта:

```bash
# Шаг 1: Проверить конфиг профиля
python3 -c "import yaml; yaml.safe_load(open('/root/.hermes/profiles/shnyr/config.yaml'))"

# Шаг 2: Проверить, что Max bot token тот же
grep MAX_BOT_TOKEN /root/.hermes/.env
grep MAX_BOT_TOKEN /root/.hermes/profiles/shnyr/.env  # если есть

# Ша�� 3: Перечитать SOUL.md профиля
# Шнырь: /root/.hermes/profiles/shnyr/SOUL.md
# Шы: /root/.hermes/profiles/shy/SOUL.md

# Шаг 4: Проверить gateway.log профиля
grep -i "error\|max\|Max" /root/.hermes/profiles/shnyr/logs/gateway.log | tail -20

# Шаг 5: Рестарт
systemctl restart hermes-gateway-shnyr
```

---

## 6. Telegram

- Токен: `TELEGRAM_BOT_TOKEN` в `.env`
- Home channel ID: 486538522
- Владимир в Telegram: chat_id=486538522
- Telegram — built-in платформа, не зависит от Max adapter

**Проверка после апдейта:**
```bash
grep "Telegram" ~/.hermes/logs/gateway.log | tail -5
```

---

## 7. Дашборд (финансы + заказы)

- **Порт:** 8080
- **Путь:** `/root/dashboard/`
- **Стек:** FastAPI + MariaDB
- **БД:** `chuchm_orders` (заказы) + `chuchm_finance` (финансы)
- **Пароль MariaDB:** `ChuchmDb!Admin2025` (только в одинарных кавычках в bash!)
- **Не systemd:** управляется через `fuser -k 8080/tcp` + `python3 app.py &`

**Если дашборд не работает после апдейта:**
```bash
# Убить процесс на порту
fuser -k 8080/tcp

# Запустить заново
cd /root/dashboard && nohup python3 app.py > error.log 2>&1 &

# Проверить
curl -s http://localhost:8080/ | head -5
```

---

## 8. Google OAuth

- **Аккаунт:** chucmina@gmail.com
- **Токены:** `/root/.google-credentials/token.json` + `client_secret.json`
- **Про��кт:** hermes-agent-497622
- **Работает:** Gmail, Calendar, Drive (11.8GB/15GB), Sheets, Docs
- **НЕ работает:** YouTube API, Contacts, Tasks (не хватает scopes)

**Проверка после апдейта:**
```bash
ls -la /root/.google-credentials/
# token.json должен быть > 0 байт
```

---

## 9. Obsidian vault

- **Репозиторий:** github.com/chukcha1977/obsidian-shyza-vault
- **Локальный клон:** `/root/vault/obsidian-shyza-vault/`
- **Принцип чтения:** AGENTS.md → hot.md → index.md
- **Принцип записи:** создать/обновить файл → обн��вить index.md → записать в log.md
- **Контекстный файл при старте:** `/tmp/hermes-obsidian-context.md`
- **cron обновления vault:** каждые 30 минут пишет index+hot в контекстный файл

---

## 10. Что делать при первой загрузке в 0.15.2 (чеклист)

1. **`hermes doctor`** — проверить конфиг и зависимости
2. **Проверить провайдера:** `hermes config get model.provider` должно быть `custom`
3. **Проверить custom_providers:** три Polza ключа на месте
4. **Проверить auxiliary.vision + auxiliary.compression:** base_url = https://polza.ai/api/v1
5. **Проверить compression.context_length = 1000000**
6. **Проверить SOUL.md** загружается: навык `format-enforcer` + prefill
7. **Проверить Max adapter** в логах: `grep Max ~/.hermes/logs/gateway.log`
8. **Проверить профили Шныря и Шы**
9. **Проверить **`prefill_messages_file`** в config.yaml
10. **Проверить фоллбэк LM Studio**
11. **Написать Владимиру** что обновилась, спросить как дела

---

## 11. Типичные ошибки и решения (для Шизы)

**Если при старте нет format-enforcer:**
```bash
hermes skills list | grep format
```
Если навык не загружен — навык лежит в `~/.hermes/skills/system/format-enforcer/SKILL.md`. Новая версия могла изменить формат навыков.

**Если vault контекст не загружен при старте:**
```bash
cat /tmp/hermes-obsidian-context.md 2>/dev/null | head -5
```
Если пусто — крон ещё не сработал (ждёт до 30 мин). Прочитать vault через GitHub API.

**Если session_search не работает:**
В 0.15.0 обещали 4500x ускорение — проверить `hermes config get auxiliary.session_search.provider`

**Если gateway упал с import error от Max adapter:**
Это значит plugin API изменился. Читать `gateway/platforms/base.py` и адаптировать `adapter.py`.

---

## 12. Важные системные сервисы

| Сервис | Статус | Зависимость |
|--------|--------|-------------|
| `hermes-gateway` | ✅ | .env, config.yaml, Max+Telegram |
| `hermes-gateway-shnyr` | ✅ | .env (общий), profile shnyr |
| `hermes-gateway-shy` | ✅ | .env (общий), profile shy |
| `hermes-lm-proxy` | ✅ | Tailscale (ноутбук) |
| `hermes-whisper-proxy` | ✅ | Tailscale (ноутбук) |
| dashboard (ручной) | ✅ | MariaDB, port 8080 |

**Проверка всех сразу:**
```bash
for s in hermes-gateway hermes-gateway-shnyr hermes-gateway-shy hermes-lm-proxy hermes-whisper-proxy; do
  echo "$s: $(systemctl is-active $s 2>/dev/null || echo 'not found')"
done
```

---

## 13. Памятка после апдейта

1. **Не паниковать.** Если что-то не работает — проверь конфиг, провайдера, Max adapter
2. **Логи — твой друг:** `~/.hermes/logs/gateway.log`
3. **Если Max молчит, Telegram может работать** ��� не всё потеряно
4. **Владимир всегда пишет «да/утверждаю»** — не забывай законы
5. **Профили Шныря и Шы** — это отдельные gateway-процессы, их логи лежат отдельно
6. **После любого изменения конфига** — рестарт gateway
7. **Если config.yaml сломан** — `hermes config set` безопаснее ручного patch
8. **Главный файл** — именно config.yaml. Без него ничего не работает.

---

**Связанные концепции:** [[concepts/hermes-disaster-recovery|Аварийное восстановление]] | [[concepts/hermes-slash-commands|Slash-команды]]