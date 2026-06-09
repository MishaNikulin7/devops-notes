# Docker сервисы

## Цель

Документировать Docker-контейнеры, которые работают на моём VPS-сервере.

## Запущенные контейнеры

| Контейнер | Образ | Статус |
|---|---|---|
| 3x-ui | ghcr.io/mhsanaei/3x-ui:latest | Up 13 days |
| zabbix-web-nginx-mysql | zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest | Up 13 days (healthy) |
| zabbix-server-mysql | zabbix/zabbix-server-mysql:alpine-7.0-latest | Up 13 days |
| mysql-server | mysql:8.0-oracle | Up 13 days |

## Используемые команды

```bash
docker ps
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
docker logs 3x-ui
docker restart 3x-ui
docker exec -it 3x-ui /bin/sh
