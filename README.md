# Homework03 :grinning:

1. **1-4 таски ДЗ:**
scriptreplay 1.timefile 1.typescript
1. **5 таск ДЗ:**
scriptreplay 2.timefile 2.typescript

## №1 1-4 таски ДЗ:

1. **Уменьшаем том под / до 8G**

Первым делом установим пакет [xfsdump](https://linux.die.net/man/8/xfsdump) для снятия копии с тома на файловой системе **XFS**

`# yum install -y xfsdump`
    
Подготовим временный том для **/** раздела:
   
`# pvcreate /dev/sdb`

`# vgcreate vg_root /dev/sdb`

`# lvcreate -n lv_root -l +100%FREE /dev/vg_root`

Создадим на нем файловую систему и смонтируем его, чтобы перенести туда данные:

`# mkfs.xfs /dev/vg_root/lv_root`

`# mount /dev/vg_root/lv_root /mnt`

Cкопируем все данные с **/** раздела в **/mnt**:

`# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt`

Переконфигурируем grub для того, чтобы при старте перейти в новый **/**

Сымитируем текущий **root** -> сделаем в него [chroot](https://wiki.archlinux.org/index.php/Chroot_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)) и обновим **grub** и обновим образ [initrd](https://ru.wikipedia.org/wiki/Initrd):

`# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done`

`# chroot /mnt/`

`# grub2-mkconfig -o /boot/grub2/grub.cfg`

```# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done```

Далее, для того, чтобы при загрузке был смонтирован нужный **root** нужно в файле */boot/grub2/grub.cfg* заменить

> rd.lvm.lv=VolGroup00/LogVol00

на

> rd.lvm.lv=vg_root/lv_root

Перезагружаемся. Меняем размер старой VG и вовзращаем на него рут. Удаляем старый LV размеров в 40G и создаем новый на 8G:

`# lvremove /dev/VolGroup00/LogVol00`

`# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00`

Создаем файловую систему, монтируем, возвращаем данные обратно:

```# mkfs.xfs /dev/VolGroup00/LogVol00 && mount /dev/VolGroup00/LogVol00 /mnt && xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt```

Переконфигурируем grub

`# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done`

`# chroot /mnt/`

`# grub2-mkconfig -o /boot/grub2/grub.cfg`

```# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done```

2. **Выделяем том под /var в зеркало**
