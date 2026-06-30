---
title: Windows Hermes — установка и архитектура
type: concept
created: 2026-06-22
updated: 2026-06-22
status: active
---

# Windows Hermes: Windows-Шиза

## Что это
На ноутбуке Владимира (Windows, RTX 3070, Tailscale 100.100.142.77) установлен автономный экземпляр Hermes Agent — полная копия Шизы на Windows. Запущен как отдельный профиль `windows-shiza` с собственным ботом в Max Messenger.

## Архитектура
- **VPS-Шиза** (100.84.167.53 через Tailscale) — оркестратор, стратег, работает с БД/дашбордом/сайтом
- **Windows-Шиза** (100.100.142.77) — локальный агент на ноутбуке, управляет Windows, LM Studio, ComfyUI
- **Связь:** Tailscale mesh VPN + Kanban board + Max Messenger (через отдельных ботов)
- **Модель:** deepseek/deepseek-v4-flash на Polza.ai (тот же провайдер, что у VPS-Шизы)
- **Fallback:** LM Studio на :1234 (локальный LLM, бесплатно)

## Что может Windows-Шиза
- PowerShell: управление службами, процессами, реестром, файловой системой
- Скриншоты через .NET Forms
- Управление Task Scheduler и автозагрузкой
- Брандмауэр Windows (сетевой доступ, открытие портов)
- Работа с LM Studio (:1234) — локальный vision/LLM
- Работа с ComfyUI (:8188) — генерация изображений по запросу VPS-Шизы

## Установка
Пошаговая инструкция: `SETUP_GUIDE.md` в профиле
Архив файлов: `/root/.hermes/profiles/windows-shiza.tar.gz`

## Статус
- [x] SOUL.md (душа Windows-Шизы) — создан
- [x] config.yaml (шаблон профиля) — создан
- [x] .env (переменные окружения) — создан
- [x] prefill_interaction_laws.json — создан
- [x] SETUP_GUIDE.md (инструкция установки) — создан
- [x] Архив профиля (tar.gz) — создан
- [ ] Установка на ноутбук — требуется Владимир
- [ ] Первый запуск — требуется Владимир
- [ ] Тестовая связь VPS↔Windows — после установки