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
