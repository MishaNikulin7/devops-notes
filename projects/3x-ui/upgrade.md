# 3x-ui Upgrade Runbook (Docker VPS)

## 📌 Цель
Обновление 3x-ui без потери VPN пользователей и без простоя.

---

## 📁 Хранение данных (ВАЖНО)

/root/x-ui-db → /etc/x-ui

📌 Это означает:
- /root/x-ui-db = данные на VPS
- /etc/x-ui = данные внутри контейнера

👉 Docker связывает их через volume (-v)

---

## 🔍 1. Проверка состояния

docker ps | grep 3x-ui
docker images | grep 3x-ui

Ожидаем:
- контейнер RUNNING v.2.8.4
- порты 2053 / 4443 / 5443 / 7443

---

## 💾 2. Backup

tar -czvf xui-backup-$(date +%F).tar.gz /root/x-ui-db

- файл backup создан

---

## 📦 3. Скачать новую версию

docker pull ghcr.io/mhsanaei/3x-ui:v3.3.1

- Downloaded image

---

## 🔁 4. Переключение

docker stop 3x-ui
docker rm 3x-ui

---

## 🧪 5. Запуск новой версии

docker ps
docker logs 3x-ui --tail 50

Ожидаем:

новая версия активна
старая сохранена

⚠️ ВАЖНО
НЕ удалять /root/x-ui-db

