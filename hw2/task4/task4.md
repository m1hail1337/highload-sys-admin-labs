## Задание 4. NUMA и cgroups.

### Демонстрация кол-ва NUMA-нод

Для демонстрации количества NUMA-нод на устройстве выполним команду:

```shell
numactl -H
```

Вывод команды:

```
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
node 0 size: 15770 MB
node 0 free: 281 MB
node distances:
node   0 
  0:  10
```

Из вывода видно, что на машине присутствует 1 NUMA-нода на ~16Gb

### Ограничение работы процессов при помощи systemd

**MemoryMax** - устанавливает максимальный лимит физической памяти (RAM) для юнита.

**CPUWeight** устанавливает относительный приоритет доступа к CPU времени. Например, если сервис A имеет weight=200, а сервис B weight=100, то A получит в 2 раза больше CPU времени

Для ограничения памяти и CPU для процессов используем следующую команду:

```shell
sudo systemd-run --unit=highload-stress-test --slice=testing.slice -p "MemoryMax=150M" -p "CPUWeight=100" stress --cpu 1 --vm 1 --vm-bytes 150M --timeout 30s
```

Здесь мы с помощью `systemd-run` сначала задаем ограничения, а затем запускаем процесс использующий их.

Мониторинг процесса был выполнен с помощью следующей команды:

```shell
systemd-cgtop /testing.slice
```

Вывод:

```shell
Control Group                                                                                                                                                                       Tasks   %CPU   Memory  Input/s Output/s
/testing.slice                                                                                                                                                                          3  198.2   150.0M   111.9K       0B
/testing.slice/highload-stress-test.service                                                                                                                                             3  198.2   149.9M        -        -
```
Видно, что при запросе 150М и ограничении в 150M процесс нормально работает. Заменим `--vm-bytes 150M` на `--vm-bytes 300M`. Запустим команду и выполним мониторинг:

```
Control Group                                                                                                                                                                       Tasks   %CPU   Memory  Input/s Output/s
/testing.slice                                                                                                                                                                          -      -   224.0K        -        -
```

Видно, что в нашем слайсе нет запущенного сервиса, посмотрим его статус с помощью команды:

```shell
systemctl status highload-stress-test.service
```

Вывод:

```
× highload-stress-test.service - /usr/bin/stress --cpu 1 --vm 1 --vm-bytes 300M --timeout 30s
     Loaded: loaded (/run/systemd/transient/highload-stress-test.service; transient)
  Transient: yes
     Active: failed (Result: oom-kill) since Wed 2025-10-22 18:33:02 MSK; 1min 40s ago
    Process: 31079 ExecStart=/usr/bin/stress --cpu 1 --vm 1 --vm-bytes 300M --timeout 30s (code=killed, signal=TERM)
   Main PID: 31079 (code=killed, signal=TERM)
        CPU: 626ms

окт 22 18:33:02 mihail-T58-V systemd[1]: Started /usr/bin/stress --cpu 1 --vm 1 --vm-bytes 300M --timeout 30s.
окт 22 18:33:02 mihail-T58-V stress[31079]: stress: info: [31079] dispatching hogs: 1 cpu, 0 io, 1 vm, 0 hdd
окт 22 18:33:02 mihail-T58-V systemd[1]: highload-stress-test.service: A process of this unit has been killed by the OOM killer.
окт 22 18:33:02 mihail-T58-V systemd[1]: highload-stress-test.service: Failed with result 'oom-kill'.
```

Видно, что процесс после превышения лимита памяти был убит `OOM killer`'ом.