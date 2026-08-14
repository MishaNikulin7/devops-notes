# Docker Services

## Обзор

На VPS запущено несколько Docker-контейнеров, которые обслуживают
Zabbix и 3x-ui.

Docker используется для изоляции сервисов, управления сетями
и публикации необходимых портов на хост.

## Запущенные контейнеры

| Контейнер | Image | Назначение |
|---|---|---|
| `3x-ui` | `ghcr.io/mhsanaei/3x-ui:v3.3.1` | Веб-панель 3x-ui |
| `zabbix-docker-zabbix-web-nginx-mysql-1` | `zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest` | Веб-интерфейс Zabbix |
| `zabbix-docker-zabbix-server-1` | `zabbix/zabbix-server-mysql:alpine-7.0-latest` | Zabbix Server |
| `zabbix-docker-mysql-server-1` | `mysql:8.0-oracle` | MySQL для Zabbix |

Текущее состояние контейнеров можно проверить командой:

```bash
docker ps
```

Для более компактного вывода:

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"
```

## Публикация портов

Docker может публиковать порт контейнера на сетевой интерфейс VPS.

Формат:

```text
host_ip:host_port -> container_port
```

### Zabbix Web

```text
0.0.0.0:8081  -> 8080/tcp
0.0.0.0:14443 -> 9443/tcp
```

Порт `8081` на хосте перенаправляется на HTTP-порт `8080`
внутри контейнера Zabbix Web.

```text
VPS :8081
   |
   v
Docker
   |
   v
Zabbix Web :8080
```

Порт `14443` аналогично перенаправляется на `9443`.

### Zabbix Server

```text
0.0.0.0:10051 -> 10051/tcp
```

Порт `10051` опубликован на хосте и используется Zabbix Server.

### MySQL

MySQL показывает внутренние порты:

```text
3306/tcp
33060/tcp
```

При этом они не опубликованы на интерфейс VPS.

MySQL доступен другим контейнерам через Docker-сеть `database`.

### 3x-ui

Контейнер публикует несколько портов:

```text
0.0.0.0:4443 -> 4443/tcp
0.0.0.0:5443 -> 5443/tcp
0.0.0.0:7443 -> 7443/tcp
127.0.0.1:2053 -> 2053/tcp
```

Особенность порта `2053`:

```text
127.0.0.1:2053
```

Он привязан только к loopback-интерфейсу VPS и не публикуется
на всех сетевых интерфейсах.

Этот порт используется как backend для Nginx reverse proxy.

## Docker-сети

На VPS используются стандартные и пользовательские Docker-сети.

Проверить их можно командой:

```bash
docker network ls
```

Основные сети Zabbix:

| Docker network | Driver |
|---|---|
| `zabbix-docker_frontend` | `bridge` |
| `zabbix-docker_backend` | `bridge` |
| `zabbix-docker_database` | `bridge` |
| `zabbix-docker_tools_frontend` | `bridge` |

Также контейнер `3x-ui` подключен к стандартной сети:

```text
bridge
```

## Подключение контейнеров к сетям

### 3x-ui

```text
bridge -> 172.17.0.2
```

### Zabbix Web

```text
zabbix-docker_frontend -> 172.16.238.2
zabbix-docker_backend  -> 172.16.239.2
zabbix-docker_database -> 172.19.0.3
```

### Zabbix Server

```text
zabbix-docker_frontend       -> 172.16.238.3
zabbix-docker_backend        -> 172.16.239.3
zabbix-docker_database       -> 172.19.0.4
zabbix-docker_tools_frontend -> 172.16.240.2
```

### MySQL

```text
zabbix-docker_database -> 172.19.0.2
```

Таким образом, один контейнер может быть подключен сразу
к нескольким Docker-сетям и иметь отдельный IP-адрес в каждой из них.

## Взаимодействие сервисов

Упрощенно Docker-часть инфраструктуры выглядит так:

```text
                     VPS
                      |
        +-------------+-------------+
        |                           |
        v                           v
      3x-ui                      Zabbix
   172.17.0.2                      |
                                   |
                 +-----------------+-----------------+
                 |                 |                 |
                 v                 v                 v
            Zabbix Web       Zabbix Server         MySQL
            172.19.0.3        172.19.0.4        172.19.0.2
                 \                 |                 /
                  +----------------+----------------+
                           database
                         172.19.0.0/16
```

Zabbix Web и Zabbix Server также подключены к другим сетям,
которые используются для разделения компонентов стека.

Подробная схема сетей описана в документации Zabbix.

## Docker Compose

Zabbix управляется через Docker Compose.

Текущие сервисы можно проверить:

```bash
docker compose -f /opt/zabbix-docker.old/zabbix-docker/compose.yaml ps
```

Основные Compose-сервисы:

```text
mysql-server
zabbix-server
zabbix-web-nginx-mysql
```

## Диагностика

Посмотреть запущенные контейнеры:

```bash
docker ps
```

Посмотреть сети:

```bash
docker network ls
```

Проверить параметры контейнера:

```bash
docker inspect <container>
```

Посмотреть логи:

```bash
docker logs <container>
```

Посмотреть последние 50 строк:

```bash
docker logs --tail 50 <container>
```

Проверить сети конкретного контейнера:

```bash
docker inspect -f \
'{{range $name, $net := .NetworkSettings.Networks}}{{$name}} -> {{$net.IPAddress}}{{println}}{{end}}' \
<container>
```
