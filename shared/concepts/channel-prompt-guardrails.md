---
type: concept
title: Guardrails в channel_prompt — урок по додумыванию чеков
summary: Урок 03.06.2026 — guardrails для чеков нужно прописывать в channel_prompt Max, а не в навыке family-expenses-receipts. Анализ ошибки додумывания данных.
complexity: beginner
domain: hermes-agent
aliases:
  - "channel_prompt guardrails"
  - "channel_prompt vs skill"
  - "правила в channel_prompt"
created: 2026-06-02
updated: 2026-06-02
tags:
  - concept
  - hermes
  - channel_prompt
  - receipts
  - lesson
status: seed
related:
  - "concepts/family-accounting-rules"
  - "concepts/groq-receipt-vision"
  - "concepts/hermes-architecture-reference"
sources:
  - session: 20260602_180302_4315fb08
---

# Guardrails в channel_prompt — урок по додумыванию чеков

## Инцидент (2 июня 2026)

В группе «Семейный учет» агент (Шиза) начала додумывать содержимое чеков — грубое нарушение правила «только факты, не додумывать». Владимир заметил и указал на проблему.

## Root cause

- **Критические правила** («не додумывать содержимое чека», «Tesseract fallback», «контроль количества чеков») были прописаны **только в навыке** `family-expenses-receipts` (SKILL.md)
- **Channel_prompt** для группы содержал только краткую инструкцию: «распознай → покажи → спроси 'Записать?'»
- **Навык не загружается автоматически** — его нужно явно вызывать через `skill_view()`
- Агент работал по channel_prompt, а в нём не было запрета на додумывание

## Решение (исправлено Владимиром)

В channel_prompt для чата `-74772047542372` добавлены явные правила:

1. **«При получении фото чека — ПЕРВЫМ ДЕЙСТВИЕМ загрузи навык family-expenses-receipts через skill_view»**
2. **«НЕ ДОДУМЫВАТЬ содержимое чека. Только то, что vision реально распознала.»**
3. **«Количество чеков на фото = количество чеков, которые ты показываешь.»**

## Уроки

### ⚠️ Критическое: guardrails должны быть в channel_prompt, а не в навыке
- Навык — это **инструмент**, который агент может загрузить или не загрузить
- Channel_prompt — это **конституция**, которая работает всегда
- Правила безопасности, запреты, обязательные процедуры — только в channel_prompt

### 📌 Второстепенное: channel_prompt должен ссылаться на навык
- Даже если все правила в навыке — channel_prompt должен содержать инструкцию: «перед ответом загрузи skill_view(name='...')»

### 📌 Третье: channel_prompt обновлять при изменении навыка
- Навык эволюционирует, channel_prompt — нет. После изменений навыка пров��рять, не нужно ли дополнить channel_prompt.

## Статус
🟢 Исправлено. Все три правила внедрены в channel_prompt.