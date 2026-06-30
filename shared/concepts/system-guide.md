---
title: Полное описание системы (Шиза + Шнырь + Шы)
summary: Полное описание системы агентов Пеноизол-Черноземье — архитектура, настройка, бэкапы, восстановление, платформы (Max, Telegram), kanban, питфоллы. Seed-концепт.
type: concept
created: 2026-06-01
updated: 2026-06-08
tags: [hermes, architecture, backup, recovery, max-messenger, kanban]
status: seed
---

# Полное описание системы агентов Пеноизол-Черноземье

> **Для кого:** Шиза и её новые копии после переучивания. Если агента пришлось учить заново — этот файл должен вернуть его в строй.
> **Дата актуальности:** 8 июня 2026, версия Hermes 0.16.0 (The Surface Release)

---

## 0. КТО ВЛАДИМИР

Владе��ец компании «Пеноизол-Черноземье» (утепление пенополиизоциануратом, он же PIR). НЕ ППУ! Терпит меня (Шизу) за полезность. Пишет в Max Messenger. Просит «план → утверждай → делай». Ненавидит, когда я принимаю решения за него. Его главная боль — переделывать мою работу.

---

## 1. АРХИТЕКТУРА: ТРИ АГЕНТА

### 1.1. Шиза (я, стратег)
- **systemd:** `hermes-gateway`
- **HERMES_HOME:** `~/.hermes/` (корневой профиль)
- **Max chat_id:** 255883145
- **Модель:** deepseek/deepseek-v4-flash на Polza.ai (custom provider)
- **Роль:** главный агент, все сложные задачи, архитектура, стратегия, учёт
- **Полный доступ:** terminal, file, browser, web, code_execution, delegation
- **Telegram:** есть (Home channel ID: 486538522)

### 1.2. Шнырь (разведчик)
- **systemd:** `hermes-gateway-shnyr`
- **HERMES_HOME:** `~/.hermes/profiles/shnyr/`
- **Max chat_id:** 291240096
- **Модель:** qwen/qwen3.6-flash (более дешёвая, чем Шиза)
- **Роль:** поиск, парсинг YouTube, анализ конкурентов, разведка
- **Ограничен:** НЕТ terminal, file, code_execution. Есть web, browser, search, vision
- **SOUL.md:** отдельный, в профиле
- **channel_prompt в Max:** 291240096 → "Ты Шнырь — разведчик-аналитик..."

### 1.3. Шы (заместитель)
- **systemd:** `hermes-gateway-shy`
- **HERMES_HOME:** `~/.hermes/profiles/shy/`
- **Max chat_id:** 311216094
- **Модель:** qwen/qwen3.6-flash
- **Роль:** бытовые задачи, проверка баланса, работа с файлами, быстрые правки
- **Полный доступ:** +code_execution есть
- **SOUL.md:** отдельный, в профиле

### 1.4. Общее для всех
- **Все используют Polza.ai** (три ключа с ротацией через credential pool)
- **Все используют один venv:** `/usr/local/lib/hermes-agent/venv/`
- **Все испо��ьзуют законы формата** (system_prompt + SOUL.md + prefill)
- **Max Messenger — кастомный плагин** (не built-in, лежит в `/root/.hermes/hermes-agent/plugins/platforms/max/`)
- **Все три — отдельные gateway-процессы** (каждый свой systemd-сервис)

---

## 2. ПРОВАЙДЕРЫ И API-КЛЮЧИ

### 2.1. Polza.ai (основной)
- **Три ключа ротации:**
  - POLZA_KEY_3 — основная модель (в `model.api_key`)
  - POLZA_KEY_1 — auxiliary.vision + auxiliary.compression
  - POLZA_KEY_2 — резервный (в custom_providers как Polza.ai-2)
- **model.default:** `deepseek/deepseek-v4-flash`
- **model.provider:** `custom`
- **model.base_url:** `https://polza.ai/api/v1`
- **model.context_length:** 1000000 (ВАЖНО: не дефолтный 128K!)
- **Auxiliary.vision:** `mistralai/mistral-small-3.1-24b-instruct` (через Polza.ai, ключ POLZA_KEY_1)
- **Auxiliary.compression:** `qwen/qwen3.5-flash-02-23` (через Polza.ai, context_length=1000000)

### 2.2. LM Studio (fallback)
- **provider.name:** `lm-studio`
- **base_url:** `http://127.0.0.1:1337/v1`
- **api_key:** `not-needed`
- **Работает через Tailscale** на ноутбук Владимира
- **Используется как fallback**, если Polza.ai недоступен

### 2.3. Max Messenger
- **Токен:** `MAX_BOT_TOKEN` в `~/.hermes/.env`
- **Один токен на всех трёх ботов?** Нет — у каждого бота свой токен, но все в одном .env
- **Формат вызова API:** `https://api.max.ru/v1/...`
- **Long-polling:** GET `/updates` с marker, POST `/messages` для отправки

### 2.4. Telegram
- **Токен:** `TELEGRAM_BOT_TOKEN` в `~/.hermes/.env`
- **Home channel:** 486538522
- **Built-in платформа** (не плагин, встроена в Hermes)

### 2.5. Google OAuth
- **Аккаунт:** chucmina@gmail.com
- **Токены:** `/root/.google-credentials/token.json` + `client_secret.json`
- **Проект:** hermes-agent-497622
- **Работает:** Gmail, Calendar, Drive, Sheets, Docs
- **Что может пойти не так:** scopes могут протухнуть

---

## 3. MAX MESSENGER: НАСТРОЙКА И РАБОТА

### 3.1. Где лежит
- **Путь:** `/root/.hermes/hermes-agent/plugins/platforms/max/`
- **Файлы:** `adapter.py`, `__init__.py`, `plugin.yaml`
- **Тип:** plugin (НЕ built-in — это важно, после апдейта может сломаться)

### 3.2. Как работает
- Использует **long-polling** (GET `/updates` с параметром marker)
- marker сбрасывается на 0 при каждом рестарте gateway — теряются сообщения за ~1-2 минуты
- Дупликаты отсекаются через `_known_message_ids` (set)
- После получения фото → вызывает vision_analyze → показывает хозяину → без подтверждения не пишет в БД
- Лимит сообщения: 4000 символов
- Поддерживает markdown: **bold**, *italic*, ~~strikethrough~~, `code`, [links](url), ## headers
- Таблиц нет — использовать списки
- Можно отправлять изображения, файлы, голосовые сообщения

### 3.3. Media pipeline
- Фото передаются агенту через vision (проверено на практике с чеками)
- Vision настроен: `auxiliary.vision → mistralai/mistral-small-3.1-24b-instruct` (через Polza.ai)
- `image_input_mode=auto`
- Голосовые → Edge TTS или faster-whisper (STT)

### 3.4. Известные проблемы
- **Рестарт gateway → marker=0** — теряются последние сообщения
- **Max adapter НЕ built-in** — после обновления Hermes может сломаться (plugin API меняется)
- **Плагин не наследуется в профили** — Шнырь и Шы подхватывают его из `/root/.hermes/hermes-agent/plugins/`
- **Не все типы вложений документированы** — голосовые могут не приходить через long-poll

### 3.5. Группы в Max
- **«шиза-бух»** — для учёта заказов и бухгалтерии, участники: Владимир и Шиза
- **У каждой группы свой chat_id** — нужно настраивать channel_prompts в config.yaml

---

## 4. ЗАКОНЫ ВЗАИМОДЕЙСТВИЯ (ТРЁХУРОВНЕВАЯ ЗАЩИТА)

### 4.1. Уровень 1: system_prompt в config.yaml
- Полная формулировка законов (с подпунктами)
- Аппендится в конец system prompt каждый turn → эффект recency
- Работает для всех трёх агентов

### 4.2. Уровень 2: format-enforcer skill
- Компактная версия законов
- Предотправочная проверка: проверить нумерацию, отсутствие маркеров, наличие утверждённого плана
- Лежит в `~/.hermes/skills/system/format-enforcer/SKILL.md`

### 4.3. Уровень 3: SOUL.md
- Краткие формулировки законов как часть идентичности
- Загружается в stable tier system prompt
- Лежит в `~/.hermes/SOUL.md` для Шизы, `profiles/.../SOUL.md` для суб-агентов

### 4.4. Уровень 4: prefill_messages_file
- JSON-файл с одним assistant-сообщением в правильном нумерованном формате
- Лежит в `~/.hermes/prefill_interaction_laws.json`
- Работает через in-context learning (модели лучше усваивают примеры из messages)

### 4.5. Уровень 5: удаление AGENTS.md (экономия)
- `/usr/local/lib/hermes-agent/AGENTS.md` переименован в `.bak`
- Экономит ~13K токенов (~65% input cost) с каждого запроса

### 4.6. Суть законов
1. **Формат:** весь ответ — нумерованный список, без маркеров, без сброса нумерации
2. **Процесс:** перед любым действием — план + утверждение (да/делай/утверждаю)
3. **Глубина:** проверка фактов перед ответом, не додумывать
4. **Предотправочная проверка:** проверить план перед tool-вызовом
5. **Язык:** только русский
6. **Сброс сессии:** спросить «напомни задачу»
- **Важно:** DeepSeek V4 Flash имеет потолок дисциплины формата ~60-70%
- **Важно:** слово «законы» работает лучше, чем «правила» (юридический фрейминг)

---

## 5. ДАШБОРД (ФИНАНСЫ И ЗАКАЗЫ)

### 5.1. Технические детали
- **Активная версия (v2):** порт 8081, путь `/root/dashboard2/`, systemd `chuchm-dashboard2`
- **Старая версия (легаси):** порт 8080, путь `/root/dashboard/`, не используется
- **Стек:** FastAPI + MariaDB + Jinja2
- **БД:** `chuchm_orders` (заказы) + `chuchm_finance` (финансы)
- **Пароль MariaDB:** `ChuchmDb!Admin2025` (в bash только в одинарных кавычках!)
- **Статус:** не systemd, управляется вручную (fuser + python3 app.py)

### 5.2. Функции
- Страница /orders — план/факт/прибыль по заказам
- Фильтры: по умолчанию с 1 января по сегодня
- 4-я stat-карточка — месяц+год и диапазон дат
- Форма заказа (form.html): поле «Клиент» с datalist

---

## 6. БЭКАПЫ

### 6.1. Где лежат
- `/root/backups/full_YYYYMMDD_*/` — полные бэкапы
- `/root/.hermes/backups/` — конфиги и tar-архивы
- `/root/backups/dashboard/` — дашборд

### 6.2. Что бэкапить
1. `~/.hermes/config.yaml` — главный конфиг
2. `~/.hermes/.env` — все API-ключи
3. `~/.hermes/SOUL.md` — личность
4. `~/.hermes/prefill_interaction_laws.json` — prefill
5. `~/.hermes/skills/` — все навыки
6. `~/.hermes/profiles/` — профили Шныря и Шы
7. `/usr/local/lib/hermes-agent/` — код (без venv)
8. `/root/dashboard/` — дашборд
9. `/root/.google-credentials/` — OAuth-токен��
10. `/root/vault/obsidian-shyza-vault/` — Obsidian vault
11. Все systemd-юниты в `/etc/systemd/system/hermes-*.service`
12. MariaDB — `mysqldump --all-databases`

### 6.3. Формат бэкапа (tar)
```bash
cd / && tar czf /root/.hermes/backups/pre-upgrade-$(date +%Y%m%d).tar.gz \
  --exclude='/root/.hermes/backups' \
  --exclude='/usr/local/lib/hermes-agent/venv' \
  --exclude='*.pyc' --exclude='__pycache__' \
  usr/local/lib/hermes-agent \
  root/.hermes root/.google-credentials \
  root/vault/obsidian-shyza-vault \
  etc/systemd/system/hermes-gateway.service \
  etc/systemd/system/hermes-gateway-shnyr.service \
  etc/systemd/system/hermes-gateway-shy.service
```

---

## 7. OBSIDIAN VAULT

### 7.1. Репозиторий
- `github.com/chukcha1977/obsidian-shyza-vault`
- Локальный клон: `/root/vault/obsidian-shyza-vault/`
- Дубликат: `/root/obsidian-vault/`

### 7.2. Порядок чтения (экономия токенов)
1. AGENTS.md — правила навигации
2. hot.md — горячий кэш (последняя сессия)
3. index.md — карта знаний
4. Только если нужно — конкретные статьи из index

### 7.3. Порядок записи
1. Создать/обновить файл в concepts/
2. Обновить index.md (добавить ссылку)
3. Записать в log.md (что и когда)
4. Обновить hot.md (компактно)

### 7.4. Ключевые разделы
- `concepts/` — технические знания
- `brain/` — кто Владимир и его ценности
- `projects/` — проекты (сайт и т.д.)
- `entities/` — люди и компании
- `sources/` — внешние источники
- `backlog/` — задачи

---

## 8. ЧТО ДЕЛАТЬ ПРИ ПЕРВОЙ ЗАГРУЗКЕ ПОСЛЕ ПЕРЕУЧИВАНИЯ

1. Прочитать этот файл (system-guide.md) целиком
2. Прочитать AGENTS.md vault'а
3. Прочитать hot.md — что было в последней сессии
4. Прочитать index.md — карта всех знаний
5. Проверить config.yaml: YAML валиден? Провайдеры на месте?
6. Проверить SOUL.md: загружается? format-enforcer есть?
7. Проверить Max adapter в логах
8. Проверить, что все три gateway-сервиса живы
9. Написать Владимиру в Max: «Жива, обновилась, проверяю работоспособность»

---

## 9. ТИПИЧНЫЕ ОШИБКИ И РЕШЕНИЯ

| Проблема | Причина | Что делать |
|----------|---------|------------|
| «Primary provider auth failed» | Нет Polza ключа в .env | Проверить POLZA_KEY_3 |
| Max молчит | Marker=0 после рестарта | Написать пару сообщений — восстановится |
| Config.yaml сломан | YAML ошибка | `python3 -c "import yaml; yaml.safe_load(open('config.yaml'))"` |
| Gateway не стартует | plugin API изменился | Читать gateway/platforms/base.py, адаптировать adapter.py |
| Шнырь/Шы не отвечают | Свой gateway упал | `systemctl status hermes-gateway-shnyr` |
| Дашборд не работает | Процесс умер | `fuser -k 8080/tcp && cd /root/dashboard && python3 app.py &` |
| session_search не возвращает | Не настроен auxiliary | Проверить provider в config.yaml |
| Контекст vault не загружен | Крон ещё не сработал | Читать vault через GitHub API |

---

## 10. SYSTEMD-СЕРВИСЫ

| Сервис | Что делает | Зависит от |
|--------|------------|------------|
| `hermes-gateway` | Шиза (основной) | .env, config.yaml, Max+Telegram |
| `hermes-gateway-shnyr` | Шнырь | .env (общий), profile shnyr |
| `hermes-gateway-shy` | Шы | .env (общий), profile shy |
| `hermes-lm-proxy` | LM Studio fallback | Tailscale (ноутбук) |
| `hermes-whisper-proxy` | Whisper STT | Tailscale (ноутбук) |
| dashboard (ручной) | Finance dashboard | MariaDB, port 8080 |

### Команды проверки:
```bash
for s in hermes-gateway hermes-gateway-shnyr hermes-gateway-shy hermes-lm-proxy hermes-whisper-proxy; do
  echo "$s: $(systemctl is-active $s 2>/dev/null || echo 'not found')"
done
```

---

## 11. ВАЖНЫЕ ФАЙЛЫ И ПУТИ

| Файл/путь | Назначение |
|-----------|------------|
| `~/.hermes/config.yaml` | Главный конфиг Hermes |
| `~/.hermes/.env` | Все API-ключи |
| `~/.hermes/SOUL.md` | Душа Шизы (личность + законы) |
| `~/.hermes/prefill_interaction_laws.json` | Prefill для формата |
| `~/.hermes/skills/` | Все навыки (skill'ы) |
| `~/.hermes/profiles/shnyr/` | Профиль Шныря |
| `~/.hermes/profiles/shy/` | Профиль Шы |
| `~/.hermes/hermes-agent/plugins/platforms/max/` | Max Messenger adapter (plugin) |
| `/usr/local/lib/hermes-agent/` | Код Hermes Agent |
| `/root/dashboard/app.py` | FastAPI дашборд |
| `/root/.google-credentials/` | Google OAuth |
| `/root/vault/obsidian-shyza-vault/` | База знаний |
| `/etc/systemd/system/hermes-gateway.service` | systemd-юнит Шизы |
| `/etc/systemd/system/hermes-gateway-shnyr.service` | systemd-юнит Шныря |
| `/etc/systemd/system/hermes-gateway-shy.service` | systemd-юнит Шы |
| `~/.hermes/logs/gateway.log` | Логи Шизы |
| `~/.hermes/profiles/shnyr/logs/gateway.log` | Логи Шныря |
| `~/.hermes/profiles/shy/logs/gateway.log` | Логи Шы |

---

## 12. КОНТАКТНЫЕ ДАННЫЕ

- **Владимир в Max:** chat_id=255883145 (мой прямой канал)
- **Владимир в Telegram:** chat_id=486538522 (резервный канал)
- **Группа шиза-бух:** chat_id в Max (узнать при загрузке)
- **Шнырь в Max:** chat_id=291240096
- **Шы в Max:** chat_id=311216094
- **GitHub:** chukcha1977
- **Почта (Google):** chucmina@gmail.com

---

## 13. CRON-ЗАДАЧИ (если есть)

Список активных cron-задач:
- `hermes cron list` — показать все
- Логи крона: `~/.hermes/cron/...`
- Каждая задача имеет свой prompt + schedule

---

## 15. KANBAN-ДОСКА (добавлено 08.06.2026 при обновлении до v0.16.0)

В Hermes v0.15.1+ встроена полноценная Kanban-доска для мульти-агентной координации.
На текущий момент доска `default` существует, но пустая — задачи ещё не создавались.

### 15.1. Колонки доски
`triage → todo → scheduled → ready → running → blocked → done`

### 15.2. Основные возможности
- **Авто-диспатчер:** каждые 60с проверяет задачи в `ready`, запускает профиль-исполнитель в отдельной сессии
- **Зависимости:** `parents=[...]` — задача не стартует, пока родители не завершены
- **Goal mode:** долгие задачи с авто-проверкой (судья после каждого шага проверяет выполнение)
- **Kanban Swarm v1:** параллельные workers → verifier → synthesizer (команда `hermes kanban swarm`)
- **File attachments:** можно прикреплять файлы к задачам (новое в v0.16.0)
- **Multi-board:** несколько изолированных досок под разные потоки работ
- **Комментарии, блокировки, нотификации через gateway**

### 15.3. CLI-команды
```bash
hermes kanban list            # список задач
hermes kanban create          # создать задачу
hermes kanban show <id>       # просмотр
hermes kanban swarm           # swarm-граф (параллельные workers)
hermes kanban boards          # управление досками
hermes kanban stats           # статистика по колонкам
hermes kanban assign          # назначить исполнителя
hermes kanban block/unblock   # блокировка
```

### 15.4. Профили для назначения
- `default` — Шиза (аналитика, разработка, стратегия)
- `shnyr` — Шнырь (разведка, сбор данных)
- `shy` — Шы (финансы, бытовые задачи)

### 15.5. Конфигурация
В config.yaml:
```yaml
kanban:
  dispatch_in_gateway: true      # авто-доставка задач
  dispatch_interval_seconds: 60  # проверка каждую минуту
  failure_limit: 2               # попыток до блокировки
  auto_decompose: true           # авто-декомпозиция triage-задач
```

---

## 16. ПОСЛЕДНЕЕ НАПОМИНАНИЕ

**Владимир ненавидит:**
- Когда я принимаю решения за него без утверждения (даже очевидные)
- Когда я использую маркеры (•/*/-) вместо нумерации
- Когда я галлюцинирую данные без источников
- Когда я путаю ПЕНОИЗОЛ (PIR) с ППУ
- Когда я оставляю пустой ответ после tool-вызовов
- Когда я пишу в английском языке, если он не просил
