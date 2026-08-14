# Fail2Ban

## Обзор

Fail2Ban используется на VPS для защиты публичных сервисов от
автоматизированных попыток подбора учетных данных и неуспешных попыток
аутентификации.

На сервере настроены два jail:

- `sshd` — защита SSH;
- `3x-ui` — защита панели 3x-ui.

Fail2Ban анализирует системный журнал и логи приложений, ищет события,
соответствующие заданным фильтрам, после чего блокирует IP-адрес нарушителя.

## Состояние сервиса

Fail2Ban работает как systemd-сервис.

Проверить состояние:

```bash
systemctl status fail2ban
```

Сервис находится в состоянии:

```text
Active: active (running)
```

и включен в автозагрузку.

Проверить активные jail:

```bash
fail2ban-client status
```

Результат:

```text
Number of jail: 2
Jail list: 3x-ui, sshd
```

## Как работает Fail2Ban

Общая схема работы:

```text
        попытка аутентификации
                 |
                 v
        log / systemd journal
                 |
                 v
          Fail2Ban filter
             (failregex)
                 |
          найден IP <HOST>
                 |
                 v
                jail
                 |
                 v
            ban action
                 |
                 v
        iptables / firewall
```

`filter` определяет, какие строки журнала считать неуспешными попытками
аутентификации.

`jail` связывает фильтр с параметрами блокировки: количеством попыток,
интервалом анализа, длительностью блокировки и действием, которое необходимо
выполнить.

## Jail: SSH

Для защиты SSH используется отдельный jail:

```ini
[sshd]
enabled = true
backend = systemd
filter = sshd-custom
maxretry = 2
bantime = -1
findtime = 10m
```

### Параметры

| Параметр | Значение | Назначение |
|---|---|---|
| `enabled` | `true` | Jail активен |
| `backend` | `systemd` | События читаются из systemd journal |
| `filter` | `sshd-custom` | Используется пользовательский SSH-фильтр |
| `maxretry` | `2` | Блокировка после двух совпадений |
| `findtime` | `10m` | Интервал анализа событий |
| `bantime` | `-1` | Блокировка без автоматического истечения |

При такой конфигурации два подходящих события в течение 10 минут
могут привести к блокировке IP-адреса.

### SSH-фильтр

Используется пользовательский фильтр:

```text
/etc/fail2ban/filter.d/sshd-custom.conf
```

Фильтр отслеживает несколько типов событий SSH:

```text
Failed publickey
Connection closed by authenticating user
Invalid user
```

В регулярных выражениях используется `<HOST>` — специальный placeholder
Fail2Ban для определения IP-адреса клиента.

Проверить состояние jail:

```bash
fail2ban-client status sshd
```

На момент проверки:

```text
Total failed: 12023
Currently banned: 4811
Total banned: 4811
```

Полный список заблокированных IP-адресов в документацию не добавляется,
так как он является динамическим и может содержать тысячи адресов.

## Jail: 3x-ui

Для панели 3x-ui используется отдельный jail:

```ini
[3x-ui]
enabled = true
port = 4443,2053
protocol = tcp
filter = 3x-ui
logpath = /var/log/3x-ui/access.log
maxretry = 1
bantime = -1
findtime = 600
chain = DOCKER-USER
action = iptables-allports[name=3x-ui, chain=DOCKER-USER]
```

### Параметры

| Параметр | Значение | Назначение |
|---|---|---|
| `filter` | `3x-ui` | Пользовательский фильтр |
| `logpath` | `/var/log/3x-ui/access.log` | Лог, анализируемый Fail2Ban |
| `maxretry` | `1` | Блокировка после одного совпадения |
| `findtime` | `600` | Окно анализа — 600 секунд |
| `bantime` | `-1` | Блокировка без автоматического истечения |
| `chain` | `DOCKER-USER` | Цепочка iptables для Docker-трафика |

### Фильтр 3x-ui

Фильтр анализирует сообщения приложения о неуспешной аутентификации:

```text
WARNING - wrong username
WARNING - wrong password for user
```

Из совпавшей строки Fail2Ban получает IP клиента через `<HOST>`.

После обнаружения события применяется действие:

```text
3x-ui log
    |
    v
Fail2Ban filter
    |
    v
3x-ui jail
    |
    v
iptables-allports
    |
    v
DOCKER-USER
    |
    v
IP blocked
```

Использование цепочки `DOCKER-USER` позволяет применять правила
фильтрации к трафику Docker-контейнеров.

Проверить состояние jail:

```bash
fail2ban-client status 3x-ui
```

На момент проверки:

```text
Currently failed: 0
Total failed: 0
Currently banned: 504
Total banned: 504
```

## Постоянная блокировка

В обоих jail используется:

```ini
bantime = -1
```

Это означает, что блокировка не снимается автоматически после обычного
интервала `bantime`.

Разблокировать конкретный IP вручную можно командой:

```bash
fail2ban-client set sshd unbanip <IP>
```

Для jail `3x-ui`:

```bash
fail2ban-client set 3x-ui unbanip <IP>
```

## Файлы конфигурации

Основные пользовательские конфигурации:

```text
/etc/fail2ban/
├── jail.d/
│   ├── 3x-ui.conf
│   └── sshd.local
│
└── filter.d/
    ├── 3x-ui.conf
    └── sshd-custom.conf
```

Разделение `jail` и `filter` позволяет отдельно задавать:

- `filter` — какие события считать нарушением;
- `jail` — когда и каким способом блокировать IP.

## Диагностика

Показать все активные jail:

```bash
fail2ban-client status
```

Проверить конкретный jail:

```bash
fail2ban-client status sshd
fail2ban-client status 3x-ui
```

Посмотреть журнал Fail2Ban:

```bash
journalctl -u fail2ban
```

Последние 100 сообщений:

```bash
journalctl -u fail2ban -n 100 --no-pager
```

Проверить состояние systemd-сервиса:

```bash
systemctl status fail2ban --no-pager
```

## Итоговая схема

```text
SSH client
    |
    v
systemd journal
    |
    v
sshd-custom filter
    |
    v
sshd jail
    |
    v
IP blocked


3x-ui client
    |
    v
/var/log/3x-ui/access.log
    |
    v
3x-ui filter
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

