# Fail2Ban Troubleshooting

## 📌 Цель

Диагностика и восстановление доступа при блокировке IP-адреса Fail2Ban.

## 🔍 Просмотр заблокированных IP

Пример:

```
Status for the jail: nginx-http-auth
|- Currently banned: 1
`- Banned IP list:
   194.150.xxx.xxx
```

## 🔎 Поиск конкретного IP

Проверить, заблокирован ли определённый IP в jail:

```bash
fail2ban-client status sshd | grep -o "98.70.50.166"
```

Если IP заблокирован, команда выведет:

```text
98.70.50.166
```

Если вывода нет — IP в данном jail не найден.

---

## 🔍 Проверка всех заблокированных IP в jail

```bash
fail2ban-client status sshd
```

Пример:

```text
Banned IP list:
98.70.50.166 45.22.11.10
```

---

## 🔓 Разблокировать IP

```bash
fail2ban-client set nginx-http-auth unbanip YOUR_IP
```

---

## ⚙ Добавить свой IP в белый список

Открыть:

```bash
vi /etc/fail2ban/jail.conf
```

Добавить или изменить строку:

```ini
ignoreip = 127.0.0.1/8 ::1 HOME_IP
```

После изменения:

```bash
systemctl restart fail2ban
```

---

## 🧪 Проверка после изменений

```bash
fail2ban-client status
```

Проверить список jail:

```bash
fail2ban-client status nginx-http-auth
```

Проверить, что домашний IP отсутствует в списке заблокированных.

---

## 📝 Что произошло в моём случае

После перезапуска контейнеров Docker доступ к панели 3x-ui пропал.

Причина оказалась не в Docker и не в 3x-ui.

Fail2Ban заблокировал мой домашний IP после нескольких попыток подключения к панели.

После разблокировки IP доступ к панели и VPN был восстановлен.

---

## ⚠️ Вывод

Перед поиском проблемы в Docker, Nginx или 3x-ui всегда проверить:

- работает ли Fail2Ban;
- не заблокирован ли мой IP;
- какие jail активны;
- нет ли моего IP в списке Banned IP.
