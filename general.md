# CEH Practical – General Workflow

## 1. Scan Ports

### Scan 1 – Top 1000 Ports

```bash
nmap -oA iloverobot-q -vvv iloverobot
```

**Output formats:**
- `.xml` – for tools that process results automatically
- `.gnmap` – grepable Nmap output
- `.nmap` – standard output

---

### Scan 2 – Full Port Scan (Background) – All 65535 Ports

```bash
nmap -p- -oA iloverobot-a -vvv iloverobot
```

---

### Scan 3 – Focused Scan

```bash
nmap -p22,80,445 -sV -sC -oA iloverobot-p -vvv iloverobot
```

**Options:**
- `-sV` = service version detection
- `-sC` = default NSE scripts
- `-vvv` = verbose output

---

## 2. Identify Services Automatically

```bash
searchsploit apache 2.4.49
```

Or:

```bash
nmap -sV -oX scan.xml target
searchsploit --nmap scan.xml
```

---

## 3. Identify Services Manually

| Port | Service | Auth Required | Notes | Commands |
|--------|---------|---------|---------|---------|
| 21 | FTP | YES/NO | Check anonymous login → enumerate files → download sensitive data | ```bash\nnmap -p21 --script ftp-anon target\nftp target\nanonymous\n<no password>\nnmap --script ftp-anon -p21 target\n``` |
| 22 | SSH | YES | Brute force / credential reuse | ```bash\nhydra -l root -P rockyou.txt ssh://target\nssh user@target\n``` |
| 25 | SMTP | YES/NO | Enumerate users | ```bash\nsmtp-user-enum -M VRFY -U users.txt -t target\n``` |
| 53 | DNS | YES/NO | Zone transfer / reconnaissance | ```bash\ndig axfr @target domain.local\ndnsrecon -d domain.local -t axfr\n``` |
| 80 | HTTP | NO | Inspect page, Gobuster, Nikto | ```bash\nhttp://ip-address\nhttp://ip-address/robots.txt\nwhatweb http://target\nnikto -h http://target\ngobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt -x php,txt,bak\n``` |
| 88 | Kerberos | YES/NO | User enumeration → AS-REP Roasting → Kerberoasting | ```text\nAS-REP Roasting\nKerberoasting (request TGT using credentials)\n``` |
| 137,138,139 | NetBIOS | * | SMB-related enumeration | ```bash\nnbtstat -A target\n``` |
| 389 | LDAP | YES/NO | LDAP enumeration / anonymous bind | ```bash\nldapsearch -x -h target\n``` |
| 445 | SMB | * | Enumerate shares → anonymous login → spider files → enumerate users → password spray | ```bash\nsmbmap -H target\nsmbmap -H target -u anonymous -p ''\nsmbclient -L //target/ -U anonymous\ncrackmapexec smb target -u anonymous -p ''\ncrackmapexec smb target -u anonymous -p '' -M spider_plus -o READ_ONLY=true\nimpacket-lookupsid anonymous:'@target\ncrackmapexec smb target -u users.txt -p 'Password123!'\n``` |
| 3306 | MySQL | YES/NO | Try default credentials / DB enumeration | ```bash\nmysql -h target -u root -p\n``` |
| 3389 | RDP | YES | Requires credentials. Brute force may lock accounts. User must belong to Remote Desktop Users group. | ```bash\nxfreerdp /u:user /p:password /v:target\ncme rdp 10.10.1.0/24 -u users.txt -p 'password'\ncrackmapexec rdp 10.10.1.0/24 -u users.txt -p 'password'\n``` |
| 5985 | WinRM | YES | Remote PowerShell shell | ```bash\nevil-winrm -i target -u user -p password\n``` |
| 6667 | IRC | YES/NO | Check IRC vulnerabilities | ```bash\nsearchsploit unrealircd\n``` |
| 7990 | Atlassian Applications | * | JIRA, Confluence, etc. | ```bash\nnmap -p5985,7990,9389 -sV -sC -A enter-porty enter\n``` |
| 9389 | AD Web Services | * | Active Directory related | ```bash\ncrackmapexec smb target\n``` |
| 47001 | WinRM (Alt) | YES | Alternate WinRM port | ```bash\nevil-winrm -i target -u user -p password\n``` |

---

## Local Enumeration (If You Have Access)

```bash
netstat -tulpn
```

---

## 4. Exploit

Perform service-specific exploitation based on findings.

---

## 5. Get Credentials or a Shell

### If Credentials Are Obtained

Try:

- RDP
- WinRM
- SMB
- MSSQL
- SSH

### If You Have a Shell

#### Local Enumeration

- winPEAS
- PowerView

#### Privilege Escalation

Check privileges and escalation paths.

### If Shell Is Unstable

Upgrade to a TTY:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Or:

```bash
shell -t
```

---

# Local Enumeration Checklist

```cmd
net user
```

List users and identify active accounts.

```cmd
whoami /priv
```

```cmd
whoami /groups
```

```cmd
ipconfig /all
```

```cmd
systeminfo
```

```cmd
net localgroup administrators
```

### Check Previously Logged-In Users

```cmd
dir C:\Users
```

---

## 6. Privilege Escalation

### Windows

- winPEAS
- PowerUp
- Seatbelt
- SharpUp
- PowerView

### Linux

- LinPEAS
- sudo -l
- SUID binaries
- Capabilities
- Cron jobs
- Writable services
- Kernel exploits
