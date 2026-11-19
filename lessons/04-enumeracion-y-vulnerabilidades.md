# Lección 04: Enumeración de servicios y búsqueda de vulnerabilidades

> Todo se practica contra mi laboratorio (Metasploitable 2 en la Red NAT `LabHacking`).
> En esta lección **no se explota nada**: se identifica y se documenta.

## Escaneo frente a enumeración
- **Escaneo** (nmap): qué puertos hay abiertos y qué servicio parece correr en cada uno.
- **Enumeración**: sacar información detallada de cada servicio (recursos compartidos,
  usuarios, directorios web, banners, configuraciones débiles).

## Vocabulario clave
| Término | Significado |
|---|---|
| Vulnerabilidad | Fallo o debilidad en un software o configuración |
| CVE | Identificador público de una vulnerabilidad (`CVE-AÑO-NÚMERO`) |
| CVSS | Puntuación de gravedad de 0 a 10 (Baja, Media, Alta, Crítica) |
| Exploit | Código que aprovecha una vulnerabilidad |
| Payload | Lo que se ejecuta una vez explotada (por ejemplo, una shell) |
| PoC | Prueba de concepto: demuestra que el fallo existe |

Fuentes: NVD (nvd.nist.gov), cve.mitre.org y Exploit-DB.

## searchsploit (Exploit-DB sin conexión)
| Comando | Qué hace |
|---|---|
| `searchsploit vsftpd 2.3.4` | Busca exploits para ese software y versión |
| `searchsploit -e "vsftpd 2.3.4"` | Búsqueda exacta |
| `searchsploit --cve 2011-2523` | Busca por CVE |
| `searchsploit -x ID` | Muestra el código para leerlo |
| `searchsploit -m ID` | Copia el exploit a la carpeta actual |
| `sudo apt update && sudo apt install exploitdb` | Actualiza la base de datos |

## Enumeración por servicio
| Servicio | Comandos |
|---|---|
| FTP (21) | `ftp IP` con usuario `anonymous`; `nmap --script ftp-anon -p21 IP` |
| SMB (139/445) | `smbclient -L //IP -N --option='client min protocol=NT1'`; `enum4linux -a IP` |
| HTTP (80) | `curl -I http://IP`; `whatweb http://IP`; `nikto -h http://IP`; `gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt` |
| SMTP (25) | `nc -nv IP 25` y después `VRFY root` |
| Telnet (23) | `nc -nv IP 23` (el banner ya revela información) |
| NFS (2049) | `showmount -e IP`; `rpcinfo -p IP` |
| MySQL (3306) | `nmap --script mysql-empty-password,mysql-info -p3306 IP` |
| Cualquier puerto | `nc -nv IP PUERTO` para ver el banner |

## Flujo de trabajo
1. Escaneo con `-sV` → software y versión.
2. `searchsploit` y NVD → ¿hay vulnerabilidades conocidas?
3. Enumerar el servicio para confirmar cómo está configurado.
4. Valorar la gravedad (CVSS) y priorizar.
5. Documentar: servicio, versión, CVE, gravedad, evidencia y cómo se corrige.

## Qué he aprendido
Entendí la diferencia entre escanear y enumerar: nmap me dice qué hay y la enumeración
me dice cómo está configurado. Aprendí a convertir una versión en una vulnerabilidad
concreta con `searchsploit` y NVD, y a leer un CVE y su puntuación CVSS para priorizar.
Practiqué cómo enumerar FTP, SMB, HTTP, SMTP, NFS y MySQL, y vi que muchos servicios
revelan información útil solo con conectarse, como los banners. También entendí que
el escaneo por defecto no mira todos los puertos, y que por eso `-p-` es imprescindible
para no dejar servicios sin revisar. Y que documentar bien (versión, CVE, gravedad y
solución) es lo que convierte un hallazgo en un informe útil.
