script -t 2> timingfile
scriptreplay timingfile
script -a typescript --timing=timingfile
less typescript

mkdir /tmp/homesnap
cd /tmp/homesnap
-->%-------------------

# Выделить отдельный lv под /home 
lvcreate -n LogVolHOME -L 2G /dev/VolGroup00
# Форматировать том в ext4
mkfs.ext4 /dev/VolGroup00/LogVolHOME
# Монтировать
mkdir /mnt/LogVolHOME
mount /dev/VolGroup00/LogVolHOME /mnt/LogVolHOME
# Копировать директорию /home на новый lv
cp -aR /home/* /mnt/LogVolHOME
ll /mnt/LogVolHOME/
ll /home/
rm -rf /home/*
ll /home/
umount /mnt/LogVolHOME
mount /dev/VolGroup00/LogVolHOME /home
# Править /etc/fstab для автоматического монтирования /home
echo "`blkid | grep HOME | awk '{print $2}'` /home ext4 defaults 0 0" >> /etc/fstab 

# reboot
ll /home/
df -hT

# Проверить снепшоты.

# Создать файлы в /home/
touch /home/file{1..20}
ll /home/
# Снять снепшот
lvcreate -s -L 50MB -n homedir_snap VolGroup00/LogVolHOME
# Показать % занятого места в снепшоте.
lvs
# Удалить часть файлов.
rm -f /home/file{11..20}
ll /home/
# Восстановиться из снапшота
umount /home
# lsof | grep '/home'
# kill -9 1343
# Восстановится и snap удалится сам.
lvconvert --merge /dev/VolGroup00/homedir_snap
mount /home
# reboot
