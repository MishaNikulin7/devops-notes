# MySQL

## Обзор

MySQL используется как база данных для Zabbix и развернут в Docker Compose.

В текущем стеке используется образ:

```text
mysql:8.0-oracle
```

MySQL не публикует порт базы данных напрямую на хост и доступен другим
компонентам через внутреннюю Docker-сеть `database`.

## Контейнер

| Параметр | Значение |
|---|---|
| Service | `mysql-server` |
| Image | `mysql:8.0-oracle` |
| Database | `zabbix` |
| Docker network | `database` |
| Container IP | `172.19.0.2` |
| MySQL port | `3306` |

Порт `3306` не опубликован на VPS.

MySQL подключен только к Docker-сети `database`:

```text
database
172.19.0.0/16
      |
      +-------------------+-------------------+
      |                   |                   |
      v                   v                   v
    MySQL             Zabbix Web         Zabbix Server
 172.19.0.2           172.19.0.3          172.19.0.4
    :3306
```

Сеть `database` имеет параметр `Internal=true` и используется для
внутреннего взаимодействия компонентов стека с базой данных.

MySQL не подключен к сетям `frontend` и `backend`.

## Docker DNS

Docker предоставляет встроенный DNS, благодаря которому контейнеры могут
обращаться друг к другу по DNS-именам вместо использования жестко заданных
IP-адресов.

MySQL доступен внутри Docker-сети по имени:

```text
mysql-server
```

Проверить разрешение имени можно из контейнера Zabbix Server:

```bash
docker exec zabbix-docker-zabbix-server-1 \
  getent hosts mysql-server
```

Результат:

```text
172.19.0.2    mysql-server
```

Таким образом:

```text
Zabbix Server
      |
      | mysql-server
      v
  Docker DNS
      |
      | 172.19.0.2
      v
    MySQL
 172.19.0.2:3306
```

Использование DNS-имени позволяет не привязывать конфигурацию приложений
к конкретному IP-адресу контейнера.

При пересоздании контейнера его IP-адрес может измениться, при этом
DNS-имя остается доступным для других контейнеров в соответствующей
Docker-сети.

## Проверка

Проверить состояние MySQL:

```bash
docker compose ps mysql-server
```

Посмотреть последние сообщения в логах:

```bash
docker compose logs --tail 50 mysql-server
```

Проверить сети и параметры контейнера:

```bash
docker inspect zabbix-docker-mysql-server-1
```

Проверить DNS-разрешение MySQL из Zabbix Server:

```bash
docker exec zabbix-docker-zabbix-server-1 \
  getent hosts mysql-server
```
