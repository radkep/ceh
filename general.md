# 1.SCAN PORTS

## Scan 1 – Top 1000 ports

```bash
nmap -oA iloverobot-q -vvv iloverobot
```

-oA - ze wszystkimi dostępnymi formatami - XML dla systemu co dalej przetwarza wyniki, systemy do generowania dokumentacji, gnmap - grepowalny nmap

## Scan 2 – Full port scan (BACKGROUND) -scans all 65535 ports

```bash
nmap -p- -oA iloverobot-a -vvv iloverobot
```

## Scan 3 – Focused scan

```bash
nmap -p22,80,445 -sV -sC -oA iloverobot-p -vvv iloverobot
```

-sV wykrywanie wersji usługi
-sC uruchom domyśle skrytpy nmap
-vvv- verbose

# 2. IDENTIFY PORTS (after scone is DONE) - automatically

```bash
searchsploit apache 2.4.49
```

or:

```bash
nmap -sV -oX scan.xml target
searchsploit --nmap scan.xml
```

# 3. IDENTIFY PORTS (after scone is DONE) -MANUALLY

| PORT        |                 | auth required | Comment                                                                                                                          | Command                                                                                                                                                                                                                                                                                                                                       |
| ----------- | --------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 21          | FTP             | YES/NO        | Check anonymous login → enumerate files → download sensitive data                                                                | `nmap -p21 -sc joker (check anonymous login)`<br>`ftp target`<br>`anonymous: <no password>`<br>`nmap --script ftp-anon -p21 target`                                                                                                                                                                                                           |
| 22          | SSH             | YES           | Brute force / credential reuse                                                                                                   | `hydra -l root -P rockyou.txt ssh://target`<br>`ssh user@target`                                                                                                                                                                                                                                                                              |
| 25          | SMTP            | YES/NO        | Enumerate users                                                                                                                  | `smtp-user-enum -M VRFY -U users.txt -t target`                                                                                                                                                                                                                                                                                               |
| 53          | DNS             | YES/NO        | Zone transfer / recon. I would not start with it first                                                                           | `dig axfr @target domain.local`<br>`dnsrecon -d domain.local -t axfr`                                                                                                                                                                                                                                                                         |
| 80          | HTTP            | NO            | → inspect page<br>→ gobuster (enumarate the folders)<br>→ nikto                                                                  | `http:// ip_address`<br>`http:// ip_address/robots.txt`<br>`whatweb http://target`<br>`nikto -h http://target`<br>`gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak`                                                                                                                                      |
| 88          |                 | YES/NO        | Enumerate users (we need to have some username) → AS-REP roasting → Kerberoasting                                                | `AES-Rep-Roasting`<br>`z poprawnym TGT(creds) : Kerberoasting`                                                                                                                                                                                                                                                                                |
| 137,138,139 | Netbios         | *             | Does not exist alone. SMB-related enumeration                                                                                    | `nbtstat -A target`                                                                                                                                                                                                                                                                                                                           |
| 389         | LDAP            | YES/NO        | Requires login data (login/password). LDAP enumeration / anonymous bind                                                          | `ldapsearch -x -h target`<br>`-x simple authenticatyion`<br>`-h LDAP server`                                                                                                                                                                                                                                                                  |
| 445         | SMB             | *             | Enumerate shares → anonymous login → spider files → enumerate users → password spray<br><br>→ smbmap<br>→ crackmapexec           | `smbmap -H target`<br>`smbmap -H target -u anonymous -p ''`<br>`smbclient -L //target/ -U anonymous`<br>`crackmapexec smb target -u anonymous -p ''`<br>`crackmapexec smb target -u anonymous -p '' -M spider_plus -o READ_ONLY=true`<br>`impacket-lookupsid 'anonymous:'@target`<br>`crackmapexec smb target -u users.txt -p 'Password123!'` |
| 3306        | MYSQL           | YES/NO        | Try default creds / DB enumeration:                                                                                              | `mysql -h target -u root -p`                                                                                                                                                                                                                                                                                                                  |
| 3389        | RDP             | YES           | Requires login/password -brute force will not work (user will be blocked)<br>User need to be in the group "remote desktop users" | `xfreerdp /u:user /p:password /v:target`<br>`cme rdp 10.10.1.0/24 -u users.txt -p "password"`<br>`crackmapexec rdp 10.10.1.0/24 -u users.txt -p "password"`                                                                                                                                                                                   |
| 5985        | WinRM           | YES           | Remote PowerShell shell                                                                                                          | `evil-winrm -i target -u user -p password`                                                                                                                                                                                                                                                                                                    |
| 6667        | IRC             | ES/NO         | Check IRC vulnerabilities                                                                                                        | `searchsploit unrealircd`                                                                                                                                                                                                                                                                                                                     |
| 7990        |                 |               | Microfost IIS, JIRA                                                                                                              | `namp -p5985,7990,9389 -sV-sC -aA enter-porty enter`                                                                                                                                                                                                                                                                                          |
| 9389        | AD Web Services |               | Active Directory related                                                                                                         | `crackmapexec smb target`                                                                                                                                                                                                                                                                                                                     |
| 47001       | WinRM           | winrm         | Alternate WinRM                                                                                                                  | `evil-winrm -i target -u user -p password`                                                                                                                                                                                                                                                                                                    |

## local enumeration

```bash
netstat -tulpn
```

# 3. Exploit

# 4. Get creds or shell

## IF creds:

→ RDP / WinRM/SMB/MSSQL/SSH

## IF shell:

→ Local enumeration
→ winPEAS, PowerWiew
→ privilige escalation

### if shell not working (example error : "must be run from the terminal ")

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
shell -t
```

# 5.Local Enumeration

→ net user
→ list c:\users to check active users (user which has been loggen in at least once )
→ whoami /priv
→ whoami /groups
→ ipconfig /all
→ systeminfo
→ net localgroup administrators

# 6. Privilege escalation
