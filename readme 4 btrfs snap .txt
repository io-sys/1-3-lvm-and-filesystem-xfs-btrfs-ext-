script -t 2> timingfile
scriptreplay timingfile
script -a typescript --timing=timingfile
less typescript

mkdir /home/btrfssnap
cd /home/btrfssnap
-->%-------------------

# * поставить btrfs - с кешем, снэпшотами - разметить здесь каталог /opt

# Форматировать в btrfs диск sdb
mkfs.btrfs -L opt /dev/sdb
lsblk -f
# Показать
btrfs filesystem show
# Для монтирования каталог
mkdir /mnt/btrfs
# Монтировать с кешем
mount -o space_cache /dev/sdb /mnt/btrfs
cd /mnt/btrfs

# Создать раздел btrfs 
btrfs subvolume create @opt
# Показать все subvolume заделы btrfs 
btrfs subvolume list -a /mnt/btrfs/
btrfs filesystem du @opt -h
ll /opt/

# Перенос директории /opt на созданный и смонтированный раздел 
ll /opt/
# Каталог пустой копировать не надо.
# Копировать -a (всё) -R (рекурсивно)
cp -aR /opt/* /mnt/btrfs/@opt/    # rsync -avHPSAX /var/ /mnt/  # пакет ставить

# править fstab для автоматического монтирования /opt
blkid
df -hT
blkid | grep opt | awk '{print $3}'
# Нужно монтировать btrfs раздел @opt в директорию /opt с кешем
echo "`blkid | grep opt | awk '{print $3}'` /opt btrfs defaults,space_cache,subvol=@opt 0 0" >> /etc/fstab

cat /etc/fstab
df -hT
# reboot

# тест снапшотов btrfs 

cd /opt/
# Создать файлы для теста
touch file1 file2 file3 file4
echo 'Test file' > file5

# Создать снепшот
	# /tmp/0/@ - с чего снять снепшот @ (/)
	# /tmp/0/@snap - имя subvol в котором будет снепшот 
btrfs subvol snapshot /opt/ ./snap
ll
pwd
# Удалить
rm -f file5
ll ./snap/
# Восстановить из снепшота удаленный файл.
cp ./snap/file5 .
ll
cat file5

# Показать все разделы со снапшотами в директории /opt
btrfs subvol list -a /opt/
# Удалить раздел со снапшотом snap
btrfs subvol delete --verbose snap
btrfs filesystem du /opt/
