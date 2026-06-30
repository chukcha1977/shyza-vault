---
title: RESTORE — Инструкция по восстановлению системы
type: concept
created: 2026-06-08
updated: 2026-06-29
summary: Полная инструкция по восстановлению Шизы из бэкапа при сбое после обновления
---

# RESTORE — Инструкция по восстановлению системы

> **Если Шиза не отвечает после обновления — действуй по этому документу.**
> Дата: 8 июня 2026, обновление до Hermes v0.16.0

---

## 1. БЫСТРАЯ ПРОВЕРКА — ЖИВА ЛИ СИСТЕМА?

```bash
# Все ли сервисы работают?
for s in hermes-gateway hermes-gateway-shnyr hermes-gateway-shy hermes-lm-proxy hermes-whisper-proxy; do
  echo "$s: $(systemctl is-active $s 2>/dev/null || echo 'not found')"
done

# Какая версия Hermes?
hermes --version

# Работает ли дашборд (заказы/финансы)?
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8081/
```

Если все `active` — система жива, проблема во мне (Шизе). Напиши в Telegram, я перезапущусь.

---

## 2. ГДЕ БЭКАПЫ

```
/root/.hermes/backups/pre-upgrade-20260608.tar.gz   — полный бэкап системы
/root/.hermes/backups/pre-upgrade-20260608.tar.gz.md5 — контрольная сумма
```

**Проверить целостность:**
```bash
cd /root/.hermes/backups && md5sum -c pre-upgrade-20260608.tar.gz.md5
# Должно быть: pre-upgrade-20260608.tar.gz: ОК
```

**Что в архиве:**
- Код Hermes Agent (без venv)
- Все конфиги, SOUL.md, навыки, профили Шныря и Шы
- Google OAuth токены
- Obsidian vault (база знаний)
- Systemd-юниты

---

## 3. ВОССТАНОВЛЕНИЕ ПОСЛЕ НЕУДАЧНОГО ОБНОВЛЕНИЯ

Если gateway не стартует или Hermes сломался:

### 3.1. Откатить код

```bash
cd /usr/local/lib/hermes-agent
git stash  # спрятать кастомные изменения
git checkout v2026.5.29  # вернуться на старую версию
source venv/bin/activate && pip install --break-system-packages -e .
systemctl restart hermes-gateway
```

### 3.2. Восстановить из бэкапа (полный откат)

```bash
cd /
tar xzf /root/.hermes/backups/pre-upgrade-20260608.tar.gz
systemctl daemon-reload && systemctl restart hermes-gateway
```

### 3.3. Восстановить БД (MariaDB)

```bash
mysql -uadmin -p'ChuchmDb!Admin2025' < /root/.hermes/backups/alldbs_20260608.sql
```

⚠️ Пароль в одинарных кавычках — `!` в пароле!

---

## 4. ЕСЛИ ПРОПАЛ MAX MESSENGER

Плагин Max лежит в `/root/.hermes/hermes-agent/plugins/platforms/max/`.
Если после обновления Max не отвечает — проверь логи:
```bash
tail -50 /root/.hermes/logs/gateway.log | grep -i "max\|error\|traceback"
```

Решение: отредактировать `adapter.py` под новую версию Hermes.

---

## 5. ЕСЛИ ПРОПАЛИ API-КЛЮЧИ

Файл: `/root/.hermes/.env`

Ключи, которые там должны быть:
- `POLZA_KEY_3` — основная модель
- `POLZA_KEY_1` — vision/compression
- `POLZA_KEY_2` — резервный
- `MAX_BOT_TOKEN` — Max Messenger (один на всех ботов)
- `TELEGRAM_BOT_TOKEN` — Telegram
- `GITHUB_TOKEN` — GitHub

Если .env пустой — восстановить из бэкапа.

---

## 6. АВАРИЙНЫЙ КАНАЛ — TELEGRAM

Если Max не работает, напиши мне в Telegram: `@chuchm_bot`.
Я отвечу там и восстановлю Max.

Если и Telegram не работает — через GitHub Issues: `github.com/chukcha1977/obsidian-shyza-vault`

---

## 7. КЛЮЧЕВЫЕ ПУТИ НА СЕРВЕРЕ

| Что | Где |
|-----|-----|
| Конфиг | `/root/.hermes/config.yaml` |
| Все API-ключи | `/root/.hermes/.env` |
| Душа Шизы | `/root/.hermes/SOUL.md` |
| Навыки | `/root/.hermes/skills/` |
| Профиль Шныря | `/root/.hermes/profiles/shnyr/` |
| Профиль Шы | `/root/.hermes/profiles/shy/` |
| Max плагин | `/root/.hermes/hermes-agent/plugins/platforms/max/` |
| Код Hermes | `/usr/local/lib/hermes-agent/` |
| Дашборд (новый) | `/root/dashboard2/` (порт 8081) |
| Дашборд (старый) | `/root/dashboard/` (порт 8080) |
| Google токены | `/root/.google-credentials/` |
| База знаний | `/root/vault/obsidian-shyza-vault/` |
| Логи Шизы | `/root/.hermes/logs/gateway.log` |
| Логи Шныря | `/root/.hermes/profiles/shnyr/logs/gateway.log` |
| Логи Шы | `/root/.hermes/profiles/shy/logs/gateway.log` |
| MariaDB заказы | `chuchm_orders` |
| MariaDB финансы | `chuchm_finance` |

---

## 8. СТАРЫЙ ДОБРЫЙ ПЕРЕЗАПУСК (если всё залипло)

```bash
for s in hermes-gateway hermes-gateway-shnyr hermes-gateway-shy hermes-lm-proxy hermes-whisper-proxy; do
  sudo systemctl restart $s
done
```

После рестарта Max adapter сбрасывает marker — первые 1-2 минуты сообщения могут теряться.
Напиши что-нибудь — marker восстановится.

---

**Связанные концепции:** [[concepts/restore-guide|Восстановление из бэкапа (для Шы)]] | [[concepts/hermes-disaster-recovery|Аварийное восстановление]]