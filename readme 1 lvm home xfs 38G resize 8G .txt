Шаги по уменьшению тома под / (root) до 8G
	1. добавить новый диск в существующий pv
	2. расширить существующий vg на добавленный диск из pv
	3. создать lv размером 8 Гб в существующем vg на новом диске pv
	4. создать файловую систему xfs на lv 8 Гб и примонтировать
	5. переместить данные / (root) на новый lv 8 Гб
	6. править конфиги: grub, initrd, fstab - на новый lv и перезагрузиться
	7. удалить старый lv / (root) на 40 Гб
	8. переместить на pv удаленного lv 40 Гб внутри vg новый lv 8 Гб  
	9. удалить pv из lvm,который был использован для нового lv 8 Гб и сейчас пустой после перемещения
--------->%-------------
mkdir /home/resizehome
cd /home/resizehome
script -t 2> timingfile
--------->%-------------
# Подготовка 
yum install xfsdump

# Уменьшить раздел под / (root) до 8 Гб

df -hT
vgs
lvs
lsblk
lvmdiskscan
# Добавить диск в существующий pv
pvcreate /dev/sdb
# Проверить
lvmdiskscan
pvs -va
# Расширить существующий vg
vgextend VolGroup00 /dev/sdb
# Проверить free space 10 Гб в pv
vgs
# Создать lv в существующем vg
lvcreate -n LogVol02 -L 8G VolGroup00
# Разметить ФС xfs 
mkfs.xfs /dev/mapper/VolGroup00-LogVol02
lsblk -f

# Перенести / (root) 

mkdir /mnt/LogVol02
ll /mnt/
lvs
mount /dev/VolGroup00/LogVol02 /mnt/LogVol02
df -hT
xfsdump -J - / | xfsrestore -J - /mnt/LogVol02
ll /mnt/LogVol02

# В chroot запись не работает.
# chroot --> grub2 --> initrd --> grub.cfg --> reboot


# chroot - grub2
Опция --bind позволяет смонтировать часть файловой структуры в другой каталог, не удаляя при этом исходную точку монтирования.
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/LogVol02/$i; done
# exit для script + save

chroot /mnt/LogVol02/
# chroot /mnt/LogVol02/
grub2-mkconfig -o /boot/grub2/grub.cfg

# chroot - initrd
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
s/.img//g"` --force; done 

# chroot - grub.cfg
cp /boot/grub2/grub.cfg /boot/grub2/grub.bkp1
cat /boot/grub2/grub.cfg | grep rd.lvm.lv
# Для того, чтобы при загрузке был смонтирован / (root) нужно в файле
#     заменить rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=VolGroup00/LogVol02
vi /boot/grub2/grub.cfg 
# Проверить
cat /boot/grub2/grub.cfg | grep rd.lvm.lv
# Выйти из chroot
exit
reboot

# cd /home/resizehome
# script -a typescript --timing=timingfile

# После reboot
df -hT
lsblk

# Удалить lv00 40 Гб и pvmove 
lvs
lvremove /dev/mapper/VolGroup00-LogVol00

lsblk
# Переместить lv на sda3 на место удаленного lv 38 Гб
pvmove -n VolGroup00/LogVol02 /dev/sdb /dev/sda3
lsblk

# Удалить пустой pv /dev/sdb из vg 
pvs
# -a (-all) удалить все пустые pv из vg 
vgreduce --all VolGroup00
vgs
pvs
# Удалить pv /dev/sdb
pvremove  /dev/sdb --verbose
pvs
# Готово

