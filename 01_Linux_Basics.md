# Linux Basics

## Navigation & Dateisystem

```bash
pwd                          # aktuelles Verzeichnis
ls -la                       # Dateien + versteckte + Berechtigungen
cd /pfad                     # Verzeichnis wechseln
cd ..                        # ein Verzeichnis zurück
mkdir ordner                 # Verzeichnis erstellen
rm datei                     # Datei löschen
rm -rf ordner                # Ordner rekursiv löschen
cp quelle ziel               # kopieren
mv quelle ziel               # verschieben / umbenennen
cat datei.txt                # Datei ausgeben
less datei.txt               # Datei seitenweise lesen
head -n 20 datei.txt         # erste 20 Zeilen
tail -n 20 datei.txt         # letzte 20 Zeilen
find / -name "*.txt" 2>/dev/null   # Datei suchen
grep "suchbegriff" datei.txt       # in Datei suchen
grep -r "suchbegriff" /pfad/       # rekursiv suchen
which tool                   # Pfad eines Tools
echo "text"                  # Text ausgeben
echo "text" > datei.txt      # in Datei schreiben (überschreiben)
echo "text" >> datei.txt     # an Datei anhängen
```

## Berechtigungen

```bash
chmod +x datei               # ausführbar machen
chmod 777 datei              # alle Rechte für alle
chmod 644 datei              # rw-r--r--
chmod 600 id_rsa             # nur Owner lesen (SSH Key!)
chown user:gruppe datei      # Besitzer ändern
ls -l                        # Berechtigungen anzeigen
# Ausgabe: -rwxrwxrwx  (user|group|others)
# r=4, w=2, x=1
```

## Benutzer & Gruppen

```bash
whoami                       # aktueller User
id                           # UID, GID, Gruppen
id username                  # Info über anderen User
cat /etc/passwd              # alle User
cat /etc/shadow              # Passwort-Hashes (root nötig)
su username                  # User wechseln
su -                         # zu root wechseln
sudo -l                      # erlaubte sudo Befehle
sudo su                      # root Shell via sudo
adduser username             # User hinzufügen
passwd username              # Passwort setzen
groups                       # eigene Gruppen
cat /etc/group               # alle Gruppen
```

## Prozesse & System

```bash
ps aux                       # alle Prozesse
ps aux | grep apache         # nach Prozess filtern
kill PID                     # Prozess beenden
kill -9 PID                  # Prozess erzwingen beenden
top / htop                   # Live Prozess-Ansicht
uname -a                     # Kernel + System Info
hostname                     # Hostname
uptime                       # Laufzeit des Systems
df -h                        # Festplattenplatz
free -h                      # RAM-Nutzung
env                          # Umgebungsvariablen
history                      # Befehlshistorie
```

## Wichtige Verzeichnisse

```bash
/etc/passwd         # Benutzer-Datenbank
/etc/shadow         # Passwort-Hashes
/etc/crontab        # System Cron Jobs
/tmp/               # temporäre Dateien (immer beschreibbar)
/var/www/html/      # Apache Webroot
/home/user/         # Home-Verzeichnisse
/root/              # root Home
/usr/share/wordlists/rockyou.txt   # Kali Wortliste
/usr/share/webshells/              # Kali Webshells
/usr/share/nmap/scripts/           # Nmap NSE Scripts
/usr/share/seclists/               # SecLists Wortlisten
```

## Nützliche Befehle für Pentesting

```bash
# Basis-Enumeration nach Shell-Zugriff
whoami && id && uname -a && hostname && ip a && ps aux

# Beschreibbare Dateien finden
find / -writable -type f 2>/dev/null

# SUID Binaries finden
find / -user root -perm -4000 -print 2>/dev/null

# Passwörter in Dateien suchen
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null

# SSH Key stehlen
cat /root/.ssh/id_rsa
cat /home/user/.ssh/id_rsa

# Cron Jobs prüfen
crontab -l
cat /etc/crontab
ls -la /etc/cron*

# Netzwerk-Verbindungen
ss -tulpn          # offene Ports
netstat -tulpn     # alternative
```

## Encoding / Decoding

```bash
# Base64
echo -n "text" | base64                    # encodieren
echo "dGV4dA==" | base64 -d               # decodieren
base64 datei.txt > datei.b64              # Datei encodieren
base64 -d datei.b64 > datei.txt           # Datei decodieren

# URL Encoding (Python)
python3 -c "import urllib.parse; print(urllib.parse.quote('text /mit sonderzeichen'))"
python3 -c "import urllib.parse; print(urllib.parse.unquote('text%20%2Fmit%20sonderzeichen'))"

# KeePass Datenbank encodieren (für Transfer)
[convert]::ToBase64String((Get-Content -path "C:\users\bob\documents\Database.kdbx" -Encoding byte))
# Linux decodieren:
echo "<base64string>" | base64 -d > database.kdbx
```
