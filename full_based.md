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




























