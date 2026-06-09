# 3x-ui

## Цель
Развёртывание панели управления 3x-ui на VPS в Docker.

## Контейнер
- Имя: 3x-ui
- Образ: ghcr.io/mhsanaei/3x-ui:latest
- Статус: Up 13 дней
- Порты: 2053, 4443, 5443, 7443

## Используемые команды
```bash
docker ps
docker logs 3x-ui --tail 50
docker restart 3x-ui
docker exec -it 3x-ui /bin/sh
docker exec -it 3x-ui x-ui status
