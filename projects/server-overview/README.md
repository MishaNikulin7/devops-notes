# Обзор VPS-сервера

## Цель

Документирование моего VPS-сервера, который используется для практики Linux-администрирования, Docker, мониторинга и базовой безопасности.

## Информация о сервере

- Провайдер: RackNerd
- Тип: VPS / KVM
- ОС: AlmaLinux 9.7
- Ядро: Linux 5.14
- Архитектура: x86_64
- RAM: 1.7 GiB
- Swap: 1.0 GiB
- Диск: 28 GB
- Использовано диска: 8.2 GB
- Свободно: 19 GB

## Основные сервисы

- Docker
- Nginx
- Zabbix Server
- MySQL
- 3x-ui
- Fail2ban
- Firewalld
- Certbot

## Docker-контейнеры

На сервере запущены контейнеры:

- 3x-ui
- Zabbix Web Nginx MySQL
- Zabbix Server MySQL
- MySQL 8.0

## Безопасность

Fail2ban установлен напрямую в систему и работает как systemd-сервис.

Активные jail:

- sshd
- 3x-ui

## Сетевые порты

На сервере используются порты:

- 22 — SSH
- 80 — HTTP
- 443 — HTTPS
- 8081 — Zabbix Web
- 10051 — Zabbix Server
- 2053, 4443, 5443, 7443 — 3x-ui

## Что я практиковал

- Работа с Linux-сервером
- Проверка состояния системы
- Работа с systemd
- Работа с Docker
- Проверка открытых портов через ss
- Настройка Fail2ban
- Работа с Git и GitHub через SSH
- Работа с ветками и Pull Request

## Команды, которые использовались

```bash
hostnamectl
free -h
df -h
docker ps
systemctl status fail2ban
fail2ban-client status
ss -tulpn
