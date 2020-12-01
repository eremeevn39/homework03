# Homework03 :grinning:

1. **1-4 таски ДЗ:**
scriptreplay 1.timefile 1.typescript
1. **5 таск ДЗ:**
scriptreplay 2.timefile 2.typescript

## №1 1-4 таски ДЗ:

1. **Уменьшить том под / до 8G**

Первым делом установим пакет xfsdump для снятия копии с тома на файловой системе XFS

`# yum install -y xfsdump`
    
Подготовим временный том для / раздела:
   
`# pvcreate /dev/sdb`

`# vgcreate vg_root /dev/sdb`

`# lvcreate -n lv_root -l +100%FREE /dev/vg_root`

Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные:

`# mkfs.xfs /dev/vg_root/lv_root`

`# mount /dev/vg_root/lv_root /mnt`

Cкопируем все данные с / раздела в /mnt:

`# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt`

Переконфигурируем grub для того, чтобы при старте перейти в новый /

Сымитируем текущий **root** -> сделаем в него [chroot](https://wiki.archlinux.org/index.php/Chroot_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)) и обновим **grub**:

`# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done`

`# chroot /mnt/`

`# grub2-mkconfig -o /boot/grub2/grub.cfg`
