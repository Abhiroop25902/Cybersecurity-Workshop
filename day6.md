# CTF

-   Flag: text file which shows some kind of checkpoint
-   Important Sites:
    -   https://github.com/gchq/CyberChef

## Boot to Root

    - penetrate VM to root

## Cryptography

1. Encoding
    - transform data into another format using a scheme that is publicly available
    - No key
    - Eg: ASCII, EBCDIC, base64, hex/binary
        - ASCII -> 128 characters
        - EBCDIC -> extended ASCII -> 8 bin size per char
        - base64 -> ends with `=`
2. Encryption

    - Transform data in order to keep it secret
    - can be reversed by knowing the KEY and ALGORITHM
    - Eg: XOR, ciphers (Caesar, Enigma)
    - [kerckhoff's principle](https://www.google.com/search?q=kerckhoff%27s+principle&oq=kerck&aqs=edge.1.69i57j0i512l5j0i10i512j0i512l2.2288j0j1&sourceid=chrome&ie=UTF-8)
    - [security through obscurity](https://www.google.com/search?q=security+through+obscurity&sxsrf=AOaemvKhdMksve3P6Yw8iMAOePYePJbniA%3A1640029628194&ei=vN3AYYKdC5-RseMPna-J0AU&ved=0ahUKEwiC66K7kvP0AhWfSGwGHZ1XAloQ4dUDCA8&uact=5&oq=security+through+obscurity&gs_lcp=Cgdnd3Mtd2l6EAMyBQgAEIAEMgUIABCABDIFCAAQgAQyBQgAEIAEMgUIABCABDIFCAAQgAQyBQgAEIAEMgUIABCABDIFCAAQgAQyBQgAEIAESgQIQRgASgQIRhgAUABYAGCRBWgAcAJ4AIABvgKIAb4CkgEDMy0xmAEAoAECoAEBwAEB&sclient=gws-wiz)
    - Enigma Machine -> if we put scrambled output as input -> we wil get sam output -> use whis propety to find correct config of machine -> then decrypt the scrambled output

3. Hashing

    - [CIA Model](https://www.google.com/search?q=cia+model&oq=CIA+model&aqs=edge.0.0i512j0i20i263i512j0i67j0i512l4j0i67j69i64.2240j0j1&sourceid=chrome&ie=UTF-8) -> Integrity
    - Fixed Length String
    - Generated based on input
    - One Way (unless brute forced)
    - Eg: SHA1, MD5
        - MD5 is easy to crack

4. Obfuscation
    - Making something harder to understand (basically unreadable)
    - Obstacle to reverse engineering
    - Eg: jsfuck, COW language

## OSINT

HINT: google clues to get more info and eventually find the ans

## Reverse Engineering

Prerequisite -> `sudo apt install default-jdk`
[Ghidra](https://github.com/NationalSecurityAgency/ghidra)
