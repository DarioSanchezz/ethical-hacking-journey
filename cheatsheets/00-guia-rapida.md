# Guía rápida: "quiero hacer X → uso Y"

Guía para consultar cuando no recuerdo un comando. Está ordenada por **lo que quiero
conseguir**, no por herramienta. `IP` = la IP del objetivo (en mi lab, Metasploitable).

> Solo se practica en mi laboratorio o con autorización por escrito.

**Última actualización:** Lección 08

---

## 0. El método (la receta que repito en cada máquina)

1. **Descubrir** el equipo → `nmap -sn RED/24`
2. **Escanear** todos los puertos → `sudo nmap -sS -p- -T4 IP`
3. **Identificar** versiones y scripts → `nmap -sC -sV IP`
4. **Enumerar** cada puerto abierto → ver sección 11
5. **Buscar vulnerabilidades** de cada versión → `searchsploit` y NVD
6. **Documentar** (writeup en GitHub)

---

## 1. Moverme por Linux
| Quiero... | Comando |
|---|---|
| Saber quién soy | `whoami` |
| Saber en qué carpeta estoy | `pwd` |
| Ver qué hay (con ocultos y detalles) | `ls -la` |
| Entrar en una carpeta / volver a la mía | `cd carpeta` / `cd ~` |
| Crear una carpeta | `mkdir nombre` |
| Escribir texto en un archivo | `echo "texto" > archivo` |
| Leer un archivo | `cat archivo` (o `less archivo` para archivos largos) |
| Copiar archivos | `cp origen destino` |
| Ejecutar como administrador | `sudo comando` |
| Buscar una palabra en una salida | `comando \| grep palabra` |
| Ver el manual de un comando | `man comando` (salir con `q`) |
| Ver la ayuda rápida | `comando --help` |
| Ver ejemplos rápidos | `tldr comando` |

## 2. Trabajar más rápido en la terminal
| Quiero... | Cómo |
|---|---|
| Autocompletar | `Tab` |
| Buscar un comando que ya usé | `Ctrl + R` y escribir parte del comando |
| Ver mi historial | `history \| grep palabra` |
| Cancelar un comando colgado | `Ctrl + C` |
| Copiar / pegar en la terminal de Kali | `Ctrl + Shift + C` / `Ctrl + Shift + V` |
| Poner el teclado en español (temporal) | `setxkbmap es` |
| Teclado en español (permanente) | `sudo dpkg-reconfigure keyboard-configuration` |

## 3. Mi red
| Quiero saber... | Comando |
|---|---|
| Mi IP, máscara, MAC y estado | `ip a` (resumen: `ip -br a`) |
| Mi puerta de enlace | `ip route` (línea `default via`) |
| Si hay conexión con una IP | `ping -c 4 IP` (`-c 4` = enviar 4 y parar) |
| Si falla la conexión o el DNS | `ping -c 4 8.8.8.8` y `ping -c 4 google.com` |
| Qué puertos tengo abiertos yo | `sudo ss -tulpn` |
| Por qué routers pasa un paquete | `traceroute IP` |
| Arrancar / parar un servicio | `sudo systemctl start ssh` / `stop ssh` |

## 4. DNS y web
| Quiero... | Comando |
|---|---|
| Resolver un nombre a IP | `dig nombre` / `dig +short nombre` |
| Ver servidores de correo / DNS / texto | `dig nombre MX` / `NS` / `TXT` |
| Ver solo las cabeceras HTTP (servidor y versión) | `curl -I http://IP` |
| Ver toda la petición y respuesta | `curl -v http://IP` |

## 5. Descubrir equipos y escanear puertos (nmap)
| Quiero... | Comando |
|---|---|
| Ver qué equipos hay vivos | `nmap -sn 10.0.2.0/24` |
| Escaneo básico (1000 puertos comunes) | `nmap IP` |
| Escaneo SYN, el habitual | `sudo nmap -sS IP` |
| **Todos** los puertos | `sudo nmap -sS -p- -T4 IP` |
| Solo ciertos puertos | `nmap -p 21,22,80 IP` |
| Los 100 / N puertos más comunes | `nmap -F IP` / `--top-ports N` |
| Servicio y **versión** | `sudo nmap -sV IP` |
| Scripts por defecto + versión (el combo) | `sudo nmap -sC -sV IP` |
| Sistema operativo | `sudo nmap -O IP` |
| Todo en uno (ruidoso) | `sudo nmap -A IP` |
| Un script concreto / categoría | `nmap --script nombre -p PUERTO IP` / `--script vuln` |
| Puertos UDP | `sudo nmap -sU --top-ports 20 IP` |
| Ir más rápido | `-T4` |
| Ver más detalle mientras escanea | `-v` |
| Guardar el resultado | `-oA nombre` (crea `.nmap`, `.xml`, `.gnmap`) |

Estados de un puerto: **open** (servicio escuchando), **closed** (sin servicio),
**filtered** (firewall), **open|filtered** (no se sabe, típico en UDP).

## 6. Buscar vulnerabilidades
| Quiero... | Comando o sitio |
|---|---|
| Saber si una versión tiene exploits | `searchsploit nombre versión` |
| Búsqueda exacta | `searchsploit -e "vsftpd 2.3.4"` |
| Buscar por CVE | `searchsploit --cve 2011-2523` |
| Leer el código de un exploit (sin ejecutarlo) | `searchsploit -x ID` |
| Copiar un exploit a mi carpeta | `searchsploit -m ID` |
| Ver gravedad (CVSS) y descripción de un CVE | `nvd.nist.gov` |
| Que nmap asocie versiones con CVEs | `nmap -sV --script vulners IP` |
| Actualizar la base de datos | `sudo apt update && sudo apt install exploitdb` |

Gravedad CVSS: 0,1-3,9 baja · 4,0-6,9 media · 7,0-8,9 alta · 9,0-10,0 crítica.

## 7. Hablar directamente con un puerto
| Quiero... | Comando |
|---|---|
| Ver el banner de un puerto | `nc -nv IP puerto` |
| Comprobar si existe un usuario en el correo | `nc -nv IP 25` y luego `VRFY nombre` (252 = existe, 550 = no) |
| Salir de la conexión | `QUIT` o `Ctrl + C` |

## 8. Enumerar servicios
| Quiero... | Comando |
|---|---|
| Probar FTP anónimo | `ftp IP` (usuario `anonymous`) |
| Listar carpetas compartidas SMB | `smbclient -L //IP -N --option='client min protocol=NT1'` |
| Enumerar Samba en bloque (usuarios, recursos) | `enum4linux -a IP` |
| Identificar tecnologías de una web | `whatweb http://IP` |
| Buscar fallos web conocidos (ruidoso) | `nikto -h http://IP` |
| Buscar directorios ocultos | `gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt` |
| Ver carpetas compartidas por NFS | `showmount -e IP` |
| Ver servicios RPC | `rpcinfo -p IP` |
| Ver si MySQL tiene contraseña vacía | `nmap --script mysql-empty-password -p3306 IP` |

## 9. "Veo este puerto → enumero con..."
| Puerto | Servicio | Herramienta |
|---|---|---|
| 21 | FTP | `ftp` anónimo, `ftp-anon` |
| 22 | SSH | anotar versión |
| 23 | Telnet | `nc` (banner) |
| 25 | SMTP | `nc` + `VRFY` |
| 53 | DNS | `dig` |
| 80 / 443 / 8180 | Web | `curl -I`, `whatweb`, `gobuster`, `nikto`, navegador |
| 111 / 2049 | RPC / NFS | `rpcinfo`, `showmount` |
| 139 / 445 | SMB | `smbclient`, `enum4linux` |
| 3306 | MySQL | script `mysql-empty-password` |
| Otro raro | ? | `nc -nv IP puerto` |
| Siempre | cualquier versión | `searchsploit` y NVD |

## 10. Puertos conocidos
21 FTP · 22 SSH · 23 Telnet · 25 SMTP · 53 DNS · 80 HTTP · 110 POP3 · 111 RPC ·
139/445 SMB · 143 IMAP · 443 HTTPS · 2049 NFS · 3306 MySQL · 3389 RDP ·
5432 PostgreSQL · 5900 VNC · 8180 Tomcat (en Metasploitable)

## 11. Laboratorio
| Quiero... | Cómo |
|---|---|
| Actualizar Kali | `sudo apt update && sudo apt full-upgrade -y` |
| Cambiar la contraseña | `passwd` |
| Instalar un programa | `sudo apt install nombre` |
| Volver a un estado limpio de la VM | Instantáneas (snapshots) de VirtualBox |
| Copiar texto entre la VM y Windows | Dispositivos → Portapapeles compartido → Bidireccional |
| Compartir una carpeta con Windows | Carpetas compartidas + `sudo usermod -aG vboxsf $USER` y reiniciar |

## 12. Acceso remoto
| Quiero... | Comando |
|---|---|
| Entrar por terminal a otra máquina | `ssh usuario@IP` |
| Usar otro puerto | `ssh -p PUERTO usuario@IP` |
| Ver por qué falla la conexión | `ssh -v usuario@IP` |
| Activar el servidor SSH | `sudo systemctl enable --now ssh` |
| Red privada entre mis equipos | Tailscale (`sudo tailscale up`) |

## 13. Git y GitHub
| Quiero... | Comando |
|---|---|
| Descargar un repositorio | `git clone URL` |
| Ver qué ha cambiado | `git status` |
| Marcar todos los cambios | `git add .` |
| Guardar una instantánea | `git commit -m "mensaje"` |
| Subirlo a GitHub | `git push` |
| Bajar cambios de GitHub | `git pull` |
| Ver el historial resumido | `git log --oneline` |

## 14. Cuando algo falla
| Problema | Causa y solución |
|---|---|
| `ping` en Linux no termina | Falta `-c`: `ping -c 4 IP` |
| `smbclient` falla negociando el protocolo | Añadir `--option='client min protocol=NT1'` |
| `whatweb` se queda colgado | Comprobar con `curl -I --max-time 10 http://IP`; usar `--open-timeout 10 --read-timeout 15` |
| `git status` dice `not a git repository` | Estás en otra carpeta, o falta la carpeta oculta `.git` |
| Aviso `LF will be replaced by CRLF` | Es solo un aviso de Windows. Se ignora |
| La VM no ve internet | Revisar que el adaptador esté en Red NAT y la interfaz `UP` |
| Un puerto no aparece en el escaneo | Probablemente fuera de los 1000 comunes: usa `-p-` |

---

---

## 15. Explotación con Metasploit
| Quiero... | Comando |
|---|---|
| Arrancar Metasploit | `sudo msfconsole` |
| Buscar un exploit por software | `search nombre` |
| Seleccionar un módulo | `use ruta/del/modulo` |
| Ver qué hay que configurar | `options` (o `show options`) |
| Poner la IP objetivo | `set RHOSTS IP` |
| Ver payloads disponibles | `show payloads` |
| Elegir un payload | `set payload nombre` |
| Lanzar el exploit | `exploit` (o `run`) |
| Confirmar que estoy dentro | `whoami`, `id`, `hostname` |
| Salir de la shell / de msfconsole | `exit` |
| Borrar un payload que se ha quedado "pegado" de antes | `unset payload` |
| Borrar todas las variables guardadas en la sesión | `unset all` |

**Flujo fijo:** `search` → `use` → `options` → `set RHOSTS IP` → `exploit`.

## 16. Dentro de una sesión de Meterpreter
| Quiero... | Comando |
|---|---|
| Ver todos los comandos disponibles | `help` |
| Ver información del sistema | `sysinfo` |
| Saber con qué usuario estoy (equivalente a whoami) | `getuid` |
| Bajar a una shell normal del sistema | `shell` |
| Salir de esa shell y volver a Meterpreter | `exit` o `Ctrl+D` |
| Dejar la sesión abierta y volver a msfconsole | `background` |
| Ver / volver a una sesión | `sessions -l` / `sessions -i N` |
| Ver usuarios del sistema (desde `shell`) | `cat /etc/passwd` |
| Ver hashes de contraseñas, solo como root (desde `shell`) | `cat /etc/shadow` |

## Pendiente de añadir en próximas lecciones
Escalada de privilegios, contraseñas (Hydra), Wireshark y detección con SIEM.
