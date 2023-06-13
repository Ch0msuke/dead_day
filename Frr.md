## Frr
### RTR-L
```
apt install -y frr
nano /etc/frr/daemons 
```
```
ospfd=yes
zebra=yes
```
```
reboot
systemctl restart frr	
vtysh
conf t
router ospf
network 192.168.200.0/24 area 0
network 10.10.10.0/30 area 0
do wr
exit
```
### RTR-R
```
apt install -y frr
nano /etc/frr/daemons 
```
```
ospfd=yes
zebra=yes
```
```
reboot
systemctl restart frr
vtysh
conf t
router ospf
network 172.16.100.0/24 area 0
network 10.10.10.0/30 area 0
do wr
exit
```
