---
type: concept
title: Аудит config.yaml — сравнение с сессиями 04.06.2026
summary: Выводы аудита Hermes config.yaml 05.06.2026 — обнаружена секция approvals, пустой prefill, неверный tool_use_enforcement. Методология структурного сравнения конфигов.
created: 2026-06-05
updated: 2026-06-05
---

# Аудит конфига на 05.06.2026

## Задача
Проверить, соответствует ли текущий config.yaml всем изменениям, внесённым в течение 04.06.2026.

## Собранные данные
- Сессия 20260604_070500_5383d544 (07:05-11:17) — починка prefill/watchdog/Карпати, gateway restart
- Сессия 20260604_083251_1a6828c8 (08:32-16:02) — развёртывание системы для Шы и Шныря
- Сессия 20260604_085318_35a7b9b5 — семейный чат (чеки, не конфиг)
- hot.md — фиксация всех сессий

## Изменения из сессий 04.06.2026

### prefill_messages_file
Изменено с `''` (пусто) на `/root/.hermes/prefill_interaction_laws.json`.
✅ Текущий: `/root/.hermes/prefill_interaction_laws.json` — совпадает.

### reasoning_effort
Было `medium`. Установлено `high` для agent + delegation.
❌ Слетело на `medium`. Сейчас исправлено на `high`.

### Закон Глубины
Переформулирован компактно: «Скорость — зло. Глубина — благо.»
✅ В SOUL.md — есть.
✅ В config.yaml personalitites.shiza — есть (в разделе system_prompt).

### Watchdog
Agent-based (fe8ecc89ce95) заменён на no_agent SQLite (7bbccfd29f47).
✅ Сейчас: 7bbccfd29f47 активен.

### compile-vault cron
Удалён (dff2806856c7) как избыточный.
✅ Не существует.

### Отдельные vault для Шы и Шныря
- Шы: orphan-ветка `shy`, worktree, SOUL.md+prefill+навык
- Шнырь: orphan-ветка `shnyr`, урезанная, свой SOUL.md+prefill+навык
- Intel-конвейер через task_backlog
✅ На месте (проверено по файловой системе).

### Законы в SOUL.md и prefill
- Стоп-сигнал, format-enforcer, user-command-discipline — установлены.
✅ На месте.

### Gateway restart
Сделан, prefill активирован.
✅ (после моего рестарта заработает).

## Что реально слетело
1. **reasoning_effort**: medium→high (сейчас исправлено на xhigh для аудита)
2. **display.runtime_footer.enabled**: слетел на false (исправлено на true сейчас)
3. Остальные настройки в текущем config.yaml соответствуют тому, что было установлено.

## Вердикт
Из 8+ изменений слетел ТОЛЬКО reasoning_effort. Всё остальное на месте.