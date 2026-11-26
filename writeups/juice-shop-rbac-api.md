# Control de acceso roto (RBAC) en la API de OWASP Juice Shop

**Entorno:** laboratorio propio, contenedor Docker de OWASP Juice Shop (`bkimminich/juice-shop`)
**Objetivo:** `http://10.0.2.15:3000`
**Categoría:** Broken Access Control (escalada vertical de privilegios)
**Fecha:** 26/11/2026

## Objetivo
Comprobar si un usuario autenticado sin privilegios de administrador puede acceder
a funciones y datos reservados para el rol admin, tanto en lectura como en
escritura, llamando directamente a la API en vez de usar la interfaz visible.

## Hallazgo 1: la ruta de administración no está protegida, solo oculta
**Ruta:** `http://10.0.2.15:3000/#/administration`
**Resultado:** accesible directamente con sesión de admin, sin estar enlazada en
ningún menú de la aplicación. La propia app confirmó: *"Ha resuelto correctamente
el desafío: Admin Section (Access the administration section of the store)"*.
**Análisis:** ocultar un enlace en la interfaz no es un control de acceso; solo
retrasa el descubrimiento. Al no existir como ruta real de servidor (es una ruta
de cliente de Angular), herramientas de fuerza bruta de directorios como
`gobuster` no la habrían encontrado — la vía real de descubrimiento es inspeccionar
el JavaScript de la aplicación.

## Hallazgo 2: `/api/Users/` devuelve el listado completo con un token de usuario normal

**Procedimiento:**
1. Login como admin por SQLi (`' OR 1=1--`) → token de admin obtenido vía `curl -X POST /rest/user/login`.
2. `curl http://10.0.2.15:3000/api/Users/ -H "Authorization: Bearer <token admin>"` → devuelve el listado completo de usuarios (control positivo, confirma que el endpoint funciona).
3. Registro de un usuario normal nuevo desde el navegador (cuenta sin ningún privilegio).
4. Login de ese usuario normal, con sus credenciales reales, por el mismo endpoint de login.
5. `curl http://10.0.2.15:3000/api/Users/ -H "Authorization: Bearer <token usuario normal>"`.

**Resultado:** el paso 5 devolvió **el mismo listado completo de usuarios** que con
el token de administrador.

**Análisis:** el servidor valida que el token sea **válido**, pero no comprueba el
**rol** del usuario antes de servir un recurso administrativo. Cualquier cuenta,
incluso creada en el momento, tiene el mismo acceso de lectura que un admin sobre
este endpoint.

## Hallazgo 3: el mismo usuario normal puede borrar datos de otros usuarios

**Procedimiento:**
```
curl -s -X DELETE http://10.0.2.15:3000/api/Feedbacks/3 -H "Authorization: Bearer <token usuario normal>"
```
**Resultado:** la reseña con `id: 3` (de otro usuario, visible previamente en el
panel de administración) fue eliminada con éxito. Confirmado recuperando
`GET /api/Feedbacks/` y comprobando que ese `id` ya no aparece.

**Análisis:** esto va un paso más allá que un simple fallo de lectura: un usuario
cualquiera, sin ningún privilegio, puede **modificar o destruir datos que
pertenecen a otros usuarios o a la aplicación**, usando el mismo patrón de API
genérica (`/api/<Modelo>/<id>`) sin ninguna comprobación de autorización.

## Análisis general
Los tres hallazgos comparten la misma causa raíz: la aplicación distingue roles
**solo en la interfaz** (qué botones y rutas se muestran), pero no repite esa
comprobación en el servidor, donde realmente importa. Esto convierte cualquier
llamada directa a la API, sin pasar por la interfaz, en una vía de escalada de
privilegios completa — tanto para leer como para destruir información.

## Incidencia real de laboratorio
Al intentar complementar esta auditoría con un escaneo automático de OWASP ZAP
(Active Scan contra Juice Shop), la máquina virtual se quedó sin recursos
(ZAP en Java + Docker + navegador simultáneos) y se reinició sola a mitad del
escaneo. Se confirmó la causa con `dmesg | grep -i -E "out of memory|oom"`. Queda
pendiente repetir el escaneo automatizado con más memoria asignada a la VM.

## Cómo se previene
- Comprobar el **rol del usuario en el servidor**, en cada endpoint sensible, no
  solo ocultar la opción en el frontend.
- Aplicar el principio de **mínimo privilegio** por defecto: denegar salvo que se
  demuestre explícitamente que el rol tiene permiso.
- Revisar especialmente los endpoints de **escritura/borrado** (POST, PUT,
  DELETE), ya que su impacto si fallan es mucho mayor que uno de solo lectura.
- No asumir que una ruta de cliente "no listada" equivale a protegida; auditar
  también a nivel de API, directamente, como se ha hecho aquí.

## Qué aprendí
Que el control de acceso roto no necesita ninguna técnica sofisticada para
explotarse: basta con una cuenta recién creada y conocer la URL del endpoint.
Aprendí a diferenciar una ruta de cliente (SPA) de una ruta real de servidor, y
por qué eso cambia completamente qué herramienta sirve para encontrarla. También
aprendí a diagnosticar un fallo real de la máquina virtual (quedarse sin memoria)
con evidencia del propio kernel en vez de suponer la causa, y a hacer todo el
proceso de autenticación y prueba de una API completamente desde `curl`, sin
depender de las limitaciones de la versión gratuita de Burp.

## Siguiente paso
Repetir el escaneo automatizado con OWASP ZAP con más recursos en la VM, y
empezar a introducir Active Directory como próximo entorno de práctica.
