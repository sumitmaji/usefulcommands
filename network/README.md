
- Tcpdump
```shell
tcpdump -nn -q -i eth0
tcpdump -A -vvv -s 9999 -i eth1 port 80 > /tmp/headers
```
- netstat
```shell
netstat -an | more
netstat -lnp
netstat -u -ap
netstat -tulpn
```
- lsof
```shell
lsof -i :389
```
- namp
```shell
namp -p 80 192.168.1.100
```

- Ip Tables
```shell
iptables -L -n
iptables -t filter -L -n
iptables -t filter -L -v
iptables -t filter -S
```
- Deleting a rule in ip table
```shell
iptables -t nat -D PREROUTING -p tcp --dport 9090 -j DNAT --to-destination 11.0.0.4:8080
```

- Adding rule ip table
```shell
iptables -t nat -A PREROUTING -p tcp --dport 32319 -j DNAT --to-destination 11.0.0.2:32319
iptables -t nat -A PREROUTING -p tcp --dport 30001 -j DNAT --to-destination 11.0.0.2:30001
iptables -t nat -A PREROUTING -p tcp --dport 30340 -j DNAT --to-destination 11.0.0.2:30340
```

- netsh
```shell
netsh interface portproxy add v4tov4 listenport=433 listenaddress=192.168.1.100 connectport=433 connectaddress=192.168.1.101
netsh interface portproxy delete v4tov4 listenport=5002 listenaddress=192.168.1.5
```