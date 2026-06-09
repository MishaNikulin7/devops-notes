# MySQL

## Цель
Документирование базы данных MySQL, используемой на VPS для Zabbix и других сервисов.

## Контейнер
- Имя: zabbix-docker-mysql-server-1
- Образ: mysql:8.0-oracle
- Статус: Up 13 дней
- Порт: 3306

## Используемые команды
```bash
docker ps
docker logs zabbix-docker-mysql-server-1 --tail 20
docker exec -it zabbix-docker-mysql-server-1 mysql -uroot -p
