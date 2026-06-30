---
title: Архитектура сайта penoizol-chernozem.ru
summary: Архитектура сайта на Битрикс — хостинг Beget, Nginx, PHP-FPM, MySQL. Структура страниц, алиасы, sitemap, админка.
created: 2026-05-27
updated: 2026-05-27
type: concept
tags: [website, penoizol, architecture]
---

# Архитектура сайта penoizol-chernozem.ru

> Как устроен сайт, на чём работает, где размещён.

## Движок
- Основной сайт — **1С-Битрикс** (шаблон Concept/Kraken)
- Документ-рут: `/home/c/chuchm/promo.penoizol-chernozem.ru/public_html/`
- Старая Joomla лежит в `penoizol-chernozem.ru/public_html/` — **не обслуживается**

## Хостинг
- **Beget** (Россия, 87.236.16.9)
- Антибот блокирует headless Chrome (белый экран на cp.beget.com)
- SFTP работает: доступ `chuchm_shyza` / `p7m8NsHk&JCY`

## Домены и поддомены
- `penoizol-chernozem.ru` — основной
- `promo.penoizol-chernozem.ru` — промо-поддомен
- `chat.penoizol-chernozem.ru` — дашборд (VPS, не Beget)
- `perevozki-voronezg.penoizol-voronezh.ru` — перевозки

## Структура
- 6 страниц в Bitrix (главная, услуги, FAQ, контакты, о компании, блог)
- 7 статей в Joomla (мигрировать)
- 10 тем на форуме Kunena (переработать в статьи)

## Связанные проекты
-   [[projects/site/Сайт (проект)|Текущие работы по сайту]]