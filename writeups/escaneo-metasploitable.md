# Escaneo y análisis de Metasploitable 2 con nmap

**Entorno:** laboratorio propio (Kali Linux → Metasploitable 2) en una Red NAT de VirtualBox aislada
**Objetivo escaneado:** `10.0.2.3` (IP privada del laboratorio)
**Categoría:** Reconocimiento y enumeración
**Fecha:** 17/11/2026

## Objetivo
Identificar qué servicios expone Metasploitable 2, qué versiones corren y cuáles
representan un riesgo, como primera fase de un test de intrusión.

## Comando ejecutado
```bash
sudo nmap -sC -sV -oA escaneo_meta 10.0.2.3
```
- `-sV`: detecta servicio y versión de cada puerto abierto.
- `-sC`: ejecuta los scripts NSE por defecto.
- `-oA escaneo_meta`: guarda la salida en los tres formatos.

Resultado completo: [`lab/scans/escaneo_meta.nmap`](../lab/scans/escaneo_meta.nmap).
El escaneo tardó unos 21 segundos.

## Resultado general
- El equipo respondía y mostraba **23 puertos TCP abiertos** de los 1000 más comunes
  (los otros 977 estaban cerrados).
- El sistema es Linux (Debian/Ubuntu antiguo), y la MAC corresponde a una tarjeta
  virtual de VirtualBox.

## Hallazgos
| Puerto | Servicio y versión | Qué observé | Riesgo |
|---|---|---|---|
| 21 | vsftpd 2.3.4 | El script `ftp-anon` indica que se permite **login anónimo** | Acceso sin credenciales; además, esta versión tiene una puerta trasera conocida (CVE-2011-2523) |
| 22 | OpenSSH 4.7p1 | Claves de host DSA de 1024 bits | Versión muy antigua y criptografía débil |
| 23 | Linux telnetd | Telnet abierto | Acceso remoto sin cifrar: credenciales en claro |
| 25 | Postfix smtpd | Soporta **SSLv2** con cifrados débiles y certificado caducado en 2010; permite `VRFY` | Cifrado roto y enumeración de usuarios |
| 53 | ISC BIND 9.4.2 | Revela su versión (`bind.version`) | Versión antigua que filtra información |
| 80 | Apache 2.2.8 (Ubuntu) DAV/2 | Título "Metasploitable2 - Linux" | Servidor y aplicaciones web desactualizados |
| 111 / 2049 | rpcbind y NFS | `rpcinfo` lista nfs, mountd y nlockmgr | NFS puede exponer carpetas compartidas |
| 139 / 445 | Samba 3.0.20 | Firma de mensajes SMB **deshabilitada** | Versión con vulnerabilidad de ejecución de comandos (CVE-2007-2447) |
| 512-514 | rexec, rlogin, rsh | Servicios "r" accesibles | Obsoletos, sin seguridad real |
| 1099 | Java RMI (GNU Classpath) | Registro RMI expuesto | Posible ejecución remota de código |
| 1524 | bindshell | Nmap lo identifica como **"Metasploitable root shell"** | Un shell de root escuchando sin autenticación |
| 2121 | ProFTPD 1.3.1 | Segundo servidor FTP | Versión antigua |
| 3306 | MySQL 5.0.51a | Responde a cualquier cliente de red | Base de datos expuesta |
| 5432 | PostgreSQL 8.3.x | Certificado caducado en 2010 | Base de datos expuesta y versión muy antigua |
| 5900 | VNC (protocolo 3.3) | Solo autenticación por contraseña | Escritorio remoto con protección débil |
| 6000 | X11 | "access denied" | Servicio gráfico expuesto a la red |
| 6667 | UnrealIRCd (Unreal3.2.8.1) | Versión identificada por `irc-info` | Versión con puerta trasera conocida |
| 8009 | Apache Jserv (AJP 1.3) | Conector AJP abierto | No debería exponerse fuera de Tomcat |
| 8180 | Apache Tomcat 5.5 | Panel de Tomcat accesible | Versión obsoleta y panel de administración |

## Análisis
- **Superficie de ataque enorme:** 23 servicios, la mayoría innecesarios para cualquier equipo.
- **Versiones antiguas y reconocibles:** `-sV` permite cruzar cada versión con bases de
  datos de vulnerabilidades, que es el siguiente paso.
- **Credenciales sin proteger:** FTP anónimo, Telnet y los servicios "r" no cifran nada.
- **Configuración débil de cifrado:** SSLv2 en SMTP y certificados caducados desde 2010.
- **Servicios con puerta trasera deliberada:** vsftpd 2.3.4, UnrealIRCd y el bindshell del puerto 1524.

## Cómo se previene
- Desactivar todo servicio que no sea necesario y filtrar el resto con un firewall.
- Mantener el software actualizado y parcheado.
- Sustituir protocolos sin cifrar por equivalentes seguros (SSH en lugar de Telnet, SFTP en lugar de FTP).
- Deshabilitar el FTP anónimo y el SSLv2, y renovar certificados caducados.
- No exponer bases de datos a la red; limitar el acceso por IP.
- Activar la firma de mensajes en SMB.
- Hacer inventarios y escaneos periódicos propios para detectar lo que queda expuesto.

## Qué aprendí
Que el reconocimiento determina todo lo que viene después: la versión exacta de un
servicio decide si hay algo que explotar. Con solo `-sC -sV` salieron ya detalles
muy útiles (FTP anónimo, SSLv2, firma SMB desactivada). Vi también que la misma
información que usa un atacante es la que necesita un defensor para saber qué tiene
expuesto.

## Siguiente paso
Escaneo de todos los puertos (`-p-`), detección de sistema operativo (`-O`) y
enumeración de los servicios más interesantes con búsqueda de vulnerabilidades
concretas (Lección 04).
