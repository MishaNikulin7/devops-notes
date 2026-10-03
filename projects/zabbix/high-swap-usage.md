# High swap usage - Zabbix alert investigation

## Проблема

Zabbix обнаружил предупреждение:
Linux: High swap space usage

На сервере было замечено повышенное использование Swap.

---

# Что такое Swap

Swap — это область на диске, которую Linux использует как дополнительную виртуальную память.

Когда оперативной памяти (RAM) становится недостаточно, ядро Linux может перемещать редко используемые данные из RAM в Swap.

Swap помогает:

- избежать аварийного завершения процессов;
- сохранить работоспособность системы при нехватке памяти.

Но Swap значительно медленнее RAM, потому что используется диск.

Поэтому постоянное активное использование Swap может говорить о нехватке памяти.

---

# Исходное состояние сервера

Сервер:
OS: AlmaLinux 9.8
RAM: 1.7 GiB
Swap: 1.0 GiB

Запущенные сервисы:

- Docker
- MySQL
- Zabbix Server
- Zabbix Agent
- Nginx
- 3x-ui

Роли сервисов:

- Nginx — веб-сервер и обратный прокси.
- Docker — запуск контейнерных сервисов.
- MySQL — база данных для Zabbix.
- Zabbix Server — сбор и обработка метрик.
- Zabbix Agent — отправка информации о состоянии сервера.
- 3x-ui — сервис управления Xray.

---

# Диагностика

Проверка памяти:

```bash
free -h

Проверка Swap:
swapon --show

Проверка активности Swap:
vmstat 1 5

Результат:
Swap использовался примерно на 500 MiB.
При этом:
si = 0
so = 0
Это означало, что активного обмена данными между RAM и Swap не было.

Что такое vm.swappiness
vm.swappiness — параметр ядра Linux, который определяет насколько активно система использует Swap.
Важно:

Это не процент использования Swap.
Например:
vm.swappiness=60
не означает:
использовать 60% Swap
Это означает:
Linux более охотно переносит неактивные данные памяти в Swap
cat /proc/sys/vm/swappiness
60
Постоянное сохранение

Создан файл:
/etc/sysctl.d/99-swappiness.conf
Содержимое:
vm.swappiness=20
Применение:
sysctl --system

Проверка после перезагрузки

После reboot проверено:

Kernel:
uname -r

Docker:
docker ps

Zabbix Agent:
systemctl status zabbix-agent

Nginx:
systemctl status nginx
Все сервисы успешно запустились.

Результат

До изменения:
Swap used: ~500 MiB
После перезагрузки:
Swap used: ~5 MiB
Zabbix предупреждение:
Linux: High swap space usage
исчезло.

## Как добавить дополнительный swap-файл размером 1 ГБ

Этот способ не изменяет существующий swap-раздел.  
К уже существующему swap просто добавляется ещё один swap-файл.

### 1. Проверить текущий swap

```bash
swapon --show
free -h

2. Создать файл размером 1 ГБ
fallocate -l 1G /swapfile

Проверить:
ls -lh /swapfile

3. Ограничить права доступа
Swap-файл должен быть доступен только root:
chmod 600 /swapfile

Проверить:
ls -l /swapfile

Должно быть примерно так:
-rw------- 1 root root 1.0G ... /swapfile

4. Подготовить файл как swap
mkswap /swapfile

5. Подключить swap
swapon /swapfile

Проверить:
swapon --show

После этого должен появиться /swapfile.
6. Добавить автоподключение после перезагрузки
Добавить строку в /etc/fstab:
echo '/swapfile none swap sw 0 0' >> /etc/fstab

Проверить:
grep '/swapfile' /etc/fstab

Ожидаемый результат:
/swapfile none swap sw 0 0

7. Финальная проверка
free -h
swapon --show

В моём случае получилось:
Swap: 2.0Gi

Активные swap-области:
/dev/vda2 partition 1024M
/swapfile file      1024M

Таким образом, к существующему swap размером 1 ГБ был добавлен ещё один swap-файл размером 1 ГБ.
Итоговый объём swap:
1 ГБ → 2 ГБ

Если понадобится добавить ещё 1 ГБ в будущем
Например, можно создать второй файл:
fallocate -l 1G /swapfile2
chmod 600 /swapfile2
mkswap /swapfile2
swapon /swapfile2
echo '/swapfile2 none swap sw 0 0' >> /etc/fstab

Проверить:
swapon --show
free -h

Тогда суммарный swap станет:
3 ГБ

Важный момент: **не запускай повторно**
```bash
fallocate -l 1G /swapfile
