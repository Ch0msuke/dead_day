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
