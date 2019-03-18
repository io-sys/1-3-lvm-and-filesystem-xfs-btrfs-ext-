script -t 2> timingfile
scriptreplay timingfile
script -a typescript --timing=timingfile
less typescript

mkdir /tmp/homesnap
cd /tmp/homesnap
-->%-------------------

# �������� ��������� lv ��� /home 
lvcreate -n LogVolHOME -L 2G /dev/VolGroup00
# ������������� ��� � ext4
mkfs.ext4 /dev/VolGroup00/LogVolHOME
# �����������
mkdir /mnt/LogVolHOME
mount /dev/VolGroup00/LogVolHOME /mnt/LogVolHOME
# ���������� ���������� /home �� ����� lv
cp -aR /home/* /mnt/LogVolHOME
ll /mnt/LogVolHOME/
ll /home/
rm -rf /home/*
ll /home/
umount /mnt/LogVolHOME
mount /dev/VolGroup00/LogVolHOME /home
# ������� /etc/fstab ��� ��������������� ������������ /home
echo "`blkid | grep HOME | awk '{print $2}'` /home ext4 defaults 0 0" >> /etc/fstab 

# reboot
ll /home/
df -hT

# ��������� ��������.

# ������� ����� � /home/
touch /home/file{1..20}
ll /home/
# ����� �������
lvcreate -s -L 50MB -n homedir_snap VolGroup00/LogVolHOME
# �������� % �������� ����� � ��������.
lvs
# ������� ����� ������.
rm -f /home/file{11..20}
ll /home/
# �������������� �� ��������
umount /home
# lsof | grep '/home'
# kill -9 1343
# ������������� � snap �������� ���.
lvconvert --merge /dev/VolGroup00/homedir_snap
mount /home
# reboot
