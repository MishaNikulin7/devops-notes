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

cp /root/x-ui-db/x-ui.db \
/root/x-ui-db/backups/x-ui.db.$(date +%F).bak

- файл backup создан

После выполнения:

```text
/root/x-ui-db/backups/
└── x-ui.db.2026-07-09.bak
```

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

## Восстановление из резервной копии

### 1. Остановить контейнер

```bash
docker stop 3x-ui
```

### 2. Удалить контейнер

```bash
docker rm 3x-ui
```

### 3. Восстановить базу

```bash
cp /root/x-ui-db/backups/x-ui.db.2026-07-09.bak \
/root/x-ui-db/x-ui.db
```

### 4. Запустить контейнер

```bash
docker run -d \
  --name 3x-ui \
  -v /root/x-ui-db:/etc/x-ui \
  -p 2053:2053 \
  -p 4443:4443 \
  -p 5443:5443 \
  -p 7443:7443 \
  --restart unless-stopped \
  ghcr.io/mhsanaei/3x-ui:v3.3.1
```

### 5. Проверить состояние

```bash
docker ps | grep 3x-ui
docker logs 3x-ui --tail 30
```

Убедиться:

- панель открывается;
- VPN подключается;
- пользователи и inbound восстановлены.

## ⚠️ Важно

Не удалять рабочие файлы SQLite:

```text
x-ui.db
x-ui.db-shm
x-ui.db-wal
system_metrics.gob
```

Резервные копии хранить только в каталоге:

```text
/root/x-ui-db/backups/
```

