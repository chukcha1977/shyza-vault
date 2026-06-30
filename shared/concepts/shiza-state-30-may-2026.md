---
type: concept
title: Состояние Шизы после сессии от 30 мая 2026
summary: Запись инцидента 30.05.2026 — слёт формата ответа, 4 краха (неполный ответ, нарушение нумерации, игнорирование указаний «по пункту N»).
created: 2026-05-31
updated: 2026-05-31
---

# Состояние системы 30 мая 2026

## Инцидент: слёт формата ответа

**Факт:** 30 мая Шиза перестала форматировать ответы нумерованным списком.

**Диагностика:** `approvals.mode` сменился с `manual` на `smart`, добавился `personalities.shiza` в конфигах, `format-enforcer` был расширен пропусками автономных действий.

**Решение:** полный restore конфига из `config.yaml.bak.20260524.v2`, синхронизация profiles/shy и profiles/shnyr.

## Шнырь: 5 критических ошибок обнаружено

1. Урезанный system_prompt
2. Отсутствует навык format-enforcer
3. Пустая persistent memory
4. Минимальный prefill (отсутствует AGENTS.md)
5. Отсутствует LAW OF DEPTH

Все ошибки восстановлены. Gateway перезапущен.

## Шы: SIGTERM timeout

`hermes-gateway-shy.service` упал в `failed (Result: timeout)` — stuck background task не отпустил процесс вовремя. systemd ждал 180s → SIGKILL. Перезапущен через systemctl restart.

## Telegram bot token conflict

Шиза и Шы борятся за один Telegram token (PID 303080). Решено на уровне gateway: каждый профиль должен иметь свой token.

## Дашборд /orders: CSS visual bug on mobile

CSS rules одинаковы в index.html, list.html, style.css (`padding: 2px 1px 2px 1px; line-height: 1.6`). Причина — рендеринг на телефоне или Service Worker. Решение не внедрено.

## Service Worker v15

Существует в `/root/dashboard/static/sw.js`. Предположительно влияет на кэш на мобильных устройствах.

## Google OAuth: refresh token revalidation

Refresh-токен Google (YouTube/Gmail/Drive) ревокеднут. API возвращает 401. Требуется полный OAuth флоу — интерактивная авторизация в браузере.
