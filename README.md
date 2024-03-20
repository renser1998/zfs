1.1 Создаем zfs пулы

   [root@zfs vagrant]# zpool create otus1 mirror /dev/sdb /dev/sdc
   [root@zfs vagrant]# zpool create otus2 mirror /dev/sdd /dev/sde
   [root@zfs vagrant]# zpool create otus3 mirror /dev/sdf /dev/sdg
   [root@zfs vagrant]# zpool create otus4 mirror /dev/sdh /dev/sdi

[root@zfs vagrant]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   126K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -

1.2.Устанавливаем сжатие
[root@zfs vagrant]# zfs set compression=lzjb otus1
[root@zfs vagrant]# zfs set compression=lz4 otus2
[root@zfs vagrant]# zfs set compression=gzip-9 otus3
[root@zfs vagrant]# zfs set compression=zle otus4
[root@zfs vagrant]# zfs get all | grep compression
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local


1.3. Загрузили файл на пулы
[root@zfs vagrant]# ls -l /otus*
/otus1:
total 22069
-rw-r--r--. 1 root root 41025184 Mar  2 08:53 pg2600.converter.log
/otus2:
total 17995
-rw-r--r--. 1 root root 41025184 Mar  2 08:53 pg2600.converter.log
/otus3:
total 10960
-rw-r--r--. 1 root root 41025184 Mar  2 08:53 pg2600.converter.log
/otus4:
total 40091
-rw-r--r--. 1 root root 41025184 Mar  2 08:53 pg2600.converter.log

1.4. Видим разницу сжатия одного и того же файла
[root@zfs vagrant]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.7M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.2M   313M     39.2M  /otus4

2.1 Скачиваем и разархивируем archive.tar.gz
Saving to: 'archive.tar.gz'
2024-03-20 09:06:38 (7.82 MB/s) - 'archive.tar.gz' saved [7275140/7275140]

[root@zfs ~]# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb

2.2 Проверим, возможно ли импортировать данный каталог в пул

[root@zfs ~]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE

2.3 Импорт пула и проверка статуса
[root@zfs ~]# zpool import -d zpoolexport/ otus
[root@zfs ~]# zpool status
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

2.4 Просмотр параметров ипортированного пула
   [root@zfs ~]# zfs get available otus
   NAME  PROPERTY   VALUE  SOURCE
   otus  available  350M   -
[root@zfs ~]# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
[root@zfs ~]# zfs get compression otus
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local
[root@zfs ~]#  zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local

3.1 Скачали файл снапшота otus_task2.file
[root@zfs ~]# ls
anaconda-ks.cfg  archive.tar.gz  original-ks.cfg  otus_task2.file  zpoolexport

3.2 Восстановление fs
[root@zfs ~]# zfs receive otus/testdata < otus_task2.file 

3.3 Поиск и вывод secret_message
[root@zfs otus]# find /otus -name "secret_message"
/otus/testdata/task1/file_mess/secret_message
[root@zfs otus]# cat /otus/testdata/task1/file_mess/secret_message
https://otus.ru/lessons/linux-hl/
