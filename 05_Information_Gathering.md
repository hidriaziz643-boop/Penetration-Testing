# Information Gathering

## OSINT

```bash
# WHOIS
whois megacorpone.com

# TheHarvester – E-Mails, Subdomains, Hosts
theHarvester -d megacorpone.com -b google
theHarvester -d megacorpone.com -b all

# Shodan – Internet-verbundene Geräte
# https://www.shodan.io/ → Host:IP oder nach Diensten suchen

# Maltego – grafische OSINT Analyse (GUI Tool)
```

## Nmap – Network Scanner

```bash
# Standard Scan
nmap -sS -sV -A 192.168.6.10

# Wichtige Optionen:
# [-p <port>]        spezifischer Port  z.B. -p 80,443 oder -p 1-1000
# [-p-]              alle 65535 Ports
# [--top-ports=100]  Top 100 Ports
# [-sn]              Ping Scan – nur Host Discovery, kein Port Scan
# [-Pn]              Skip Host Discovery (treat all hosts as online)
# [-sU]              UDP Scan (dauert lange)
# [-sV]              Version Detection (Dienst-Version erkennen)
# [-O]               OS Detection
# [-A]               OS + Version + Script Scanning + Traceroute
# [-sT]              TCP connect() Scan (vollständiger Handshake)
# [-sS]              SYN Stealth Scan (Hinweis: bei VPN/SSH-Tunnel/Proxy beachten)
# [-T<0-5>]          Timing Template (0=paranoid, 5=insane)
# [-oG datei.txt]    Ergebnis in Datei speichern (grepable)
# [-oX datei.xml]    XML-Ausgabe
# [-oA prefix]       alle Formate gleichzeitig

# Port-Status: open | filtered | closed | unfiltered

# Nmap Script Engine (NSE)
# [-sC]              Standard-Scripts
# [--script=<name>]  spezifisches Script

# Beispiele
nmap -sV -p 80,443 10.10.10.10
nmap -sS -sV -A -p- 10.10.10.10
nmap -sn 10.10.10.0/24                    # alle Hosts im Netz finden
nmap -v -p 139,445 -oG smb.txt 10.11.1.1-254

# Automator Script
# https://github.com/21y4d/nmapAutomator
# Nmap Mindmap: https://github.com/Ignitetechnologies/Mindmap/blob/main/Nmap/nmap%20HD.png
```

## Masscan – schneller Port Scanner

```bash
masscan -p1-65535 10.10.10.10 --rate=1000
# --rate : Pakete pro Sekunde (viel schneller als nmap)
# Für ersten Überblick, dann nmap für Details
```

## Banner Grabbing

```bash
nc -vn 192.168.6.44 21    # Banner Grabbing mit netcat
# -v : verbose
# -n : numerische IP (kein DNS)
# Zeigt: Dienst + Version direkt beim Verbinden
```

## Port Enumeration Ressourcen

```
"<portnummer> enumeration" → Google
https://ptestmethod.readthedocs.io/en/latest/LFF-IPS-P2-VulnerabilityAnalysis.html
https://github.com/Ignitetechnologies/Mindmap/blob/main/Enumeration/Enumeration%20HD.png
https://book.hacktricks.xyz/network-services-pentesting/pentesting-<port>
```
