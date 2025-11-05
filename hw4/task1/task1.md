## Задание 1. Создание Server Pool для backend.

Создадим папки backend1 и backend2:

```shell
mkdir backend1 backend2
```

Создадим index.html для каждого сервера:

```shell
touch backend1/index.html && echo "<h1>Response from Backend Server 1</h1>" > backend1/index.html
touch backend2/index.html && echo "<h2>*** Response from Backend Server 2 ***</h2>" > backend2/index.html
```

Запустим сервера в разных терминалах на портах 8081 и 8082:

```shell
# Терминал 1
python3 -m http.server 8081 --directory backend1
# Терминал 2
python3 -m http.server 8082 --directory backend2
```

Проверим, что они запустились с помощью `curl` в третьем терминале:

```
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://localhost:8081
<h1>Response from Backend Server 1</h1>
mihail@mihail-T58-V:~/IdeaProjects/highload-sys-admin-labs$ curl http://localhost:8082
<h2>*** Response from Backend Server 2 ***</h2>
```

Сервера отвечают содержанием созданного `index.html`. Все работает.