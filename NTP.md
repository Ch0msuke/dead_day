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
systemctl restart chronyd
```
### RTR-L RTR-R WEB-L WEB-R
```
nano /etc/chrony/chrony.conf
```
Вместо строки pool пишем
```
server 192.168.200.200
systemctl restart chronyd
```
### CLI
ntp server 3.3.3.1
