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
Раскомментить 
```
forwarders {
	4.4.4.1;
};
```
Вставить в конце
```
dnssec-validation no;
	recursion yes;
	allow-recursion {internal;};
	liste-on-v6 { any; };
};
include "/etc/bind/internal.acl"; 
```
```
nano /etc/bind/internal.acl
```
```
acl "internal" {
	172. 16.100.0/24;
	192.168.100.0/24;
	10.10.10.0/30;
	127.0.0.0/8;
};
```
```
nano /etc/bind/named.conf.default-zones
```
```
zone “int.demo.wsr” {
	type master;
	file “/etc/bind/db.int.demo”;
        };
zone “100.168.192.in-addr.arpa” {
	type master;
	allow-query {internal;};
	file “/etc/bind/db.192”;
        };
zone “100.16.172.in-addr.arpa” {
	type master;
	allow-query {internal;};
	file “/etc/bind/db.172”;
        };
```
```
cp /etc/bind/db.127 /etc/bind/db.int.demo
nano /etc/bind/db.int.demo
```
```
$TTL	604800
@	IN	SOA	int.demo.wsr. root.int.demo.wsr. (
			1		; Serial
			604800		; Refresh
			86400		; Retry
			2413200		; Expire
			604800  )	; Negative Cache TTL
;
@	IN	NS	int.demo.wsr.			
	IN	A	192.168.100.200
web-l	IN	A	192.168.100.100
web-r	IN	A	172.16.100.100
srv	IN	A	192.168.100.200
rtr-l	IN	A	192.168.100.254
rtr-r	IN	A	172.16.100.254
webapp	IN	A	172.16.100.100
	IN	A	192.168.100.100
ntp	IN	CNAME	srv
dns	IN	CNAME	srv
```
```
cp /etc/bind/db.int.demo /etc/bind/db.192
nano /etc/bind/db.192
```
```
$TTL	604800
@	IN	SOA	int.demo.wsr. root.int.demo.wsr. (
			1		; Serial
			604800		; Refresh
			86400		; Retry
			2413200		; Expire
			604800  )	; Negative Cache TTL
;
@	IN	NS	int.demo.wsr.			
	IN	A	192.168.100.200
100	IN	PTR	web-l.int.demo.wsr.
254	IN	PTR	rtr-l.int.demo.wsr.
200	IN	PTR	srv.int.demo.wsr.

```
```
cp /etc/bind/db.192 /etc/bind/db.172
nano /etc/bind/db.172
```
```
$TTL	604800
@	IN	SOA	int.demo.wsr. root.int.demo.wsr. (
			1		; Serial
			604800		; Refresh
			86400		; Retry
			2413200		; Expire
			604800  )	; Negative Cache TTL
;
@	IN	NS	int.demo.wsr.			
	IN	A	192.168.100.200
100	IN	PTR	web-r.int.demo.wsr.
254	IN	PTR	rtr-r.int.demo.wsr.
```
```
rndc reload 
systemctl restart bind9
systemctl enable bind9
```

## NTP
### ALL
```
apt install -y chrony
systemctl enable chrony
```
### ISP
```
nano /etc/chrony/chrony.conf
```
Вместо строки pool пишем
```
server 127.127.1.0 iburst
local stratum 3
allow 4.4.4.100
allow 3.3.3.10
allow 5.5.5.100
```
```
systemctl restart chronyd
```
### SRV
```
nano /etc/chrony/chrony.conf
```
```
server 4.4.4.1 iburst
allow 192.168.100.0/24
allow 172.16.100.0/24
allow 10.10.10.0/30
```
```
systemctl restart chronyd
```
### RTR-L RTR-R WEB-L WEB-R
```
nano /etc/chrony/chrony.conf
```
Вместо строки pool пишем
```
server 192.168.200.200
```
```
systemctl restart chronyd
```
### CLI
ntp server 3.3.3.1

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
## SSH
### ALL
```
apt install -y  openssh-server ssh
systemctl start sshd
systemctl status sshd
```
подключение
```
ssh root@"name/IP"
```














