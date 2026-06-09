# Fail2ban

## Цель
Защита SSH и сервисов Docker от brute-force атак.

## Состояние сервиса
systemctl status fail2ban
fail2ban-client status
fail2ban-client status 3x-ui
fail2ban-client status sshd

## Настройка фильтров
vi /etc/fail2ban/jail.d/3x-ui.conf
vi /etc/fail2ban/jail.local
systemctl restart fail2ban

## Что практиковал
- Настройка Fail2ban
- Проверка состояния jail
- Перезапуск сервиса
