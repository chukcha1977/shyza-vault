---
title: Правила для агента
type: meta
created: 2026-05-28
updated: 2026-06-30
---

# AGENTS.md — правила навигации

> Читай этот файл ПЕРВЫМ при старте любой сессии. Он дешевле SCHEMA.md.

## Единый порядок загрузки (SessionStart)

1. Прочитать AGENTS.md (этот файл) — 1-2 сек
2. Прочитать hot.md — что было в прошлый раз
3. Прочитать index.md — где что лежит
4. Только если нужно углубиться — читать конкретные статьи из index
5. shared/brain/about.md и shared/brain/values.md — читать только если ты не знаешь, кто Владимир

## Экономия токенов

- НЕ читать все файлы подряд — навигация через index.md
- НЕ лезть в shared/raw/ без явной необходимости
- hot.md — твой горячий кэш, он компактнее index
- Если в hot.md есть ответ на вопрос — не перечитывай концепты
- session_search дешевле, чем перечитывать старые файлы vault

## Структура vault

```
shared/             # ОБЩЕЕ ДЛЯ ВСЕХ АГЕНТОВ
  brain/            — стратегический контекст (КТО Владимир, проекты, ценности)
  entities/         — люди и компании
  concepts/         — знания (КАК устроено)
  sources/          — внешние источники с выводами
  raw/              — immutable, не трогать
  _templates/       — шаблоны Obsidian Templater

agents/             # ЛИЧНЫЕ ПАПКИ АГЕНТОВ
  shiza/            — VPS-Шиза (дашборд, заказы, Hermes-архитектура)
    projects/       — проекты (ЧТО делаем)
    backlog/        — задачи и приоритеты
    plans/          — планы
    queries/        — SQL-запросы
    надо-разобраться/ — темы для изучения
  windows-shiza/    — Windows-Шиза (STIKS, ComfyUI, Windows)
  shnyr/            — Шнырь (разведка)
  shy/              — Шы (оперативный помощник)
  shiva/            — Шива (помощница Windows-Шизы)

AGENTS.md           — этот файл (правила навигации)
SCHEMA.md           — ДНК системы (структура, workflow, шаблоны)
index.md            — карта зн��ний (все статьи с ссылками)
hot.md              — горячий кэш (последняя сессия)
log.md              — лог изменений
```

## Когда что использовать

| Ситуация | Что читать |
|----------|-----------|
| Старт сессии | hot.md → index.md |
| Вопрос по проекту | index → agents/shiza/projects/ |
| Технический вопрос | index → shared/concepts/ |
| «Кто это?» | index → shared/entities/ |
| Нужна цифра из БД | SQL в chuchm_orders |
| Повторный вопрос | hot.md, не перечитывать концепты |
| Работа в шиза-бух | shared/concepts/costing-methodology.md |
| Работа в семейный учет | shared/concepts/family-accounting-rules.md |

## Кто что пишет

| Агент | Пишет в |
|-------|---------|
| VPS-Шиза (я) | agents/shiza/ |
| Windows-Шиза | agents/windows-shiza/ |
| Шнырь | agents/shnyr/ |
| Шы | agents/shy/ |
| Шива | agents/shiva/ |
| Все (с разрешения Владимира) | shared/ |

## Обновление знаний

- Новые факты → shared/concepts/ с YAML frontmatter и викиссылками
- Новый проект → agents/shiza/projects/ с папкой проекта
- После каждой сессии: обновить hot.md
- После новых записей: обновить index.md + log.md
- Не дублировать: перед записью проверить, что нет в vault
- Не копировать данные из MariaDB — в vault только анализ
- **Обязательно использовать шаблоны из shared/_templates/** при создании новых страниц (concept, entity, source, project). Заполнять все поля frontmatter. Не создавать страницы без шаблона.

## Питфоллы

- Не перечитывать SCHEMA.md каждый раз — он большой
- hot.md не должен превышать 200 строк
- shared/raw/ — immutable, не редактировать
- Если не уверен в источнике — укажи в YAML
- SOUL.md лежит в `/root/.hermes/SOUL.md`, а НЕ в `/root/SOUL.md` — старый путь не существует
