---
title: Инструкция по восстановлению Шизы из бэкапа
summary: Пошаговая инструкция для Шы — как восстановить Шизу из полного бэкапа при краше: распаковка, проверка конфигов, перезапуск сервисов.
type: guide
created: 2026-05-31
updated: 2026-05-31
tags: [backup, restore, disaster-recovery]
---

# Восстановление системы агентов из полного бэкапа

## Где лежит последний бэкап

```
/root/.hermes/backups/pre-upgrade-20260531.tar.gz
```

Рядом лежит файл `.md5` с контрольной суммой.

## Полный состав архива

- `/usr/local/lib/hermes-agent/` — сам Hermes Agent
- `/root/.hermes/` — конфиги, стейты, сессии, скиллы, профили суб-агентов
- `/root/.google-credentials/` — OAuth-токены Google
- `/root/vault/obsidian-shyza-vault/` — Obsidian vault
- `/etc/systemd/system/hermes-*.service` — systemd-юниты
- `/root/.hermes/config.yaml` — дублирован в корень архива

## Пошаговая инструкция

### 1. Подготовка

```bash
# Убедись, что бэкап на месте
ls -lh /root/.hermes/backups/pre-upgrade-20260531.tar.gz
cat /root/.hermes/backups/pre-upgrade-20260531.tar.gz.md5
```

### 2. Остановка всех сервисов

```bash
systemctl stop hermes-gateway-shnyr.service
systemctl stop hermes-gateway-shy.service
systemctl stop hermes-gateway.service
systemctl stop hermes-whisper-proxy.service
systemctl stop hermes-lm-proxy.service

# Проверить, что все встали
systemctl is-active hermes-gateway.service hermes-gateway-shy.service hermes-gateway-shnyr.service hermes-lm-proxy.service hermes-whisper-proxy.service
# Должно вернуть: inactive inactive inactive inactive inactive
```

### 3. Проверка целостности архива

```bash
cd /
md5sum -c /root/.hermes/backups/pre-upgrade-20260531.tar.gz.md5
# Должно быть: pre-upgrade-20260531.tar.gz: ОК
```

### 4. Распаковка

```bash
cd /
tar -xzf /root/.hermes/backups/pre-upgrade-20260531.tar.gz
```

### 5. Восстановление systemd-юнитов

Файлы уже распакованы в `/etc/systemd/system/`. Перечитай их:

```bash
systemctl daemon-reload
```

### 6. Запуск сервисов

```bash
systemctl start hermes-lm-proxy.service
systemctl start hermes-whisper-proxy.service
systemctl start hermes-gateway.service
systemctl start hermes-gateway-shy.service
systemctl start hermes-gateway-shnyr.service

# Проверка
systemctl is-active hermes-gateway.service hermes-gateway-shy.service hermes-gateway-shnyr.service hermes-lm-proxy.service hermes-whisper-proxy.service
# Должно быть: active active active active active
```

### 7. Проверка работоспособности агентов

```bash
# Проверить логи
journalctl -u hermes-gateway.service -n 20 --no-pager
journalctl -u hermes-gateway-shy.service -n 20 --no-pager
journalctl -u hermes-gateway-shnyr.service -n 20 --no-pager
```

Если всё зелёное — агенты работают. Если какой-то упал, смотри конкретную ошибку в его логах.

## Питфоллы

- **Критично:** распаковывать ОБЯЗАТЕЛЬНО из корня (`cd /`), иначе пути разъедутся
- **Архив не содержит** venv Hermes Agent — после восстановления пересоздай: `cd /usr/local/lib/hermes-agent && python -m venv venv && source venv/bin/activate && pip install -e ".[dev]"`
- **OAuth-токены** могут протухнуть — если Google API перестанет работать, перевыпусти через `hermes google oauth`
- **Базы данных:** `.db`-файлы в архиве — делай бэкап перед восстановлением поверх старого, если не уверен
- Если система обновилась (ядро, пакеты) — старый Python в архиве уже не актуален, venv нужно пересоздать обязательно

## Контакты

Если восстановление не удалось — пиши мне в Telegram @chukcha1977 или в Max Messenger.

---

**Связанные концепции:** [[concepts/restore|RESTORE — инструкция]] | [[concepts/hermes-disaster-recovery|Аварийное восстановление]]
