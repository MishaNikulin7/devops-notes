# Fail2ban

## Цель

Защита VPS-сервера от автоматизированных атак на панель управления 3x-ui.

## Состояние сервиса

Проверка:

systemctl status fail2ban

Результат:

Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled)

Active: active (running)

Main PID: 662 (fail2ban-server)

## Активные правила защиты

Проверка:

fail2ban-client status

Результат:

Number of jail: 2

Jail list:

- 3x-ui
- sshd

## Статистика защиты 3x-ui

Проверка:

fail2ban-client status 3x-ui

Результат на момент документирования:

Currently failed: 0

Total failed: 76398

Currently banned: 495

Total banned: 495

## Используемые команды

systemctl status fail2ban

fail2ban-client status

fail2ban-client status 3x-ui

fail2ban-client status sshd

systemctl restart fail2ban

