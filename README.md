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
