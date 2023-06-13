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
```
Вставить в конец файла 
```
conn enc_gre
  	auto=start
	authby=secret
	type=tunnel
	ike=aes-sha1;modp2048
	left=4.4.4.100
	leftprotoport=gre
	right=5.5.5.100
	rightprotoport=gre
```
```
nano /etc/ipsec.secrets
```
```
4.4.4.100 5.5.5.100 : PSK “pass”
```
```
systemctl enable ipsec
```
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
## iptables
### RTR-L
```
apt install -y iptables
iptables -A INPUT -i ens192 -p icmp -j ACCEPT
iptables -A INPUT -i ens192 -p gre -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -i ens192 -p udp --dport 500 -j ACCEPT
iptables -A INPUT -i ens192 -p udp --dport 4500 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables -t nat -A PREROUTING -i ens192 -p udp --dport 2244 -j DNAT --to-destination 192.168.200.100:22
iptables -t nat -A PREROUTING -i ens192 -p udp --dport 53 -j DNAT --to-destination 192.168.200.200
iptables -t nat -A PREROUTING -s 3.3.3.0/24 -j DNAT --to-destination 192.168.200.200 (docker)
iptables -t nat -A PREROUTING -s 3.3.3.0/24 -j DNAT --to-destination 192.168.200.100 (docker)
iptables -A INPUT -m conntrack -ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack -ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens192 -j DROP
apt install -y iptables-persistent
```
### RTR-R
```
apt install -y iptables
iptables -A INPUT -i ens192 -p icmp -j ACCEPT
iptables -A INPUT -i ens192 -p gre -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -i ens192 -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -i ens192 -p udp --dport 500 -j ACCEPT
iptables -A INPUT -i ens192 -p udp --dport 4500 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables -t nat -A PREROUTING -i ens192 -p tcp –dport 2222 -j DNAT --to-destination 172.16.100.100:22
iptables -t nat -A PREROUTING -s 3.3.3.0/24 -j DNAT –to-destination 172.16.100.100
iptables -A INPUT -m conntrack -ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack -ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens192 -j DROP
apt install -y iptables-persistent
```
## Docker
### WEB-L WEB-R
add docker.iso
```
apt install docker*
systemctl enable docker
mkdir /mnt/docker
mount /dev/sr0 /mnt/docker
docker image load -i /mnt/docker/app.tar
docker run -d -p 80:80 --restart=always app:latest
```
## DNS
### ISP
```
apt install -y bind9
nano /etc/bind/named.conf.options
```
Вставить
```
dnssec-validation no;
```
```
nano /etc/bind/named.conf.default-zones
```
В конце пишем
```
zone "demo.wsr" {
	type master;
	file "/etc/bind/db.demo";
};
```
```
cp /etc/bind/db.127 /etc/bind/db.demo
nano /etc/bind/db.demo
```
```
$TTL	10
@		IN	SOA	demo.wsr. root.demo.wsr. (
			1		; Serial
			604800		; Refresh
			86400		; Retry
			2413200		; Expire
			604800  )	; Negative Cache TTL
;
@		IN	NS	demo.wsr.			
		IN	A	3.3.3.1
isp		IN	A	3.3.3.1
www		IN	A	4.4.4.100
		IN	A	5.5.5.100
internet	IN	CNAME	isp

int		IN	NS	int.demo.wsr.
		IN	A	4.4.4.100
```
```
rndc reload
systemctl restart bind 9	
systemctl enable bind9
```
journalctl -xe в помощь если ошибки
### SRV
```
apt install -y bind9
nano /etc/bind/named.conf.options
```








