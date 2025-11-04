## Задание 3. Настройка фаервола/Host Firewalling.

### Создание запрещающего правила

Создадим правило, которое будет запрещать подключение по порту 8080 с помощью `iptables`:

```shell
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP
```

Это правило дропает все tcp пакеты, приходящие на порт 8080. Проверим, что политика применилась:

```shell
sudo iptables -L -n -v | grep 8080
```

Вывод:

```
    0     0 DROP       tcp  --  lo     *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080
```

Далее, запустим анализ tcp трафика на порт 8080 утилитой `tcpdump`:

```shell
sudo tcpdump -n -i lo tcp port 8080
```

Вывод:

```shell
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on lo, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

Анализ трафика запущен, поднимем HTTP-сервер и с помощью `curl` отправим несколько запросов (аналогично заданию 1). После этого в анализе tcp трафика появились строки:

```
22:46:25.669315 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 144998007 ecr 0,nop,wscale 7], length 0
22:46:26.721448 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 144999060 ecr 0,nop,wscale 7], length 0
22:46:27.746668 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145000085 ecr 0,nop,wscale 7], length 0
22:46:28.770216 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145001108 ecr 0,nop,wscale 7], length 0
22:46:29.793808 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145002132 ecr 0,nop,wscale 7], length 0
22:46:30.820932 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145003159 ecr 0,nop,wscale 7], length 0
22:46:32.865376 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145005204 ecr 0,nop,wscale 7], length 0
22:46:36.900877 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145009239 ecr 0,nop,wscale 7], length 0
22:46:44.961402 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145017300 ecr 0,nop,wscale 7], length 0
22:47:01.345452 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145033684 ecr 0,nop,wscale 7], length 0
22:47:33.601375 IP 127.0.0.1.45436 > 127.0.0.1.8080: Flags [S], seq 2147353725, win 65495, options [mss 65495,sackOK,TS val 145065940 ecr 0,nop,wscale 7], length 0
```

Из сообщений видно, что клиент (`curl`) пытается соединиться с нашим сервером по TCP протоколу (`Flags [S]` - значит, что отправляет запрос SYN),
но не получает SYN-ACK ответов, это говорит о том, что firewall работает.
