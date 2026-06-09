# Nginx с SSL

## Цель
Документирование веб-сервера и TLS сертификатов.

## Проверка конфигурации
nginx -T
systemctl reload nginx

## Установка Certbot
dnf install certbot python3-certbot-nginx -y
certbot --nginx -d m.nikulin.dev
certbot --nginx -d zbx.nikulin.dev

## Автоматическое обновление сертификатов
systemctl status certbot-renew.timer
systemctl enable --now certbot-renew.timer
certbot renew --dry-run

## Что практиковал
- Настройка Nginx
- Настройка SSL сертификатов
- Проверка автоматического обновления
