Как работает LVM Mirror
	1. создать vg из двух дисков, 
	2. создать раздел внутри vg при указании опции -m1 будут созданы 2тома на разных pv
-->%-------------------

# /var на LVM Mirror
lsblk
# Записать метаданные на диски sdd и sde
pvcreate /dev/sdd /dev/sde
pvs
# Создать vg из 2 дисков с именем VolGroupVAR
vgcreate VolGroupVAR /dev/sdd /dev/sde
vgs
# Создать raid1 на lvm из 2 дисков
lvcreate --type raid1 --mirrors 1  -L 900M  -n LogVolVAR VolGroupVAR /dev/sdd /dev/sde
lsblk
lvdisplay

# Форматировать lv где raid1 в ФС ext4 для копирования на том директорию /var
mkfs.ext4 /dev/VolGroupVAR/LogVolVAR

mkdir /mnt/LogVolVAR
ll /mnt/LogVolVAR
mount /dev/VolGroupVAR/LogVolVAR /mnt/LogVolVAR
df -hT
# Копировать -a всё -R рекурсивно
cp -aR /var/* /mnt/LogVolVAR
ll /mnt/LogVolVAR/
# Cодержимое старого /var в папку /tmp/oldvar 
mkdir /tmp/oldvar && mv /var/* /tmp/oldvar 
# Монтировать новый /var на raid 1 lvm в каталог /var 
umount /mnt/LogVolVAR/
mount /dev/VolGroupVAR/LogVolVAR /var 
# Правка fstab для автоматического монтирования /var
echo "`blkid | grep VAR: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab

# reboot
lsblk
df -hT
lvmdiskscan
