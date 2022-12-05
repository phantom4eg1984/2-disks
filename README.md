1.  Добавить в Vagrantfile еще дисков
Добавить диски в систему можно с помощью следующих команд:
 :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250,
                        :port => 5
                },

Я добавлю в виртуальную машину 8 дисков, из которых соберу RAID 10.

Команды которыми можно воспользоваться для просмотра информации после запуска вирутальной машины что бы увидеть какие блочные устройства есть в системе:
[root@otuslinux ~]# lshw -short | grep disk
/0/100/1.1/0.0.0    /dev/sdi  disk        42GB VBOX HARDDISK
/0/100/d/0          /dev/sda  disk        262MB VBOX HARDDISK
/0/100/d/1          /dev/sdb  disk        262MB VBOX HARDDISK
/0/100/d/2          /dev/sdc  disk        262MB VBOX HARDDISK
/0/100/d/3          /dev/sdd  disk        262MB VBOX HARDDISK
/0/100/d/4          /dev/sde  disk        262MB VBOX HARDDISK
/0/100/d/5          /dev/sdf  disk        262MB VBOX HARDDISK
/0/100/d/6          /dev/sdg  disk        262MB VBOX HARDDISK
/0/100/d/7          /dev/sdh  disk        262MB VBOX HARDDISK

[root@otuslinux ~]# fdisk -l
Disk /dev/sda: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdf: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdg: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdb: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdd: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdc: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sde: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdh: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdi: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ef1a

   Device Boot      Start         End      Blocks   Id  System

/dev/sdi1   *        2048    83886079    41942016   83  Linux

[root@otuslinux ~]# lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  250M  0 disk 
sdb      8:16   0  250M  0 disk 
sdc      8:32   0  250M  0 disk 
sdd      8:48   0  250M  0 disk 
sde      8:64   0  250M  0 disk 
sdf      8:80   0  250M  0 disk 
sdg      8:96   0  250M  0 disk 
sdh      8:112  0  250M  0 disk 
sdi      8:128  0   40G  0 disk 
`-sdi1   8:129  0   40G  0 part /

[root@otuslinux ~]# lsscsi
[1:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda 
[2:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdb 
[3:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdc 
[4:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdd 
[5:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sde 
[6:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdf 
[7:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdg 
[8:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdh 
[30:0:0:0]   disk    ATA      VBOX HARDDISK    1.0   /dev/sdi 

2. Собрать R10:
Занулить суперблок на всех устройствах которые будут использоваться в RAID:
[root@otuslinux ~]# mdadm --zero-superblock --force /dev/sd{a..h}
mdadm: Unrecognised md component device - /dev/sda
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf
mdadm: Unrecognised md component device - /dev/sdg
mdadm: Unrecognised md component device - /dev/sdh

Создаем RAID 10 из 8 дисков:
[root@otuslinux ~]# mdadm --create --verbose /dev/md0 -l 10 -n 8 /dev/sd{a..h}
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.


Убедиться, что RAID-массив проинициализирован корректно можно просмотрев файл /proc/mdstat. В этом файле отражается текущее состояние RAID-массива:

[root@otuslinux ~]# cat /proc/mdstat 

Personalities : [raid10] 

md0 : active raid10 sdh[7] sdg[6] sdf[5] sde[4] sdd[3] sdc[2] sdb[1] sda[0]

     1015808 blocks super 1.2 512K chunks 2 near-copies [8/8] [UUUUUUUU]

     unused devices: <none>


[root@otuslinux ~]# mdadm -D /dev/md0

/dev/md0:

           Version : 1.2

     Creation Time : Mon Dec  5 09:25:50 2022

        Raid Level : raid10
        Array Size : 1015808 (992.00 MiB 1040.19 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 8
     Total Devices : 8


       Persistence : Superblock is persistent
       Update Time : Mon Dec  5 09:25:56 2022
             State : clean 

    Active Devices : 8
   Working Devices : 8
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 60e7ae05:d873001c:90f99dfb:80dd04c6
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync set-A   /dev/sda
       1       8       16        1      active sync set-B   /dev/sdb
       2       8       32        2      active sync set-A   /dev/sdc
       3       8       48        3      active sync set-B   /dev/sdd
       4       8       64        4      active sync set-A   /dev/sde
       5       8       80        5      active sync set-B   /dev/sdf
       6       8       96        6      active sync set-A   /dev/sdg
       7       8      112        7      active sync set-B   /dev/sdh


3. Прописать собранный рейд в конф, чтобы рейд собирался при загрузке:
Система сама не запоминает какие RAID-массивы ей нужно создать и какие компоненты в них входят. Эта информация находится в файле mdadm.conf:
[root@otuslinux ~]# mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid10 num-devices=8 metadata=1.2 name=otuslinux:0 UUID=60e7ae05:d873001c:90f99dfb:80dd04c6
   devices=/dev/sda,/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf,/dev/sdg,/dev/sdh
[root@otuslinux ~]# echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
[root@otuslinux ~]# mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

4. Создать GPT раздел и 5 партиций:

Создаем GPT раздел:
[root@otuslinux ~]# parted -s /dev/md0 mklabel gpt

Создаем 5 разделов:
[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 0% 20%
[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 20% 40%
[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 40% 60%
[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 60% 80%
[root@otuslinux ~]# parted /dev/md0 mkpart primary ext4 80% 100%

Проверяем что все разделы были успешно созданы:
[root@otuslinux ~]# lsblk                                                 

NAME      MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINT

sda         8:0    0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdb         8:16   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdc         8:32   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdd         8:48   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sde         8:64   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdf         8:80   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdg         8:96   0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdh         8:112  0  250M  0 disk   

`-md0       9:0    0  992M  0 raid10 

  |-md0p1 259:0    0  196M  0 md     

  |-md0p2 259:1    0  198M  0 md     

  |-md0p3 259:2    0  200M  0 md     

  |-md0p4 259:3    0  198M  0 md     

  `-md0p5 259:4    0  196M  0 md     

sdi         8:128  0   40G  0 disk   

`-sdi1      8:129  0   40G  0 part   /

Как видим все разделы были успешно созданы!

Создаем файловую систему ext4 на каждом разделе:
[root@otuslinux vagrant]# for i in $(seq 1 5); do mkfs.ext4 /dev/md0p$i; done

Создаем 5 новых каталогов:
[root@otuslinux vagrant]# mkdir /home/vagrant/dir{1..5}

Монтируем разделы в эти каталоги:
[root@otuslinux vagrant]# for i in $(seq 1 5); do mount /dev/md0p$i /home/vagrant/dir$i; done

[root@otuslinux vagrant]# df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  489M     0  489M   0% /dev
tmpfs          tmpfs     496M     0  496M   0% /dev/shm
tmpfs          tmpfs     496M  6.8M  489M   2% /run
tmpfs          tmpfs     496M     0  496M   0% /sys/fs/cgroup
/dev/sdi1      xfs        40G  5.1G   35G  13% /
tmpfs          tmpfs     100M     0  100M   0% /run/user/1000
/dev/md0p1     ext4      186M  1.6M  171M   1% /home/vagrant/dir1
/dev/md0p2     ext4      188M  1.6M  173M   1% /home/vagrant/dir2
/dev/md0p3     ext4      190M  1.6M  175M   1% /home/vagrant/dir3
/dev/md0p4     ext4      188M  1.6M  173M   1% /home/vagrant/dir4
/dev/md0p5     ext4      186M  1.6M  171M   1% /home/vagrant/dir5

5. Сломать/починить raid:
Что бы сломать raid можно воспользоваться командой опцией fail и remove.
[root@otuslinux vagrant]# mdadm /dev/md0 --fail /dev/sda
[root@otuslinux vagrant]# mdadm /dev/md0 --remove /dev/sda

Для восстановления необходимо добавить новый диск в пул, после этого произойдет rebuild и RAID будет полностью восстановлен.
[root@otuslinux vagrant]# mdadm /dev/md0 --add /dev/sda


