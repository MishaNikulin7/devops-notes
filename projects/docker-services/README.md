# Docker сервисы

## Цель
Документирование всех Docker-контейнеров, работающих на VPS.

## Контейнеры
- 3x-ui
- Zabbix Web Nginx MySQL
- Zabbix Server MySQL
- MySQL 8.0

## Команды проверки
docker ps
docker logs 3x-ui
docker logs zabbix-server-mysql
docker logs zabbix-web-nginx-mysql

## Старт и рестарт контейнеров
docker start 3x-ui
docker restart 3x-ui
docker exec -it 3x-ui /bin/sh

## Порты
- 3x-ui: 2053, 4443, 5443, 7443
- Zabbix Web: 8081, 14443
- Zabbix Server: 10051
- MySQL: 3306, 33060

## Примечание
Пароли и конфиденциальные данные не публикуются.
