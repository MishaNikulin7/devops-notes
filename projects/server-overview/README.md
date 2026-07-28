# Обзор VPS-сервера

## Цель

Документирование моего VPS-сервера, который используется для практики Linux-администрирования, Docker, мониторинга и базовой безопасности.

Сервер используется как учебная DevOps-среда для изучения:

- Linux;
- systemd;
- Docker;
- Nginx;
- мониторинга;
- безопасности;
- работы с Git и GitHub.

---

# Информация о сервере

Проверка:

```bash
hostnamectl показывает основную информацию о системе:
 Static hostname: racknerd-ed4bda6
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: a2664927dfc7abac44926bd9b88a25e6
         Boot ID: bb937335a72e4460affa3086a9f7a69b
  Virtualization: kvm
Operating System: AlmaLinux 9.8 (Olive Jaguar)
     CPE OS Name: cpe:/o:almalinux:almalinux:9::baseos
          Kernel: Linux 5.14.0-687.26.1.el9_8.x86_64
    Architecture: x86-64
 Hardware Vendor: Red Hat
  Hardware Model: KVM
Firmware Version: 1.16.0-4.module_el8.9.0+3659+9c8643f3

lscpu показывает информацию о процессоре
CPU(s):                                  2
On-line CPU(s) list:                     0,1
Model name:                              Intel(R) Xeon(R) CPU E5-2690 v4 @ 2.60GHz
BIOS Model name:                         RHEL 7.6.0 PC (i440FX + PIIX, 1996)

df -h показывает использование файловой системы
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        834M     0  834M   0% /dev
tmpfs           854M     0  854M   0% /dev/shm
tmpfs           342M  6.6M  335M   2% /run
/dev/vda1        28G  9.3G   18G  35% /

ip -br a Команда показывает активные сетевые интерфейсы и назначенные IP-адреса
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.*.*.*/24 -/64
br-96f07bdf53ae  UP             172.16.239.1/24 -/64
docker0          UP             172.17.0.1/16 -/64
br-f0de33dcc956  DOWN           172.18.0.1/16
br-35667cdfdc9f  UP             172.19.0.1/16 -/64
br-74bd160b3469  DOWN           172.20.0.1/16
br-838c983249d4  UP             172.16.240.1/24 -/64
br-93c64d998b53  UP             172.16.238.1/24 -/64

docker ps Команда показывает запущенные контейнеры
CONTAINER ID   IMAGE                                             COMMAND                  CREATED         STATUS                  PORTS                                                                                                                                                                                NAMES
d84759813372   ghcr.io/mhsanaei/3x-ui:v3.3.1                     "/app/DockerEntrypoi…"   2 weeks ago     Up 11 hours             0.0.0.0:2053->2053/tcp, [::]:2053->2053/tcp, 0.0.0.0:4443->4443/tcp, [::]:4443->4443/tcp, 0.0.0.0:5443->5443/tcp, [::]:5443->5443/tcp, 0.0.0.0:7443->7443/tcp, [::]:7443->7443/tcp   3x-ui
3b0156b77a3f   zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest   "docker-entrypoint.sh"   13 months ago   Up 11 hours (healthy)   8443/tcp, 0.0.0.0:8081->8080/tcp, [::]:8081->8080/tcp, 0.0.0.0:14443->9443/tcp, [::]:14443->9443/tcp                                                                                 zabbix-docker-zabbix-web-nginx-mysql-1
6b6c3d6ce07f   zabbix/zabbix-server-mysql:alpine-7.0-latest      "/usr/bin/docker-ent…"   13 months ago   Up 11 hours             0.0.0.0:10051->10051/tcp, [::]:10051->10051/tcp                                                                                                                                      zabbix-docker-zabbix-server-1
b0b67de64347   mysql:8.0-oracle                                  "docker-entrypoint.s…"   13 months ago   Up 11 hours             3306/tcp, 33060/tcp                                                                                                                                                                  zabbix-docker-mysql-server-1
