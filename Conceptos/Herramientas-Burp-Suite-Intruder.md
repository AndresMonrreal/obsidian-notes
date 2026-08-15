# Burp Suite — Intruder (a fondo)

Relacionado: [[Herramientas-Burp-Suite]] · [[Herramientas-Burp-Suite-Repeater]] · [[Herramientas-Hydra]] · [[OWASP-Top-10-2025]]

## Qué es
Herramienta de fuzzing/fuerza bruta integrada en Burp: toma una request capturada y manda muchas variaciones automatizadas, sustituyendo valores en posiciones marcadas — comparable en función a Wfuzz o ffuf por línea de comandos. Sirve para fuerza bruta de logins (usuario/contraseña desde un wordlist) o fuzzing de rutas/endpoints/vhosts.

> ⚠️ En **Community**, Intruder está limitado en velocidad (rate-limited) — mucho más lento que en Professional. Sigue siendo útil para aprender el concepto, pero en ataques grandes conviene considerar alternativas de CLI (ver el caso práctico en [[Web-Pentest-Credential-Stuffing-IDOR-Ejemplo]]).

## Interfaz — 4 sub-pestañas
El campo **Target** se autocompleta si la request llegó desde Proxy (`Ctrl+I` o clic derecho → *Send to Intruder*).

| Pestaña | Qué hace |
|---|---|
| **Positions** | elegir el tipo de ataque y marcar dónde van los payloads en la request |
| **Payloads** | definir qué valores insertar en cada posición marcada |
| **Resource Pool** | asignación de recursos entre tareas automatizadas — solo relevante en Pro |
| **Settings** | comportamiento del ataque (ej. marcar requests que contengan cierto texto, manejo de redirects 3xx) |

## Positions — marcar dónde van los payloads
Burp intenta detectar solas las posiciones más probables, resaltadas en verde entre signos de sección (`§`). Botones:
- **Add §**: marca una posición manualmente (seleccionar texto en el editor y hacer clic).
- **Clear §**: borra todas las posiciones marcadas — lienzo en blanco para definir las propias.
- **Auto §**: vuelve a autodetectar las posiciones más probables (útil tras un `Clear §`).

## Payloads — 4 secciones
- **Payload Sets**: elegir para qué posición se configura un set y qué tipo de payload usar. Con ataques de un solo set (Sniper, Battering Ram) solo hay una opción en el dropdown sin importar cuántas posiciones haya marcadas. Con ataques multi-set (Pitchfork, Cluster Bomb) hay un ítem por posición — el orden en el dropdown sigue el orden de aparición en la request, de arriba hacia abajo / izquierda a derecha.
- **Payload Settings**: opciones específicas del tipo elegido — con "Simple list" se puede agregar a mano, pegar líneas, o cargar un archivo (`Load`). ⚠️ cuidado con wordlists enormes, pueden colgar Burp.
- **Payload Processing**: reglas aplicadas a cada payload antes de mandarlo — ej. **Add prefix** (agrega texto al inicio) / **Add suffix** (agrega texto al final), capitalizar, saltar payloads que matcheen un regex.
- **Payload Encoding**: por default Burp aplica URL-encoding a los payloads para que viajen seguros — se puede ajustar qué caracteres codificar o desactivarlo del todo.

## Los 4 tipos de ataque
Usando de ejemplo `username=§pentester§&password=§Expl01ted§` (2 posiciones):

### Sniper (default)
Un solo payload set, recorre **una posición a la vez** — agota el wordlist completo en la primera posición (dejando la otra en su valor original), luego pasa a la siguiente.
```
requests = numberOfWords × numberOfPositions
```
Con wordlist `[burp, suite, intruder]` y 2 posiciones → 6 requests (3 en `username`, luego 3 en `password`). Ideal para fuzzear **una sola posición** (fuerza bruta de un parámetro, fuzzing de un endpoint tipo `/products/§N§`).

### Battering ram
Un solo payload set, pero mete **el mismo payload en todas las posiciones a la vez**, en cada request.
```
requests = numberOfWords
```
Con el mismo wordlist → 3 requests (`username=burp&password=burp`, `username=suite&password=suite`, etc.). Útil para probar el mismo valor repetido en varios campos simultáneamente (ej. race conditions, o "¿el mismo valor funciona como user y password a la vez?").

### Pitchfork
Un payload set **por posición** (hasta 20), iterados **en paralelo** — toma el ítem N de cada lista y los sustituye juntos en la misma request.
```
requests = length del payload set MÁS CORTO
```
Con `usernames: [joel, harriet, alex]` y `passwords: [J03l, Emma1815, Sk1ll]` → 3 requests, emparejando línea a línea (`joel:J03l`, `harriet:Emma1815`, `alex:Sk1ll`). **Se detiene en cuanto la lista más corta se agota** — si las listas tienen largos distintos, el resto de la lista más larga nunca se prueba. Ideal para **credential stuffing** (pares usuario:contraseña ya emparejados de una filtración).

### Cluster bomb
Un payload set por posición (hasta 20), pero prueba **cada combinación posible** entre todos los sets.
```
requests = length(set1) × length(set2) × ... × length(setN)
```
Con 3 usuarios × 3 contraseñas → 9 requests (todas las combinaciones). Genera mucho tráfico — útil cuando **no se sabe qué contraseña corresponde a qué usuario** (fuerza bruta real de credenciales, no credential stuffing).

## Burp Macros — manejar tokens/cookies que cambian en cada request
Cuando un login trae un **CSRF token** y/o **cookie de sesión que cambia con cada carga de la página**, un ataque de Intruder normal falla porque cada request necesita un token fresco distinto — el "recursive grep" simple no sirve si la respuesta es un redirect. La solución: una **macro** que hace una petición previa (ej. `GET /login/`) antes de cada intento, y extrae los valores frescos.

**Configuración (Settings → Sessions):**
1. **Macros** → *Add* → elegir en el historial la request `GET` que devuelve un token/cookie fresco (ej. `/admin/login/`) → nombrarla.
2. **Session Handling Rules** → *Add* → en **Scope**: limitar a la herramienta **Intruder** solamente, y limitar la **URL Scope** al sitio objetivo (o "Use suite scope" si ya se definió un scope global).
3. En **Details** → **Rule Actions** → *Add* → **Run a Macro** → seleccionar la macro creada.
4. Restringir qué actualiza la macro: **"Update only the following parameters and headers"** (ej. agregar `loginToken`) y **"Update only the following cookies"** (ej. agregar `session`) — sin esto, la macro sobrescribiría *todos* los parámetros de cada request, no solo el token/cookie.

Con esto, cada request del ataque va precedida por la macro, que refresca automáticamente el token y la cookie antes de mandar el intento real — el resto del flujo (Pitchfork con wordlists de usuario/contraseña) funciona igual que un credential stuffing normal.

## Referencia rápida
| Elemento | Qué hace |
|---|---|
| `§...§` | marca los límites de una posición de payload |
| `Add §` / `Clear §` / `Auto §` | marcar manual / borrar todo / autodetectar posiciones |
| Sniper | 1 set, 1 posición a la vez — `words × positions` requests |
| Battering ram | 1 set, todas las posiciones a la vez — `words` requests |
| Pitchfork | 1 set por posición, en paralelo — `min(len(sets))` requests |
| Cluster bomb | 1 set por posición, todas las combinaciones — `∏ len(sets)` requests |
| Add prefix / Add suffix | reglas de Payload Processing para agregar texto antes/después de cada payload |
| Macro + Session Handling Rule | refresca tokens/cookies que cambian antes de cada request del ataque |

Caso práctico completo (credential stuffing + IDOR, replicado con curl cuando la GUI iba lenta): [[Web-Pentest-Credential-Stuffing-IDOR-Ejemplo]].
