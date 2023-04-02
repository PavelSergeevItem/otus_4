**Стенд с Vagrant c ZFS**

 **1. Описание задания.**

  Определить алгоритм с наилучшим сжатием.

  Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4). Создать 4 файловых системы на каждой применить свой алгоритм сжатия. Для сжатия использовать либо текстовый файл, либо группу файлов: определить настройки пула с помощью команды zfs import, собрать ZFS pool. 
Командами zfs определить настройки: размер хранилища, тип pool,значение recordsize, какое сжатие используется, какая контрольная сумма используется.

  Работа со снапшотами. Скопировать файл из удаленной директории [https://drive.google.com/file/d/1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG/view?usp=sharing], восстановить файл локально. zfs receive найти зашифрованное сообщение в файле secret_message

**2. Выполенение задания.**

1. Создал виртуальную машину основываясь на примере указанным в методичке. Создал вагрантфайл, запустил его командой ``vagrant up``. Подключился к виртуальной машине командой ``vagrant ssh``. Получил корневой доступ ``sudo -i``
2. Начал анализ определения алгоритма с наилучшим сжатием.
3. Создал 4 пула используюя поочередно команды.
```
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi
```
4. Добавил разные алгоритмы сжатия в каждую файловую систему, использую следующие команды
```
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```
5. Скачал текстовый файл по во все пулы, для примера заполнения использовал команду
``do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done``
6. Проверил, сколько места занимает один и тот же файл в разных пулах. 

``zfs list``
```
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.6M   330M     21.5M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.0M   313M     38.9M  /otus4
```

7. Проверил степень сжатия файлов.

``zfs get all | grep compressratio | grep -v ref``
```
otus1  compressratio         1.80x                  -
otus2  compressratio         2.21x                  -
otus3  compressratio         3.63x                  -
otus4  compressratio         1.00x                  -
```
Проанализировав вывод команды ``zfs get all | grep compressratio | grep -v ref``, пришел к выводу, что алгоритм gzip-9 самый эффективный по сжатию.

8. Скачал архив в домашний каталог. Команда ниже. 
``wget -O archive.tar.gz --no-check-certificate 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'``
9. Разархировал архив при помощи команды: ``tar -xzvf archive.tar.gz``
10. Импортировал пул полученный из архива в ОС применив команду: ``zpool import -d zpoolexport/ newotus``
11. Командой ``zpool status`` получил информацию о составе импортированного пула, вывод команды ниже.

```
pool: newotus  
  state: ONLINE  
   scan: none requested  
 config:


        NAME                            STATE     READ WRITE CKSUM
        newotus                         ONLINE       0     0     0
          mirror-0                      ONLINE       0     0     0
            /root/zpoolexport/filea     ONLINE       0     0     0
            /root/zpoolexport/fileb     ONLINE       0     0     0
            
errors: No known data errors
```
12. Потом определил настройки командой ``zpool get all newotus``  
```
[root@zfs ~]# zfs get all newotus
NAME  PROPERTY              VALUE                  SOURCE
newotus  type                  filesystem             -
newotus  creation              Fri May 15  4:00 2020  -
newotus  used                  2.04M                  -
newotus  available             350M                   -
newotus  referenced            24K                    -
newotus  compressratio         1.00x                  -
newotus  mounted               yes                    -
newotus  quota                 none                   default
newotus  reservation           none                   default
newotus  recordsize            128K                   local
newotus  mountpoint            /otus                  default
newotus  sharenfs              off                    default
newotus  checksum              sha256                 local
newotus  compression           zle                    local
newotus  atime                 on                     default
newotus  devices               on                     default
newotus  exec                  on                     default
newotus  setuid                on                     default
newotus  readonly              off                    default
newotus  zoned                 off                    default
newotus  snapdir               hidden                 default
newotus  aclinherit            restricted             default
newotus  createtxg             1                      -
newotus  canmount              on                     default
newotus  xattr                 on                     default
newotus  copies                1                      default
newotus  version               5                      -
newotus  utf8only              off                    -
newotus  normalization         none                   -
newotus  casesensitivity       sensitive              -
newotus  vscan                 off                    default
newotus  nbmand                off                    default
newotus  sharesmb              off                    default
newotus  refquota              none                   default
newotus  refreservation        none                   default
newotus  guid                  14592242904030363272   -
newotus  primarycache          all                    default
newotus  secondarycache        all                    default
newotus  usedbysnapshots       0B                     -
newotus  usedbydataset         24K                    -
newotus  usedbychildren        2.01M                  -
newotus  usedbyrefreservation  0B                     -
newotus  logbias               latency                default
newotus  objsetid              54                     -
newotus  dedup                 off                    default
newotus  mlslabel              none                   default
newotus  sync                  standard               default
newotus  dnodesize             legacy                 default
newotus  refcompressratio      1.00x                  -
newotus  written               24K                    -
newotus  logicalused           1020K                  -
newotus  logicalreferenced     12K                    -
newotus  volmode               default                default
newotus  filesystem_limit      none                   default
newotus  snapshot_limit        none                   default
newotus  filesystem_count      none                   default
newotus  snapshot_count        none                   default
newotus  snapdev               hidden                 default
newotus  acltype               off                    default
newotus  context               none                   default
newotus  fscontext             none                   default
newotus  defcontext            none                   default
newotus  rootcontext           none                   default
newotus  relatime              off                    default
newotus  redundant_metadata    all                    default
newotus  overlay               off                    default
newotus  encryption            off                    default
newotus  keylocation           none                   default
newotus  keyformat             none                   default
newotus  pbkdf2iters           0                      default
newotus  special_small_blocks  0                      default
[root@zfs ~]#  available  
```
13. Скачал файл, указанный в задании. ``wget -O otus_task2.file --no-check-certificate "https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download"``
14. Восстановил файловую систему из снепшота: ``zfs receive otus/test@today < otus_task2.file``
15. Нашел файл с именем "sevret_message": ``find /otus/test -name "secret_message"``
16. Посмотрел содержимое найденного файла, при помощи команды: ``cat /otus/test/task1/file_mess/secret_message.``
Содержимое файла это url: https://github.com/sindresorhus/awesome
17. Написал баш скрипт, добавил его в вагрантфайл. Bash-скриптом, который будет конфигурировать сервер.
18. Написал readme на гитхабе. Закинул вагрантфайл в свой репозиторий.
