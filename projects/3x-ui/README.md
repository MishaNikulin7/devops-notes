# 3x-ui

## Обзор

3x-ui развернут на VPS в Docker-контейнере.

Сервис работает в стандартной Docker bridge-сети, использует bind mount
для хранения данных и интегрирован с Nginx и Fail2Ban.

## Контейнер

| Параметр | Значение |
|---|---|
| Container | `3x-ui` |
| Image | `ghcr.io/mhsanaei/3x-ui:v3.3.1` |
| Status | `running` |
| Restart policy | `unless-stopped` |
| Network mode | `bridge` |
| Container IP | `172.17.0.2` |
| Gateway | `172.17.0.1` |

Контейнер запускается через:

```text
/app/DockerEntrypoint.sh
```

с аргументом:

```text
./x-ui
```

## Сеть

Контейнер подключен к стандартной Docker-сети `bridge`.

```text
VPS
 |
 | Docker bridge
 | Gateway: 172.17.0.1
 |
 +---- 172.17.0.2
          |
        3x-ui
```

IP `172.17.0.2` является внутренним адресом контейнера в Docker-сети
и используется для связи внутри хоста.

## Порты

Контейнер публикует следующие порты:

| Host | Container | Доступ |
|---|---|---|
| `127.0.0.1:2053` | `2053/tcp` | Только локально на VPS |
| `0.0.0.0:4443` | `4443/tcp` | Все IPv4-интерфейсы |
| `0.0.0.0:5443` | `5443/tcp` | Все IPv4-интерфейсы |
| `0.0.0.0:7443` | `7443/tcp` | Все IPv4-интерфейсы |

Порты `4443`, `5443` и `7443` также опубликованы на IPv6-интерфейсах.

Порт `2053` привязан только к loopback-интерфейсу:

```text
127.0.0.1:2053
```

Поэтому он не публикуется напрямую на внешних интерфейсах VPS.

## Reverse Proxy

Nginx использует локальный порт `2053` как backend для 3x-ui.

```text
Internet
   |
   | HTTPS :443
   v
 Nginx
   |
   | 127.0.0.1:2053
   v
 Docker
   |
   v
3x-ui:2053
```

Внешнее HTTPS-соединение завершается на Nginx, после чего запрос
проксируется к локально опубликованному порту контейнера.

Такой вариант позволяет не публиковать web backend 3x-ui напрямую
на внешних интерфейсах сервера.

## Persistent Storage

Для хранения данных используется bind mount:

```text
/root/x-ui-db -> /etc/x-ui
```

Схема:

```text
VPS filesystem
/root/x-ui-db
      |
      | bind mount
      v
3x-ui container
/etc/x-ui
```

Данные хранятся на файловой системе VPS отдельно от жизненного цикла
контейнера.

При пересоздании контейнера содержимое `/root/x-ui-db` на хосте
сохраняется.

## Restart Policy

Для контейнера настроена политика:

```text
unless-stopped
```

Docker автоматически запускает контейнер после перезапуска Docker daemon
или сервера, если контейнер ранее не был остановлен вручную.

## Fail2Ban

Для 3x-ui настроен отдельный Fail2Ban jail.

Fail2Ban анализирует:

```text
/var/log/3x-ui/access.log
```

и использует пользовательский фильтр для обнаружения неуспешных попыток
аутентификации.

Общая схема:

```text
3x-ui
  |
  v
access.log
  |
  v
Fail2Ban filter
  |
  v
3x-ui jail
  |
  v
iptables / DOCKER-USER
  |
  v
IP blocked
```

Блокировка выполняется через цепочку `DOCKER-USER`, что позволяет
применять правила фильтрации к Docker-трафику.

Подробная конфигурация Fail2Ban описана в:

```text
projects/fail2ban/
```

## Диагностика

Состояние контейнера:

```bash
docker ps --filter "name=3x-ui"
```

Последние сообщения контейнера:

```bash
docker logs 3x-ui --tail 50
```

Проверка опубликованных портов:

```bash
docker port 3x-ui
```

## Архитектура

```text
                     Internet
                        |
                        | HTTPS :443
                        v
                      Nginx
                        |
                        | 127.0.0.1:2053
                        v
                 Docker port mapping
                        |
                        v
                  3x-ui container
                    172.17.0.2
                        |
              +---------+---------+
              |                   |
              v                   v
          /etc/x-ui          access.log
              |                   |
         bind mount            Fail2Ban
              |                   |
              v                   v
       /root/x-ui-db         DOCKER-USER
                                  |
                                  v
                              IP blocked
```
