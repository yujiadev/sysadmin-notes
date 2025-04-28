# Manage Network

Private IPv4 address 
| Class  | IP Address Range | CIDR | Number of Addresses |
| :----- | :---------------- | :---- | ---------------- |
|  A     | 10.0.0.0-10.255.255.255 | 10.0.0.0/8 | 16777216 | 
|  B     | 172.16.0.0-172.31.255.255 | 172.16.0.0/12 | 1048576	|
|  C     | 192.168.0.0-192.168.255.255 | 192.168.0.0/16 | 65536 |

## Tools and Commands

| Command | Description |
| ------- | ----------- |
| ip [-s] link, ip addr | Shows the link status and IP address information for all network interfaces |
| ip addr add 192.168.56.150/24 dev eth0 | Assigns an IP address and netmask to the eth0 interface |
| ip neigh | Shows the ARP table |
| ip route | Display the routing table |
| ss -tupna | Shows all listening and non-listening socketts, along with the program to which to they belong |

ping 
traceroute

### Configure a Network Adapter with ip
```bash 
ip addr add 172.16.0.50/24 dev eth0
```
To configure permanent chagnes to the network configuration, use **nmcli** or manually modify the configuration files in **/etc/NetworkManager/system-connections**. (Not Recommanded)

|     Command       |           Description         |
| ----------------- | ----------------------------- |
| ip link set *eth0* up | Activates the specified interface |
| ip link set *eth0* down | Deactivates the specified interface |
| ip addr flush dev *eth0* | Removes all IP addresses from the specified interface |
| ip link set *eth0* txqlen *N* | Changes the length of the transmit queue for the specified interface |
| ip link set *eth0* mut *N* | Sets the maximum transmission unit as *N*, in bytes |

```bash 
yujia@workstation:~/Desktop/sysadmin-notes$ ip neigh show
# IP address     inteface        mac address       STALE ARP cache timeout/ Reachable
192.168.1.11 dev enp0s25 lladdr 08:00:27:b3:bc:31 REACHABLE 
192.168.1.1 dev enp0s25 lladdr 20:25:d2:f1:01:30 STALE 
fe80::1 dev enp0s25 lladdr 20:25:d2:f1:01:30 router STALE 
fe80::2225:d2ff:fef1:130 dev enp0s25 lladdr 20:25:d2:f1:01:30 router STALE 
```

```bash 
# Display the routing table
yujia@workstation:~/Desktop/sysadmin-notes$ ip route
default via 10.8.0.1 dev tun0 proto static metric 50 
default via 192.168.1.1 dev enp0s25 proto dhcp src 192.168.1.5 metric 100 
10.8.0.0/24 dev tun0 proto kernel scope link src 10.8.0.3 metric 50 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.18.0.0/16 dev br-109d1ab6cfb0 proto kernel scope link src 172.18.0.1 linkdown 
172.19.0.0/16 dev br-c8dc7a908be4 proto kernel scope link src 172.19.0.1 linkdown 
192.168.1.0/24 dev enp0s25 proto kernel scope link src 192.168.1.5 metric 100 
192.168.1.11 dev enp0s25 proto static scope link metric 50 
```

### Display Network Connections with ss

-a: all network, -t: TCP, -u: UDP, -n: numeric format, -p: process ID

```bash 
yujia@workstation:~/Desktop/sysadmin-notes$ ss -tuna4
Netid    State     Recv-Q    Send-Q             Local Address:Port         Peer Address:Port   Process    
udp      UNCONN    0         0                     127.0.0.54:53                0.0.0.0:*                 
udp      UNCONN    0         0                  127.0.0.53%lo:53                0.0.0.0:*                 
udp      ESTAB     0         0            192.168.1.5%enp0s25:68            192.168.1.1:67                
udp      UNCONN    0         0                        0.0.0.0:33698             0.0.0.0:*                 
udp      UNCONN    0         0                        0.0.0.0:5353              0.0.0.0:*                 
tcp      LISTEN    0         4096                  127.0.0.54:53                0.0.0.0:*                 
tcp      LISTEN    0         4096                   127.0.0.1:11434             0.0.0.0:*                 
tcp      LISTEN    0         128                    127.0.0.1:33211             0.0.0.0:*                 
tcp      LISTEN    0         4096                   127.0.0.1:631               0.0.0.0:*                 
tcp      LISTEN    0         4096               127.0.0.53%lo:53                0.0.0.0:*                 
tcp      ESTAB     0         0                       10.8.0.3:37300        20.44.10.122:443               
tcp      ESTAB     0         30                      10.8.0.3:47786       140.82.114.25:443               
tcp      ESTAB     0         0                    192.168.1.5:57566        192.168.1.11:7899              
tcp      ESTAB     0         0                       10.8.0.3:41352        34.237.73.95:443    
```