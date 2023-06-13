## IP
### ALL
```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.conf
sysctl -p

```
```
hostnamectl set-hostname “name_VM”
```
### ALL
```
nano /etc/resolv.conf 
```
### ISP
```
nameserver 3.3.3.1
search demo.wsr
```
### ALL
```
nameserver 192.168.200.200
search int.demo.wsr
```
### ALL
```
echo "4.4.4.100 RTR-L" >> /etc/hosts 
echo "192.168.200.254 RTR-L" >> /etc/hosts 
echo "10.10.10.2 RTR-L" >> /etc/hosts 
echo "5.5.5.100 RTR-R" >> /etc/hosts 
echo "172.16.100.254 RTR-R" >> /etc/hosts 
echo "10.10.10.1 RTR-R" >> /etc/hosts
echo "192.168.200.200 SRV" >> /etc/hosts 
echo "192.168.200.100 WEB-L" >> /etc/hosts 
echo "172.16.100.100 WEB-R" >> /etc/hosts 
echo "4.4.4.1 ISP" >> /etc/hosts 
echo "5.5.5.1 ISP" >> /etc/hosts 
echo "3.3.3.1 ISP" >> /etc/hosts 
echo "3.3.3.10 CLI" >> /etc/hosts 
```
### ALL
```
nano /etc/network/interfaces
```
### ISP
```
auto ens192
iface ens192 inet static
address 4.4.4.1
netmask 255.255.255.0

auto ens224
iface ens224 inet static
address 5.5.5.1
netmask 255.255.255.0

auto ens256
iface ens256 inet static
address 3.3.3.1
netmask 255.255.255.0
```
### RTR-L
```
auto ens224
iface ens224 inet static
address 192.168.200.254
netmask 255.255.255.0

auto ens192
iface ens192 inet static
address 4.4.4.100
netmask 255.255.255.0
gateway 4.4.4.1

auto tunnel1
iface tunnel1 inet tunnel
address 10.10.10.1
netmask 255.255.255.252
mode gre
local 4.4.4.100
endpoint 5.5.5.100
ttl 255
```
### RTR-R
```
auto ens224
iface ens224 inet static
address 172.16.100.254
netmask 255.255.255.0

auto ens192
iface ens192 inet static
address 5.5.5.100
netmask 255.255.255.0
gateway 5.5.5.1

auto tunnel1
iface tunnel1 inet tunnel
address 10.10.10.2
netmask 255.255.255.252
mode gre
local 5.5.5.100
endpoint 4.4.4.100
ttl 255
```
### SRV
```
auto ens192
iface ens192 inet static
address 192.168.200.200
netmask 255.255.255.0
gateway 192.168.200.254
```
### WEB-L
```
auto ens192
iface ens192 inet static
address 192.168.200.100
netmask 255.255.255.0
gateway 192.168.200.254
```
### WEB-R
```
auto ens192
iface ens192 inet static
address 172.16.100.100
netmask 255.255.255.0
gateway 172.16.100.254 
```
### ALL
```
systemctl restart networking
```
## IPsec
### RTR-L RTR-R
```
apt install -y libreswan
nano /etc/ipsec.conf
//вставить в конец файла 
conn enc_gre
  	auto=start
	authby=secret
	type=tunnel
	ike=aes-sha1;modp2048
	left=4.4.4.100
	leftprotoport=gre
	right=5.5.5.100
	rightprotoport=gre
nano /etc/ipsec.secrets
4.4.4.100 5.5.5.100 : PSK “pass”
systemctl enable ipsec
```
## Frr
### RTR-L
```
apt install -y frr
nano /etc/frr/daemons 
ospfd=yes
zebra=yes
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























