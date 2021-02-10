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
