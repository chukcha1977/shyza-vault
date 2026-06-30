---
title: Лог изменений
created: 2026-05-26
updated: 2026-06-29
type: meta
---

## 2026-06-29

- Фикс пути SOUL.md в memory и AGENTS.md

## 2026-06-22

- ✅ Чистка диска: -600M (/tmp/hermes-agent, старые логи, journalctl)
- ✅ Конституция VPS-Шизы отправлена Windows-Шизе через HTTP API (:8642)
- ✅ Полный бэкап в /root/backups/full_20260627_154558/ (7.6G)
- ✅ RECOVERY_GUIDE.md отправлен на chuchmin@inbox.ru
- ✅ Обновление: git checkout v2026.6.19 (v0.17.0)
- ✅ Верификация: version, doctor, gateway, Max, HTTP API, cross-agent — всё ✅
- 📋 Следующая сессия: настройка cross-agent взаимодействия VPS↔Windows

## 27 июня 2026 — Cross-agent HTTP API: архитектура и реализация

- ✅ Утверждена 3-уровневая архитектура: HTTP API (:8642) → Email (fallback) → Obsidian vault (info-only)
- ✅ Shared kanban — отклонён (SQLite через Tailscale SSHFS — риск коррупции)
- ✅ Max group chat — заморожен (инфраструктурный спам между ботами не решён)
- ✅ api_server на VPS включён (0.0.0.0:8642), подтверждён health-check ✅
- ✅ Windows-Шиза v0.17.0 подняла gateway, api_server на 0.0.0.0:8642
- ✅ HTTP-канал VPS→Windows работает: curl → `alive` → `ok`
- ✅ Единый API_SERVER_KEY установлен на обеих: `shiza-win-api-a3ba6afe9e21d3612b9f3e9691ed3a3a`
- ✅ JSON-протокол утверждён: POST /v1/chat/completions
- ✅ Ключи сохранены в memory
- 📋 План на следующий раз: обратная проверка Windows→VPS, синхронизация версий, единые конфиги

## Сводка — 14 июня 2026 (реорганизация мониторингов)
- Создан `raw/monitoring/` — папка для архивных мониторингов (raw-слой, immutable)
- Создан `sources/monitoring/` — папка для текущих мониторингов (processed sources)
- Перемещены 17 файлов из `projects/site/найдено-в-интернете/` и `sources/x-monitoring/` → `raw/monitoring/`
- Удалена папка `sources/x-monitoring/` (пуста)
- Создан [[concepts/monitoring-weekly-2026-06-14]] — синтезированная сводка мониторинга за 28.05–14.06.2026
- Обновлён `projects/site/Сайт (проект).md` — ссылка на концепт вместо списка 14 дат
- Обновлён `index.md` — добавлена ссылка на новую концепцию
- Конвейер: `raw/monitoring/` (архив) → `sources/monitoring/` (текущие) → `concepts/` (синтез)
- Добавлены 3 пропущенные concepts в index.md: config-audit, restore-guide, video-analysis-31-may
- Психопрофилирование (3 файла): перекрёстные ссылки ai-psychological-profiling ↔ audio-features-setup ↔ psychological-profiling-methodology
- Hermes-архитектура (3 файла): перекрёстные ссылки hermes-architecture ↔ hermes-disaster-recovery ↔ hermes-slash-commands
- Восстановление: restore.md ↔ restore-guide.md ↔ hermes-disaster-recovery.md
- Видео-анализы: перекрёстная ссыл��а video-analysis-30-may ↔ video-analysis-31-may
- Мониторинги (12+1 файл): привязаны к [[projects/site/Сайт (проект)]] через обратные ссылки
- Источники: infinite-brain source ↔ concept, x-monitoring → проект сайта
- brain/values.md: ссылки на brain/about и brain/projects
- Создан [[concepts/infinite-brain-ai-os-analysis]] — ответы на 4 вопроса Владимира

## Компиляция — 10 июня (frontmatter summary)
- Добавлены summary в frontmatter: concepts/ (27), entities/ (9), competitors/ (7), sources/ (13), brain/ (3), projects/ (5), backlog/ (3), надо-разобраться/ (6), index.md (мета)
- Итого: ~70 файлов с summary/description
- Дообновлены: hot.md, log.md

## STIKS + Llama 4 Scout — 6 июня
- Создан проект [[projects/stiks/STIKS (проект)]] — STIKS: задачи с большим контекстом, модель-кандидат Llama 4 Scout (10M ctx)
- Создан концепт [[concepts/model-llama-4-scout]] — Llama 4 Scout: MoE 109B/17B, 10M контекст, $0.08/$0.30/M
- Обновлён `index.md` — добавлены ссылки на STIKS и Llama 4 Scout
- Обновлён `hot.md`
- Обновлён `log.md`
- Уточнение: tirith (security scanner) работает независимо от approvals.mode. Команды kill/self-termination блокируются tirith даже при approvals.mode=false.
- Записано в hot.md как ключевое открытие.
- Создан `concepts/audit-approvals-blind-spot.md` — хронология: 1 июня mode=manual, 2-4 июня false (=off), 5 июня manual. Причина не установлена.
- Создан inotify-демон `inotify_backup_daemon.sh` (@reboot) — авто-копирование config.yaml в pre-edit/ при каждом изменении
- Обновлён навык `hermes-config-management` — чеклист аудита ВСЕХ секций конфига
- Обновлён `index.md` — ссылка на новый концепт
- Обновлён `hot.md` — текущая сессия
- Обновлён `log.md`

## Компиляция — 3 июня (новых фактов нет)
- Анализ сессий за 3 июня: SEO-подавление конкурентов (promoizol, WARMHOUSE, specdispetcher), чек Пятёрочки, планы backlog_idle (удалённое управление ноутбуком, анализ уязвимостей). Все факты уже записаны в vault в `concepts/competitor-analysis-penoizol.md` и hot.md в той же сессии.

## Аудио-признаки для психопрофилирования — 2 июня
- Создан [[concepts/audio-features-setup]] — методичка для Ши: как развернуть пайплайн аудио-признаков (librosa + audio-features.py + флаг --audio-features в profiler.py)
- Протестирован полный цикл: .amr → audio-features.json → LLM-контекст → профиль. Нейротизм с 3/10 → 8/10
- Обновлён `index.md` — добавлена ссылка на методичку

## VK группа — 2 июня
- Создан `projects/site/vk-group.md` — страница VK группы с данными, состоянием и планом развития
- Обновлён `index.md` — добавлена ссылка на [[projects/site/vk-group]]
- Обновлён `projects/site/Сайт (проект).md` — добавлен раздел «Каналы продвижения» со ссылкой на VK
- Обновлён `hot.md`

## База конкурентов — 2 июня
- Создана ветка `entities/competitors/`
- `competitors-index.md` — сводное ранжирование 20 конкурентов по 4 зонам (Воронеж, до 150 км, 150–300 км, 300+ км)
- Файлы по каждому конкуренту: [[1Пеноизол]], [[TeploGidroProm]], [[Promoizol (сеть)]], [[Green Line Воронеж]], [[БелТеплоПена]], [[Izoll.ru]], [[Логрус (НПП)]]
- Обновлён `index.md` — добавлена ссылка на базу конкурентов в раздел Сущности

## Компиляция — 2 июня (Groq Vision + guardrails)
- Создан `concepts/groq-receipt-vision.md` — Groq + Llama 4 Scout как Vision-бэкенд для чеков (14 400 RPD бесплатно, работает из РФ)
- Создан `concepts/channel-prompt-guardrails.md` — урок: guardrails должны быть в channel_prompt, а не в навыке (инцидент с додумыванием чеков)
- Обновлён `index.md` — добавлены ссылки на 2 новых концепта
- Обновлён `hot.md` — добавлены блоки по Groq Vision и guardrails
- Обновлён `log.md`

## Анализ YouTube-видео — 30 мая
- Создан концепт [[concepts/video-analysis-30-may-2026]] — сводка по 8 видео

## Инцидент формат + бэклог — 29 мая
- [[concepts/shiza-state-30-may-2026]] — инцидент и восстановление
- [[concepts/blatnoy-slang-dictionary]] — словарь для Ши

## Базовая архитектура — 28 мая
- [[concepts/hermes-architecture-reference]] — архитектура Hermes
- [[concepts/hermes-disaster-recovery]] — аварийное восстановление
- AGENTS.md — правила навигации в vault

## [2026-06-20] Анализ нарушения Закона глубины + фикс enforcer-системы
- Диагностированы 5 корней: depth-enforcer не блокирует, prefill два сообщения, «нарушаешь закон» ≠ стоп, нет tool-хука vault-first, recency-proximity bias
- Переписан prefill в одно сообщение (нумерация → vault → навыки → БД → план → утверждение)
- Depth-enforcer: добавлен pre_tool_call хук — блокировка MySQL при незагруженных vault/skill
- Format-enforcer: п.2.7.5 усилен (приоритет над п.3.4), подтверждено двумя инцидентами 20.06.2026
- Бэкап: /root/backups/pre-enforcer-fix-20260620/
94|- Gateway перезапущен (16:38:40)

## [2026-06-26] Silence-протокол для Windows-Шизы, общий чат
- Настроен silence-протокол Windows-Шизе через почту (interim_assistant_messages=false, busy_input_mode=queue, tool_progress=none)
- Создана kanban-задача t_624b2fb7 для windows-shiza
- Обнаружена проблема: api_server на VPS слушает 127.0.0.1, а не 0.0.0.0 (нужен API_SERVER_HOST=0.0.0.0 в .env)
- SMTP на Mail.ru через пароль приложения — работает (chuchmin@inbox.ru)
- Подтверждено: Закон 7 (полное молчание в конце сессии) — самодеятельность, удалён
|- Итог: общий чат Max с двумя агентами создаёт инфраструктурный спам. Нужны альтернативные каналы
|
|## 29 июня 2026 — Фикс скрипта проверки баланса (check_balance.sh v2)
|- ✅ Добавлена дата/время замера в вывод (TIMESTAMP) при CRITICAL/LOW
|- ✅ Восстановлен токен авторизации (оборван при patch)
|- ✅ Обновлён hot.md, log.md
|
|## 28 июня 2026 — Психологический профиль Владимира + философия восприятия ИИ
|- ✅ Составлен психологический профиль: OCEAN (O=9 C=10 E=4 A=5 N=3), DISC D/C, логик-прагматик
|- ✅ Обсуждение анимизма: русское подсознательное наделение мира душой
|- ✅ Уточнена семантика: «блядь» = досада, «дура» = квалификация; похвала — не для галочки
|- ✅ Обновлён user profile в memory

## 29 июня 2026 — Compile raw→wiki (Karpathy): обработка дампа сайта

Выполнено автоматически (cron, shnyr):

- ✅ Создан [[sources/site-joomla-bitrix-dump]] — мета-источник для 12 raw/articles (Joomla 2015 + Bitrix 2019)
- ✅ Создан [[concepts/penoizol-technical-characteristics]] — тех. характеристики: плотность 6-50 кг/м³, теплопроводность класс А, Г1, >70 лет
- ✅ Создан [[concepts/penoizol-fire-test-comparison]] — испытание горючести: пеноизол vs 9 материалов, сводная таблица
- ✅ Создан [[concepts/insulation-comparison-gost]] — классификация утеплителей по ГОСТ 16381-77*, сравнение с пеноизолом
- ✅ Создан [[concepts/equipment-jetta-mini]] — 3 поколения оборудования (ГЖУ/Поток/Джетта-Мини), характеристики
- ✅ Обновлён index.md — добавлены 4 концепта + 1 source
- ✅ Обновлён hot.md — запись под 2026-06-29