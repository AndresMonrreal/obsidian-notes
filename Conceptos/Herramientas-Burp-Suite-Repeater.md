# Burp Suite — Repeater (a fondo)

Relacionado: [[Herramientas-Burp-Suite]] · [[Bases-de-Datos-SQL]] · [[HTTP-Peticiones-y-Respuestas]] · [[Fingerprinting-Stacks-Web-CVEs]]

## Qué es y para qué sirve
Repeater deja modificar y reenviar una request capturada tantas veces como haga falta, con interfaz gráfica — ideal para pruebas manuales repetitivas con cambios pequeños: SQLi manual, bypass de filtros de WAF, ajustar parámetros de un formulario. También se puede armar una request desde cero, como usando curl a mano.

## Interfaz — 6 secciones
| Sección | Ubicación | Qué hace |
|---|---|---|
| **Request List** | arriba a la izquierda | lista de todas las requests mandadas a Repeater; cada nueva aparece aquí |
| **Request Controls** | debajo de la lista | botones Send, Cancel, e historial de navegación (adelante/atrás) |
| **Request/Response View** | centro, la mayoría del espacio | editar la request a la izquierda, ver la respuesta a la derecha tras mandarla |
| **Layout Options** | arriba a la derecha del Request/Response | horizontal (default), vertical, o pestañas separadas |
| **Inspector** | lado derecho | desglose visual/editable de la request y respuesta, más intuitivo que el editor crudo |
| **Target** | arriba del Inspector | IP/dominio destino — se rellena solo cuando la request llega desde otro módulo de Burp |

## Enviar una request a Repeater
Con **Intercept ON** en Proxy, capturar la request → clic derecho → **Send to Repeater** (o `Ctrl+R`). Al llegar, Target e Inspector ya muestran info, pero no hay respuesta todavía — el botón **Send** dispara la petición y llena la vista de Response.

Editar cualquier parte de la request (headers, método, body) directo en el editor de texto y volver a mandar actualiza la Response — los botones de historial junto a Send permiten moverse entre versiones anteriores de la misma request.

## Vistas de la respuesta (4 botones sobre el response box)
- **Pretty** (default): la respuesta cruda con formato mejorado para legibilidad.
- **Raw**: la respuesta exactamente como la mandó el servidor, sin tocar.
- **Hex**: representación byte a byte — útil con archivos binarios.
- **Render**: la página como se vería en un navegador real — poco usado en Repeater (el foco suele ser el código fuente), pero disponible.

Botón adicional **Show non-printable characters (`\n`)**: muestra caracteres normalmente invisibles — cada línea de una respuesta HTTP termina en `\r\n` (retorno de carro + salto de línea), relevante para interpretar headers correctamente.

## Inspector — desglose de la request/response
Disponible tanto en Proxy como en Repeater, en el lado derecho:
- **Request Attributes**: método, ruta, protocolo (HTTP/1 vs HTTP/2) — editable.
- **Request Query Parameters**: datos mandados por la URL (`?redirect=false`).
- **Request Body Parameters**: igual que los query params, pero específico de peticiones POST.
- **Request Cookies**: lista editable de cookies mandadas.
- **Request Headers**: ver/editar/agregar/quitar cualquier header — útil para probar cómo reacciona el servidor ante headers inesperados.
- **Response Headers**: solo lectura (no se puede controlar lo que manda el servidor) — visible solo después de mandar la request y recibir respuesta.

## Ejercicio 1 — agregar un header personalizado
Capturar `GET /` en Proxy → Send to Repeater → Send (para ver el HTML normal) → con Inspector (o editando a mano), agregar un header nuevo:
```
FlagAuthorised: True
```
Volver a mandar y revisar si el servidor reacciona distinto ante ese header (a veces un endpoint de prueba responde algo especial si detecta un header "mágico" como este).

**Atajo por curl**, si la GUI va lenta — manda exactamente la misma request sin tocar Burp:
```bash
curl -H "FlagAuthorised: True" http://IP/
```

## Ejercicio 2 — fuzzing manual de un endpoint numérico
Muchas apps tienen rutas tipo `/products/N` donde `N` es un ID entero. El objetivo: mandar valores "extremos" en vez del entero normal, buscando provocar un **500 Internal Server Error** (falla no manejada) en vez de un 404 controlado — un 500 revela que el input no se valida/sanitiza antes de usarse internamente (ej. en una query a BD o una búsqueda en array).

Flujo con Repeater: capturar una request válida (`/products/3`) → Send to Repeater → cambiar el número al final por cada valor y mandar:
- Negativo: `/products/-1`
- Cero: `/products/0`
- Decimal: `/products/1.5`
- Texto: `/products/abc`
- Número gigantesco: `/products/99999999999999999999`
- Vacío: `/products/`

**Atajo por curl**, probando varios de un jalón:
```bash
for val in -1 0 1.5 abc 99999999999999999999 "' OR '1'='1"; do
  echo "=== /products/$val ==="
  curl -s -o /dev/null -w "%{http_code}\n" "http://IP/products/$val"
done
```
`for val in ...; do ... done` prueba cada valor de la lista uno por uno · `-s -o /dev/null` descarta el cuerpo de la respuesta · `-w "%{http_code}\n"` imprime solo el código de estado — así se ve rápido cuál valor da 500 sin leer HTML completo cada vez. Una vez identificado el valor que rompe el endpoint, pedirlo completo (sin `-o /dev/null`) para ver el cuerpo de la respuesta — ahí suele venir el detalle del error (o una flag, en un lab).

## Ejercicio 3 — Union SQL Injection manual, paso a paso
Objetivo: explotar una inyección SQL de tipo UNION en el parámetro `ID` de `/about/ID` para extraer datos de una tabla (ej. notas internas del CEO).

### Paso 1 — Confirmar la vulnerabilidad
```
GET /about/2' HTTP/1.1
```
Agregar una comilla simple (`'`) después del ID suele bastar para romper una query SQL mal construida. Un `500 Internal Server Error` como respuesta confirma que la query se rompió.

### Paso 2 — Leer el error verboso (si el servidor lo expone)
Un mensaje de error mal manejado puede filtrar la query real completa, por ejemplo:
```
Invalid statement: SELECT firstName, lastName, pfpLink, role, bio FROM people WHERE id = 2'
```
Esto de un solo golpe revela: el nombre de la tabla (`people`) y las 5 columnas seleccionadas (`firstName, lastName, pfpLink, role, bio`) — ahorra los pasos de enumeración manual de tabla/columnas que normalmente haría falta.

### Paso 3 — Enumerar los nombres reales de columna
```
/about/0 UNION ALL SELECT column_name,null,null,null,null FROM information_schema.columns WHERE table_name="people"
```
`UNION ALL SELECT` une el resultado de una segunda query completamente distinta a la original — ambas deben devolver el mismo número de columnas, de ahí los `null` de relleno · el ID se cambia de `2` a `0` (uno que no existe) para que la query legítima original no devuelva ningún registro real — así la única fila que llega a la página es la de la query inyectada · `information_schema.columns` es la tabla estándar de metadatos de MySQL que lista las columnas de cualquier tabla del sistema, filtrando aquí por `table_name="people"`.

El primer resultado (columna `id`) aparece insertado donde normalmente iría `firstName` (la primera de las 5 columnas originales) — confirmando que la inyección funciona y dónde se renderiza en la página.

### Paso 4 — Ver todas las columnas de un jalón con `group_concat()`
```
/about/0 UNION ALL SELECT group_concat(column_name),null,null,null,null FROM information_schema.columns WHERE table_name="people"
```
`group_concat()` junta todos los valores de una columna en un solo string (separados por coma por default) — en vez de ver una fila a la vez, aparecen todas las columnas juntas: `id, firstName, lastName, pfpLink, role, shortRole, bio, notes`.

### Paso 5 — Extraer el dato objetivo
Con el nombre de tabla (`people`), columna objetivo (`notes`), y el ID del registro (`1`, confirmado revisando el perfil en `/about/`):
```
/about/0 UNION ALL SELECT notes,null,null,null,null FROM people WHERE id = 1
```
`notes,null,null,null,null` coloca el contenido de esa columna en la posición donde normalmente iría `firstName` — por eso aparece en el lugar del nombre/título en la página renderizada.

**Vía curl** (URL-encodeando los espacios como `%20`):
```bash
curl "http://IP/about/0%20UNION%20ALL%20SELECT%20notes,null,null,null,null%20FROM%20people%20WHERE%20id%20=%201"
```

## Referencia rápida
| Acción | Cómo |
|---|---|
| Enviar una request capturada a Repeater | clic derecho → *Send to Repeater*, o `Ctrl+R` |
| Cambiar vista de la respuesta | botones Pretty / Raw / Hex / Render sobre el response box |
| Ver caracteres invisibles (`\r\n`) | botón `\n` junto a las vistas |
| Editar headers/cookies/params visualmente | panel **Inspector**, lado derecho |
| Confirmar SQLi simple | agregar `'` al final de un parámetro numérico y mandar |
| Enumerar columnas por SQLi | `UNION ALL SELECT column_name,null,... FROM information_schema.columns WHERE table_name="X"` |
| Ver todas las columnas de un jalón | envolver en `group_concat(column_name)` |
