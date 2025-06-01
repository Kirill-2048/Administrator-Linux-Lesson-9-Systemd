# Lesson-9-Systemd Part 1.
# Мониторинг лога на предмет наличия ключевого слова

1.	Cоздание файла с конфигурацией для сервиса в директории /etc/default

root@ubuntu:~# cat > /etc/default/watchlog

#Configuration file for my watchlog service

#File and word in that file that we will be monit

WORD="ALERT"

LOG=/var/log/watchlog.log

2.	Создание файла /var/log/watchlog.log

root@ubuntu:~# cat > /var/log/watchlog.log

#File /var/log/watchlog.log

#This file contains key word

#Key word is ALERT

3.	Создание скрипта

root@ubuntu:~# touch /opt/watchlog.sh

root@ubuntu:~# nano /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

root@ubuntu:~# chmod +x /opt/watchlog.sh

4.	Создание юнита для сервиса

root@ubuntu:~# cat > /etc/systemd/system/watchlog.service

[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

5.	Создание юнита для таймера

root@ubuntu:~# cat > /etc/systemd/system/watchlog.timer

[Unit]
Description=Run watchlog script every 30 second
[Timer]
#Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target

6.	Проверка результата

root@ubuntu:~# systemctl start watchlog.service
root@ubuntu:~# systemctl start watchlog.timer
root@ubuntu:~# grep -R "I found word" /var/log/syslog

Вывод:

/var/log/syslog:2025-06-01T09:03:25.804639+00:00 ubuntu root: 2025-06-01 09:03:25: I found word, Master!
/var/log/syslog:2025-06-01T09:06:08.172767+00:00 ubuntu root: 2025-06-01 09:06:08: I found word, Master!

Сообщение получено.
