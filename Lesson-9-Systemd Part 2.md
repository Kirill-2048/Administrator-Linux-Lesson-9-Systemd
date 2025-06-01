## Lesson-9-Systemd Part 2.

## Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта.

1.	Устанавливаем spawn-fcgi и необходимые для него пакеты.

root@ubuntu:~# apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y

2.	Создаем файл с настройками для будущего сервиса /etc/spawn-fcgi/fcgi.conf.

root@ubuntu:/etc# touch /etc/spawn-fcgi/fcgi.conf

root@ubuntu:/etc# nano /etc/spawn-fcgi/fcgi.conf

SOCKET=/var/run/php-fcgi.sock

OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

3.	Создаем юнит-файл.

root@ubuntu:/# touch /etc/systemd/system/spawn-fcgi.service

root@ubuntu:/# nano /etc/systemd/system/spawn-fcgi.service

[Unit]

Description=Spawn-fcgi startup service by Otus

After=network.target

[Service]

Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]

WantedBy=multi-user.target

4.	Проверка работоспособности.

root@ubuntu:/# systemctl start spawn-fcgi

root@ubuntu:/# systemctl status spawn-fcgi

Вывод:

  spawn-fcgi.service - Spawn-fcgi startup service by Otus
  
     Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; preset: enabled)
     
     Active: active (running) since Sun 2025-06-01 15:37:32 UTC; 1min 0s ago
     
   Main PID: 16940 (php-cgi)
      Tasks: 33 (limit: 4599)
     Memory: 14.4M (peak: 14.7M)
        CPU: 19ms
     CGroup: /system.slice/spawn-fcgi.service
             ├─16940 /usr/bin/php-cgi
             ├─16941 /usr/bin/php-cgi
             ├─16942 /usr/bin/php-cgi
             ├─16943 /usr/bin/php-cgi
             ├─16944 /usr/bin/php-cgi
             ├─16945 /usr/bin/php-cgi
             ├─16946 /usr/bin/php-cgi
             ├─16947 /usr/bin/php-cgi
             ├─16948 /usr/bin/php-cgi
             ├─16949 /usr/bin/php-cgi
             ├─16950 /usr/bin/php-cgi
             ├─16951 /usr/bin/php-cgi
             ├─16952 /usr/bin/php-cgi
             ├─16953 /usr/bin/php-cgi
             ├─16954 /usr/bin/php-cgi
             ├─16955 /usr/bin/php-cgi
             ├─16956 /usr/bin/php-cgi
             ├─16957 /usr/bin/php-cgi
             ├─16958 /usr/bin/php-cgi
             ├─16959 /usr/bin/php-cgi
             ├─16960 /usr/bin/php-cgi
             ├─16961 /usr/bin/php-cgi
             ├─16962 /usr/bin/php-cgi
             ├─16963 /usr/bin/php-cgi
             ├─16964 /usr/bin/php-cgi
             ├─16965 /usr/bin/php-cgi
             ├─16966 /usr/bin/php-cgi
             ├─16967 /usr/bin/php-cgi
             ├─16968 /usr/bin/php-cgi
             ├─16969 /usr/bin/php-cgi
             ├─16970 /usr/bin/php-cgi
             ├─16971 /usr/bin/php-cgi
             └─16972 /usr/bin/php-cgi

июн 01 15:37:32 ubuntu systemd[1]: Started spawn-fcgi.service - Spawn-fcgi startup service by Otus.
