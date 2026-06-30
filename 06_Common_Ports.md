# Common Ports – Enumeration

---

## Port 21 – FTP

```bash
# Banner Grabbing
nc -vn 10.10.10.10 21

# Anonymous Login prüfen
ftp 10.10.10.10
# User: anonymous  /  Pass: (leer oder E-Mail)

# Nmap Scripts
nmap --script ftp-anon -p 21 10.10.10.10       # anonymous login check
nmap --script ftp-* -p 21 10.10.10.10

# Brute Force
hydra -l user -P /usr/share/wordlists/rockyou.txt ftp://10.10.10.10
```

---

## Port 22 – SSH

```bash
# SSH-Version prüfen
nc -vn 10.10.10.10 22
nmap --script ssh-* -p 22 10.10.10.10

# Login
ssh user@10.10.10.10
ssh -i id_rsa user@10.10.10.10    # mit Key

# Brute Force
hydra -l user -P rockyou.txt ssh://10.10.10.10 -t 4
# -t 4 : 4 Threads (mehr kann Ban auslösen)
```

---

## Port 25 – SMTP

```bash
# Banner Grabbing + Enumeration
nc -vn 10.11.1.217 25

# Interessante SMTP-Befehle:
# VRFY username   → prüft ob User existiert
# EXPN gruppe     → listet Mitglieder einer Gruppe

# Brute Force (SMTP Auth)
hydra -l email@yahoo.com -P passwortliste.lst -s 465 -S -v -V -t 1 smtp.mail.yahoo.com smtp
# -s 465 : Port
# -S     : SSL/TLS verwenden
# -v -V  : verbose (jeder Versuch)
# -t 1   : 1 Thread (SMTP ist empfindlich)
```

---

## Port 53 – DNS

```bash
# Name Server suchen
host -t ns megacorpone.com

# Mail Server suchen
host -t mx megacorpone.com

# Zone Transfer versuchen (zeigt alle Records wenn ungeschützt)
host -l megacorpone.com ns1.megacorpone.com

# dnsrecon – Zone Transfer mit AXFR
dnsrecon -d megacorpone.com -t axfr

# dnsenum – vollständige DNS Enumeration
dnsenum th-deg.de

# nslookup
nslookup -type=NS megacorpone.com

# Hinweis: Port 53 ist selten von Firewalls geblockt
# → ideal für Reverse Shell Listener
```

---

## Port 80 / 443 – HTTP / HTTPS

```bash
# Developer Tools: CTRL + Shift + I
# robots.txt und sitemap.xml prüfen → oft versteckte Pfade

# Directory Bruteforce
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak
# -x : Dateiendungen mitscannen (z.B. findet login.php, config.bak, readme.txt)

# Weitere Wortlisten:
# /usr/share/seclists/Discovery/Web-Content/
# /usr/share/wordlists/dirbuster/

# Vulnerability Scanner
nikto -h 192.168.1.1                          # Schwachstellenscan

# WordPress
wpscan -u <IP> --threads 10 --wordlist /usr/share/wordlists/rockyou.txt --username admin

# WordPress User Enumeration
for i in {1..100}; do curl -s -L -i http://<IP>/?author=$i | grep -E -o "\" title=\"View all posts by [a-z0-9A-Z\-\.]*|Location:.*" | sed 's/\// /g' | cut -f 6 -d ' ' | grep -v "^$"; done

# Tools
# Nuclei       – Vulnerability Scanner (Subdomain Finder)
# Burp Suite   – Proxy, Scanner, Intruder, Repeater
# sqlmap       – SQL Injection
# ffuf         – schneller Directory Fuzzer
# feroxbuster  – rekursiver Directory Scanner
# DIRB / DirBuster / Dirsearch / OWASP ZAP

# Ressourcen
# https://hackertarget.com/attacking-wordpress/
```

---

## Port 111 – NFS (Network File System)

```bash
# NFS Shares enumerieren
nmap -sV -p 111 --script=rpcinfo 10.11.1.1-254

# NSE Scripts für NFS
ls -l /usr/share/nmap/scripts/nfs*
nmap -p 111 --script nfs* 10.11.1.72

# Share mounten
mkdir /tmp/nfs_share
sudo mount -o nolock 10.11.172:/home /tmp/nfs_share/
cd /tmp/nfs_share && ls -la
```

---

## Port 139 / 445 – SMB  |  Port 137 (UDP)

```bash
# Ressourcen:
# https://www.noobsec.net/oscp-cheatsheet/#port-139445---smb
# https://0xdf.gitlab.io/2024/03/21/smb-cheat-sheet.html

# enum4linux – SMB Enumeration (Windows + Linux)
sudo enum4linux -U <ip>       # Userliste
sudo enum4linux -S <ip>       # Shares
sudo enum4linux -a <ip>       # alles

# smbclient – Shares auflisten und verbinden
smbclient -L //10.10.201.219/                      # alle Shares (Linux-Ziel: Forward Slash)
smbclient //10.10.10.10/share -U username
# Nach Verbindung:
#   get <datei>    → Datei herunterladen (z.B. id_rsa)
#   put <datei>    → Datei hochladen
#   exit           → Verbindung beenden

# Nmap NSE Scripts für SMB
ls -l /usr/share/nmap/scripts/smb*
nmap --script smb-vuln* -p 139,445 10.10.10.10
nmap -v -p 139,445 -oG smb.txt 10.11.1.1-254

# Windows Ziel (Port 135 = MSRPC)
nmap -p 135,445 --script=smb-vuln-ms* 10.11.1.5

# nbtscan
nbtscan -r 10.11.1.0/24

# Metasploit
use auxiliary/scanner/smb/smb_version
set RHOSTS 10.10.10.10
run

# SMB Mindmap: https://github.com/Ignitetechnologies/Mindmap/blob/main/Enumeration/Enumeration%20HD.png
```

---

## Port 161 – SNMP (UDP)

```bash
# Community Strings bruteforcen
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.10.10.10

# SNMP Walk (Community String: public)
snmpwalk -c public -v1 10.10.10.10
snmpwalk -c public -v2c 10.10.10.10 1.3.6.1.4.1.77.1.2.25   # User-Liste (Windows)

# snmpcheck
snmpcheck -t 10.10.10.10 -c public

# Nmap
nmap -sU -p 161 --script snmp-* 10.10.10.10
```

---

## Port 389 / 636 – LDAP / LDAPS

```bash
# LDAP Enumeration
ldapsearch -x -H ldap://10.10.10.10 -b "dc=domain,dc=local"
ldapsearch -x -H ldap://10.10.10.10 -b "dc=domain,dc=local" "(objectClass=user)"

# enum4linux mit LDAP
enum4linux -a 10.10.10.10

# Nmap
nmap -p 389 --script ldap-* 10.10.10.10
```

---

## Port 1433 – MSSQL

```bash
# Nmap
nmap -p 1433 --script ms-sql-* 10.10.10.10

# Impacket mssqlclient
impacket-mssqlclient sa:password@10.10.10.10
# SQL Befehle:
# SELECT @@version
# EXEC xp_cmdshell 'whoami'    (wenn aktiviert → RCE!)

# Metasploit
use auxiliary/scanner/mssql/mssql_login
```

---

## Port 3306 – MySQL

```bash
# Verbinden
mysql -h 10.10.10.10 -u root -p
mysql -h 10.10.10.10 -u root           # ohne Passwort versuchen

# SQL Befehle
SHOW DATABASES;
USE datenbankname;
SHOW TABLES;
SELECT * FROM users;

# Nmap
nmap -p 3306 --script mysql-* 10.10.10.10

# Brute Force
hydra -l root -P rockyou.txt mysql://10.10.10.10
```

---

## Port 3389 – RDP (Remote Desktop)

```bash
# Verbinden
xfreerdp /u:Administrator /p:password /v:10.10.10.10
rdesktop 10.10.10.10 -u Administrator -p password

# BlueKeep prüfen (CVE-2019-0708)
nmap -p 3389 --script rdp-vuln-ms12-020 10.10.10.10

# Metasploit Scanner
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS 10.10.10.10
run

# Brute Force
hydra -l administrator -P rockyou.txt rdp://10.10.10.10
```

---

## Port 6379 – Redis

```bash
# Verbinden
redis-cli -h 10.10.10.10

# Redis Befehle
INFO server          # Server-Info
CONFIG GET *         # alle Konfiguration
KEYS *               # alle Keys
GET keyname          # Wert lesen
```

---

## Common Ports – Übersicht

| Port | Protokoll | Dienst | Kategorie |
|------|-----------|--------|-----------|
| 21 | TCP | FTP | Linux/Windows |
| 22 | TCP | SSH | Linux |
| 25 | TCP | SMTP | Linux/Windows |
| 53 | TCP/UDP | DNS | Linux/Windows |
| 80 | TCP | HTTP | Linux/Windows |
| 111 | TCP | NFS/RPC | Linux |
| 135 | TCP | MSRPC | Windows |
| 137 | UDP | NetBIOS | Windows |
| 139 | TCP | SMB/NetBIOS | Windows |
| 161 | UDP | SNMP | Linux/Windows |
| 389 | TCP | LDAP | Windows (AD) |
| 443 | TCP | HTTPS | Linux/Windows |
| 445 | TCP | SMB | Windows |
| 636 | TCP | LDAPS | Windows (AD) |
| 1433 | TCP | MSSQL | Windows |
| 3306 | TCP | MySQL | Linux/Windows |
| 3389 | TCP | RDP | Windows |
| 5985 | TCP | WinRM | Windows |
| 6379 | TCP | Redis | Linux/Windows |
| 8080 | TCP | HTTP Alt | Linux/Windows |
| 8443 | TCP | HTTPS Alt | Linux/Windows |
| 27017 | TCP | MongoDB | Linux/Windows |
