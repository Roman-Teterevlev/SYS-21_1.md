### Roman Teterevlev
### Disaster recovery и Keepalived

### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### Ответ:


------


### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Ответ:

```
#!/bin/bash
HOST="127.0.0.1"
PORT="80"
FILE=/var/www/html/index.nginx-debian.html

exec 3> /dev/tcp/${HOST}/${PORT}
if [ $? -eq 0 ] && [ -f "$FILE" ] ; then exit 0 ; else exit 1 ; fi

```

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check {
    script "/etc/keepalived/check.sh"
    interval 3
}

vrrp_instance VI_1 {
    state MASTER
    interface ens3
    virtual_router_id 10
    priority 100
    advert_int 1

    virtual_ipaddress {
            172.23.10.10/17
        }

        track_script {
           check
        }
}

```

```
global_defs {
    script_user root
    enable_script_security
}

vrrp_script check {
    script "/etc/keepalived/check.sh"
    interval 3
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens3
    virtual_router_id 10
    priority 50
    advert_int 1

    virtual_ipaddress {
            172.23.10.10/17
        }

        track_script {
           check
        }
}

```

------


