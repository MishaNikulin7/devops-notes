# Zabbix

## Цель
Мониторинг VPS-сервера с помощью Zabbix, развернутого в Docker.

## Контейнеры
| Контейнер | Образ | Статус |
|-----------|-------|--------|
| zabbix-server-mysql | zabbix/zabbix-server-mysql:alpine-7.0-latest | Up 13 дней |
| zabbix-web-nginx-mysql | zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest | Up 13 дней (healthy) |

## Порты
- Zabbix Web: 8081, 14443
- Zabbix Server: 10051

## Используемые команды
```bash
docker ps
docker logs --tail 20 zabbix-docker-zabbix-server-1
docker logs --tail 20 zabbix-docker-zabbix-web-nginx-mysql-1
