# Lección 11: Control de acceso por roles (RBAC) y primer contacto con OWASP ZAP

> Práctica contra OWASP Juice Shop (contenedor Docker propio).

## Control de acceso roto: la categoría #1 del OWASP Top 10
No es casualidad que vaya por delante incluso de la inyección SQL: es el fallo más
común y el más fácil de introducir sin darse cuenta.

| Tipo | Qué significa | Ejemplo visto |
|---|---|---|
| **Escalada horizontal** | Un usuario normal accede a los datos de **otro usuario normal** | IDOR de la cesta (Lección 08) |
| **Escalada vertical** | Un usuario normal accede a funciones **reservadas para un rol superior** | Lo de hoy: listar y borrar datos de admin |

La causa suele ser la misma de siempre: **el servidor confía en que, si el botón no
se ve, nadie va a llamar a esa función** — y no vuelve a comprobar el rol cuando la
petición llega directamente a su endpoint.

## OWASP ZAP: qué es y qué pasó al usarlo
ZAP es como Burp, pero gratuito y con **escaneo automático** incluido (en Burp
Community eso es de pago). Instalación:
```bash
sudo apt install -y zaproxy
zaproxy &
```
Al lanzar un **Active Scan** contra Juice Shop desde "Quick Start" (URL de
objetivo + spider tradicional + Attack), la VM se quedó sin recursos y se
reinició sola a mitad de escaneo (ZAP en Java + Docker + Firefox a la vez piden
bastante RAM). Se confirma con:
```bash
dmesg | grep -i -E "out of memory|oom"
```
**Lección de laboratorio:** un escaneo automático completo necesita más recursos
de los que parece; conviene no correr Burp/ZAP, Docker y el navegador a la vez en
una VM justa de RAM, y asignar más memoria (VM apagada → Configuración →
Sistema → Memoria base) antes de reintentarlo.

## Encontrar una ruta "oculta" de verdad: SPA vs servidor real
`gobuster`/`dirb` buscan carpetas y archivos que existen **de verdad en el
servidor**. Una ruta como `/#/administration` es de **Angular, del lado del
cliente** (el `#` lo delata) — no es una carpeta real, así que la fuerza bruta de
directorios **no la encuentra nunca**, por bueno que sea el diccionario.
La forma real de encontrar rutas de cliente ocultas es inspeccionar el propio
JavaScript que el navegador descarga (ahí vive la configuración de rutas de
Angular), con la pestaña Red del navegador o descargando el bundle con `curl` y
buscando patrones como `path:"..."` dentro.

## Auditar una API por completo con `curl`, sin pasar por el navegador
Cada paso usado hoy, en orden:
```bash
# 1. Login por SQLi, conseguir el token de admin
curl -s -X POST http://IP:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"' OR 1=1--\",\"password\":\"x\"}"

# 2. Probar un endpoint de API con ese token (control positivo)
curl -s http://IP:3000/api/Users/ -H "Authorization: Bearer TOKEN"

# 3. Registrar un usuario normal (en el navegador) y loguearlo igual que en el paso 1,
#    pero con sus credenciales reales en vez del bypass de SQLi

# 4. Repetir el paso 2 con el token del usuario normal → comprobar si también funciona

# 5. Probar escritura/borrado, no solo lectura
curl -s -X DELETE http://IP:3000/api/Feedbacks/3 -H "Authorization: Bearer TOKEN"
```
`-X POST`/`-X DELETE` cambian el método HTTP (por defecto curl usa GET).
`-H "Authorization: Bearer TOKEN"` es la cabecera que algunos endpoints
(distintos a los que usan cookie) exigen para autenticar.

## Resultado: doble confirmación del fallo
Con el token de un usuario **normal**, sin ningún privilegio:
- `GET /api/Users/` devolvió el listado **completo** de usuarios, igual que con el
  admin.
- `DELETE /api/Feedbacks/3` **borró de verdad** una reseña de otro usuario.

No es solo "ver algo que no debería" — es la capacidad de **destruir datos ajenos**
sin ningún permiso especial, solo con una cuenta cualquiera recién registrada.

## Qué he aprendido
Entendí la diferencia entre escalada horizontal y vertical, y vi en la práctica
por qué el control de acceso roto es la categoría más importante del OWASP Top 10:
aquí ni siquiera hizo falta ningún truco de inyección, solo llamar directamente a
un endpoint con un token válido cualquiera. Aprendí una distinción técnica
importante que no tenía clara: la fuerza bruta de directorios (`gobuster`) no
sirve para encontrar rutas de una SPA como Angular, porque esas rutas no existen
como archivos en el servidor — hay que mirar el JavaScript del cliente para
encontrarlas de verdad. También tuve un incidente real de laboratorio (la VM se
quedó sin memoria y se reinició sola al correr ZAP, Docker y Firefox a la vez),
que diagnostiqué con `dmesg` en vez de asumir cualquier cosa. Y, sobre todo,
aprendí a hacer todo el proceso de autenticación y prueba de una API completamente
por `curl`, sin depender del navegador ni de la búsqueda (de pago) de Burp
Community, lo cual es además más rápido y más fácil de documentar con evidencia
exacta.

Detalle completo con evidencia real en
[writeups/juice-shop-rbac-api.md](../writeups/juice-shop-rbac-api.md).
