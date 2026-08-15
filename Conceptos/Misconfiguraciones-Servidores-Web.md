# Misconfiguraciones de Servidores Web (Apache, Nginx, Node/Express, Python HTTP Server)

Relacionado: [[Fingerprinting-Stacks-Web-CVEs]] · [[Content-Discovery-Manual-OSINT-y-Gobuster]] · [[HTTP-Peticiones-y-Respuestas]] · [[Reconocimiento-Manual-Web-y-DevTools]] · [[Linux-Herramientas-y-Admin]]

## Idea central
El servidor web en sí (no la aplicación que corre encima) moldea qué misconfiguraciones son posibles, qué rutas vale la pena revisar, y qué herramientas conviene usar. Identificar el software correcto es el primer paso, no un formalismo.

## 1. Identificar el servidor — 3 fuentes de fingerprint

### Header `Server`
```bash
curl -sI http://IP:PUERTO
```
`-s` silencioso · `-I` HEAD request, solo cabeceras (sin bajar el body).

| Software | Header `Server` típico |
|---|---|
| Apache2 | `Apache/2.4.x (Ubuntu)` |
| Python `http.server` | `SimpleHTTP/0.6 Python/3.xx.x` |
| Node.js Express | (ninguno — ni Express ni Node lo mandan por default) |
| Nginx | `nginx/1.xx.x (Ubuntu)` |

### Header `X-Powered-By`
Cuando `Server` viene vacío o genérico (típico de Express), revisar este header — Express lo manda automático salvo que el dev lo desactive: `X-Powered-By: Express`.

### Páginas de error 404 por default
```bash
curl -s http://IP:PUERTO/nonexistent-page-xyz
```
`-s` sin `-I` (se necesita el body completo, no solo headers). Cada software tiene una "huella visual" distinta incluso con headers suprimidos:
- **Python**: texto plano simple.
- **Nginx**: incluye la versión en el pie de página HTML.
- **Apache**: incluye su nombre en el cuerpo de la página.

## 2. Python `http.server` — sin ningún control de acceso
```bash
python3 -m http.server 8000   # referencia: así se levanta (comando del desarrollador, no algo que atacarías)
```
Sirve **todo** el directorio de trabajo tal cual — sin autenticación, sin `.htaccess` equivalente, sin lista negra, **incluyendo dotfiles como `.env`** (a diferencia de Apache/Nginx, donde el listado de directorio está desactivado por default y se puede restringir por ruta).

```bash
curl -s http://IP:8000/                # listado de directorio si no hay index.html
curl -s http://IP:8000/.env            # dotfiles se sirven igual que cualquier archivo
curl -s http://IP:8000/backup.zip -o backup.zip && unzip backup.zip -d backup-contents/
```
Buscar especialmente `.env` (credenciales/config) y archivos de respaldo (`.zip`, `.tar.gz`) — a veces contienen código fuente, dumps de base de datos, o configuración que nunca debió quedar pública.

> No hay vulnerabilidad que explotar aquí — el servidor funciona exactamente como fue diseñado. El hallazgo es que está corriendo en un lugar donde no debería, sirviendo archivos que no deberían ser públicos. Si aparece en un engagement, **asumir que todo el directorio de trabajo es legible**.

## 3. Apache — version disclosure, directory listing, mod_status, backups
```bash
curl -sI http://IP:80 | grep -i server
```
Ubuntu por default usa `ServerTokens OS` — incluye SO + versión exacta en el header.

### Directory Listing (`Options +Indexes`)
```bash
curl -s http://IP/files/
```
Si una ruta no tiene `index.html`, Apache lista el contenido (nombre, tamaño, fecha) — leer cada archivo que aparezca; developers dejan ahí CSVs, notas internas o backups.

### `mod_status` / `/server-status`
```bash
curl -s http://IP/server-status
```
Viene habilitado por default en Ubuntu con restricción `Require local` (solo localhost) — pero una directiva `Require all granted` en cualquier virtual host puede **sobrescribir silenciosamente** esa restricción, exponiéndolo a cualquier IP sin tocar la config del módulo. **Revisar `/server-status` siempre, incluso en servidores que parecen usar configuración default.** Expone conexiones activas, rutas solicitadas, total de peticiones desde el arranque, y la versión exacta del servidor.

### Archivos sin enlazar — Gobuster
```bash
gobuster dir -u http://IP:80 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -x bak,txt -t 20
```
`-x bak,txt` agrega esas extensiones a cada palabra del wordlist (busca `archivo.bak`, `archivo.txt`, etc., no solo `archivo`) — así aparecen backups y configs viejas que nunca quedaron enlazadas desde ninguna página. Encontrar un `.bak` casi siempre vale la pena revisarlo. `.htpasswd` accesible da un hash de contraseña crackeable offline y confirma que alguna ruta usa HTTP Basic Auth.

```bash
curl -s http://IP:80/backup.bak
```

**Flujo completo Apache**: header de versión → navegar cualquier directorio listable → `/server-status` → Gobuster con extensiones para archivos sin enlazar.

## 4. Node.js / Express — modo desarrollo filtrando de más
A diferencia de Apache/Nginx (sirven archivos estáticos de una ruta configurada), Express corre **código de aplicación** — cada request lo decide el código, no una configuración de servidor.

### Fingerprint y versión de la app
```bash
curl -sI http://IP:3000          # X-Powered-By: Express
curl -s http://IP:3000           # a veces la raíz devuelve JSON de estado con "version"
```

### Errores verbosos (stack traces)
El manejador de errores default de Express suprime stack traces si `NODE_ENV=production` — **pero un manejador de errores personalizado puede filtrarlos sin importar `NODE_ENV`** (patrón muy común: código de debug que nunca se endureció antes de salir a producción).
```bash
curl -s http://IP:3000/api/users | python3 -m json.tool
```
Un `500` desde un endpoint de API vale la pena investigar — el stack trace revela rutas internas del servidor, nombres de módulos, y a veces la query de base de datos que falló.

### Endpoints de debug — enumerar rutas y variables de entorno
```bash
curl -s http://IP:3000/api/routes        # lista cada ruta registrada (si el dev dejó este endpoint)
curl -s http://IP:3000/api/debug/env     # variables de entorno completas — credenciales, flags
```
`/api/routes` suele funcionar leyendo la propiedad interna `app._router.stack` de Express — un patrón de misconfiguración reconocido, aunque esa estructura interna cambia entre versiones de Express (Express 5 rompió implementaciones pensadas para Express 4). `NODE_ENV=development` en un servidor de producción es en sí misma una señal de despliegue sin endurecer.

### Archivos estáticos (`express.static()`)
```bash
curl -s http://IP:3000/static/config.js
```
JS de cliente a veces trae URLs de API internas, hostnames internos, o flags de debug como constantes — "pensado para ser público" no es lo mismo que "solo contiene información pública".

> `express.static()` responde `404` para dotfiles (`.env`, etc.) por default — **al revés** que el servidor HTTP de Python, que los sirve igual que cualquier archivo. Un `404` en una ruta estática de Express para `.env` no confirma que el archivo no exista, solo que el middleware lo está bloqueando por esa vía — haría falta shell o una ruta distinta para alcanzarlo.

**Flujo completo Node/Express**: headers confirman el framework → provocar un error para ver qué se filtra → endpoint de debug para enumerar rutas → revisar variables de entorno → seguir la lista de rutas hasta archivos estáticos.

## 5. Nginx — mismo patrón, vocabulario propio
```bash
curl -sI http://IP:8080 | grep -i server
```
Equivalente a `ServerTokens` de Apache: la directiva **`server_tokens`** (default `on`) controla el header `Server` y la versión en páginas de error simultáneamente — `server_tokens off` las suprime de ambos lados a la vez.

```bash
curl -s http://IP:8080/nonexistent-path   # si el Server header está suprimido, el 404 confirma si server_tokens realmente está off
```

### `autoindex` (equivalente a `Options +Indexes` de Apache)
```nginx
location /files/ {
    autoindex on;
    root /var/www/nginx/;
}
```
Directiva legítima y documentada (útil para file-sharing interno real) — el problema es usarla en una ruta con archivos sensibles o sin control de acceso.
```bash
curl -s http://IP:8080/files/
```

### `stub_status` / `/nginx_status` (equivalente a `mod_status` de Apache)
```nginx
location /nginx_status {
    stub_status;
    allow all;  # debería ser: allow 127.0.0.1; deny all;
}
```
```bash
curl -s http://IP:8080/nginx_status
# Active connections: 1
# server accepts handled requests
# 15 15 15
# Reading: 0 Writing: 1 Waiting: 0
```
Los tres números de la segunda línea, en orden: conexiones aceptadas totales, conexiones manejadas totales, peticiones totales desde el arranque. No es explotable directo, pero confirma monitoreo interno expuesto y sugiere que puede haber otros endpoints igual de mal protegidos.

> Con acceso a shell en un host Nginx, la configuración vive en `/etc/nginx/sites-available/` — leerla directo muestra exactamente qué directorios están expuestos y qué módulos están habilitados.

## 6. Auditoría universal: headers de seguridad
Ningún servidor manda estos headers por default — requieren configuración activa en cualquiera de los 4 tipos:

| Header | Protege contra | Valor típico |
|---|---|---|
| `X-Frame-Options` | Clickjacking (embeber la página en un iframe de otro dominio) | `DENY` / `SAMEORIGIN` |
| `X-Content-Type-Options` | MIME sniffing (el navegador "adivinando" el tipo de contenido) | `nosniff` |
| `Content-Security-Policy` | Restringe de dónde pueden cargar scripts/estilos/recursos | `default-src 'self'` |
| `Referrer-Policy` | Qué se manda en el header `Referer` al navegar a otro sitio | `no-referrer` / `strict-origin` |
| `Strict-Transport-Security` | Fuerza HTTPS en peticiones futuras (solo aplica sobre HTTPS) | `max-age=31536000` |

> `X-Frame-Options` está técnicamente reemplazado por `Content-Security-Policy: frame-ancestors`, que da control más fino — un sitio moderno bien endurecido puede omitir `X-Frame-Options` a propósito si ya cubre esto vía CSP. Al reportar un hallazgo de clickjacking, revisar ambos antes de marcarlo como ausente.

```bash
for port in 80 8000 3000 8080; do
  echo "=== Port $port ===";
  curl -sI http://IP:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|strict-transport|referrer-policy" \
    || echo "(no security headers found)";
done
```
El bucle repite el mismo chequeo contra varios puertos sin escribir el comando N veces · `grep -iE "a|b|c"` busca cualquiera de los headers (`-i` ignora mayúsculas, `-E` regex extendida para el `|`) · `|| echo "..."` asegura una salida explícita incluso cuando `grep` no encuentra nada (en vez de quedarse en silencio).

## 7. Nikto — automatizar todo lo anterior
```bash
nikto -h http://IP:80 -nointeractive
```
`-nointeractive` evita que Nikto se detenga esperando confirmación en ciertos pasos. Detecta en segundos varios de los mismos hallazgos vistos a mano: `/server-status` expuesto, archivos `.bak`, directory listing, headers de seguridad ausentes — cada hallazgo aparece prefijado con `+` en el output, con un código `OSVDB-XXXX` de referencia (ej. `OSVDB-3268` = directory indexing detectado).

> Para un escaneo más rápido: `nikto -h IP -Tuning 123` restringe a las categorías de chequeo más comunes (los códigos de tuning van concatenados, no separados por coma).

## Tabla comparativa — el mismo patrón, 4 vocabularios distintos
| Misconfiguración | Apache | Python HTTP | Node.js | Nginx |
|---|---|---|---|---|
| Disclosure de versión en headers | Sí | Sí | Parcial (vía `X-Powered-By`) | Sí |
| Directory listing | `Options +Indexes` | Comportamiento default (todo el directorio) | N/A | `autoindex on` |
| Endpoint de estado/debug expuesto | `/server-status` (`mod_status`) | N/A | `/api/debug/env`, `/api/routes` | `/nginx_status` (`stub_status`) |
| Archivos sensibles accesibles | backups, `.htpasswd` | `.env`, archivos comprimidos | `config.js` estático | archivos en directorios con `autoindex` |
| Headers de seguridad ausentes | Sí (default) | Sí (default) | Sí (default) | Sí (default) |

## Lección central
Las configuraciones default priorizan la facilidad de despliegue sobre la seguridad — version disclosure, directory listing y páginas de estado vienen activadas por default con fines de diagnóstico, para facilitarle la vida al administrador. Quitarlas o restringirlas requiere una acción deliberada que casi nadie hace. Encontrar estos patrones en un engagement real no indica negligencia — indica que nadie revisó los defaults.
