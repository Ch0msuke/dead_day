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
