# Lección 08: Burp Suite y primeras vulnerabilidades web reales

> Práctica contra OWASP Juice Shop (contenedor Docker propio), app oficial de
> entrenamiento. Nunca contra sistemas sin autorización.

## Burp Suite: qué es y para qué
Proxy de intercepción: se coloca entre el navegador e internet, dejando ver, pausar,
editar y reenviar cada petición antes de que salga. Versión usada: **Community**
(gratuita), suficiente para interceptar, usar Repeater y Decoder.
```
Navegador → Burp Suite (127.0.0.1:8080) → Servidor web
```

## Configuración del proxy en Firefox
Configuración → red → proxy manual: Host `127.0.0.1`, puerto `8080`, marcar también
para HTTPS.

### Problema real: Firefox excluye `localhost` del proxy por defecto
Firefox, por seguridad, no manda a través del proxy el tráfico a `localhost`/`127.0.0.1`
aunque todo esté bien configurado. **Solución:** acceder a la app por la IP real de
Kali (`http://10.0.2.15:3000`) en vez de `localhost`, o vaciar el campo "No usar proxy
para" en la configuración de red de Firefox.

## Capturar peticiones
Proxy → Intercept → "Intercept is on". Con eso activado cada petición se pausa hasta
pulsar **Forward**. En la práctica, mejor navegar con Intercept **desactivado** y
activarlo justo antes de la acción a capturar.

## Hallazgo 1: ruta oculta sin control de acceso
```
http://10.0.2.15:3000/#/score-board
```
Accesible directamente sin haber iniciado sesión. **Control de acceso roto**: una ruta
no listada en el menú sigue siendo pública si no se protege explícitamente en el
servidor.

## Hallazgo 2: inyección SQL en el login
En el campo email del login:
```
' OR 1=1--
```
Contraseña: cualquiera. La consulta SQL generada queda, aproximadamente:
```sql
SELECT * FROM users WHERE email = '' OR 1=1--' AND password = '...'
```
`'` cierra la comilla de la consulta · `OR 1=1` condición siempre verdadera · `--`
comenta el resto (la comprobación de contraseña). Resultado: login como el primer
usuario de la base de datos, el administrador.

## Hallazgo 3: IDOR en la cesta de la compra
Las SPA (Juice Shop está hecha en Angular) no muestran en la URL la petición real al
servidor: hay que mirarla en la pestaña Red del navegador o en Burp. La petición real es:
```
GET /rest/basket/N
```
Cambiando `N` por el ID de otro usuario, el servidor devolvió los datos de **otra
cesta**, sin comprobar que perteneciera al usuario autenticado. Esto es un **IDOR**
(*Insecure Direct Object Reference*).

## Qué he aprendido
Aprendí a usar Burp Suite como proxy de intercepción, y me encontré con un problema
real (la exclusión de `localhost` en Firefox) que tuve que diagnosticar y resolver
usando la IP real de la máquina en vez de `localhost`. Practiqué una inyección SQL
clásica para saltarme el login, y entendí la lógica exacta de por qué `' OR 1=1--`
funciona a nivel de la consulta SQL generada. También aprendí que en aplicaciones de
página única (SPA) la URL visible no es la petición real, y hay que mirar el tráfico de
red para encontrar los identificadores que usa la API; con eso confirmé un IDOR real al
acceder a la cesta de otro usuario solo cambiando un número. El hilo común de los tres
hallazgos es el mismo: la aplicación confía en que el usuario no va a mirar donde no
debe, en vez de comprobar permisos en el servidor en cada petición.

Detalle completo con evidencia real en
[writeups/juice-shop-primeras-vulnerabilidades.md](../writeups/juice-shop-primeras-vulnerabilidades.md).
