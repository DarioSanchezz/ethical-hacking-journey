# Chuleta de enumeración y búsqueda de vulnerabilidades

## Todos los puertos y SO
```bash
sudo nmap -sS -p- -T4 -oA escaneo_completo IP
sudo nmap -O -oA escaneo_so IP
```

## searchsploit
```bash
searchsploit vsftpd 2.3.4
searchsploit --cve 2011-2523
searchsploit -x ID        # leer
searchsploit -m ID        # copiar
```

## FTP
```bash
ftp IP                    # usuario: anonymous
nmap --script ftp-anon -p21 IP
```

## SMB
```bash
smbclient -L //IP -N --option='client min protocol=NT1'
enum4linux -a IP
```

## Web
```bash
curl -I http://IP
whatweb http://IP
nikto -h http://IP
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt
```

## Banners, SMTP y Telnet
```bash
nc -nv IP 25              # luego: VRFY root
nc -nv IP 23
```

## NFS y RPC
```bash
showmount -e IP
rpcinfo -p IP
```

## Gravedad CVSS
0,1-3,9 Baja · 4,0-6,9 Media · 7,0-8,9 Alta · 9,0-10,0 Crítica
