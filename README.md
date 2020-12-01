# Homework03 :grinning:

1. **1-4 таски ДЗ:**
scriptreplay 1.timefile 1.typescript
1. **5 таск ДЗ:**
scriptreplay 2.timefile 2.typescript

## №1 1-4 таски ДЗ:

1. **Уменьшить том под / до 8G**
   1. Первым делом установим пакет xfsdump для снятия копии с тома на файловой системе XFS

```# yum install -y xfsdump```
    
   1. Подготовим временный том для / раздела:
   
```# pvcreate /dev/sdb```

```# vgcreate vg_root /dev/sdb```

```# lvcreate -n lv_root -l +100%FREE /dev/vg_root```
