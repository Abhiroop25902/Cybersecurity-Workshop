# Internet Basics

-   type `ipconfig` in powershell
-   code like `192.168.33.123` is an IPv4 address
-   unique identifier for devices in internet

## Two Sets of Protocol

1. IPv4 (Internet Protocol v4)
    - Eg: `192.168.1.5`
    - all numbers are 8 bit, making whole address 32 bit (255.255.255.255)
    - everyone can't have same IP address, so this is small
2. IPv6 (Internet Protocol v6)
    - Eg: `fedc:ba98:7654:3210:adbf:bbff:2922:ffff`
    - 128 bits (more number of user) -> 340 undecillion unique address space
    - Hexadecimals
    - First two hexes are global prefix (fedc:ba98) -> Worldwide Definition
    - next hex is subnet (7654)
    - next four are interface ID (3210:adbf:bbff:2922:ffff) -> device specific
    - Simplified IPv6 writing:
        - Eg: `fe80::b0d2:cdd4:f1a1:52e6%43`
        - `::`is related to how you write simplified IPv6 addresses
        - The number after the `%` is the scope ID

[youtube video](https://www.youtube.com/watch?v=aor29pGhlFE&ab_channel=Techquickie)

## Static and Dynamic IP Address:

-   Generally IP address are dynamic(assigned randomly when a device gets connected to router), but in cases you want it to be fixed for a lot of time, we can have it static (used in website, server ip, etc)
-   Dynamic IP are given using DHCP (Dynamic Host Configuration Protocol) -> It will automatically assign you IP address to device connecting to routers (this protocol is done in router)
-   By any chance two devices have same IP, there will be clash and DHCP will auto change IP of one to fix it
-   DHCP increases security as IP changes, and also brings ease of use

## IP Classes

-   Class A
    -   1.0.0.1 to 126.255.255.255
    -   16 million on each of 127 networks (biggest)
    -   for ISP's
-   Class B
    -   128.1.0.1 to 191.255.255.254
    -   65,000 hosts on each of 16,000 networks
    -   for Hospitals, Office, and companies
-   Class C
    -   192.0.1.1 to 223.255.254.254
    -   Supports 254 Hosts on each of 2 million networks
    -   Home Network devices
-   Class D
    -   224.0.0.0 ro 239.255.255.255
    -   Reserved for Multicast group
-   Class E
    -   240.0.0.0 to 254.255.255.254
    -   Reserved for future, research, and development purpose

### 127.0.0.1

-   This is the IP address of our localhost, this is how our PC can connect to itself

## MAC Addressing

-   Unique ID which is hardwired to your Network Card
-   Eg: C8 60 00 BA 95 65 (this will never change)
    -   First 3 parts (C8 60 00 BA) is OUI (Organizationally Unique Identifier)
        -   used for trusting devices in a subnet
    -   Last 3 parts (BA 95 65) is UAA/Extended Identifier / Device Identifier
        -   Unique Address assigned by vendor
-   We generally hide MAC Address by something call spoofing, it can be done physically or by software

## The OSI (Open Systems Interconnection) Model

-   7 Layers
    1. Physical:
        - Electrical + Physical spec for devices
        - cable + connectors
        - data in bits
        - Eg: RJ45 Cable
        - Hacking method: Sniffing
    2. Data Link:
        - Provide connection between hosts on the same network (Ethernet MAC Address)
        - Any device which allows you to connect you to internet
        - Eg: NIC (Network Interface Card), Switches
        - Hacking method: Spoofing
    3. Network:
        - Connects hosts on a different networks IPv4 + IPv6
        - routing of data packets
        - logical address, IP Address (Router)
        - talk to router happens in network layer
        - Hacking method: Man in the middle (MITM)
    4. Transport:
        - Enable transfer of data TCP/UDP
        - End to End connections
        - when you go to `www.google.com` server is giving info to our browser via packets
        - this is what happens in Transport Layer
        - Hacking method: Reconnaissance/DOS/DDOS
    5. Session:
        - Controls Dialogue between computers
        - Controls Termination and restarts
        - hackers can use Session high-jacking
        - server like google needs to identify unique user, and this is done via cookies
        - all cookies are in Later 5
        - Hacking method: Highjacking
    6. Presentation:
        - Encryption/Decryption compression
        - Context for communication between layers
        - HTML + CSS + browser side JS
        - Hacking method: Phishing
    7. Application:
        - High level APTs, HTTP, FTP, SMTP
        - Hacking method: Exploit
-   Layer 1, 2 is Hardware, and Layer 3 to Layer 7 is Software Layer
-   Layer 4,5,6 are called maidsafe compliments
-   Acronym "please do not throw salami pizza anywhere"

### why don't websites open when we block all cookies?

Server need cookies to create session, if session is not made layer 6 and 7 will not load

## TCP, UDP, and handshake

-   TCP: Transmission Control Protocol [click here](https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:the-internet/xcae6f4a7ff015e7d:transporting-packets/a/transmission-control-protocol--tcp)

    -   Eg: netflix uses TCP, and hence it can know where you stopped your video
    -   Eg: gamer server uses TCP (look into `server run abandoning`)
    -   How hacker can do their stuff?
        -   we give SYN, but never send ACK, server will keep giving you SYN-ACK, i.e server will allocate resources for client, i.e we are taking resources for an actual user
        -   do this with many people and server will just crash for actual users

-   UDP: User Datagram Protocol [click here](https://www.khanacademy.org/computing/computers-and-internet/xcae6f4a7ff015e7d:the-internet/xcae6f4a7ff015e7d:transporting-packets/a/user-datagram-protocol-udp)

    -   server will keeps spamming till client accepts, hence it cannot store which packets are received or not
    -   inefficient than TCP but fast and insecure than TCP
    -   many free video player works in UDP (free anime sites)

-   TCP vs UDP [click here](http://www.steves-internet-guide.com/tcp-vs-udp/)

[video](https://www.youtube.com/watch?v=PpsEaqJV_A0&ab_channel=Techquickie)

## Common Port

-   TCP

    -   FTP(21) : file transfer protocol
    -   SSH (22) : secure shell
    -   TELNET(23) : used in old and legacy router
    -   SMTP(25) : simple mail transfer protocol (send mail) (legacy)
    -   DNS(53) : dns protocol (dns converts site name to ip address)
    -   HTTP (80) : hypertext transfer protocol
    -   HTTPS (443) : hypertext transfer protocol secure
    -   POP3 (110) : Post office Protocol (send email to receiver)
    -   SMB (139 and 445) : server message block, file transfer for windows
    -   IMAP (143) : (receives mail)

-   UDP
    -   DNS(53) : [click here](http://www.steves-internet-guide.com/tcp-vs-udp/)
    -   DHCP (67, 68) : dynamic host transfer protocol
    -   TFTP (69) : trivial file transfer protocol
    -   SNMP (161) : service network management protocol (not used much)

## Unicast, Broadcast, and Multicast

[click here](https://erg.abdn.ac.uk/users/gorry/course/intro-pages/uni-b-mcast.html)

-   If last number in Ipv4 is 255 -> that IP is used in broadcast
-   Unicast -> two different subnets
-   Broadcast -> inside subnet
-   Multicast -> Unicast between subnets + broadcast inside subnets
-   Broadcast used
    -   ARP Poisoning -> with this you can get MITM attack
    -   [mac spoofing](<https://en.wikipedia.org/wiki/MAC_spoofing#:~:text=MAC%20spoofing%20is%20a%20technique,(NIC)%20cannot%20be%20changed.&text=Essentially%2C%20MAC%20spoofing%20entails%20changing,computer's%20identity%2C%20for%20any%20reason.>)

## Sub-Netting

Eg: 192.168.0.1

-   subnetting happens between the octets (the .)
-   last two numbers are called class C, here `0.1`
-   we divide IP by their last octets to create subnets
    -   Eg: 192.126.0, 192.126.1, 192.126.2 can be different subnets
-   each subnet can have maximum 256 devices (8 bit limit)
    -   Eg: 192.126 subnet can have 256 devices with each subnet 192.126.0, 192.126.1, 192.126.2, ... each having their own 256 devices

[click here](https://www.cloudflare.com/en-in/learning/network-layer/what-is-a-subnet/)

## setup stuffs for labs

Download following

1. [Virtual Box](https://www.virtualbox.org/)
    - two ways to booting into different OS: portioning/dual-boot and Virtual Machine
    - VM can be made nad managed by Virtual Box
    - VM doesn't interfere with actual OS
2. [Kali Linux](https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/)
    - [Download](https://www.kali.org/get-kali/#kali-virtual-machines) VBox pre-made file 
    - uncompress the downloaded file to get vdi file -> open it in vbox
    - allocate max 50% RAM for the image
    - all other defaults should be OK
    - import -> agree -> wait for install 
    - open VM as usual then
    - username: kali
    - password: kali
    - In kali -> settings display -> change resolution (or install guest additions)
    
