#Я использовал предоставленный шаблон для VM. Сразу добавил для автоматической установки в Vagratyfile xfsdump и nano.
lsblk
df -H

#вывод команд

[root@lvm vagrant]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 
[root@lvm vagrant]# df -HT
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs        41G   17G   24G  67% /
devtmpfs                        devtmpfs  246M     0  246M   0% /dev
tmpfs                           tmpfs     256M     0  256M   0% /dev/shm
tmpfs                           tmpfs     256M  4.8M  251M   2% /run
tmpfs                           tmpfs     256M     0  256M   0% /sys/fs/cgroup
/dev/sda2                       xfs       1.1G   66M  998M   7% /boot
tmpfs                           tmpfs      52M     0   52M   0% /run/user/1000
tmpfs                           tmpfs      52M     0   52M   0% /run/user/0

#Видим, что слишком большой объём. Смотрим, что занимает столько места.

du -hd 1 /

#определяем, что это каталог Vagrant в /, внутри которого хранятся наши vdi диски, Vagrantfile и т.д. 
#меняем Vagrantfile: размещаем диски не в каталоге с Vagrantfile
#Пересоздаём виртуалку
#Первый вариант Vagrant файла не стал размещать, сразу итоговый положил

[root@lvm vagrant]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 
[root@lvm vagrant]# df -HT
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs        41G  944M   40G   3% /
devtmpfs                        devtmpfs  246M     0  246M   0% /dev
tmpfs                           tmpfs     256M     0  256M   0% /dev/shm
tmpfs                           tmpfs     256M  4.8M  251M   2% /run
tmpfs                           tmpfs     256M     0  256M   0% /sys/fs/cgroup
/dev/sda2                       xfs       1.1G   66M  998M   7% /boot
tmpfs                           tmpfs      52M     0   52M   0% /run/user/1000

#Можно было бы загрузиться из-под live-cd, и выполнить все операции, но опять же буду идти по методичке, т.к. описанный в ней способ считаю более интересным с практической точки зрения и рациональным.

sudo su

pvcreate /dev/sdb                                   # создаём pv на диске в sdb

vgcreate vg_root /dev/sdb                           # создаём vg на диске в sdb

lvcreate -n lv_root -l +100%FREE /dev/vg_root       # создаём lv на диске в vg

mkfs.xfs /dev/vg_root/lv_root                       # Создадим файловую систему на lv_root

mount /dev/vg_root/lv_root /mnt                     # монтирование lv_root в /mnt

df -HT                                              # проверка

[root@lvm vagrant]# df -HT
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs        41G  944M   40G   3% /
devtmpfs                        devtmpfs  246M     0  246M   0% /dev
tmpfs                           tmpfs     256M     0  256M   0% /dev/shm
tmpfs                           tmpfs     256M  4.8M  251M   2% /run
tmpfs                           tmpfs     256M     0  256M   0% /sys/fs/cgroup
/dev/sda2                       xfs       1.1G   66M  998M   7% /boot
tmpfs                           tmpfs      52M     0   52M   0% /run/user/1000
/dev/mapper/vg_root-lv_root     xfs        11G   34M   11G   1% /mnt

xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt    # делаем копию системного раздела

ls -lah /mnt                                        # проверка

[root@lvm vagrant]# ls -lah /mnt
total 12K
drwxr-xr-x. 18 root    root     239 May  1 08:39 .
dr-xr-xr-x. 18 root    root     239 May  1 08:14 ..
lrwxrwxrwx.  1 root    root       7 May  1 08:38 bin -> usr/bin
drwxr-xr-x.  2 root    root       6 May 12  2018 boot
drwxr-xr-x.  2 root    root       6 May 12  2018 dev
drwxr-xr-x. 79 root    root    8.0K May  1 08:14 etc
drwxr-xr-x.  3 root    root      21 May 12  2018 home
lrwxrwxrwx.  1 root    root       7 May  1 08:38 lib -> usr/lib
lrwxrwxrwx.  1 root    root       9 May  1 08:38 lib64 -> usr/lib64
drwxr-xr-x.  2 root    root       6 Apr 11  2018 media
drwxr-xr-x.  2 root    root       6 Apr 11  2018 mnt
drwxr-xr-x.  2 root    root       6 Apr 11  2018 opt
drwxr-xr-x.  2 root    root       6 May 12  2018 proc
dr-xr-x---.  3 root    root     149 May  1 08:14 root
drwxr-xr-x.  2 root    root       6 May 12  2018 run
lrwxrwxrwx.  1 root    root       8 May  1 08:38 sbin -> usr/sbin
drwxr-xr-x.  2 root    root       6 Apr 11  2018 srv
drwxr-xr-x.  2 root    root       6 May 12  2018 sys
drwxrwxrwt.  8 root    root     193 May  1 08:14 tmp
drwxr-xr-x. 13 root    root     155 May 12  2018 usr
drwxrwxr-x.  2 vagrant vagrant   25 Apr 14 18:11 vagrant
drwxr-xr-x. 18 root    root     254 May  1 08:13 var

du -hd 1 /mnt                                       # проверка

[root@lvm vagrant]# du -hd 1 /mnt
4.0K	/mnt/vagrant
0	/mnt/srv
0	/mnt/opt
0	/mnt/mnt
0	/mnt/media
16K	/mnt/home
560M	/mnt/usr
4.0K	/mnt/tmp
269M	/mnt/var
40K	/mnt/root
30M	/mnt/etc
0	/mnt/sys
0	/mnt/run
0	/mnt/proc
0	/mnt/dev
0	/mnt/boot
858M	/mnt

for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done    # перемонтировать каталоги из цикла в /mnt

chroot /mnt/                                        # режим изоляции

grub2-mkconfig -o /boot/grub2/grub.cfg              # конфигурируем grub

cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done          #обновим initrd

sed -i 's/VolGroup00\/LogVol00/vg_root\/lv_root/g' /boot/grub2/grub.cfg       # правим grub

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config           # правим конфиг selinux, чтобы без проблем войти.

sed -i 's/VolGroup00-LogVol00/vg_root-lv_root/g' /etc/fstab                   # Правим /etc/fstab

exit                                                                          # Выходим из chroot

reboot                                                                        # перезагружаемся

lsblk                                                                         # Проверяем, видим, что всё получилось
[root@lvm vagrant]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm  
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk

# Теперь мы должны изменить старый раздел и вернуть на него /, т.е. удаляем старый раздел 40G и создаём новый 8G

lvremove /dev/VolGroup00/LogVol00                                             # удаляем старый раздел

lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00                         # создаём раздел с тем же именем

mkfs.xfs /dev/VolGroup00/LogVol00                                             # форматируем в xfs

mount /dev/VolGroup00/LogVol00 /mnt                                           # монтируем в /mnt

df -Th                                                                        # проверяем

[root@lvm vagrant]# df -Th
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/vg_root-lv_root     xfs        10G  899M  9.2G   9% /
devtmpfs                        devtmpfs  236M     0  236M   0% /dev
tmpfs                           tmpfs     244M     0  244M   0% /dev/shm
tmpfs                           tmpfs     244M  4.6M  240M   2% /run
tmpfs                           tmpfs     244M     0  244M   0% /sys/fs/cgroup
/dev/sda2                       xfs      1014M   61M  954M   6% /boot
tmpfs                           tmpfs      49M     0   49M   0% /run/user/1000
tmpfs                           tmpfs      49M     0   49M   0% /run/user/0
/dev/mapper/VolGroup00-LogVol00 xfs       8.0G   33M  8.0G   1% /mnt

xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt                      # делаем копию системного раздела

for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done    # перемонтировать каталоги из цикла в /mnt

chroot /mnt/                                        # режим изоляции

grub2-mkconfig -o /boot/grub2/grub.cfg              # конфигурируем grub

cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done          #обновим initrd

# по рекомендации из меточки, не перезагружаемся и не выходим из под chroot - мы можем заодно перенести /var.

lsblk                                                                          # посмотрим на текущую ситуацию по дискам 

[root@lvm boot]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0    8G  0 lvm  /
sdb                       8:16   0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk

# на свободных дисках создаём зеркало
pvcreate /dev/sdc /dev/sdd
vgcreate vg_var /dev/sdc /dev/sdd
lvcreate -L 950M -m1 -n lv_var vg_var

mkfs.ext4 /dev/vg_var/lv_var                                                    # создаём на lv_var новую файловую систему

mount /dev/vg_var/lv_var /mnt                                                   # монтируем lv_var в /mnt

df -Th                                                                          # проверяем

[root@lvm boot]# df -Th
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs       8.0G  899M  7.2G  11% /
devtmpfs                        devtmpfs  236M     0  236M   0% /dev
tmpfs                           tmpfs     244M  4.6M  240M   2% /run
/dev/sda2                       xfs      1014M   61M  954M   6% /boot
/dev/mapper/vg_var-lv_var       ext4      922M  2.4M  856M   1% /mnt

cp -aR /var/* /mnt/                                                             # копируем содержимое /var на /lv_var

mkdir /tmp/oldvar && mv /var/* /tmp/oldvar                                      # я считаю, что делать это не обязательно на лаборатоной машине (можно просто удалить), но в реальных боевых условиях это необходимо, потому сделаю это. 

umount /mnt                                                                     # отмонтируем lv_var
mount /dev/vg_var/lv_var /var                                                   # примонтируем lv_var в каталог /var

df -Th                                                                          # проверяем

[root@lvm boot]# df -Th
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs       8.0G  899M  7.2G  11% /
devtmpfs                        devtmpfs  236M     0  236M   0% /dev
tmpfs                           tmpfs     244M  4.6M  240M   2% /run
/dev/sda2                       xfs      1014M   61M  954M   6% /boot
/dev/mapper/vg_var-lv_var       ext4      922M  272M  587M  32% /var

# редактируем /etc/fstab
echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
sed -i 's/vg_root-lv_root/VolGroup00-LogVol00/g' /etc/fstab

exit                                                                          # Выходим из chroot
reboot                                                                        # перезагружаемся

df -Th                                                                        # проверяем

[root@lvm vagrant]# df -Th
Filesystem                      Type      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00 xfs       8.0G  899M  7.2G  11% /
devtmpfs                        devtmpfs  236M     0  236M   0% /dev
tmpfs                           tmpfs     244M     0  244M   0% /dev/shm
tmpfs                           tmpfs     244M  4.6M  240M   2% /run
tmpfs                           tmpfs     244M     0  244M   0% /sys/fs/cgroup
/dev/sda2                       xfs      1014M   61M  954M   6% /boot
/dev/mapper/vg_var-lv_var       ext4      922M  272M  586M  32% /var
tmpfs                           tmpfs      49M     0   49M   0% /run/user/1000

# удаляем веременную Volume Group
lvremove /dev/vg_root/lv_root
vgremove /dev/vg_root
pvremove /dev/sdb

# добавляем /home раздел (проделываем тоже, что и с /var выше)
lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
mkfs.xfs /dev/VolGroup00/LogVol_Home
mount /dev/VolGroup00/LogVol_Home /mnt/
cp -aRv /home/* /mnt/
rm -rf /home/*
umount /mnt
mount /dev/VolGroup00/LogVol_Home /home/
echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab

# работа со снапшотами

touch /home/file{1..20}                                                       # генерируем файлы в /home

ls -lah /home                                                                 # проверяем

[root@lvm vagrant]# ls -lah /home/
total 0
drwxr-xr-x   3 root    root    292 May  1 18:03 .
drwxr-xr-x. 18 root    root    239 May  1 14:01 ..
-rw-r--r--   1 root    root      0 May  1 18:03 file1
-rw-r--r--   1 root    root      0 May  1 18:03 file10
-rw-r--r--   1 root    root      0 May  1 18:03 file11
-rw-r--r--   1 root    root      0 May  1 18:03 file12
-rw-r--r--   1 root    root      0 May  1 18:03 file13
-rw-r--r--   1 root    root      0 May  1 18:03 file14
-rw-r--r--   1 root    root      0 May  1 18:03 file15
-rw-r--r--   1 root    root      0 May  1 18:03 file16
-rw-r--r--   1 root    root      0 May  1 18:03 file17
-rw-r--r--   1 root    root      0 May  1 18:03 file18
-rw-r--r--   1 root    root      0 May  1 18:03 file19
-rw-r--r--   1 root    root      0 May  1 18:03 file2
-rw-r--r--   1 root    root      0 May  1 18:03 file20
-rw-r--r--   1 root    root      0 May  1 18:03 file3
-rw-r--r--   1 root    root      0 May  1 18:03 file4
-rw-r--r--   1 root    root      0 May  1 18:03 file5
-rw-r--r--   1 root    root      0 May  1 18:03 file6
-rw-r--r--   1 root    root      0 May  1 18:03 file7
-rw-r--r--   1 root    root      0 May  1 18:03 file8
-rw-r--r--   1 root    root      0 May  1 18:03 file9
drwx------.  3 vagrant vagrant  74 May 12  2018 vagrant

lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home                 # снимаем снапшот

rm -f /home/file{11..20}                                                      #удалим часть файлов

ls -lah /home                                                                 # проверим

[root@lvm vagrant]# ls -lah /home/
total 0
drwxr-xr-x   3 root    root    152 May  1 18:11 .
drwxr-xr-x. 18 root    root    239 May  1 14:01 ..
-rw-r--r--   1 root    root      0 May  1 18:03 file1
-rw-r--r--   1 root    root      0 May  1 18:03 file10
-rw-r--r--   1 root    root      0 May  1 18:03 file2
-rw-r--r--   1 root    root      0 May  1 18:03 file3
-rw-r--r--   1 root    root      0 May  1 18:03 file4
-rw-r--r--   1 root    root      0 May  1 18:03 file5
-rw-r--r--   1 root    root      0 May  1 18:03 file6
-rw-r--r--   1 root    root      0 May  1 18:03 file7
-rw-r--r--   1 root    root      0 May  1 18:03 file8
-rw-r--r--   1 root    root      0 May  1 18:03 file9
drwx------.  3 vagrant vagrant  74 May 12  2018 vagrant

# процесс восстановления со снапшота

umount /home
lvconvert --merge /dev/VolGroup00/home_snap
mount /home

ls -lah /home                                                                 # проверим

[root@lvm vagrant]# ls -lah /home/
total 0
drwxr-xr-x   3 root    root    292 May  1 18:03 .
drwxr-xr-x. 18 root    root    239 May  1 14:01 ..
-rw-r--r--   1 root    root      0 May  1 18:03 file1
-rw-r--r--   1 root    root      0 May  1 18:03 file10
-rw-r--r--   1 root    root      0 May  1 18:03 file11
-rw-r--r--   1 root    root      0 May  1 18:03 file12
-rw-r--r--   1 root    root      0 May  1 18:03 file13
-rw-r--r--   1 root    root      0 May  1 18:03 file14
-rw-r--r--   1 root    root      0 May  1 18:03 file15
-rw-r--r--   1 root    root      0 May  1 18:03 file16
-rw-r--r--   1 root    root      0 May  1 18:03 file17
-rw-r--r--   1 root    root      0 May  1 18:03 file18
-rw-r--r--   1 root    root      0 May  1 18:03 file19
-rw-r--r--   1 root    root      0 May  1 18:03 file2
-rw-r--r--   1 root    root      0 May  1 18:03 file20
-rw-r--r--   1 root    root      0 May  1 18:03 file3
-rw-r--r--   1 root    root      0 May  1 18:03 file4
-rw-r--r--   1 root    root      0 May  1 18:03 file5
-rw-r--r--   1 root    root      0 May  1 18:03 file6
-rw-r--r--   1 root    root      0 May  1 18:03 file7
-rw-r--r--   1 root    root      0 May  1 18:03 file8
-rw-r--r--   1 root    root      0 May  1 18:03 file9
drwx------.  3 vagrant vagrant  74 May 12  2018 vagrant



