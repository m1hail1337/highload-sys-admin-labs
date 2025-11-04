## Задание 2. Динамическая маршрутизация с BIRD.

### Создание dummy-интерфейса

Создадим dummy-интерфейс с адресом `192.168.14.88/32` и названием `service_0`:

```shell
sudo modprobe dummy
sudo ip link add service_0 type dummy
sudo ip addr add 192.168.14.88/32 dev service_0
sudo ip link set service_0 up
```

Посмотрим последние записи вывода команды `ip a`:

```shell
ip a | tail -6
```

Вывод:

```
12: service_0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 22:c3:81:63:28:f8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.14.88/32 scope global service_0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c3:81ff:fe63:28f8/64 scope link 
       valid_lft forever preferred_lft forever
```

Видно, что интерфейс service_0 был создан успешно.

### Анонсирование адреса при помощи BIRD

Напишем конфиг для `BIRD` (в файле `/etc/bird/bird.conf`), чтобы анонсировать адреса из подсети `192.168.14.0/24`, но только если у него будет маска подсети `/32` и имя будет начинаться на `service_`:

```shell
router id 192.168.14.88;

protocol device device1 { scan time 10; }

protocol direct direct1 {
  interface "service_*";  # Разрешаем только для начинающихся с service_
  ipv4 { import all; };
}

protocol kernel kernel1 {
  persist;
  scan time 20;
  ipv4 { import none; export all; }
}

protocol rip rip1 {
  interface "enp0s3" {
    version 2;
    metric 1;
    authentication none;
  };
  ipv4 {
    import all;
    export filter {
      if (source = RTS_DEVICE) && (net ~ 192.168.14.0/24) && (net.len = 32) then  # Наше условие на подсеть + маску
        accept;
      else
        reject;
    };
  };
}
```

Аналогично, добавим еще несколько интерфейсов, чтобы проверить работу `BIRD`:

```shell
sudo ip link add service_1 type dummy
sudo ip addr add 192.168.14.1/30 dev service_1
sudo ip link add service_2 type dummy
sudo ip addr add 192.168.10.4/32 dev service_2
sudo ip link add srv_1 type dummy
sudo ip addr add 192.168.14.4/32 dev srv_1

sudo ip link set service_1 up
sudo ip link set service_2 up
sudo ip link set srv_1 up
```

С помощью утилиты `tcpdump` убедимся в правильной конфигурации фильтра:

```shell
sudo tcpdump -vv -s0 -n -i enp0s3 udp port 520
```

Вывод:

```
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:33:29.588231 IP (tos 0xc0, ttl 1, id 21453, offset 0, flags [none], proto UDP (17), length 52)
    10.0.2.15.520 > 224.0.0.9.520: [bad udp cksum Oxec49 -> 0x0e74!]
        RIPv2, Request, length: 24, routes: 1 or less
          AFI 0,    0.0.0.0/0, tag 0x0000, metric: 16, next-hop: self
        0x0000: 0102 0000 0000 0000 0000 0000 0000 0000
        0x0010: 0000 0000 0000 0010
22:33:29.588247 IP (tos 0xc0, ttl 1, id 21454, offset 0, flags [none], proto UDP (17), length 52)
    10.0.2.15.520 > 224.0.0.9.520: [bad udp cksum Oxec49 > 0x3e80!]
        RIPV2, Response, length: 24, routes: 1 or less
          AFI IPv4, 192.168.14.88/32, tag 0x0000, metric: 1, next-hop: self
        0x0000: 0202 0000 0002 0000 c0a8 0e58 ffff ffff
        0x0010: 0000 0000 0000 0001
22:33:48.614852 IP (tos 0xc0, ttl 1, id 34943, offset 0, flags [none], proto UDP (17), length 52)
    10.0.2.15.520 > 224.0.0.9.520: [bad udp cksum Oxec49 > 0x3e80!]
        RIPV2, Response, length: 24, routes: 1 or less
          AFI IPv4, 192.168.14.88/32, tag 0x0000, metric: 1, next-hop: self
        0x0000: 0202 0000 0002 0000 c0a8 0e58 ffff ffff
        0x0010: 0000 0000 0000 0001
22:34:18.568719 IP (tos 0xc0, ttl 1, id 57013, offset 0, flags [none], proto UDP (17), length 52)
    10.0.2.15.520 > 224.0.0.9.520: [bad udp cksum Oxec49 > 0x3e80!]
        RIPV2, Response, length: 24, routes: 1 or less
          AFI IPv4, 192.168.14.88/32, tag 0x0000, metric: 1, next-hop: self
        0x0000: 0202 0000 0002 0000 c0a8 0e58 ffff ffff
        0x0010: 0000 0000 0000 0001
^C
4 packets captured
4 packets received by filter
O packets dropped by kernel
```

Из вывода команды видно, что анонсируется только интерфейс `192.168.14.88/32` (service_0), другие не подходят под правило фильтрации т.к.:
- 192.168.14.1/30 (service_1) - маска /30 вместо /32
- 192.168.10.4/32 (service_2) - не из подсети 192.168.14.0/24
- 192.168.14.4/32 (srv_1) - имя не начинается на service_

