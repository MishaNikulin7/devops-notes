# Zabbix

## Цель
Развёртывание мониторинга сервера через Zabbix в Docker.

## Контейнеры
| Контейнер | Образ | Статус |
|-----------|-------|--------|
| zabbix-server-mysql | zabbix/zabbix-server-mysql:alpine-7.0-latest | Up 13 days |
| zabbix-web-nginx-mysql | zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest | Up 13 days (healthy) |

## Используемые порты
- Zabbix Web: 8081, 14443
- Zabbix Server: 10051

## Команды
```bash
docker ps
docker logs zabbix-docker-zabbix-server-1 --tail 20
docker logs zabbix-docker-zabbix-web-nginx-mysql-1 --tail 20
