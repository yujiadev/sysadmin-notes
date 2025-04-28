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
yujia@workstation:~/Desktop/sysadmin-notes$ ss -tunap4
Netid State  Recv-Q Send-Q       Local Address:Port   Peer Address:Port Process                           
udp   UNCONN 0      0               127.0.0.54:53          0.0.0.0:*                                      
udp   UNCONN 0      0            127.0.0.53%lo:53          0.0.0.0:*                                      
udp   ESTAB  0      0      192.168.1.5%enp0s25:68      192.168.1.1:67                                     
udp   UNCONN 0      0                  0.0.0.0:33698       0.0.0.0:*                                      
udp   UNCONN 0      0                  0.0.0.0:5353        0.0.0.0:*                                      
tcp   LISTEN 0      4096            127.0.0.54:53          0.0.0.0:*                                      
tcp   LISTEN 0      4096             127.0.0.1:11434       0.0.0.0:*                                      
tcp   LISTEN 0      128              127.0.0.1:33211       0.0.0.0:*                                      
tcp   LISTEN 0      4096             127.0.0.1:631         0.0.0.0:*                                      
tcp   LISTEN 0      4096         127.0.0.53%lo:53          0.0.0.0:*                                      
tcp   ESTAB  0      0                 10.8.0.3:43388 140.82.113.26:443   users:(("brave",pid=6296,fd=21)) 
tcp   ESTAB  0      0              192.168.1.5:57566  192.168.1.11:7899                                   
tcp   ESTAB  0      0                 10.8.0.3:41352  34.237.73.95:443   users:(("brave",pid=6296,fd=19)) 
```

## Network Configuration and Troubleshooting

```bash 
root@workstation:~# systemctl status NetworkManager
● NetworkManager.service - Network Manager
     Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-04-28 08:52:13 +08; 11h ago
       Docs: man:NetworkManager(8)
   Main PID: 1970 (NetworkManager)
      Tasks: 9 (limit: 76585)
     Memory: 22.8M (peak: 34.7M)
        CPU: 14.892s
     CGroup: /system.slice/NetworkManager.service
             ├─ 1970 /usr/sbin/NetworkManager --no-daemon
             ├─14940 /usr/libexec/nm-openvpn-service --bus-name org.freedesktop.NetworkManager.openvpn.Co>
             └─14945 /usr/sbin/openvpn --remote xx.xx.xx.xx 1194 tcp-client --http-proxy 192.168.1.11 789>

Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6075] device (tun0): Activation: st>
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6077] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6082] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6086] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6089] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6483] policy: set 'my-vpn1' (tun0) >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6487] policy: set 'my-vpn1' (tun0) >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6517] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6521] device (tun0): state change: >
Apr 28 18:19:17 workstation NetworkManager[1970]: <info>  [1745835557.6528] device (tun0): Activation: su
```
RHEL9 uses a service known as Network Manager to monitor and manage network settings (nmcli)
```bash 
# Display current status of network devices
root@workstation:~ nmcli dev status
DEVICE           TYPE      STATE                   CONNECTION      
enp0s25          ethernet  connected               Profile 1       
tun0             tun       connected (externally)  tun0            
lo               loopback  connected (externally)  lo              
br-109d1ab6cfb0  bridge    connected (externally)  br-109d1ab6cfb0 
br-c8dc7a908be4  bridge    connected (externally)  br-c8dc7a908be4 
docker0          bridge    connected (externally)  docker0         
vboxnet0         ethernet  unmanaged               --              
vboxnet1         ethernet  unmanaged               --       

# Restart Network Manager
root@workstation:~ systemctl restart NetworkManager
```
Sometimes the network related issue can't be resolved by restart NetworkManager, then looking into the configuration files under /etc/NetworkManager/system-connections directory is needed.

Each installed network adapter, such as ens3, gets its own **name.nmconnection** configuration file, where **name** is the name of the device or name of the connection.

```bash 
[root@localhost ~]cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=b4808ad1-7327-361c-aece-d225ff16b02c
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1745409147

[ethernet]

[ipv4]
method=auto     # DHCP

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

## Network Configuration Tool

### The nmcli Configurattion Tool
```bash 
[root@localhost ~] nmcli con show
NAME    UUID                                  TYPE      DEVICE 
enp0s3  b4808ad1-7327-361c-aece-d225ff16b02c  ethernet  enp0s3 
lo      fa19881b-9cbc-47ad-a665-c1397697c9ca  loopback  lo     

# New connection profile
nmcli con mod "eth0-work" ipv4.dns 192.168.20.1

# S~~witch to the new connection profile
nmcli con up "eth0-work"

# Prevent a connection from starting automatically at boot
nmcli con mod "eth0-work" connection.autoconnect no
```

### Configure Name Resolution

/etc/hostname
/etc/hosts 
/etc/resolv.conf
/etc/nsswitch.conf

#### /etc/nsswitch.conf
Specify database search priorities for everything from authentication to name service.

```bash 
# In order of likelihood of use to accelerate lookup.
shadow:     files
hosts:      files dns myhostname
```

Troubleshoot IPv6, "ping6 -I *eth0*" and "tracepath6"


## A VERY VERY Genernal Scenario & Solution; In PROD ETL, NET Trace, dump may All Needed
| Scenario | Solution |
| -------- | -------- |
| Networking is donw | Check physical connections. Run **ip link show** to check active interface. Run **systemctl status NetworkManager**. Check IP settings by running **ip addr show**
| Unable to access remote systems | ping, traceroute, ping6, tracepath6 |
| Current network settings lead to conflicts | Check network device configuration in **/etc/NetworkManager/system-connections** |
| Hostname not recognized | Review /etc/hostname, **hostname** command, **hostnamectl** |
| Remote hostnames not recognized | Review **/etc/hosts** and **/etc/nsswitch.conf**. Check **/etc/resolv.conf** for an appropriate DNS server IP address. Run the **dig** command to test the DNS resolution. |
