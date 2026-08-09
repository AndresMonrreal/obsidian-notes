# Testing Manual de Aplicaciones Web (Solo Navegador) — Page Source y DevTools

Relacionado: [[Reconocimiento-Activo]] · [[HTTP-Peticiones-y-Respuestas]] · [[Herramientas-Burp-Suite]] · [[OWASP-Top-10-2025]] · [[JavaScript-Web]]

## Por qué probar manual antes que con herramientas automatizadas
Revisar la aplicación a mano —solo con el navegador, sin scanners— entrena el ojo para detectar lo que un scanner automatizado no necesariamente marca: comentarios de desarrolladores, enlaces ocultos, configuraciones erróneas, lógica client-side manipulable. Las herramientas son potentes, pero un ojo entrenado y curioso encuentra más antes de siquiera lanzar la primera automatización.

## 1. Mapear la aplicación a mano
Antes de tocar cualquier herramienta, navegar el sitio completo y documentar cada página/función encontrada en una tabla simple: **Feature | Endpoint | Resumen**. Prestar atención especial a cualquier parte que **requiera interacción del usuario** (forms, login, uploads, etc.) — ahí suele vivir la superficie de ataque real.

## 2. Ver el código fuente — `view-source:`
```
view-source:https://sitio.com
```
También con clic derecho → "Ver código fuente de la página" (`Ctrl+U` en la mayoría de navegadores). Es el HTML/CSS/JS **tal como el servidor lo mandó**, antes de que JavaScript lo modifique en el navegador.

Qué buscar:
- **Comentarios HTML** (`<!-- ... -->`): mensajes de desarrolladores, notas para ellos mismos — a veces explican partes internas de la app que nunca debieron ser públicas.
- **Enlaces ocultos** (etiquetas `<a href="...">`): rutas que no aparecen en la navegación normal del sitio pero sí están en el HTML — a veces llevan a paneles internos o áreas privadas.
- **Rutas de archivos estáticos** (CSS/JS/imágenes): revisar el directorio donde viven — si el servidor tiene **directory listing** habilitado (configuración incorrecta), se ve TODO el contenido de esa carpeta, no solo los archivos que la página referencia — a veces backups, código fuente u otros archivos que nunca debieron quedar públicos.
- **Comentarios sobre el framework usado (y su versión)**: con eso se puede buscar si esa versión específica tiene CVEs públicos conocidos — un framework desactualizado es una pista de alto valor.

## 3. DevTools — Inspector (vista viva del DOM)
El código fuente (`view-source:`) **no** refleja cambios hechos por CSS/JS/interacción del usuario después de cargar — para eso está el **Inspector** (`Elements` en Chrome), que muestra el DOM tal como está en este momento.

Uso ofensivo típico: **paywalls o bloqueos puramente client-side**. Si un elemento (ej. un `<div class="premium-blocker">`) tapa contenido usando CSS (`display: block`):
1. Clic derecho sobre el elemento bloqueado → **Inspeccionar**.
2. Localizarlo en el árbol del DOM.
3. En el panel de estilos, cambiar `display: block` a `display: none` (o borrar la regla).

El contenido bloqueado aparece — el bloqueo nunca estuvo protegido del lado servidor, solo escondido visualmente. **Cualquier cosa que dependa solo del cliente para ocultar/restringir contenido no es una protección real.** El cambio es solo local a ese navegador (desaparece al refrescar) — no modifica nada en el servidor.

## 4. DevTools — Debugger / Sources (JavaScript)
Pestaña para depurar JS — **Debugger** en Firefox/Safari, **Sources** en Chrome. Como pentester, sirve para leer la lógica real del cliente, no solo para depurar errores.

- **Minificado**: todo el JS en una sola línea (se quitan tabs/espacios/saltos para reducir el tamaño del archivo) — usar **Pretty Print** (ícono `{ }`) para reformatearlo y poder leerlo.
- **Ofuscado**: además de minificado, el código se reescribe a propósito para ser difícil de entender (nombres de variables sin sentido, lógica indirecta) — dificulta pero no impide la lectura.
- **Breakpoints**: clic en el número de línea pausa la ejecución de JS justo ahí, cada vez que el navegador llega a esa línea. Útil para **congelar el estado antes de que el JS "limpie" algo** — por ejemplo, si una línea tipo `elemento['remove']()` borra un elemento del DOM apenas carga la página, un breakpoint justo ahí detiene la ejecución antes de que se ejecute ese borrado, dejando el elemento visible para inspeccionarlo con calma.

## 5. DevTools — Network (tráfico real de la app)
Registra cada request que la página hace en segundo plano — clave para **AJAX** (JavaScript pidiendo/mandando datos al servidor sin recargar la página completa, ej. un formulario de contacto que se envía "sin refrescar").

Flujo típico:
1. Abrir la pestaña Network, limpiar la lista (ícono de bote de basura) si ya está saturada.
2. Interactuar con la app (llenar y enviar un formulario, por ejemplo).
3. Aparece una nueva entrada — clic en ella para ver **headers de request/response, cookies enviadas, y la respuesta HTML/JSON completa** del servidor, incluso si esa respuesta nunca se muestra visualmente en la página.

A veces la respuesta completa de un endpoint AJAX trae más información de la que la interfaz realmente despliega — vale la pena revisar la pestaña **Response** de cada request, no solo lo que se ve en pantalla.

## 6. DevTools — Storage (qué guarda el navegador)
Pestaña **Application → Storage** (Chrome) o **Storage** (Firefox). Cuatro tipos relevantes:

| Tipo | Qué guarda |
|---|---|
| **Local Storage** | datos persistentes — sobreviven aunque se cierre el navegador |
| **Session Storage** | datos temporales — solo mientras esa pestaña/sesión sigue abierta |
| **Cookies** | datos que el *servidor* mandó y pidió guardar — lo más relevante para sesión/autenticación |
| **Cache Storage** | recursos cacheados (imágenes, scripts, respuestas de API) para carga más rápida |

**Cookies — flags de seguridad a revisar siempre:**
- **`HttpOnly`**: si está activo, JavaScript **no puede leer esa cookie** (`document.cookie` no la muestra) — protección directa contra robo de cookie de sesión vía XSS. Si una cookie de sesión NO tiene este flag, un XSS exitoso puede robarla directamente.
- **`Secure`**: la cookie solo se manda por HTTPS — sin este flag, podría viajar en texto plano si alguna petición cae a HTTP.
- **`SameSite`**: controla si la cookie se manda en peticiones que vienen de **otro sitio** — mitiga CSRF (`Strict`/`Lax` limitan el envío cross-site; `None` no protege nada por sí solo, requiere ir acompañado de `Secure`).

Revisar estos tres flags en cualquier cookie de sesión es un chequeo rápido y de alto valor: su ausencia es una debilidad real y documentable en cualquier reporte de pentest.

## Referencia rápida
| Acción | Para qué |
|---|---|
| `view-source:URL` | ver el HTML/CSS/JS tal como lo manda el servidor |
| Inspector → editar CSS del elemento | saltarse bloqueos puramente visuales/client-side |
| Debugger/Sources → Pretty Print | reformatear JS minificado/ofuscado para leerlo |
| Debugger/Sources → breakpoint en una línea | congelar la ejecución de JS antes de que "limpie" algo |
| Network → inspeccionar una request AJAX | ver headers, cookies y respuesta completa de un envío en segundo plano |
| Storage → Cookies | revisar `HttpOnly` / `Secure` / `SameSite` de cada cookie de sesión |

## Mentalidad detrás de esto
Ninguna de estas técnicas "hackea" nada — son observación cuidadosa de lo que la aplicación ya está exponiendo. Antes de correr cualquier scanner automatizado, recorrer la app a mano (comentarios, enlaces ocultos, versión del framework, comportamiento real del JS/cookies) enseña más sobre cómo pensar como atacante que cualquier herramienta por sí sola.
