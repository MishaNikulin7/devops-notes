# Nginx + SSL

## Цель
Документирование веб-сервера и TLS сертификатов на VPS.

## Состояние сервера
systemctl status nginx

## Сертификаты

| Домен | Дата окончания | Статус |
|-------|----------------|--------|
| m.nikulin.dev | 2026-08-21 | VALID |
| zbx.nikulin.dev | 2026-08-21 | VALID |

## Команды
```bash
nginx -T
systemctl reload nginx
certbot certificates
systemctl status certbot-renew.timer
