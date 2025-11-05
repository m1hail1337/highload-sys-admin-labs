## Задание 4. Балансировка L7 с помощью NGINX.

Создадим конфиг для nginx в файле `/etc/nginx/sites-available/highload-app` :

```nginx configuration
upstream backend_pool {
    server 127.0.0.1:8081 max_fails=7 fail_timeout=30s;
    server 127.0.0.1:8082 backup;
}

server {
    listen 127.0.0.1:8888;
    
    location / {
        proxy_pass http://backend_pool;
        proxy_set_header X-high-load-test 123;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 7;
        proxy_next_upstream_timeout 30s;
        
        proxy_connect_timeout 1s;
        proxy_read_timeout 1s;
    }
}
```

В первом блоке мы установили, что сервер с портом 8081 будет активным, а с 8082 - бэкапом. Это значит, что все запросы будут идти в 1,
при условии, что он работает. Когда сервер 7 раз (`max_fails=7`) не обработает запрос, весь трафик в следующие полминуты (`fail_timeout=30s`) пойдет на второй сервер.

Далее мы указываем, где будет расположен сервер nginx и какой Header добавлять при запросе через него.

Активируем конфигурацию: делаем ссылку в enabled-директорию + перезапуск сервиса:

```shell
sudo ln -sf /etc/nginx/sites-available/highload-app /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

Вывод:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Выполним запрос через `curl`:

```
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://127.0.0.1:8888
<h1>Response from Backend Server 1</h1>
```

Повторим запрос и получим более подробный анализ в `tshark`:


```shell
sudo tshark -i lo -V -Y "http" -f "tcp port 8081"
```

Анализируем порт 8081, чтобы увидеть проставленный NGINX Header (он отправляется от NGINX серверу,
поэтому при анализе трафика на 8888 мы его не увидим). Вывод:

```
Capturing on 'Loopback: lo'
 ** (tshark:128001) 06:06:41.194825 [Main MESSAGE] -- Capture started.
 ** (tshark:128001) 06:06:41.194864 [Main MESSAGE] -- File: "/tmp/wireshark_loQ69EF3.pcapng"
Frame 4: 184 bytes on wire (1472 bits), 184 bytes captured (1472 bits) on interface lo, id 0
    Interface id: 0 (lo)
        Interface name: lo
    Encapsulation type: Ethernet (1)
    Arrival Time: Nov  5, 2025 06:06:43.941659530 MSK
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1762312003.941659530 seconds
    [Time delta from previous captured frame: 0.000013717 seconds]
    [Time delta from previous displayed frame: 0.000000000 seconds]
    [Time since reference or first frame: 0.000026914 seconds]
    Frame Number: 4
    Frame Length: 184 bytes (1472 bits)
    Capture Length: 184 bytes (1472 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:tcp:http]
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
    Destination: 00:00:00_00:00:00 (00:00:00:00:00:00)
        Address: 00:00:00_00:00:00 (00:00:00:00:00:00)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: 00:00:00_00:00:00 (00:00:00:00:00:00)
        Address: 00:00:00_00:00:00 (00:00:00:00:00:00)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 170
    Identification: 0x7469 (29801)
    Flags: 0x40, Don't fragment
        0... .... = Reserved bit: Not set
        .1.. .... = Don't fragment: Set
        ..0. .... = More fragments: Not set
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 64
    Protocol: TCP (6)
    Header Checksum: 0xc7e2 [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 127.0.0.1
    Destination Address: 127.0.0.1
Transmission Control Protocol, Src Port: 47770, Dst Port: 8081, Seq: 1, Ack: 1, Len: 118
    Source Port: 47770
    Destination Port: 8081
    [Stream index: 0]
    [Conversation completeness: Incomplete, ESTABLISHED (7)]
    [TCP Segment Len: 118]
    Sequence Number: 1    (relative sequence number)
    Sequence Number (raw): 227109079
    [Next Sequence Number: 119    (relative sequence number)]
    Acknowledgment Number: 1    (relative ack number)
    Acknowledgment number (raw): 1743652312
    1000 .... = Header Length: 32 bytes (8)
    Flags: 0x018 (PSH, ACK)
        000. .... .... = Reserved: Not set
        ...0 .... .... = Nonce: Not set
        .... 0... .... = Congestion Window Reduced (CWR): Not set
        .... .0.. .... = ECN-Echo: Not set
        .... ..0. .... = Urgent: Not set
        .... ...1 .... = Acknowledgment: Set
        .... .... 1... = Push: Set
        .... .... .0.. = Reset: Not set
        .... .... ..0. = Syn: Not set
        .... .... ...0 = Fin: Not set
        [TCP Flags: ·······AP···]
    Window: 512
    [Calculated window size: 65536]
    [Window size scaling factor: 128]
    Checksum: 0xfe9e [unverified]
    [Checksum Status: Unverified]
    Urgent Pointer: 0
    Options: (12 bytes), No-Operation (NOP), No-Operation (NOP), Timestamps
        TCP Option - No-Operation (NOP)
            Kind: No-Operation (1)
        TCP Option - No-Operation (NOP)
            Kind: No-Operation (1)
        TCP Option - Timestamps: TSval 518634338, TSecr 518634338
            Kind: Time Stamp Option (8)
            Length: 10
            Timestamp value: 518634338
            Timestamp echo reply: 518634338
    [Timestamps]
        [Time since first frame in this TCP stream: 0.000026914 seconds]
        [Time since previous frame in this TCP stream: 0.000013717 seconds]
    [SEQ/ACK analysis]
        [iRTT: 0.000013197 seconds]
        [Bytes in flight: 118]
        [Bytes sent since last PSH flag: 118]
    TCP payload (118 bytes)
Hypertext Transfer Protocol
    GET / HTTP/1.0\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.0\r\n]
            [GET / HTTP/1.0\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.0
    X-high-load-test: 123\r\n                                               # Хэдер NGINX !!!
    Host: backend_pool\r\n
    Connection: close\r\n
    User-Agent: curl/7.81.0\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://backend_pool/]
    [HTTP request 1/1]

Frame 8: 106 bytes on wire (848 bits), 106 bytes captured (848 bits) on interface lo, id 0
    Interface id: 0 (lo)
        Interface name: lo
    Encapsulation type: Ethernet (1)
    Arrival Time: Nov  5, 2025 06:06:43.942171245 MSK
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1762312003.942171245 seconds
    [Time delta from previous captured frame: 0.000014776 seconds]
    [Time delta from previous displayed frame: 0.000511715 seconds]
    [Time since reference or first frame: 0.000538629 seconds]
    Frame Number: 8
    Frame Length: 106 bytes (848 bits)
    Capture Length: 106 bytes (848 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:tcp:http:data-text-lines]
Ethernet II, Src: 00:00:00_00:00:00 (00:00:00:00:00:00), Dst: 00:00:00_00:00:00 (00:00:00:00:00:00)
    Destination: 00:00:00_00:00:00 (00:00:00:00:00:00)
        Address: 00:00:00_00:00:00 (00:00:00:00:00:00)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: 00:00:00_00:00:00 (00:00:00:00:00:00)
        Address: 00:00:00_00:00:00 (00:00:00:00:00:00)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 92
    Identification: 0x8dac (36268)
    Flags: 0x40, Don't fragment
        0... .... = Reserved bit: Not set
        .1.. .... = Don't fragment: Set
        ..0. .... = More fragments: Not set
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 64
    Protocol: TCP (6)
    Header Checksum: 0xaeed [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 127.0.0.1
    Destination Address: 127.0.0.1
Transmission Control Protocol, Src Port: 8081, Dst Port: 47770, Seq: 187, Ack: 119, Len: 40
    Source Port: 8081
    Destination Port: 47770
    [Stream index: 0]
    [Conversation completeness: Incomplete, DATA (15)]
    [TCP Segment Len: 40]
    Sequence Number: 187    (relative sequence number)
    Sequence Number (raw): 1743652498
    [Next Sequence Number: 227    (relative sequence number)]
    Acknowledgment Number: 119    (relative ack number)
    Acknowledgment number (raw): 227109197
    1000 .... = Header Length: 32 bytes (8)
    Flags: 0x018 (PSH, ACK)
        000. .... .... = Reserved: Not set
        ...0 .... .... = Nonce: Not set
        .... 0... .... = Congestion Window Reduced (CWR): Not set
        .... .0.. .... = ECN-Echo: Not set
        .... ..0. .... = Urgent: Not set
        .... ...1 .... = Acknowledgment: Set
        .... .... 1... = Push: Set
        .... .... .0.. = Reset: Not set
        .... .... ..0. = Syn: Not set
        .... .... ...0 = Fin: Not set
        [TCP Flags: ·······AP···]
    Window: 512
    [Calculated window size: 65536]
    [Window size scaling factor: 128]
    Checksum: 0xfe50 [unverified]
    [Checksum Status: Unverified]
    Urgent Pointer: 0
    Options: (12 bytes), No-Operation (NOP), No-Operation (NOP), Timestamps
        TCP Option - No-Operation (NOP)
            Kind: No-Operation (1)
        TCP Option - No-Operation (NOP)
            Kind: No-Operation (1)
        TCP Option - Timestamps: TSval 518634338, TSecr 518634338
            Kind: Time Stamp Option (8)
            Length: 10
            Timestamp value: 518634338
            Timestamp echo reply: 518634338
    [Timestamps]
        [Time since first frame in this TCP stream: 0.000538629 seconds]
        [Time since previous frame in this TCP stream: 0.000014776 seconds]
    [SEQ/ACK analysis]
        [iRTT: 0.000013197 seconds]
        [Bytes in flight: 40]
        [Bytes sent since last PSH flag: 40]
    TCP payload (40 bytes)
    TCP segment data (40 bytes)
[2 Reassembled TCP Segments (226 bytes): #6(186), #8(40)]
    [Frame: 6, payload: 0-185 (186 bytes)]
    [Frame: 8, payload: 186-225 (40 bytes)]
    [Segment count: 2]
    [Reassembled TCP length: 226]
    [Reassembled TCP Data: 485454502f312e3020323030204f4b0d0a5365727665723a2053696d706c65485454502f…]
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
    Server: SimpleHTTP/0.6 Python/3.10.12\r\n
    Date: Wed, 05 Nov 2025 03:06:43 GMT\r\n
    Content-type: text/html\r\n
    Content-Length: 40\r\n
        [Content length: 40]
    Last-Modified: Wed, 05 Nov 2025 02:09:34 GMT\r\n
    \r\n
    [HTTP response 1/1]
    [Time since request: 0.000511715 seconds]
    [Request in frame: 4]
    [Request URI: http://backend_pool/]
    File Data: 40 bytes
Line-based text data: text/html (1 lines)
    <h1>Response from Backend Server 1</h1>\n
```

Мы убедились, что NGINX проставляет указанный header. Теперь проверим правило с переключением на бэкап-сервер после 7 неудачных попыток.
Отключим сервер 8081 и отправляем запросы на nginx:

```shell

```