# Burp Suite — Testing de aplicaciones web

Relacionado: [[HTTP-Web]] · [[Arquitectura-Web]] · [[Ofensiva-Pentesting]] · [[Seguridad-de-Red]]

## Qué es
Framework en Java, estándar de la industria para pentesting de aplicaciones **web y móviles** (incluyendo APIs). Captura y permite manipular todo el tráfico HTTP/HTTPS entre el navegador y el servidor: interceptar, ver y modificar **requests** antes de que lleguen al servidor y **responses** antes de que lleguen al navegador.

Ediciones:
- **Community** (gratis, no comercial) → la que usamos.
- **Professional** (escáner automático, fuzzer sin límite, Collaborator, guardar proyectos).
- **Enterprise** (escaneo continuo en servidor).

## Primer arranque (wizard inicial, Community)
Al abrir Burp, dos pantallas de configuración antes de llegar a la interfaz:
1. **New Project**: dejar seleccionado **"Temporary project in memory"** → *Next* (Community no soporta guardar proyectos en disco — esa opción solo existe en Pro).
2. **Select configuration**: dejar **"Use Burp defaults"** → *Start Burp*.

Ya dentro, en la pestaña **Proxy** → botón **"Open Browser"** abre el navegador Chromium integrado (Burp Browser), que ya trae el proxy y el certificado de Burp preconfigurados — más rápido que configurar FoxyProxy a mano (ver más abajo la alternativa manual).

Dejar **Intercept is off** para navegar normal sin que cada request se detenga a pedir Forward — todo el tráfico igual queda registrado en **Proxy > HTTP History** para revisarlo después.

## Funciones principales (Community)
| Herramienta | Uso |
|-------------|-----|
| **Proxy** | interceptar/modificar requests y responses |
| **Repeater** | capturar, modificar y reenviar la misma request muchas veces — ver [[Herramientas-Burp-Suite-Repeater]] |
| **Intruder** | rociar un endpoint con requests (fuerza bruta/fuzzing, limitado en Community) |
| **Decoder** | codificar/decodificar datos, generar hashes — ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] |
| **Comparer** | comparar dos piezas de datos (palabra o byte) — ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] |
| **Sequencer** | analizar aleatoriedad de tokens/cookies — ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] |
| **Organizer** | guardar y anotar requests de solo lectura para después — ver [[Herramientas-Burp-Suite-Decoder-Comparer-Sequencer]] |
| **Extender / BApp Store** | extensiones (Java, Python-Jython, Ruby-JRuby); ej. Logger++ |

## Extensions — administrar extensiones instaladas
- **Extensions List**: lista las extensiones cargadas en el proyecto actual — se pueden activar/desactivar individualmente.
- **Add**: instala una extensión nueva desde un archivo en disco (módulo propio o de fuera del BApp Store oficial).
- **Remove**: desinstala la extensión seleccionada.
- **Up / Down**: reordena las extensiones instaladas — el orden importa porque **determina la secuencia en la que se invocan al procesar tráfico**. Se aplican en orden **descendente**, empezando desde arriba de la lista — relevante cuando varias extensiones modifican requests y pueden llegar a interferir entre sí.
- **Details / Output / Errors**: para la extensión seleccionada — *Details* (nombre, versión, descripción), *Output* (lo que la extensión imprime al correr), *Errors* (fallos durante su ejecución — útil para debug).

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
