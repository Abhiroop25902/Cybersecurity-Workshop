# Hands On with Linux

While we are hacking, we will not have access to GUI, so we need to learn how to control Linux in CLI

## File System Directory

-   `/` -> root folder
    -   `bin` -> binaries folder, commands and programs
        -   Eg: `ls` program is in this folder only
    -   `boot` -> the programs which is used while booting, contains `grub` bootloader -> don't mess
    -   `dev` -> devices
    -   `etc` -> all config files
        -   `passwd` -> user password
        -   `shadow` -> hashed passwords
        -   `hosts` -> file used by the operating system to translate hostnames to IP-addresses, i.e local DNS lookup
    -   `home` -> your home folder
    -   `lib` -> library files (like `.dll` files)
    -   `media` -> external devices like CD, USB
    -   `mnt` -> external devices lik HDD and SSD -> don't mess up can damage data
    -   `opt` -> optionals
    -   `proc` -> process file system -> contains all info about the computer
        -   Eg: try `cat /proc/cpuinfo`
    -   `root` -> root user home directory
        -   need to switch user as admin to access, [click here](https://askubuntu.com/questions/617850/changing-from-user-to-superuser)
    -   [`sbin`](http://www.linfo.org/sbin.html) -> system files
    -   [`srv`](https://tldp.org/LDP/Linux-Filesystem-Hierarchy/html/srv.html) -> service files
    -   `tmp` -> temporary files
    -   `usr` -> application files (`usr/include` contains cpp header files)
    -   `var` -> variable files that changes

## Navigation

-   `ls <dir>` -> list file in current directory, if `<dir>` is present list files in `<dir>`
    -   files and folders have different colors
    -   `-l` -> list vertically and with permissions
    -   `-a` -> list all files, even hidden files
-   `cd <dir>` -> change directory to `<dir>`
    -   `.` -> current folder
    -   `..` -> go back current folder
    -   `~` -> home directory
-   `pwd` -> writes present working directory
-   `cls` -> clear terminal
-   `mkdir <f1> <f2>` -> make folder `<f1>` and `<f2>`
-   `rmdir` -> delete a directory
-   `touch <f1> <f2>` -> make file `<f1>` and `<f2>`
-   `rm <filename>` -> remove `<filename>` -> (don't execute rm -rf as root)
    -   `-rf` is a flag which does recursive force removal -> force permanently deletes files
-   `man <command>` -> show manual of the command
-   `nano` -> text editor

## Three types of user

-   `ROOT` -> administrator
-   `USER` -> no admin privileges, have to prefix `sudo` for doing admin tasks
    -   `sudo` give temporary root power while processing a command
-   `LLP (low level privilege)` -> admin specifies what can LLP do or not, called 'www-data'

## [Privilege Escalation](https://www.beyondtrust.com/blog/entry/privilege-escalation-attack-defense-explained)

Hacker starts form LLP, tries to get access of different users to eventually get admin privilege

One user will always have Root previledges

```
LLP
| (vertical movement)
User1 -> User2 -> User3 -> User4 ->> (lateral/horizontal movement)
                        | (vertical movement)
                        Root
```

## [Permission Management in Linux](https://techgenix.com/linux-permissions/)

## Extra tips while using linux

-   Tab completion is your friend
-   When as home user we get `$` symbol, when root we get `#` symbol

## Networking

-   `ifconfig` -> show IP config
-   `ip` -> newer IP config data printing for better regex support
    -   `ip a` -> show all (same as ifconfig)
    -   `ip r` -> traceroute command -> traces to routers present
    -   `ip n` -> arp -> address resolution protocol
-   `traceroute <addr>` -> print the route packets trace to network host (addr)
    -   Eg: `traceroute www.google.com`
-   `arp -a` -> uses [ARP](https://www.geeksforgeeks.org/how-address-resolution-protocol-arp-works/)
-   `ping <ip or address>` -> pings ip or addr to see if that server is visible or not

## installation and updating of tools

-   `sudo apt update` -> updates your local repo with online version
-   `sudo apt upgrade` -> reads the repo and updates any program that has some update
-   `sudo apt install fq` -> install package named fq if present

NOTE: apt -> Advanced Package Tool

## Hacker's Methodology

-   [explore this](https://shahmeeramir.com/how-i-hacked-into-airbnb-in-three-simple-steps-22ee0650da8f?gi=7b5ce2f015d9)

### Further Reading

-   [HTB](https://www.hackthebox.com/) -> competitive hacking
-   [Honeypot Architecture](<https://en.wikipedia.org/wiki/Honeypot_(computing)>)
-   [What is Open Port](https://en.wikipedia.org/wiki/Open_port)
-   [Signaling System 7](https://www.techtarget.com/searchnetworking/definition/Signaling-System-7)
