# Primeras vulnerabilidades web en OWASP Juice Shop

**Entorno:** laboratorio propio, contenedor Docker de OWASP Juice Shop (`bkimminich/juice-shop`)
**Objetivo:** `http://10.0.2.15:3000`
**Categoría:** Vulnerabilidades web (control de acceso, inyección)
**Fecha:** 23/11/2026

## Objetivo
Identificar y confirmar vulnerabilidades reales en una aplicación web moderna
(Angular/Node.js), usando Burp Suite como proxy de intercepción.

## Herramientas
- Burp Suite Community Edition, como proxy en `127.0.0.1:8080`.
- Firefox (Kali), configurado para usar ese proxy.
- Herramientas de desarrollador del navegador (pestaña Red), para encontrar las
  peticiones reales de la aplicación (es una SPA, la URL visible no refleja las
  llamadas a la API).

**Incidencia resuelta:** Firefox excluye `localhost`/`127.0.0.1` del proxy por
defecto, por lo que el tráfico no pasaba por Burp al visitar `localhost:3000` pese a
tener todo bien configurado. Se resolvió accediendo a la aplicación por la IP real de
Kali (`10.0.2.15`) en vez de `localhost`.

## Hallazgo 1: ruta oculta sin control de acceso
**Ruta:** `http://10.0.2.15:3000/#/score-board`
**Resultado:** accesible directamente, sin haber iniciado sesión.
**Impacto:** expone un panel de seguimiento de retos que no debería ser público.
**Causa:** la ruta no aparece en el menú, pero no está protegida por ningún control
en el servidor; ocultar un enlace no equivale a restringir el acceso.

## Hallazgo 2: inyección SQL — bypass de login
**Campo email:** `' OR 1=1--`
**Campo contraseña:** cualquier valor
**Resultado:** inicio de sesión como `admin@juice-sh.op` (administrador), sin conocer
su contraseña.
**Petición capturada en Burp:**
```
POST /rest/user/login HTTP/1.1
Content-Type: application/json

{"email":"' OR 1=1--","password":"x"}
```
**Causa:** la aplicación concatena el campo email directamente en la consulta SQL sin
parametrizarlo. `'` cierra la cadena de la consulta, `OR 1=1` fuerza una condición
siempre verdadera, y `--` comenta el resto de la consulta (incluida la comprobación de
contraseña), devolviendo el primer usuario de la base de datos.

## Hallazgo 3: IDOR en la cesta de la compra
Con sesión iniciada como administrador, la cesta mostraba `admin@juice-sh.op` y un
identificador numérico visible solo en la petición real (pestaña Red del navegador),
no en la URL de la aplicación:
```
GET /rest/basket/N
```
**Prueba:** se cambió el número `N` de la petición por el de otra cesta.
**Resultado:** el servidor devolvió los datos de **otra cesta distinta**, sin
comprobar que perteneciera al usuario autenticado.
**Causa:** el servidor confía en el identificador recibido sin verificar que el
usuario tiene permiso sobre ese recurso concreto. Es un IDOR (*Insecure Direct Object
Reference*).

## Análisis
Los tres hallazgos comparten la misma raíz: **la aplicación delega el control de
acceso en el cliente** (qué se muestra en el menú, qué ve el usuario en pantalla) en
vez de verificarlo siempre en el servidor. Es el patrón de fallo más común en
aplicaciones web reales, muy por delante de exploits complejos.

## Cómo se previene
- Proteger cada ruta y cada endpoint de la API con comprobación de autenticación y
  autorización en el servidor, nunca confiando en que el cliente no la pida.
- Usar siempre consultas parametrizadas (prepared statements) para eliminar la
  inyección SQL de raíz.
- Verificar en cada petición que el recurso solicitado (cesta, pedido, perfil...)
  pertenece al usuario autenticado, en vez de confiar en el identificador recibido.
- Auditar periódicamente rutas "ocultas" o no documentadas: deben tener la misma
  protección que las visibles.

## Qué aprendí
Que en aplicaciones modernas tipo SPA, mirar la barra de direcciones no basta: hay que
observar las peticiones reales a la API, con las herramientas de desarrollador o con un
proxy como Burp. También aprendí a diagnosticar un problema de configuración real (la
exclusión de `localhost` en el proxy de Firefox) buscando una causa en vez de asumir que
algo estaba mal configurado sin más. Y, sobre todo, vi en la práctica que la mayoría de
vulnerabilidades web no vienen de técnicas exóticas, sino de controles de acceso que
faltan o de código que confía en el usuario más de lo que debería.

## Siguiente paso
Profundizar en el control de acceso de Juice Shop (otros endpoints, roles de usuario),
y ver otras clases de inyección (XSS) y manipulación de JWT (Lección 09).
