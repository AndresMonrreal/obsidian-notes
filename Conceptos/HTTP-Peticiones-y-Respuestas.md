# HTTP — Peticiones, respuestas y headers

Relacionado: [[HTTP-Web]] · [[Cliente-Servidor-HTTP]] · [[Arquitectura-Web]] · [[Herramientas-Burp-Suite]] · [[Bases-de-Datos-SQL]]

## Componentes de una aplicación web
**Front End** (lo que ves en el navegador):
- **HTML** → instrucciones de qué mostrar (la "estructura/ADN").
- **CSS** → apariencia (colores, fuentes, layout).
- **JavaScript** → lógica y decisiones en el navegador (ver [[JavaScript-Web]]).

**Back End** (lo que no ves):
- **Web Server** → aloja y entrega el contenido.
- **Database** → almacena, modifica y recupera información.
- **Infraestructura** → servidores de aplicación, red, almacenamiento.
- **WAF (Web Application Firewall)** → filtra peticiones maliciosas antes de que lleguen al servidor (opcional).

## Anatomía de una URL
`esquema://usuario@host:puerto/path?query#fragment`

| Parte | Descripción |
|-------|-------------|
| **Scheme** | protocolo: `http` o `https` (HTTPS cifra la conexión) |
| **User** | credenciales en la URL (raro e inseguro) |
| **Host/Domain** | qué sitio accedes; ojo con **typosquatting** (dominios casi idénticos usados en phishing) |
| **Port** | 1–65535; por defecto 80 (HTTP), 443 (HTTPS) |
| **Path** | archivo/página específica en el servidor |
| **Query String** | empieza con `?`; términos de búsqueda o inputs. Modificable → riesgo de **inyección** |
| **Fragment** | empieza con `#`; sección de la página. También modificable |

## Mensajes HTTP
Dos tipos:
- **HTTP Request**: la envía el cliente para disparar acciones.
- **HTTP Response**: la envía el servidor como respuesta.

Estructura de un mensaje:
1. **Start Line** → tipo de mensaje (request line o status line).
2. **Headers** → pares clave-valor con info extra.
3. **Empty Line** → divisor obligatorio entre headers y body.
4. **Body** → los datos reales (form data en request; HTML/JSON en response).

## Request Line (línea de petición)
Formato: `MÉTODO /path HTTP/versión`
Ejemplo: `GET /login HTTP/1.1`

### Métodos HTTP
| Método | Uso | Nota de seguridad |
|--------|-----|-------------------|
| **GET** | obtener datos sin modificar | no poner tokens/passwords (viajan en claro) |
| **POST** | crear/actualizar (envía datos) | validar input → evitar **SQLi/XSS** |
| **PUT** | reemplazar/actualizar recurso | verificar autorización |
| **DELETE** | eliminar recurso | solo usuarios autorizados |
| **PATCH** | actualizar parte de un recurso | validar datos |
| **HEAD** | como GET pero solo headers (metadata) | |
| **OPTIONS** | lista métodos disponibles del recurso | |
| **TRACE** | debug de métodos; suele deshabilitarse | |
| **CONNECT** | crea conexión segura (HTTPS) | |

### Versiones HTTP
- **HTTP/0.9** (1991): solo GET.
- **HTTP/1.0** (1996): headers, más tipos de contenido.
- **HTTP/1.1** (1997): conexiones persistentes, chunked encoding. **La más usada hoy.**
- **HTTP/2** (2015): multiplexing, compresión de headers.
- **HTTP/3** (2022): usa QUIC, más rápido y seguro.

## Request Headers (comunes)
| Header | Ejemplo | Descripción |
|--------|---------|-------------|
| `Host` | `Host: tryhackme.com` | nombre del servidor destino |
| `User-Agent` | `User-Agent: Mozilla/5.0` | navegador que envía |
| `Referer` | `Referer: https://google.com/` | URL de origen |
| `Cookie` | `Cookie: user_type=student` | datos que el servidor pidió guardar |
| `Content-Type` | `Content-Type: application/json` | formato de los datos enviados |

## Request Body (formatos de datos en POST/PUT)

### URL Encoded (`application/x-www-form-urlencoded`)
Pares `key=value` separados por `&`; caracteres especiales percent-encoded.
```http
POST /profile HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Aleksandra&age=27&country=US
```

### Form Data (`multipart/form-data`)
Bloques separados por un boundary; sirve para subir archivos/binarios.
```http
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary...
```

### JSON (`application/json`)
```http
Content-Type: application/json

{ "name": "Aleksandra", "age": 27, "country": "US" }
```

### XML (`application/xml`)
```http
Content-Type: application/xml

<user><name>Aleksandra</name><age>27</age></user>
```

## Status Line y códigos de estado
`HTTP/versión CÓDIGO Reason-Phrase` → ej. `HTTP/1.1 200 OK`

| Rango | Categoría |
|-------|-----------|
| 1xx | Informational (recibido, sigue) |
| 2xx | Success (todo bien) |
| 3xx | Redirection (recurso movido) |
| 4xx | Client Error (problema en la petición) |
| 5xx | Server Error (fallo del servidor) |

Códigos frecuentes: **100** Continue · **200** OK · **301** Moved Permanently · **404** Not Found · **500** Internal Server Error.

## Response Headers
| Header | Ejemplo | Descripción |
|--------|---------|-------------|
| `Date` | `Date: Fri, 23 Aug 2024 10:43:21 GMT` | fecha/hora de la respuesta |
| `Content-Type` | `text/html; charset=utf-8` | tipo de contenido devuelto |
| `Server` | `Server: nginx` | software del servidor (a menudo se oculta: filtra info al atacante) |
| `Set-Cookie` | `Set-Cookie: sessionId=38af...` | envía cookies al cliente |
| `Cache-Control` | `Cache-Control: max-age=600` | cuánto cachear |
| `Location` | `Location: /index.html` | destino en redirecciones (3xx); validar → evitar **open redirect** |

Flags de cookie importantes: **HttpOnly** (no accesible desde JS → mitiga XSS) y **Secure** (solo por HTTPS).

## Security Headers
Analizador online: securityheaders.io

### Content-Security-Policy (CSP)
Define de qué dominios se puede cargar contenido → mitiga **XSS**. `self` = mismo dominio.
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'
```
- `default-src` → política por defecto
- `script-src` → de dónde se cargan scripts
- `style-src` → de dónde se cargan CSS

### Strict-Transport-Security (HSTS)
Fuerza HTTPS siempre.
```http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
- `max-age` → tiempo de expiración (s)
- `includeSubDomains` → aplica a subdominios
- `preload` → incluir en listas de preload de navegadores

### X-Content-Type-Options
```http
X-Content-Type-Options: nosniff
```
`nosniff` → el navegador NO adivina el MIME type; usa solo `Content-Type`.

### Referrer-Policy
Controla cuánta info del referrer se envía: `no-referrer`, `same-origin`, `strict-origin`, `strict-origin-when-cross-origin`.

## Práctica: hacer peticiones
Con Burp Suite ([[Herramientas-Burp-Suite]]), curl o el emulador de THM:
```bash
curl -X GET  http://MACHINE_IP/api/users
curl -X POST http://MACHINE_IP/api/user/2 -d '{"country":"US"}' -H "Content-Type: application/json"
curl -X DELETE http://MACHINE_IP/api/user/1
```
