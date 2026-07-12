# Burp Suite — Testing de aplicaciones web

Relacionado: [[HTTP-Web]] · [[Arquitectura-Web]] · [[Ofensiva-Pentesting]] · [[Seguridad-de-Red]]

## Qué es
Framework en Java, estándar de la industria para pentesting de aplicaciones **web y móviles** (incluyendo APIs). Captura y permite manipular todo el tráfico HTTP/HTTPS entre el navegador y el servidor: interceptar, ver y modificar **requests** antes de que lleguen al servidor y **responses** antes de que lleguen al navegador.

Ediciones:
- **Community** (gratis, no comercial) → la que usamos.
- **Professional** (escáner automático, fuzzer sin límite, Collaborator, guardar proyectos).
- **Enterprise** (escaneo continuo en servidor).

## Funciones principales (Community)
| Herramienta | Uso |
|-------------|-----|
| **Proxy** | interceptar/modificar requests y responses |
| **Repeater** | capturar, modificar y reenviar la misma request muchas veces |
| **Intruder** | rociar un endpoint con requests (fuerza bruta/fuzzing, limitado en Community) |
| **Decoder** | codificar/decodificar datos |
| **Comparer** | comparar dos piezas de datos (palabra o byte) |
| **Sequencer** | analizar aleatoriedad de tokens/cookies |
| **Extender / BApp Store** | extensiones (Java, Python-Jython, Ruby-JRuby); ej. Logger++ |

## El Dashboard (4 cuadrantes)
- **Tasks**: tareas en background (ej. "Live Passive Crawl").
- **Event log**: acciones de Burp (arranque del proxy, conexiones).
- **Issue Activity**: vulnerabilidades (solo Pro).
- **Advisory**: detalle de vulnerabilidades (solo Pro).

## Atajos de teclado
| Atajo | Pestaña |
|-------|---------|
| `Ctrl+Shift+D` | Dashboard |
| `Ctrl+Shift+T` | Target |
| `Ctrl+Shift+P` | Proxy |
| `Ctrl+Shift+I` | Intruder |
| `Ctrl+Shift+R` | Repeater |
| `Ctrl+U` | URL-encode de la selección |

## Configuración (Settings)
- **User settings** (globales): afectan toda la instalación.
- **Project settings**: solo la sesión actual (Community no guarda proyectos).
- Categorías útiles: *Sessions* (Cookie jar), *Suite* → *Updates*, *Hotkeys* (atajos), TLS por proyecto.

## El Proxy (lo más importante)
- **Intercept is on/off**: con intercept ON, cada request se retiene y aparece en el Proxy. Botones: Forward, Drop, editar, enviar a otras herramientas.
- Burp registra el tráfico igual aunque intercept esté OFF → pestaña **HTTP history** y **WebSockets history**.
- **Match and Replace**: regex para modificar requests/responses (ej. cambiar el User-Agent).

### Configurar el navegador (FoxyProxy)
1. Instalar FoxyProxy en Firefox.
2. Añadir proxy: Title `Burp`, IP `127.0.0.1`, Port `8080`.
3. Activar la config Burp desde el icono de FoxyProxy.
4. En Burp: Proxy → Intercept ON.
5. Navegar a `http://MACHINE_IP/` → la request queda capturada.

> Alternativa: **Burp Browser** (Chromium integrado, ya preconfigurado). Botón "Open Browser" en la pestaña Proxy. En Linux como root: Settings → Tools → Burp's browser → permitir sin sandbox.

## Target tab (3 sub-pestañas)
- **Site map**: árbol de las páginas visitadas (se llena solo navegando). Útil para mapear APIs.
- **Issue definitions**: lista de vulnerabilidades que busca el escáner (referencia para reportes).
- **Scope settings**: incluir/excluir dominios/IPs.

## Scoping (definir el alcance)
1. Target → clic derecho sobre el objetivo → **Add To Scope**.
2. Aceptar "detener logging de lo que no está en scope".
3. Proxy settings → "Intercept Client Requests" → activar **And URL Is in target scope**.

Así el proxy ignora todo lo que no esté en el alcance → vista más limpia.

## HTTPS a través del proxy (certificado CA)
1. Con el proxy activo, ir a `http://burp/cert` → descarga `cacert.der`.
2. Firefox: `about:preferences` → buscar "certificates" → View Certificates → Import.
3. Seleccionar `cacert.der`, marcar "Trust this CA to identify websites" → OK.

## Ejemplo de ataque: XSS reflejado
Payload: `<script>alert("Succ3ssful XSS")</script>`

Un filtro **client-side** (JavaScript, ver [[Ofensiva-Pentesting]]) bloquea caracteres especiales, pero es trivial de saltar con el proxy:
1. Proxy activo + intercept ON.
2. Enviar el formulario con datos legítimos.
3. Interceptar la request y reemplazar el campo por el payload.
4. Seleccionar el payload y hacer `Ctrl+U` (URL-encode).
5. **Forward** → el XSS se ejecuta.

Lección: **nunca confíes solo en validación client-side**; siempre valida en el servidor.
