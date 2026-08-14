# Nginx Reverse Proxy + HTTPS

Практический проект по настройке Nginx на собственном Linux VPS.

Nginx используется как точка входа для нескольких сервисов:
принимает HTTP/HTTPS-запросы, перенаправляет HTTP на HTTPS
и проксирует запросы к локальным backend-сервисам.

## Что реализовано

- Nginx reverse proxy для нескольких сервисов
- отдельные virtual hosts
- перенаправление HTTP → HTTPS
- TLS 1.2 / TLS 1.3
- Let's Encrypt + Certbot
- автоматическое обновление сертификатов через systemd timer
- проксирование запросов к локальным backend-сервисам
- диагностика через `nginx -t`, `systemctl`, `journalctl`, `ss` и `curl`

## Архитектура

```text
                         Internet
                            |
                  HTTP :80 / HTTPS :443
                            |
                            v
                         Nginx
                      Reverse Proxy
                       /          \
                      /            \
                     v              v
             m.nikulin.dev     zbx.nikulin.dev
                     |              |
                     v              v
             127.0.0.1:2053   127.0.0.1:8081
                    |                |
                    v                v
            Docker service    Docker service
```

Nginx принимает внешние подключения на портах `80` и `443`.

Backend-сервисы напрямую через Nginx не публикуются:
reverse proxy перенаправляет запросы на локальные порты.

## Virtual Hosts

Для сервисов используются отдельные конфигурации:

```text
/etc/nginx/conf.d/
├── m.nikulin.dev.conf
└── zbx.nikulin.dev.conf
```

### m.nikulin.dev

HTTPS-запросы проксируются на локальный backend:

```nginx
location / {
    proxy_pass http://127.0.0.1:2053;

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### zbx.nikulin.dev

Zabbix доступен через отдельное доменное имя.

Nginx проксирует HTTPS-запросы на сервис:

```nginx
location / {
    proxy_pass http://127.0.0.1:8081;

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

## HTTP → HTTPS

Для HTTP используется перенаправление:

```nginx
return 301 https://$server_name$request_uri;
```

Таким образом обычный HTTP-запрос перенаправляется на HTTPS.

Проверка:

```bash
curl -I http://m.nikulin.dev
curl -I http://zbx.nikulin.dev
```

## TLS

Для HTTPS используются сертификаты Let's Encrypt.

В конфигурации Nginx подключены сертификат и закрытый ключ:

```nginx
ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
```

Поддерживаемые версии TLS:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

Также настроен HSTS:

```nginx
add_header Strict-Transport-Security "max-age=63072000" always;
```

> Закрытые ключи и другие секретные данные в репозиторий не добавляются.

## Certbot

Просмотр установленных сертификатов:

```bash
certbot certificates
```

Автоматическое обновление сертификатов выполняется через systemd timer:

```bash
systemctl status certbot-renew.timer
```

Проверить расписание:

```bash
systemctl list-timers | grep certbot
```

Безопасно протестировать процедуру обновления:

```bash
certbot renew --dry-run
```

## Проверка Nginx

Перед применением изменений проверяю синтаксис конфигурации:

```bash
nginx -t
```

После успешной проверки:

```bash
systemctl reload nginx
```

Проверить состояние сервиса:

```bash
systemctl status nginx
```

## Проверка сетевых портов

```bash
ss -lntp | grep -E ':80|:443'
```

Nginx должен слушать:

```text
TCP :80   HTTP
TCP :443  HTTPS
```

Проверить backend-порты:

```bash
ss -lntp | grep -E ':2053|:8081'
```

## Проверка сервисов

Проверка внешнего HTTP/HTTPS:

```bash
curl -I https://m.nikulin.dev
curl -I https://zbx.nikulin.dev
```

Проверка backend напрямую с VPS:

```bash
curl -I http://127.0.0.1:2053
curl -I http://127.0.0.1:8081
```

Это позволяет определить, находится ли проблема на стороне
backend-сервиса или reverse proxy.

## Диагностика

При недоступности сервиса использую последовательную проверку:

```text
DNS
 |
 v
TCP :80/:443
 |
 v
Nginx
 |
 v
TLS
 |
 v
Reverse Proxy
 |
 v
Backend
```

### 1. DNS

```bash
dig m.nikulin.dev
dig zbx.nikulin.dev
```

### 2. Порты

```bash
ss -lntp
```

### 3. Состояние Nginx

```bash
systemctl status nginx
```

### 4. Конфигурация

```bash
nginx -t
```

### 5. Журналы

```bash
journalctl -u nginx -n 50
```

Для Zabbix также используется отдельный error log:

```bash
tail -f /var/log/nginx/zbx.nikulin.dev.error.log
```

### 6. Очистка логов

```bash
find /var/log/nginx -type f -mtime +30 -ls
find /var/log/nginx -type f -mtime +30 -delete
```

### 7. Backend

```bash
curl -I http://127.0.0.1:8081
```
