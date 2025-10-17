## Задание 2. Межпроцессное взаимодействие (IPC) с разделяемой памятью.

### Создание программы

Создадим программу с разделяемой памятью на языке C:

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

int main() {
    key_t key = ftok("homework_key", 65); // Generate a unique key
    int shmid = shmget(key, 1024, 0666|IPC_CREAT); // Create 1KB segment
    if (shmid == -1) {
        perror("shmget");
        exit(1);
    }
    printf("Shared memory segment created.\n");
    printf("ID: %d\nKey: 0x%x\n", shmid, key);
    printf("Run 'ipcs -m' to see it. Process will exit in 60 seconds...\n");
    
    sleep(60);
    
    shmctl(shmid, IPC_RMID, NULL); // Clean up
    printf("Shared memory segment removed.\n");
    return 0;
}
```

Скомпилируем и запустим:

```shell
gcc shm_creator.c -o shm_creator
touch homework_key
./shm_creator
```

Перед запуском в другом окне терминала я выполнил команду:

```shell
watch ipcs -m
```

После запуска в списке появилась строка с выделенным сегментом памяти:

```
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x4102d488 1081347    mihail     666        1024       0
...
```

Из вывода видно, что метрика `nattch` (number of attached processes) равна 0. Это следует из того, что мы создали сегмент
разделяемой памяти без привязки к адресному пространству процесса (функция `shmat()`), поэтому процесс (наша программа) не может его использовать.
