# Домашнее задание 2. WEB. Балансировка нагрузки


## Задание 1. Server Pool

### Шаг 1.1. Каталоги и index.html

```bash
mkdir -p ~/backend1 ~/backend2
echo '<h1>Response from Backend Server 1</h1>' > ~/backend1/index.html
echo '<h2>*** Response from Backend Server 2 ***</h2>' > ~/backend2/index.html
head ~/backend1/index.html ~/backend2/index.html


Вывод
yuri@ubuntu-hp:~$ head ~/backend1/index.html ~/backend2/index.html
==> /home/yuri/backend1/index.html <==
<h1>Response from Backend Server 1</h1>

==> /home/yuri/backend2/index.html <==
<h2>*** Response from Backend Server 2 ***</h2>

```

### Шаг 1.2. Запуск серверов в отдельных терминалах

```bash
yuri@ubuntu-hp:~$ cd ~/backend1 && python3 -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
```

```bash
yuri@ubuntu-hp:~$ cd ~/backend2 && python3 -m http.server 8082
Serving HTTP on 0.0.0.0 port 8082 (http://0.0.0.0:8082/) ...
```

### Шаг 1.3. Проверка

```bash
yuri@ubuntu-hp:~$ ss -ltnp | grep -E ':808[12]'
LISTEN 0      5                 0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=9712,fd=3))     
LISTEN 0      5                 0.0.0.0:8081       0.0.0.0:*    users:(("python3",pid=9671,fd=3))     


yuri@ubuntu-hp:~$ curl -s http://127.0.0.1:8081
<h1>Response from Backend Server 1</h1>

yuri@ubuntu-hp:~$ curl -s http://127.0.0.1:8082
<h2>*** Response from Backend Server 2 ***</h2>
```

Оба сервера слушают свои порты и отдают каждый свою страницу

## Задание 2. DNS Load Balancing (dnsmasq)

### Шаг 2.1. Конфиг

dnsmasq слушает только 127.0.0.1, поскольку на 127.0.0.53:53 работает systemd-resolved.

```bash
yuri@ubuntu-hp:~$ sudo tee /etc/dnsmasq.d/highload.conf > /dev/null <<'EOF'
listen-address=127.0.0.1
bind-interfaces
no-resolv
no-hosts
host-record=my-awesome-highload-app.local,127.0.0.1
host-record=my-awesome-highload-app.local,127.0.0.2
log-queries
EOF

yuri@ubuntu-hp:~$ sudo systemctl restart dnsmasq
yuri@ubuntu-hp:~$ systemctl status dnsmasq.service
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/lib/systemd/system/dnsmasq.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-10-25 23:27:26 MSK; 5s ago
    Process: 85716 ExecStartPre=/etc/init.d/dnsmasq checkconfig (code=exited, status=0/SUCCESS)
    Process: 85724 ExecStart=/etc/init.d/dnsmasq systemd-exec (code=exited, status=0/SUCCESS)
    Process: 85734 ExecStartPost=/etc/init.d/dnsmasq systemd-start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 85732 (dnsmasq)
      Tasks: 1 (limit: 18813)
     Memory: 604.0K
        CPU: 24ms
     CGroup: /system.slice/dnsmasq.service
             └─85732 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -7 /etc/dnsmasq.d,.dpkg-dist,.dpkg-old,.>
```

### Шаг 2.2. dig

```bash
yuri@ubuntu-hp:~$ dig @127.0.0.1 my-awesome-highload-app.local

; <<>> DiG 9.18.39-0ubuntu0.22.04.7-Ubuntu <<>> @127.0.0.1 my-awesome-highload-app.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17979
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;my-awesome-highload-app.local.	IN	A

;; ANSWER SECTION:
my-awesome-highload-app.local. 0 IN	A	127.0.0.1
my-awesome-highload-app.local. 0 IN	A	127.0.0.2

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Sun Oct 25 23:28:14 MSK 2026
;; MSG SIZE  rcvd: 90

```

* Видим `status: NOERROR`, флаг `aa` - dnsmasq отвечает авторитетно из своего конфига.
* `ANSWER: 2`, то есть заданы обе A-записи, 127.0.0.1 и 127.0.0.2.
* TTL 0, то есть локальные записи dnsmasq по умолчанию не кешируются.

### Шаг 2.3. Отказ backend2

Останавливаем backend2 и повторяем запрос:

```bash
yuri@ubuntu-hp:~$ cd ~/backend2 && python3 -m http.server 8082
Serving HTTP on 0.0.0.0 port 8082 (http://0.0.0.0:8082/) ...
127.0.0.1 - - [25/Oct/2026 23:10:57] "GET / HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.

yuri@ubuntu-hp:~/backend2$ 
```

```bash
yuri@ubuntu-hp:~$ curl -s --max-time 2 http://127.0.0.1:8082 || echo "backend2 down"
backend2 down

yuri@ubuntu-hp:~$ dig @127.0.0.1 my-awesome-highload-app.local

; <<>> DiG 9.18.39-0ubuntu0.22.04.7-Ubuntu <<>> @127.0.0.1 my-awesome-highload-app.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36575
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;my-awesome-highload-app.local.	IN	A

;; ANSWER SECTION:
my-awesome-highload-app.local. 0 IN	A	127.0.0.1
my-awesome-highload-app.local. 0 IN	A	127.0.0.2

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Sun Oct 25 23:31:31 MSK 2026
;; MSG SIZE  rcvd: 90

```

DNS-записи не изменились, поскольку dnsmasq не проверяет доступность backend-ов и продолжает отдавать адрес упавшего сервера. Часть клиентов получит ошибку соединения; некоторые клиенты попробуют второй адрес, но это зависит от клиента. Даже при удалении записи старый ответ может жить в кешах до истечения TTL.

Поэтому DNS-балансировку можно дополнять health-check-ами и балансировкой L4  /L7.

## Задание 3. L4 балансировка (IPVS)

### Шаг 3.1. dummy-интерфейс

```bash
yuri@ubuntu-hp:~$ sudo ip link add dummy1 type dummy
yuri@ubuntu-hp:~$ sudo ip addr add 192.168.100.1/32 dev dummy1
yuri@ubuntu-hp:~$ sudo ip link set dummy1 up

yuri@ubuntu-hp:~$ ip -br addr show dummy1
dummy1           UNKNOWN        192.168.100.1/32 fe80::90d3:9aff:fec8:abd9/64 
```

### Шаг 3.2. Виртуальный сервис

```bash
yuri@ubuntu-hp:~$ sudo ipvsadm -A -t 192.168.100.1:80 -s rr
yuri@ubuntu-hp:~$ sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8081 -m
yuri@ubuntu-hp:~$ sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8082 -m

yuri@ubuntu-hp:~$ sudo ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.100.1:80 rr
  -> 127.0.0.1:8081               Masq    1      0          0         
  -> 127.0.0.1:8082               Masq    1      0          0         

```

-m (NAT) нужен, поскольку порт VIP (80) отличается от портов backend-ов.

### Шаг 3.3. Проверка балансировки

```bash
Обнуляем счетчики
yuri@ubuntu-hp:~$ sudo ipvsadm -Z


yuri@ubuntu-hp:~$ for i in $(seq 1 10); do curl -s http://192.168.100.1; done
<h1>Response from Backend Server 1</h1>
<h1>Response from Backend Server 1</h1>
<h1>Response from Backend Server 1</h1>
<h1>Response from Backend Server 1</h1>
<h1>Response from Backend Server 1</h1>


yuri@ubuntu-hp:~$ sudo ipvsadm -Ln --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.100.1:80                   10       39       33     2493     2826
  -> 127.0.0.1:8081                      5       34       28     2193     2626
  -> 127.0.0.1:8082                      5        5        5      300      200

```

Ответы чередуются, у каждого real server `Conns = 5`. IPVS балансирует TCP-соединения в ядре, не разбирая HTTP, и не проверяет доступность backend-ов.

## Задание 4. L7 балансировка (nginx)

### Шаг 4.1. Конфиг

```bash
yuri@ubuntu-hp:~$ sudo rm -f /etc/nginx/sites-enabled/default
yuri@ubuntu-hp:~$ sudo tee /etc/nginx/conf.d/highload.conf > /dev/null <<'EOF'
log_format upstreamlog '"$request" status=$status upstream=$upstream_addr upstream_status=$upstream_status';

upstream backend_pool {
    zone backend_pool 64k;
    server 127.0.0.1:8081 max_fails=7 fail_timeout=60s;
    server 127.0.0.1:8082 backup;
}

server {
    listen 127.0.0.1:8888;
    access_log /var/log/nginx/highload_access.log upstreamlog;
    error_log  /var/log/nginx/highload_error.log warn;

    location / {
        proxy_pass http://backend_pool;
        proxy_set_header X-high-load-test 123;
        proxy_connect_timeout 1s;
    }
}
EOF

yuri@ubuntu-hp:~$ sudo nginx -t && sudo systemctl reload nginx
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


yuri@ubuntu-hp:~$ ss -ltnp | grep 8888
LISTEN 0      511             127.0.0.1:8888       0.0.0.0:*                                          
yuri@ubuntu-hp:~$ 
```

* `backup`, то есть 8082 работает только при недоступности 8081.
* `max_fails=7`, то есть 8081 отключается из выдачи после 7 неудачных попыток.
* `zone` - общий счётчик неудач для всех worker-ов nginx.

### Шаг 4.2. Переключение на backup

Останавливаем backend1:
```bash
yuri@ubuntu-hp:~$ cd ~/backend1 && python3 -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
127.0.0.1 - - [25/Oct/2026 23:10:45] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [25/Oct/2026 23:40:01] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [25/Oct/2026 23:40:01] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [25/Oct/2026 23:40:01] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [25/Oct/2026 23:40:01] "GET / HTTP/1.1" 200 -
192.168.100.1 - - [25/Oct/2026 23:40:01] "GET / HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.

yuri@ubuntu-hp:~/backend1$
```

Запускаем backend2:
```bash
yuri@ubuntu-hp:~/backend2$ python3 -m http.server 8082
Serving HTTP on 0.0.0.0 port 8082 (http://0.0.0.0:8082/) ...

```


```bash
Очистим логи
sudo truncate -s 0 /var/log/nginx/highload_access.log /var/log/nginx/highload_error.log

yuri@ubuntu-hp:~$ for i in $(seq 1 10); do curl -s http://127.0.0.1:8888; done
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>
<h2>*** Response from Backend Server 2 ***</h2>


yuri@ubuntu-hp:~$ sudo cat -n /var/log/nginx/highload_access.log
     1	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     2	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     3	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     4	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     5	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     6	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     7	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8081, 127.0.0.1:8082 upstream_status=502, 200
     8	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8082 upstream_status=200
     9	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8082 upstream_status=200
    10	"GET / HTTP/1.1" status=200 upstream=127.0.0.1:8082 upstream_status=200

yuri@ubuntu-hp:~$ sudo grep -cE 'connect\(\) failed' /var/log/nginx/highload_error.log
7

yuri@ubuntu-hp:~$ sudo grep 'temporarily disabled' /var/log/nginx/highload_error.log
2026/10/25 23:48:11 [warn] 92597#92597: *19 upstream server temporarily disabled while connecting to upstream, client: 127.0.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://127.0.0.1:8081/", host: "127.0.0.1:8888"
```

Видим, что запросы 1-7 идут сначала в 8081 (`upstream_status=502, 200`), после ошибки повторяются на 8082. После 7-й ошибки 8081 помечен недоступным (`temporarily disabled`), запросы 8-10 идут сразу в 8082.

### Шаг 4.3. tshark

Запустим backend1 и backend2.  Также в одном терминале tshark запустим, в другом отправим запрос через curl.

```bash
yuri@ubuntu-hp:~$ sudo tshark -i lo -f "tcp port 8888 or tcp port 8081" -Y http -O http
Running as user "root" and group "root". This could be dangerous.
Capturing on 'Loopback: lo'
 ** (tshark:93139) 23:52:43.562477 [Main MESSAGE] -- Capture started.
 ** (tshark:93139) 23:52:43.562544 [Main MESSAGE] -- File: "/tmp/wireshark_loIG7GX3.pcapng"
```

```bash
yuri@ubuntu-hp:~$ curl -s http://127.0.0.1:8888
<h1>Response from Backend Server 1</h1>
```

Вывод tshark
```bash
yuri@ubuntu-hp:~$ sudo tshark -i lo -f "tcp port 8888 or tcp port 8081" -Y http -O http

Running as user "root" and group "root". This could be dangerous.
Capturing on 'Loopback: lo'
 ** (tshark:93139) 23:52:43.562477 [Main MESSAGE] -- Capture started.
 ** (tshark:93139) 23:52:43.562544 [Main MESSAGE] -- File: "/tmp/wireshark_loIG7GX3.pcapng"
Frame 4: 144 bytes on wire (1152 bits), 144 bytes captured (1152 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 50192, Dst Port: 8888, Seq: 1, Ack: 1, Len: 78
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.1\r\n]
            [GET / HTTP/1.1\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.1
    Host: 127.0.0.1:8888\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://127.0.0.1:8888/]
    [HTTP request 1/1]

Frame 9: 184 bytes on wire (1472 bits), 184 bytes captured (1472 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 52498, Dst Port: 8081, Seq: 1, Ack: 1, Len: 118
Hypertext Transfer Protocol
    GET / HTTP/1.0\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.0\r\n]
            [GET / HTTP/1.0\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.0
    X-high-load-test: 123\r\n
    Host: backend_pool\r\n
    Connection: close\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://backend_pool/]
    [HTTP request 1/1]

Frame 13: 106 bytes on wire (848 bits), 106 bytes captured (848 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8081, Dst Port: 52498, Seq: 187, Ack: 119, Len: 40
[2 Reassembled TCP Segments (226 bytes): #11(186), #13(40)]
Hypertext Transfer Protocol
    HTTP/1.0 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.0 200 OK\r\n]
            [HTTP/1.0 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.0
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: SimpleHTTP/0.6 Python/3.10.13\r\n
    Date: Sun, 25 Oct 2026 20:53:44 GMT\r\n
    Content-type: text/html\r\n
    Content-Length: 40\r\n
        [Content length: 40]
    Last-Modified: Sun, 25 Oct 2026 20:07:38 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002702998 seconds]
    [Request in frame: 9]
    [Request URI: http://backend_pool/]
    File Data: 40 bytes
Line-based text data: text/html (1 lines)

Frame 18: 308 bytes on wire (2464 bits), 308 bytes captured (2464 bits) on interface lo, id 0
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
Transmission Control Protocol, Src Port: 8888, Dst Port: 50192, Seq: 1, Ack: 79, Len: 242
Hypertext Transfer Protocol
    HTTP/1.1 200 OK\r\n
        [Expert Info (Chat/Sequence): HTTP/1.1 200 OK\r\n]
            [HTTP/1.1 200 OK\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Response Version: HTTP/1.1
        Status Code: 200
        [Status Code Description: OK]
        Response Phrase: OK
    Server: nginx/1.18.0 (Ubuntu)\r\n
    Date: Sun, 25 Oct 2026 20:53:44 GMT\r\n
    Content-Type: text/html\r\n
    Content-Length: 40\r\n
        [Content length: 40]
    Connection: keep-alive\r\n
    Last-Modified: Sun, 25 Oct 2026 20:07:38 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.002989807 seconds]
    [Request in frame: 4]
    [Request URI: http://127.0.0.1:8888/]
    File Data: 40 bytes
Line-based text data: text/html (1 lines)

```

Видим информацию о пакетах, отправленных при запросе клиента и ответе сервера.

Пакет 1. curl --> nginx (порт 8888)
- Запись с Host: 127.0.0.1:8888\r\n

Пакет 2. nginx --> backend1 (порт 8081)
- Тот же запрос, но его уже отправил nginx.
- Появился X-high-load-test: 123.
- HTTP/1.0 и Connection: close - настройки nginx по умолчанию для запросов к upstream.
- Host: backend_pool - по умолчанию nginx подставляет имя upstream-группы.

Пакет 3. backend1 --> nginx (ответ)
- Server: SimpleHTTP/0.6 Python/3.10.12\r\n (видим, что отвечает python-сервер)

Пакет 4. nginx --> curl (ответ клиенту)
- Тело то же, но заголовки от nginx (Server: nginx, версия HTTP/1.1.)
- Server: nginx/1.18.0 (Ubuntu)\r\n


Итог

В запросе клиента (на порт 8888) заголовка `X-high-load-test` нет, в запросе nginx к 8081 есть, поскольку его добавляет nginx. Также видим два отдельных TCP-соединения (клиент-nginx и nginx-backend), в отличие от IPVS.
