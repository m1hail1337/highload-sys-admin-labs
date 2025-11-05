## Задание 3. Балансировка Layer 4 с помощью IPVS.

Создадим dummy1 интерфейс с адресом `192.168.100.1/32`:

```shell
sudo ip link add dummy1 type dummy
sudo ip addr add 192.168.100.1/32 dev dummy1
sudo ip link set dummy1 up
```

Проверка создания:

```
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ ip a | grep dummy
12: dummy1: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 192.168.100.1/32 scope global dummy1
```

Создадим VS для наших реальных серверов:

```shell
sudo ipvsadm -A -t 192.168.100.1:80 -s rr

sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8081 -m
sudo ipvsadm -a -t 192.168.100.1:80 -r 127.0.0.1:8082 -m
```

Проверим, что VS создался правильно:

```shell
sudo ipvsadm -ln
```

Вывод:

```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.100.1:80 rr
  -> 127.0.0.1:8081               Masq    1      0          0         
  -> 127.0.0.1:8082               Masq    1      0          0 
```

Тестируем виртуальный сервер curl-запросами:

```
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h2>*** Response from Backend Server 2 ***</h2>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h1>Response from Backend Server 1</h1>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h2>*** Response from Backend Server 2 ***</h2>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h1>Response from Backend Server 1</h1>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h2>*** Response from Backend Server 2 ***</h2>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://192.168.100.1
<h1>Response from Backend Server 1</h1>
```

Видно, что сервер обращается поочереди к 2 серверам (принцип round-robin). Посмотрим статистику `ipvsadm`:

```shell
sudo ipvsadm -ln --stats
```

Вывод:

```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.100.1:80                    6       38       35     2486     3248
  -> 127.0.0.1:8081                      3       19       17     1243     1586
  -> 127.0.0.1:8082                      3       19       18     1243     1662
```

Видим, что кол-во connections одинаковое.

