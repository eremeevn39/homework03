# Homework03 :grinning:

1. [**1-4 таски ДЗ:**](#1-1-4-%D1%82%D0%B0%D1%81%D0%BA%D0%B8-%D0%B4%D0%B7)
scriptreplay 1.timefile 1.typescript
1. [**5 таск ДЗ:**](#2-5-%D1%82%D0%B0%D1%81%D0%BA-%D0%B4%D0%B7-%D0%BF%D1%80%D0%BE%D0%BF%D0%B8%D1%81%D1%8B%D0%B2%D0%B0%D0%B5%D0%BC-%D0%BC%D0%BE%D0%BD%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B2-fstab-%D1%81-%D1%80%D0%B0%D0%B7%D0%BD%D1%8B%D0%BC%D0%B8-%D0%BE%D0%BF%D1%86%D0%B8%D1%8F%D0%BC%D0%B8-%D0%B8-%D1%80%D0%B0%D0%B7%D0%BD%D1%8B%D0%BC%D0%B8-%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2%D1%8B%D0%BC%D0%B8-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0%D0%BC%D0%B8-ext4-%D0%B8-xfs)
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

Перезагружаемся. Меняем размер старой **VG** и вовзращаем на него рут. Удаляем старый **LV** размеров в **40G** и создаем новый на **8G**:

`# lvremove /dev/VolGroup00/LogVol00`

`# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00`

Создаем файловую систему, монтируем, возвращаем данные обратно:

```# mkfs.xfs /dev/VolGroup00/LogVol00 && mount /dev/VolGroup00/LogVol00 /mnt && xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt```

Переконфигурируем grub

`# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done`

`# chroot /mnt/`

`# grub2-mkconfig -o /boot/grub2/grub.cfg`

```# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done```

2. **Выделяем том под __/var__ в зеркало**

Создаем зеркало:

`# pvcreate /dev/sdc /dev/sdd`

`# vgcreate vg_var /dev/sdc /dev/sdd`

`# lvcreate -L 950M -m1 -n lv_var vg_var`

Создаем на нем ФС и перемещаем туда **/var**:

`# mkfs.ext4 /dev/vg_var/lv_var`
`# mount /dev/vg_var/lv_var /mnt`
`# cp -aR /var/* /mnt/      # rsync -avHPSAX /var/ /mnt/`

Монтируем новый **var** в каталог **/var**:

`# umount /mnt`

`# mount /dev/vg_var/lv_var /var`

Правим **fstab** для автоматического монтирования **/var**:

```# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab```

Перезагружаемся и удаляем временную **Volume Group**:

`# lvremove /dev/vg_root/lv_root`
`# vgremove /dev/vg_root`
`# pvremove /dev/sdb`

3. Выделяем том под **/home**:

Создаем логический раздел:

`# lvcreate -n LogVol_Home -L 2G /dev/VolGroup00`

Создаем ФС:

`# mkfs.xfs /dev/VolGroup00/LogVol_Home`

`# mount /dev/VolGroup00/LogVol_Home /mnt/`

`# cp -aR /home/* /mnt/`

`# rm -rf /home/*`

`# umount /mnt`

`# mount /dev/VolGroup00/LogVol_Home /home/`

Правим **fstab** для автоматического монтирования **/home**

```# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab```

3. Делаем том для снапшотов **/home**:

Сгенерируем файлы в **/home/**:

`# touch /home/file{1..20}`

Делаем снапшот:

`# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home`

Удаляем часть файлов:

`# rm -f /home/file{11..20}`

Восстанавливаемся со снапшота:

`# umount /home`

`# lvconvert --merge /dev/VolGroup00/home_snap`

`# mount /home`

## №2 5 таск ДЗ: Прописываем монтирование в **fstab** с разными опциями и разными файловыми системами (**ext4** и **xfs**)
Создаем разделы **LVM**:

`# pvcreate /dev/sdb`
`# vgcreate hw03 /dev/sdb`
`# lvcreate -l+50%FREE -n hw03-01 hw03`
`# lvcreate -l+100%FREE -n hw03-02 hw03`

Форматируем разделы в разные ФС:

`# mkfs.ext4 /dev/hw03/hw03-01`
`# mkfs.xfs /dev/hw03/hw03-02`

Создаем каталоги и монтируемся в них:

`# mkdir -p /mount/hw03-01 /mount/hw03-02`
`# mount /dev/hw03/hw03-01 /mount/hw03-01`
`# mount /dev/hw03/hw03-02 /mount/hw03-02`

Добавляем параметры автомонтирования в **fstab**:

```# echo "`blkid | grep hw03-hw03--01: | awk '{print $2}'` /mount/hw03-01 ext4 defaults 0 0" >> /etc/fstab```
```# echo "`blkid | grep hw03-hw03--02: | awk '{print $2}'` /mount/hw03-02 xfs defaults 0 2" >> /etc/fstab```

Проверяем содержимое **fstab** на всякий случай:

`# cat /etc/fstab`
