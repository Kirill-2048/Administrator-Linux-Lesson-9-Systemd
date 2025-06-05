## Lesson-9-Systemd Part 3.
## Запуск нескольких инстансов nginx с разными конфигурационными файлами одновременно.

1.	Создание нового Unit для работы с шаблонами (/etc/systemd/system/nginx@.service).

root@ubuntu:~# touch /etc/systemd/system/nginx@.service

root@ubuntu:~# nano /etc/systemd/system/nginx@.service

	[Unit]

	Description=A high performance web server and a reverse proxy server

	Documentation=man:nginx(8)

	After=network.target nss-lookup.target

	[Service]

	Type=forking

	PIDFile=/run/nginx-%I.pid

	ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'

	ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'

	ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload

	ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid

	TimeoutStopSec=5

	KillMode=mixed

	[Install]

	WantedBy=multi-user.target


2.	Создание файлов конфигурации /etc/nginx/nginx-first.conf, /etc/nginx/nginx-second.conf.


root@ubuntu:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf

root@ubuntu:~# cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf

root@ubuntu:~# nano /etc/nginx/nginx-first.conf

pid /run/nginx-first.pid;

http {

…

	server {
 
		listen 9001;
  
	}
 
#include /etc/nginx/sites-enabled/*;

root@ubuntu:~# nano /etc/nginx/nginx-second.conf

pid /run/nginx-second.pid;

http {

…

	server {
 
		listen 9002;
  
	}
 
#include /etc/nginx/sites-enabled/*;

3.	Проверка синтаксиса.

nginx: the configuration file /etc/nginx/nginx-second.conf syntax is ok

nginx: configuration file /etc/nginx/nginx-second.conf test is successful

root@ubuntu:/# nginx -t -c /etc/nginx/nginx-first.conf

nginx: the configuration file /etc/nginx/nginx-first.conf syntax is ok

nginx: configuration file /etc/nginx/nginx-first.conf test is successful

4.	Проверка работоспособности.

root@ubuntu:/# systemctl start nginx@first

root@ubuntu:/# systemctl status nginx@first

● nginx@first.service - A high performance web server and a reverse proxy server

     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     
     Active: active (running) since Thu 2025-06-05 13:31:26 UTC; 17s ago
     
      …
      
июн 05 13:31:26 ubuntu systemd[1]: Started nginx@first.service - A high performance web server and a reverse proxy server.

root@ubuntu:/# systemctl start nginx@second

root@ubuntu:/# systemctl status nginx@second

● nginx@second.service - A high performance web server and a reverse proxy server

     Loaded: loaded (/etc/systemd/system/nginx@.service; disabled; preset: enabled)
     
     Active: active (running) since Thu 2025-06-05 13:32:10 UTC; 8s ago
     
       ….

июн 05 13:32:10 ubuntu systemd[1]: Started nginx@second.service - A high performance web server and a reverse proxy server.

5.	Проверка портов.

root@ubuntu:/# ss -tnulp | grep nginx

tcp   LISTEN 0      511                 0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=37863,fd=5),("nginx",pid=37862,fd=5),("nginx",pid=37861,fd=5))

tcp   LISTEN 0      511                 0.0.0.0:9001      0.0.0.0:*    users:(("nginx",pid=47506,fd=5),("nginx",pid=47505,fd=5),("nginx",pid=47504,fd=5))

tcp   LISTEN 0      511                 0.0.0.0:9002      0.0.0.0:*    users:(("nginx",pid=47783,fd=5),("nginx",pid=47782,fd=5),("nginx",pid=47781,fd=5))

tcp   LISTEN 0      511                    [::]:80           [::]:*    users:(("nginx",pid=37863,fd=6),("nginx",pid=37862,fd=6),("nginx",pid=37861,fd=6))


Вывод:

Три сервиса запущены одновременно: nginx, nginx@first, nginx@second.
