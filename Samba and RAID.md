## Samba
## RAID
### SRV
```
apt install -y mdadm
mdadm --create /dev/md0 -l 1 -n2 2 /dev/sdb /dev/sdc
mdadm --detail --scan --verbose | tee -a /etc/mdadm/mdadm.conf
mkfs.ext4 -L ourraid /dev/md0
mkdir /mnt/storage
mount /dev/md0 /mnt/storage
nano /etc/fstab
```
```
LABEL=ourraid 	/mnt/storage 	ext4 	defaults	0	2
```
```
apt install -y samba
nano /etc/samba/smb.conf
```
```
[web]
path=/mnt/storage
browsable=yes
writable=yes
guest ok=yes
read only=no
public=yes
create mask=0777
directory mask = 0777
force create mode = 0777
force directory mode = 0777
```
```
chmod 777 -R /mnt/storage
systemctl restart smdb
systemctl enable smdb
```
### WEB-L WEB-R
```
apt install -y cifs-utils
nano /etc/fstab
```
```
//192.168.200.200/web	/opt/storage	cifs	rw	0	0
```
```
systemctl enable cifs-utils
```
