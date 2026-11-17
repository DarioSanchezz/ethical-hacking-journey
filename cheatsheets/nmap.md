# Chuleta de nmap

## Descubrimiento
```bash
nmap -sn 10.0.2.0/24            # equipos vivos
```

## Escaneo de puertos
```bash
nmap IP                          # 1000 puertos más comunes
sudo nmap -sS -p- -T4 IP         # SYN, todos los puertos, rápido
nmap -F IP                       # 100 más comunes
nmap --top-ports 20 IP
sudo nmap -sU --top-ports 20 IP  # UDP
```

## Servicios, SO y scripts
```bash
sudo nmap -sV IP                 # versiones
sudo nmap -O IP                  # sistema operativo
sudo nmap -sC -sV IP             # scripts por defecto + versiones
nmap --script ftp-anon -p 21 IP
nmap --script vuln IP            # ruidoso: solo en laboratorio o con permiso
ls /usr/share/nmap/scripts | grep ftp
```

## Guardar resultados
```bash
sudo nmap -sC -sV -oA escaneo_meta IP   # .nmap, .xml y .gnmap
```

## Estados de puerto
open · closed · filtered · unfiltered · open|filtered

## Plantillas de velocidad
T0 paranoico · T1 sigiloso · T3 normal · T4 rápido · T5 agresivo
