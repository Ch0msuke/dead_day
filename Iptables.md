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
