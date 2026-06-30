---
title: Миграция с немецкого VPS на Beget
type: plan
status: draft
created: 2026-05-30
author: Шиза
tags: [migration, vps, beget, infra]
---

# Миграция с немецкого VPS на Beget VPS

## 1. Контекст и предпосылки

### 1.1. Текущая архитектура
- **Немецкий VPS** (as210546.net, 2.27.1.148): 1 vCPU, 1.9GB RAM, 30GB NVMe (91% full), Ubuntu 24.04
  - Swap полностью занят (511/511 MB) — система на пределе
  - Свободно 2.7GB диска — критический минимум
- **Beget (87.236.16.9)**: хостинг сайта penoizol-chernozem.ru (Bitrix), базы MySQL 5.7
  - Доступ: SSH chuchm_shyza, панель cp.beget.com (антибот)

### 1.2. Что работает на немецком VPS
1. **Hermes Agent** (главный шлюз) — основа, к ней привязаны Max боты
2. **Hermes Gateway — Шнырь** (профиль shnyr, профиль разведчика)
3. **Hermes Gateway — Шы** (профиль shy, профиль зам. Шизы)
4. **MariaDB** (chuchm_finance + chuchm_orders, ~126 MB)
5. **Чучм-дашборд** (FastAPI на порту 8080, /root/dashboard/)
6. **Nginx** (chat.penoizol-chernozem.ru → localhost:8080, SSL от Let's Encrypt)
7. **X-UI / Xray** (VLESS+Reality, порты 555, 2096, 2053 — обход блокировок)
8. **Tailscale** (связь с ноутбуком Владимира, IP: 100.84.167.53)
9. **LM Studio Proxy** (прокси к ноутбуку через Tailscale, порт 1337)
10. **Whisper Proxy** (fallback для распознавания речи, порт 1338)
11. **Cron-задачи** (idle-workflow, ежедневная сводка)
12. **Obsidian vault** (/root/vault/)
13. **Google OAuth токены** (/root/.google-credentials/)

### 1.3. Проблемы текущего VPS (причины миграции)
1. Диск 91% — каждый бэкап последний
2. Swap забит — производительность страдает
3. Немецкая локация — для работы с российскими сервисами (Max, Beget) лишняя задержка
4. Нет возможности расширения (30GB max на тарифе)
5. Стоимость неясна — возможно, дороже Beget VPS

---

## 2. Целевая архитектура

### 2.1. Beget VPS — рекомендуемая конфигурация
- **Дата-центр**: Россия, Санкт-Петербург (ru1)
- **Тариф**: 2 ядра / 4 GB RAM / 40 GB NVMe (33₽/день ≈ 1000₽/мес)
  - *Минимально допустимый*: 2 ядра / 2 GB / 30 GB (27₽/день ≈ 810₽/мес)
- **ОС**: Ubuntu 24.04 (есть в готовых образах Beget)
- **Сеть**: 1 Гбит/с, публичный IPv4
- **Особенности**: авто-бэкапы, DNS-хостинг, приватные сети

### 2.2. Распределение сервисов после миграции

| Компонент | Куда | Примечание |
|-----------|------|------------|
| Hermes Agent (все 3 профиля) | Beget VPS | Первичный сервис |
| MariaDB (chuchm_finance, chuchm_orders) | Beget VPS | Отдельный инстанс |
| Чучм-дашборд | Beget VPS | FastAPI на :8080 |
| Nginx + SSL | Beget VPS | chat.penoizol-chernozem.ru |
| Tailscale | Beget VPS | Для связи с ноутбуком |
| LM Studio Proxy | Beget VPS | На 127.0.0.1:1337 |
| Whisper Proxy | Beget VPS | На 127.0.0.1:1338 |
| Cron задачи | Beget VPS | Скопировать jobs.json |
| Google OAuth | Beget VPS | Скопировать папку |
| Obsidian vault | Beget VPS | Git-репозиторий |
| X-UI / Xray | ❌ НЕ ПЕРЕНОСИТЬ | На российском VPS не нужен |
| Сайт penoizol-chernozem.ru | Остаётся на Beget-хостинге | Ничего не меняется |

### 2.3. Что НЕ переносится
1. **X-UI (Xray/VPN)** — инструмент обхода блокировок. На российском VPS не имеет смысла, может привлечь внимание.
2. **Старый немецкий VPS** — после миграции остановить и отключить.
3. **Joomla-сайт** — уже не обслуживается.

---

## 3. Пошаговый план миграции

### Фаза 0: Подготовка (День 1)

#### Шаг 0.1: Создание VPS на Beget
1. Вход в панель Beget (вручную, антибот)
2. Создание VPS: Россия, 2/4/40 GB, Ubuntu 24.04
3. Получение root-доступа (IP, пароль/SSH-ключ)
4. Базовая настройка: часовой пояс, hostname, fail2ban, ufw
5. Добавление SSH-ключа Владимира и Шизы (ключ с немецкого VPS)

#### Шаг 0.2: Инвентаризация зависимостей
1. Составить точный список Python-пакетов Hermes (`pip list`)
2. Зафиксировать версии: Node.js, Python, MariaDB, Nginx
3. Скачать актуальную версию Hermes-agent с GitHub (или перенести через Git)
4. Проверить все скрытые зависимости (симлинки, переменные среды)

#### Шаг 0.3: Подготовка бэкапа на немецком VPS
1. Полный дамп обеих баз: `mysqldump --all-databases --routines --triggers > /root/full_dump.sql`
2. Архивация конфигов: `tar -czf /root/hermes-configs.tar.gz /root/.hermes/config.yaml /root/.hermes/profiles/ /root/.hermes/prefill_*.json /root/.hermes/gateway.yaml /usr/local/lib/hermes-agent/.env /etc/nginx/sites-available/ /etc/systemd/system/hermes-*.service /etc/systemd/system/chuchm-dashboard.service`
3. Копирование vault: `cd /root/vault && git push` (если есть remote) или архив
4. Копирование Google OAuth: tar -czf /root/google-creds.tar.gz /root/.google-credentials/

### Фаза 1: Развёртывание инфраструктуры на Beget VPS

#### Шаг 1.1: Базовая настройка VPS
```bash
apt update && apt upgrade -y
apt install -y nginx python3 python3-venv python3-pip mariadb-server mariadb-client \
  git curl wget ufw fail2ban certbot python3-certbot-nginx nodejs npm
# Если нужна конкретная версия Node.js
```

#### Шаг 1.2: MariaDB
1. Установка и настройка MariaDB 10.11+ (как на немце)
2. Создание пользователей admin/finance_user/orders_user с теми же паролями
3. Импорт дампа баз
4. Настройка удалённого доступа (если нужно для дашборда)
5. Верификация: `mysql -e "SELECT * FROM chuchm_orders.orders LIMIT 5;"`

#### Шаг 1.3: Hermes Agent (ядро)
1. Клонирование репозитория Hermes:
   ```bash
   git clone https://github.com/nousresearch/hermes-agent /usr/local/lib/hermes-agent
   ```
2. Установка зависимостей (uv sync или pip install -e .)
3. Создание виртуального окружения
4. Копирование `.env` с POLZA_KEY_1/2/3 и другими секретами
5. Копирование config.yaml + профили shy/shnyr
6. Создание prefill_messages.json (законы взаимодействия)
7. Настройка systemd-юнитов (hermes-gateway, hermes-gateway-shnyr, hermes-gateway-shy)
8. Запуск gateway, проверка логов

#### Шаг 1.4: Дашборд
1. Копирование /root/dashboard/ на Beget VPS
2. Установка зависимостей (pip install -r requirements.txt)
3. Настройка подключения к MariaDB (config.py)
4. Создание systemd-юнита chuchm-dashboard
5. Запуск на порту 8080, проверка

#### Шаг 1.5: Nginx + SSL
1. Копирование конфига chat (nginx sites-available)
2. Перевыпуск SSL-сертификата:
   ```bash
   certbot certonly --nginx -d chat.penoizol-chernozem.ru
   ```
   *Примечание: DNS домена должен уже указывать на IP нового VPS, иначе certbot не сможет подтвердить владение.*
3. Настройка прокси на localhost:8080
4. Добавление в sites-enabled
5. `nginx -t && systemctl reload nginx`

#### Шаг 1.6: Tailscale
1. Установка Tailscale
2. Подключение к тому же аккаунту (chucmina@)
3. Настройка разрешения на отправку через Tailscale
4. Проверка связи с ноутбуком (100.100.142.77)

#### Шаг 1.7: Прокси сервисы
1. Копирование hermes-lm-proxy.py и hermes-whisper-proxy.py
2. Создание systemd-юнитов
3. Настройка зависимости от tailscaled.service
4. Запуск, проверка

#### Шаг 1.8: Cron-задачи
1. Копирование jobs.json из /root/.hermes/cron/
2. Копирование скриптов (daily_summary.py, send_summary.sh)
3. Перезапуск cron-воркера Hermes
4. Проверка статуса джобов

#### Шаг 1.9: Google OAuth
1. Копирование /root/.google-credentials/
2. Проверка работы (Gmail/Calendar/Sheets API)

#### Шаг 1.10: Obsidian vault
1. Клонирование Git-репозитория vault (или копирование архива)
2. Настройка пути /root/vault/

### Фаза 2: Переключение с немецкого VPS на Beget

#### Шаг 2.1: Подготовка к переключению (параллельная работа)
1. Все сервисы на новом VPS запущены, но gateway **НЕ подключены к Max**
2. Проверка дашборда через локальный curl (http://127.0.0.1:8080)
3. Проверка MariaDB: данные на месте, связи работают
4. Проверка Tailscale: ноутбук видит новый VPS

#### Шаг 2.2: DNS-переключение
1. Изменить DNS-запись chat.penoizol-chernozem.ru на IP нового VPS
2. Подождать TTL (обычно 5-30 минут, но может быть до 24ч)
3. Проверить, что curl https://chat.penoizol-chernozem.ru отвечает с нового VPS

**ВАЖНО**: DNS меняется ДО запуска gateway на новом VPS, чтобы:
- certbot смог подтвердить домен
- Даунтайм чата был минимальным
- Сервер на немце продолжал работать до переключения

#### Шаг 2.3: Запуск gateway на новом VPS
1. Перевыпуск SSL (certbot)
2. Запуск всех трёх gateway
3. Проверка: gateway подключились к Max, боты отвечают

#### Шаг 2.4: Остановка старого VPS
1. Остановить gateway на немецком VPS (systemctl stop hermes-gateway*)
2. Остановить дашборд
3. Остановить все сервисы
4. Сделать финальный бэкап (на всякий случай)
5. **ОСТАНОВИТЬ VPS**, но не удалять (7 дней на возврат)

### Фаза 3: Пост-миграция

#### Шаг 3.1: Мониторинг (первые 24 часа)
1. Gateway стабильно работают? Max не жалуется?
2. Дашборд отвечает?
3. Сводка cron отработала?
4. Ноутбук Владимира всё ещё видит VPS через Tailscale?

#### Шаг 3.2: Финальные действия
1. Если всё стабильно — удалить немецкий VPS
2. Записать IP нового VPS в Obsidian (entities/Beget.md)
3. Обновить бэкап-скрипты (пути, IP)
4. Настроить автоматические бэкапы на Beget VPS
5. Обновить BACKLOG.md и dashboard

---

## 4. Оценка времени простоя

| Этап | Ожидаемый downtime | Комментарий |
|------|-------------------|-------------|
| DNS-переключение | 5-30 мин (обычно) | TTL задать минимальный 60 сек перед миграцией |
| Генерация SSL | 0 (parallel) | Выполняется после DNS |
| Перезапуск gateway | 1-2 мин | Max переподключается |
| **Итого** | **~10-40 мин** | |

---

## 5. Риски и контриеры

### 5.1. Max Messenger отвалится при смене IP
- **Риск**: HIGH. Gateway держит длительное соединение с Max. При переезде на новый IP сессия разорвётся.
- **Контрмера**: 
  - На немецком VPS gateway должен быть ОСТАНОВЛЕН, прежде чем запускать на новом (чтобы Max не путал два соединения)
  - Проверить, нужна ли перерегистрация webhook/бота
  - Max может требовать привязки IP бота — надо уточнить в документации Max Messenger

### 5.2. Tailscale из России
- **Риск**: MEDIUM. DERP-сервера Tailscale могут быть недоступны из РФ
- **Контрмера**:
  - Tailscale работает через DERP-relay; у Tailscale есть DERP в Москве (?)
  - Если не работает — настроить прямой DERP-сервер (на самом VPS)
  - Альтернатива: WireGuard напрямую к ноутбуку

### 5.3. Beget блокирует порты (например, 25 для SMTP, 443 может быть занят)
- **Риск**: MEDIUM
- **Контрмера**: Проверить открытые порты на Beget VPS тестовым listen

### 5.4. Certbot не сможет подтвердить домен до переключения DNS
- **Риск**: MEDIUM
- **Контрмера**: 
  - Сначала DNS → ждём → потом certbot → потом gateway
  - Или использовать DNS-01 challenge (certbot --manual --preferred-challenges dns) — но это сложнее

### 5.5. Пароли и секреты потеряются при копировании .env
- **Риск**: HIGH
- **Контрмера**: 
  - Тщательно сверить .env на обоих серверах
  - POLZA_KEY_1/2/3 — проверить, что работают
  - Пароли БД (admin, finance_user, orders_user) — зафиксировать

### 5.6. Cron-задачи сбиваются
- **Риск**: LOW
- **Контрмера**: jobs.json копируется как есть, cron работает через Hermes (не системный crond)

---

## 6. Точный порядок выполнения (execution checklist)

### PRE (на немецком VPS, за 1 день до миграции):
- [ ] 6.1. Дамп БД (mysqldump --all-databases --routines --triggers > /root/pre_migration_dump.sql)
- [ ] 6.2. Архив конфигов: /root/.hermes/, /etc/nginx/, /etc/systemd/system/hermes-*, /etc/systemd/system/chuchm-*
- [ ] 6.3. Архив /root/dashboard/
- [ ] 6.4. Архив /root/.google-credentials/
- [ ] 6.5. Архив /root/hermes-lm-proxy.py, /root/hermes-whisper-proxy.py
- [ ] 6.6. git push Obsidian vault (или архив)
- [ ] 6.7. pip freeze > /root/requirements.txt
- [ ] 6.8. Зафиксировать версии: python3, node, nginx, mariadb
- [ ] 6.9. Скопировать всё на временное хранилище или через scp на beget-хостинг

### STEP 1 (на Beget VPS, утро дня миграции):
- [ ] 6.10. Установка ОС, базовая безопасность, SSH-ключи
- [ ] 6.11. Установка пакетов: nginx, mariadb, python, node, certbot, tailscale
- [ ] 6.12. MariaDB: импорт дампа, создание пользователей
- [ ] 6.13. Hermes: клонирование, настройка venv, копирование .env + configs
- [ ] 6.14. Дашборд: копирование, pip install, настройка
- [ ] 6.15. Прокси: копирование, systemd-юниты
- [ ] 6.16. Obsidian vault: git clone
- [ ] 6.17. Cron: копирование jobs.json + скриптов
- [ ] 6.18. Tailscale: установка и подключение
- [ ] 6.19. Google OAuth: копирование папки
- [ ] 6.20. ПРОВЕРКА: все сервисы работают, дашборд отвечает на 127.0.0.1:8080

### STEP 2 (переключение):
- [ ] 6.21. Установить TTL DNS в 60 сек (на Beget DNS-панели)
- [ ] 6.22. Поменять A-запись chat.penoizol-chernozem.ru на новый IP
- [ ] 6.23. Дождаться распространения DNS (dig chat.penoizol-chernozem.ru)
- [ ] 6.24. Перевыпуск SSL: certbot --nginx -d chat.penoizol-chernozem.ru
- [ ] 6.25. ПРОВЕРКА: https://chat.penoizol-chernozem.ru отвечает
- [ ] 6.26. **ОСТАНОВИТЬ** gateway на немецком VPS:
  ```bash
  systemctl stop hermes-gateway
  systemctl stop hermes-gateway-shnyr
  systemctl stop hermes-gateway-shy
  systemctl stop chuchm-dashboard
  ```
- [ ] 6.27. **ЗАПУСТИТЬ** gateway на Beget VPS:
  ```bash
  systemctl start hermes-gateway
  systemctl start hermes-gateway-shnyr
  systemctl start hermes-gateway-shy
  systemctl start chuchm-dashboard
  ```
- [ ] 6.28. ПРОВЕРКА: gateway подключились к Max, боты отвечают
- [ ] 6.29. ПРОВЕРКА: дашборд отвечает на chat.penoizol-chernozem.ru
- [ ] 6.30. ПРОВЕРКА: cron idle-workflow сработал

### STEP 3 (пост-миграция):
- [ ] 6.31. Остановка остальных сервисов на немецком VPS
- [ ] 6.32. Финальный бэкап немецкого VPS
- [ ] 6.33. Наблюдение 24 часа
- [ ] 6.34. Обновление Obsidian: entities/Beget.md, лог
- [ ] 6.35. Удаление немецкого VPS (через 7 дней)

---

## 7. Бюджет

| Статья | Стоимость |
|--------|-----------|
| Beget VPS (2/4/40 GB) | 33₽/день (~1000₽/мес) |
| Немецкий VPS (до остановки) | ? (~7-10€/мес — экономия) |
| Beget-хостинг (сайт) | уже оплачен |
| **Итого дополнительно** | **~1000₽/мес** |

---

## 8. Критические вопросы до миграции

1. **Max Messenger**: Как gateway аутентифицируется? Нужна ли привязка IP? Что будет при перезапуске на новом IP?
2. **Beget VPS**: Какие порты открыты по умолчанию? Нужен ли отдельный SSH-ключ?
3. **DNS**: Где управляется DNS penoizol-chernozem.ru? На Beget DNS или у регистратора?
4. **Tailscale**: Работает ли из России в принципе? Нужны ли дополнительные DERP-сервера?
5. **X-UI**: Нужно ли сохранить конфигурацию Xray на будущее (для других целей)?