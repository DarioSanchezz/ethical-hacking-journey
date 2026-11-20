# Lección 06: Cracking de contraseñas (hashes y ataques en vivo)

> Práctica en mi laboratorio (Metasploitable 2). Nunca contra sistemas sin autorización.

## Qué es un hash y por qué no se "descifra"
Una función hash convierte una contraseña en una cadena de longitud fija, de un solo
sentido (no se puede volver atrás). "Crackear" un hash = probar contraseñas, calcular
su hash, y comparar si coincide con el que tengo.

## Tipos de hash habituales
MD5, SHA-1 (débiles) · SHA-256/512 (Linux moderno, `/etc/shadow`) · NTLM (Windows) ·
bcrypt (apps web modernas, diseñado para ser lento).

## La "sal" (salt)
Valor aleatorio distinto por usuario que se combina con la contraseña antes de
calcular el hash, para que contraseñas iguales no den el mismo hash. Se guarda junto
al hash (no es secreta).

## Formato de `/etc/shadow`
```
usuario:$id$sal$hash:...
```
`$1$`=MD5, `$5$`=SHA-256, `$6$`=SHA-512, `$2y$`/`$2b$`=bcrypt. `*` o `!` = cuenta sin
contraseña utilizable.

## Preparar los hashes
```bash
unshadow passwd.txt shadow.txt > hashes.txt
```
Combina `/etc/passwd` y `/etc/shadow` en el formato que espera John.

## John the Ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hashes.txt   # prueba variantes
```
John recuerda lo ya encontrado: en pasadas posteriores solo ataca lo que falta.

## Hashcat (misma idea, usando GPU)
```bash
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
`-m` = tipo de hash (1800 = sha512crypt) · `-a 0` = modo diccionario.

## Hydra: atacar un servicio en vivo (no un hash robado)
```bash
hydra -l usuario -p contraseña ssh://IP        # una credencial
hydra -l usuario -P lista.txt ssh://IP         # un usuario, lista de contraseñas
```
`-l`/`-p` minúscula = un solo valor · `-L`/`-P` mayúscula = una lista.

| | Cracking de hash | Ataque en vivo (Hydra) |
|---|---|---|
| Necesita | El hash robado | Solo la IP |
| Velocidad | Muy rápido, local | Lento, por red |
| Riesgo de detección | Ninguno | Alto (bloqueos, alarmas) |

## Un obstáculo real: algoritmos SSH obsoletos
Al atacar el SSH de Metasploitable (OpenSSH 4.7, de 2007) con Hydra, apareció:
```
kex error: no match for method mac algo
```
Kali moderna, por seguridad, deshabilita MACs antiguos (`hmac-md5`, `hmac-sha1`) que
Metasploitable es lo único que sabe hablar. Solución: permitir esos algoritmos en
`/etc/ssh/ssh_config` con `+algoritmo` (añade sin quitar los modernos), sin tocar la
seguridad de mis propias conexiones a otros servidores.

**Esto es, en sí mismo, un hallazgo**: un SSH que solo soporta cifrados obsoletos es
una vulnerabilidad de configuración, más allá de las contraseñas.

## Qué he aprendido
Entendí que crackear un hash es probar y comparar, no descifrar, y para qué sirve la
sal. Practiqué con John sobre los hashes reales de Metasploitable y vi que encuentra
contraseñas muy débiles en segundos (`service:service`, `sys:batman`). También aprendí
que John recuerda lo que ya encontró y solo ataca lo pendiente en pasadas posteriores,
y que con un diccionario pequeño y dirigido (sabiendo que `msfadmin` suele tener esa
contraseña) se puede confirmar algo en segundos sin recorrer 14 millones de palabras.
Con Hydra vi la diferencia entre crackear un hash robado y atacar un servicio en vivo,
mucho más lento y ruidoso. Y me encontré con un problema real de compatibilidad
criptográfica entre mi Kali moderna y un servidor SSH de 2007, que tuve que resolver
ajustando la configuración SSH de mi propio cliente, sin debilitar mi seguridad general.

Detalle completo con resultados reales en
[writeups/cracking-metasploitable.md](../writeups/cracking-metasploitable.md).
