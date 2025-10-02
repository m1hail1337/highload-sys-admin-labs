## Задание 2. Наблюдение за VFS.

В рамках работы была рассмотрена команда

```shell
cat  /etc/os-release > /dev/null
```

Для анализа исполнения команды была использована утилита `strace`. С помощью нее мы видим системные вызовы
открытия, чтения, записи и закрытия файлов. Команда для анализа вызовов:
```shell
strace -e trace=openat,read,write,close cat /etc/os-release > /dev/null
```

Вывод:

```
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
close(3)                                = 0
openat(AT_FDCWD, "/etc/os-release", O_RDONLY) = 3
read(3, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 131072) = 386
write(1, "PRETTY_NAME=\"Ubuntu 22.04.4 LTS\""..., 386) = 386
read(3, "", 131072)                     = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
+++ exited with 0 +++
```

Рассмотрим каждый вид вызовов:

* `openat` - открываем файлы кэша, системных библиотек, локали и сам читаемый файл
* `read` - читаем открытые файлы
* `write` - записываем содержимое файла в /dev/null (фактически этого не происходит, просто `strace` перехватывает вызовы до их выполнения)
* `close` - закрытие открытых системой файлов

