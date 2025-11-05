## Задание 2. DNS Load Balancing с помощью `dnsmasq`.

Создадим конфиг для сервиса `dnsmasq`. Для этого создадим файл highload-app.conf в директории `/etc/dnsmasq.d`
и добавим в него правила для резолва адресов наших python-серверов:

```shell
sudo touch /etc/dnsmasq.d/highload-app.conf
sudo vi /etc/dnsmasq.d/highload-app.conf
```

Конфиг:

```
address=/my-awesome-highload-app.local/127.0.0.1
address=/my-awesome-highload-app.local/127.0.0.2
```

Для применения конфига перезапустим сервис:

```shell
sudo systemctl restart dnsmasq
```

Проверим работу конфига с помощью утилиты `dig`:

```shell
dig @127.0.0.1 my-awesome-highload-app.local
```

Вывод:

```
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> @127.0.0.1 my-awesome-highload-app.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8595
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;my-awesome-highload-app.local. IN      A

;; ANSWER SECTION:
my-awesome-highload-app.local. 0 IN     A       127.0.0.2
my-awesome-highload-app.local. 0 IN     A       127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Wed Nov 05 05:26:21 MSK 2025
;; MSG SIZE  rcvd: 90
```

Видно, что в секции ANSWER фигурируют оба настроенных адреса для домена. Отключим один из серверов и выполним команду еще раз.

Результат:

```
; <<>> DiG 9.18.39-0ubuntu0.22.04.2-Ubuntu <<>> @127.0.0.1 my-awesome-highload-app.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24116
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;my-awesome-highload-app.local. IN      A

;; ANSWER SECTION:
my-awesome-highload-app.local. 0 IN     A       127.0.0.2
my-awesome-highload-app.local. 0 IN     A       127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Wed Nov 05 05:31:39 MSK 2025
;; MSG SIZE  rcvd: 90
```

Видно, что результат не изменился. Значит DNS сервер продолжит возвращать обе A-записи (127.0.0.1 и 127.0.0.2) независимо от состояния backend серверов. Из-за этого клиенты будут получать неработающий IP-адрес в 50% случаев, что приведет к ошибкам подключения. Пример:

```
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://127.0.0.1:8082
curl: (7) Failed to connect to 127.0.0.1 port 8082 after 0 ms: Connection refused
```