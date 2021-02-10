# zfs

1. Добавить в Свойства системы - Переменные среды - Системные переменные - Path путь к папке с установленным VirtualBox (по умолчанию C:\Program Files\Oracle\VirtualBox)
2. Создать новый репозиторий https://github.com/andmisha/zfs
3. Скачаь Vagrantfile по ссылке https://github.com/nixuser/virtlab/tree/main/zfs
4. Выполнить vagrant up
5. В официальной документации на сайте не нашел, какие алгоримты сжатия поддерживает ZFS. Сайт Oracle порадовал - https://docs.oracle.com/cd/E53394_01/html/E54801/gpxis.html
6. Посмотреть с помощью lsblk какие диски есть в нашей системе
```
[vagrant@server ~]$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0  10G  0 disk
└─sda1   8:1    0  10G  0 part /
sdb      8:16   0   1G  0 disk
sdc      8:32   0   1G  0 disk
sdd      8:48   0   1G  0 disk
sde      8:64   0   1G  0 disk
sdf      8:80   0   1G  0 disk
sdg      8:96   0   1G  0 disk
```
7. Создать пул
```
[vagrant@server ~]$ sudo zpool create pool01 mirror sdb sdc
```
8. Проверить статус ZFS
```
[vagrant@server ~]$ zfs list
NAME     USED  AVAIL     REFER  MOUNTPOINT
pool01  82.5K   832M       24K  /pool01
```
и
```
[vagrant@server ~]$ zpool status
  pool: pool01
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        pool01      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors
```
9. Создать файловую систему на пуле pool01 
```
[vagrant@server ~]$ sudo zfs create pool01/gzip
```
10. Проверить, что файловая система создалась и примонтирована
```
[vagrant@server gzip]$ zfs get mounted
NAME         PROPERTY  VALUE    SOURCE
pool01       mounted   yes      -
pool01/gzip  mounted   yes      -
```
11. Скопировать файл Война и мир на нее и проверить, что скопировался
```
[vagrant@server ~]$ sudo cp War_and_Peace.txt /pool01/gzip/
[vagrant@server ~]$ ls -l
total 0
-rw-rw-r--. 1 vagrant vagrant 0 Feb 10 21:31 War_and_Peace.txt
[vagrant@server ~]$ cd /pool01/gzip/
[vagrant@server gzip]$ ls -l
total 1
-rw-r--r--. 1 root root 0 Feb 10 21:41 War_and_Peace.txt
```
12. Включить сжатие gzip
```
[vagrant@server gzip]$ sudo zfs set compress=gzip pool01/gzip
```
13. Проверить коэффициент сжатия
```
[vagrant@server gzip]$ sudo zfs get compression,compressratio pool01/gzip
NAME         PROPERTY       VALUE     SOURCE
pool01/gzip  compression    gzip      local
pool01/gzip  compressratio  1.00x     -
```
14. Далее я каждый раз удалял файловую систему, создавал заново и копировал файл Война и мир
```
[vagrant@server ~]$ sudo zfs destroy pool01/gzip
```
15. Включить сжатие lz4
```
[vagrant@server gzip]$ sudo zfs set compress=lz4 pool01/lz4
```
16. Проверить коэффициент сжатия
```
[vagrant@server ~]$ sudo zfs get compression,compressratio pool01/lz4
NAME        PROPERTY       VALUE     SOURCE
pool01/lz4  compression    lz4       local
pool01/lz4  compressratio  1.00x     -
```
17. Включить сжатие lzjb
```
[vagrant@server gzip]$ sudo zfs set compress=lzjb pool01/lzjb
```
18. Проверить коэффициент сжатия
```
[vagrant@server ~]$ sudo zfs get compression,compressratio pool01/lzjb
NAME         PROPERTY       VALUE     SOURCE
pool01/lzjb  compression    lzjb      local
pool01/lzjb  compressratio  1.00x     -   -
```
19. Включить сжатие zle
```
[vagrant@server gzip]$ sudo zfs set compress=zle pool01/zle
```
20. Проверить коэффициент сжатия
```
[vagrant@server ~]$ sudo zfs get compression,compressratio pool01/zle
NAME        PROPERTY       VALUE     SOURCE
pool01/zle  compression    zle       local
pool01/zle  compressratio  1.04x     -
```
Вывод - видимо файл не очень удачный для проверки сжатия, лучший результат показал алгоритм zle.

---
21. Скачать архив с файлами https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg и распаковать в /home/vagrant/zpoolexport
22. Выполнить команду zpool import -d ${PWD}/zpoolexport/ как я понимаю для проверки импортируемого пула
```
[vagrant@server ~]$ sudo zpool import -d ${PWD}/zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                                 ONLINE
          mirror-0                           ONLINE
            /home/vagrant/zpoolexport/filea  ONLINE
            /home/vagrant/zpoolexport/fileb  ONLINE
```            
23. Выполнить импорт пула
```
[vagrant@server ~]$ sudo zpool import -d ${PWD}/zpoolexport/ otus
```
24. Проверить статус (так как я делал на 1ВМ, то статус выводится для ранее созданного пула)
```
[vagrant@server ~]$ zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                                 STATE     READ WRITE CKSUM
        otus                                 ONLINE       0     0     0
          mirror-0                           ONLINE       0     0     0
            /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
            /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

  pool: pool01
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        pool01      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors
```
и
```
[vagrant@server ~]$ zpool list
NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus     480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
pool01   960M   264K   960M        -         -     0%     0%  1.00x    ONLINE  -
```
25. Командой zfs list вывести все файловые системы
```
[vagrant@server ~]$ zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            2.04M   350M       24K  /otus
otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
pool01           208K   832M     25.5K  /pool01
pool01/zle        25K   832M       25K  /pool01/zle
```
26. Командой zfs вывел настройки пула - тип, recordsize, контрольную сумму, алгоритм сжатия и размер хранилища
```
[vagrant@server ~]$ sudo zfs get type,recordsize,checksum,compress,available otus/hometask2
NAME            PROPERTY     VALUE       SOURCE
otus/hometask2  type         filesystem  -
otus/hometask2  recordsize   128K        inherited from otus
otus/hometask2  checksum     sha256      inherited from otus
otus/hometask2  compression  zle         inherited from otus
otus/hometask2  available    350M        -
```

---
27. Скачать файл https://drive.google.com/file/d/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG/view?usp=sharing и распаковать в /home/vagrant/
28. Выполнить импорт снапшота
```
[vagrant@server ~]$ sudo zfs receive otus/storage/task2 < otus_task2.file
```
29. Вывести список всех снапшотов
```
[vagrant@server ~]$ zfs list -t snapshot
NAME                 USED  AVAIL     REFER  MOUNTPOINT
otus/storage@task2     0B      -     2.83M  -
```
30. Восстановить файлы из снапшота
```
[vagrant@server ~]$ sudo zfs rollback otus/storage@task2
```
31. Проверить содержимое директории
```
[vagrant@server ~]$ ls -l /otus/storage/task2
total 2589
-rw-r--r--. 1 root    root          0 May 15  2020 10M.file
-rw-r--r--. 1 root    root     727040 May 15  2020 cinderella.tar
-rw-r--r--. 1 root    root         65 May 15  2020 for_examaple.txt
-rw-r--r--. 1 root    root          0 May 15  2020 homework4.txt
-rw-r--r--. 1 root    root     309987 May 15  2020 Limbo.txt
-rw-r--r--. 1 root    root     509836 May 15  2020 Moby_Dick.txt
drwxr-xr-x. 3 vagrant vagrant       4 Dec 18  2017 task1
-rw-r--r--. 1 root    root    1209374 May  6  2016 War_and_Peace.txt
-rw-r--r--. 1 root    root     398635 May 15  2020 world.sql
```
