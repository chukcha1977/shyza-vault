---
type: concept
title: Google OAuth — проблема scopes при обновлении токена
summary: Проблема invalid_scope при обновлении Google OAuth-токена — новый токен включает только scopes из первого запроса. Решение: явно указывать все scopes при обновлении.
complexity: intermediate
domain: devops
aliases:
  - "invalid_scope"
  - "Google токен"
  - "Drive scope"
created: 2026-05-29
updated: 2026-05-29
tags:
  - concept
  - google
  - oauth
  - token
  - devops
status: seed
related:
  - "concepts/hermes-slash-commands"
sources:
  - session: 20260528_235813_bafbe3e8
---

# Google OAuth — проблема scopes при обновлении токена

## Проблема

При попытке обновления Google OAuth-токена с новым scope (добавление `drive`) возникает ошибка:

```
google.auth.exceptions.RefreshError: ('invalid_scope: Bad Request', {'error': 'invalid_scope', 'error_description': 'Bad Request'})
```

## Причина

Google OAuth-токен «запоминает» набор scopes, с которыми был создан. Если скрипт пытается обновить токен, передавая расширенный список scopes (включая новые, которых не было при создании), Google возвращает `invalid_scope`.

**Нельзя использовать один токен для разных наборов scopes.**

## Проявление

- Токен `/root/.hermes/google_token.json` был создан со scopes для почты/календаря.
- Скрипт `google_api.py` (из навыка google-workspace) содержит `SCOPES` с полным списком, включая `https://www.googleapis.com/auth/drive`.
- При рефреше токен пытается получить Drive-доступ — ошибка.

## Решение

**Создать отдельный токен для Drive** — отдельный скрипт бэкапа со своим файлом `google_token.json`, который изначально содержит Drive scope.

Пример: `/root/.hermes/scripts/backup_to_drive.sh` — создан 28 мая 2026, использует отдельную процедуру аутентификации с Drive scope. При первом запуске проходит OAuth flow заново (через браузер).

## Уроки

1. Google токены привязаны к scopes на момент создания — расширение scopes требует новой аутентификации.
2. Для разных операций (почта, календарь, drive) лучше использовать разные токены.
3. Проверять `SCOPES` в скрипте перед попыткой рефреша.

## Статус

🟡 Скрипт бэкапа создан, но не выполнен до конца (прерван пользователем 28 мая). Требуется перезапуск и проверка.