# Billu Box

The devices which will hack is termed as box

## step 0:

`sudo su`

## step 1: Foothold

We gather as much info as possible

we use nmap (network mapper) to find ip of the box to hack

1. do ifconfig to find your NAT IP (let it be `192.168.110.131`)
   Eg Output:

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.110.131  netmask 255.255.255.0  broadcast 192.168.110.255
        inet6 fe80::20c:29ff:fe74:24c3  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:74:24:c3  txqueuelen 1000  (Ethernet)
        RX packets 229  bytes 16344 (15.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 95  bytes 7707 (7.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8  bytes 400 (400.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 400 (400.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

2. `nmap -sP 192.168.110.1/24` -> ping hosts in the given subnet (i.e search from `192.168.110.1` to `192.168.110.255`)
    - Eg: `nmap -sP 192.168.110.147/24` scans from 192.168.110.147 to 255 hosts

Eg Output ->

```
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-14 09:02 EST
Nmap scan report for 192.168.110.1
Host is up (0.00070s latency).
MAC Address: 00:50:56:C0:00:08 (VMware)
Nmap scan report for 192.168.110.2
Host is up (0.00050s latency).
MAC Address: 00:50:56:EC:6B:0C (VMware)
Nmap scan report for 192.168.110.132
Host is up (0.00097s latency).
MAC Address: 00:0C:29:66:04:55 (VMware)
Nmap scan report for 192.168.110.254
Host is up (0.00053s latency).
MAC Address: 00:50:56:F8:38:E4 (VMware)
Nmap scan report for 192.168.110.131
Host is up.
Nmap done: 256 IP addresses (5 hosts up) scanned in 2.84 seconds
```

so we have 192.168.110.1, 192.168.110.2, 192.168.110.132, 192.168.110.254, 192.168.110.131

out of these mine is 192.168.110.131, so one of the ip are the box

192.168.110.1, 192.168.110.2 these are the VM gateways so we can ignores them for now

so these two are possible target 192.168.110.132, 192.168.110.254

3. `nmap -sV 192.168.110.132 192.168.110.254` -> check if "service" is running it the given ports
    -   scan all the services (application layer) port of the given ips

```
Nmap scan report for 192.168.110.132
Host is up (0.0010s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
111/tcp  open  rpcbind 2-4 (RPC #100000)
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 00:0C:29:66:04:55 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 192.168.110.254
Host is up (0.00068s latency).
All 1000 scanned ports on 192.168.110.254 are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
MAC Address: 00:50:56:F8:38:E4 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (2 hosts up) scanned in 18.13 seconds
```

So maybe our box is 192.168.110.132 

it is the ubuntu machine, so we can confirm this ip is of BilluBox

OPTIONAL : `nmap -sV 192.168.110.1\24` will scan open ports of all the ip in the subnet
