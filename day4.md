# Eternal Blue

Windows 7 VM -> SMB exploit -> change admin passwd and do [wannacry](https://www.kaspersky.co.in/resource-center/threats/ransomware-wannacry)

## Step 0

`sudo su`

## Step 1: Find IP

1. `ifconfig`

    ```
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.110.131  netmask 255.255.255.0  broadcast 192.168.110.255
            inet6 fe80::20c:29ff:fe74:24c3  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:74:24:c3  txqueuelen 1000  (Ethernet)
            RX packets 230  bytes 22906 (22.3 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 47  bytes 4684 (4.5 KiB)
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

    My IP --> `192.168.110.131`

2. `nmap -sP 192.168.110.1/24`

    ```
    starting Nmap 7.92 ( https://nmap.org ) at 2021-12-16 08:48 EST
    Nmap scan report for 192.168.110.2
    Host is up (0.0017s latency).
    Nmap scan report for 192.168.110.131
    Host is up (0.00048s latency).
    Nmap scan report for 192.168.110.133
    Host is up (0.0075s latency).
    Nmap done: 256 IP addresses (3 hosts up) scanned in 2.75 seconds
    ```

    Possible IP : `192.168.110.2`, `192.168.110.133`

    `192.168.110.2` -> VMware NAT router

    so windows IP -> `192.168.110.133`

3. `nmap -sV 192.168.110.133`

    ```
    Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-16 08:50 EST
    Nmap scan report for 192.168.110.133
    Host is up (0.0011s latency).
    Not shown: 991 closed tcp ports (conn-refused)
    PORT      STATE SERVICE      VERSION
    135/tcp   open  msrpc        Microsoft Windows RPC
    139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
    49152/tcp open  msrpc        Microsoft Windows RPC
    49153/tcp open  msrpc        Microsoft Windows RPC
    49154/tcp open  msrpc        Microsoft Windows RPC
    49155/tcp open  msrpc        Microsoft Windows RPC
    49156/tcp open  msrpc        Microsoft Windows RPC
    49157/tcp open  msrpc        Microsoft Windows RPC
    Service Info: Host: WIN-845Q99OO4PP; OS: Windows; CPE: cpe:/o:microsoft:windows

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 61.45 seconds
    ```

    confirm windows IP is `192.168.110.133`

    !!SMB ports 139 and 455 are open
    !!135 -> default Windows RPC

    [More Info](https://www.google.com/search?q=smb+port&oq=smb+port&aqs=edge.0.0i433i512j0i512j0i131i433i512j0i512l5j69i64.1597j0j1&sourceid=chrome&ie=UTF-8)

    [More about RPC](https://docs.microsoft.com/en-us/windows/win32/rpc/rpc-start-page)

## Optional Step 1.1: Windows specific way of finding

1. `nbtscan 192.168.68.1/24`

    - nbtscan -> only netbios -> only SMB Port (windows specific)

    ```
    Doing NBT name scan for addresses from 192.168.110.1/24

    IP address       NetBIOS Name     Server    User             MAC address
    ------------------------------------------------------------------------------
    192.168.110.133  WIN-845Q99OO4PP  <server>  <unknown>        00:0c:29:a2:0d:8b
    192.168.110.255 Sendto failed: Permission denied

    ```
