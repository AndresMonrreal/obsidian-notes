---
tags:
  - pre-security
  - web
  - HTTP
  - HTTPS
  - URL
  - status-codes
  - headers
  - cookies
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🌍 HTTP, URLs, Headers y Cookies

---

## HTTP vs HTTPS

| | HTTP | HTTPS |
|-|------|-------|
| **Nombre** | HyperText Transfer Protocol | HTTP Secure |
| **Puerto** | 80 | 443 |
| **Cifrado** | ❌ Sin cifrado — tráfico visible | ✅ Cifrado TLS — tráfico protegido |
| **Verificación** | ❌ No garantiza el servidor real | ✅ Certifica que hablas con el servidor correcto |
| **Uso** | Obsoleto para datos sensibles | Estándar actual |

> [!warning] HTTP sin cifrado
> En HTTP, cualquiera en la misma red puede leer tu tráfico con herramientas como **Wireshark**. Nunca ingreses contraseñas o datos personales en sitios sin HTTPS (sin el 🔒 en el navegador).

HTTP/HTTPS se desarrolló entre 1989–1991 por **Tim Berners-Lee**.

---

## URL — Uniform Resource Locator

Una URL es la instrucción completa de **cómo y dónde** acceder a un recurso en Internet.

```
https://user:password@tryhackme.com:443/room/dns?id=1#task3
  │      │       │          │         │    │      │    │
  │      │       │          │         │    │      │    └── Fragment
  │      │       │          │         │    │      └─────── Query String
  │      │       │          │         │    └────────────── Path
  │      │       │          │         └─────────────────── Port
  │      │       │          └───────────────────────────── Host
  │      └───────┴──────────────────────────────────────── User:Password
  └─────────────────────────────────────────────────────── Scheme
```

### Componentes de una URL

| Componente | Descripción | Ejemplo |
|------------|-------------|---------|
| **Scheme** | Protocolo de acceso | `http`, `https`, `ftp` |
| **User:Password** | Credenciales (opcional, poco común) | `admin:password123` |
| **Host** | Dominio o IP del servidor | `tryhackme.com` |
| **Port** | Puerto de conexión (opcional si es estándar) | `:8080` |
| **Path** | Ruta al recurso en el servidor | `/blog/article` |
| **Query String** | Parámetros adicionales (empieza con `?`) | `?id=1&sort=asc` |
| **Fragment** | Sección específica de la página (solo en el navegador, no llega al servidor) | `#task3` |

---

## Haciendo un Request HTTP

### Request mínimo

```http
GET / HTTP/1.1
```

### Request completo (con headers)

```http
GET /room/dns HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://tryhackme.com/
```

| Línea | Significado |
|-------|-------------|
| `GET /room/dns HTTP/1.1` | Método + ruta + versión HTTP |
| `Host: tryhackme.com` | Qué sitio quiero (el servidor puede alojar varios) |
| `User-Agent: Mozilla/5.0` | Qué navegador soy (el servidor adapta la respuesta) |
| `Referer: https://...` | Desde qué página vengo |
| *(línea en blanco)* | Indica el fin del request |

### Response del servidor

```http
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html
Content-Length: 98

<html>
<head><title>TryHackMe</title></head>
<body>Welcome To TryHackMe.com</body>
</html>
```

| Línea | Significado |
|-------|-------------|
| `HTTP/1.1 200 OK` | Versión + status code + mensaje |
| `Server: nginx/1.15.8` | Software del servidor web |
| `Content-Type: text/html` | Tipo de contenido que devuelve |
| `Content-Length: 98` | Tamaño en bytes de la respuesta |
| *(línea en blanco)* | Fin de los headers, empieza el body |
| HTML | El contenido real de la página |

---

## Métodos HTTP

| Método | Acción | Ejemplo de uso |
|--------|--------|----------------|
| **GET** | Obtener información | Cargar una página, buscar algo |
| **POST** | Enviar datos (crear) | Registrarse, enviar formulario, login |
| **PUT** | Enviar datos (actualizar) | Editar perfil, actualizar un registro |
| **DELETE** | Eliminar un recurso | Borrar cuenta, eliminar un post |

> [!info] GET vs POST
> - **GET**: los datos van en la URL (`?user=andres`) — visibles en el historial y logs
> - **POST**: los datos van en el body del request — no visibles en la URL

---

## HTTP Status Codes

El primer número define la **categoría**:

| Rango | Categoría | Descripción |
|-------|-----------|-------------|
| **1xx** | Informational | El servidor recibió la primera parte, sigue enviando |
| **2xx** | Success | Request exitoso |
| **3xx** | Redirection | El recurso está en otra parte |
| **4xx** | Client Error | El problema es del cliente (tu request está mal) |
| **5xx** | Server Error | El problema es del servidor |

### Códigos más importantes

| Código | Nombre | Cuándo aparece |
|--------|--------|----------------|
| **200** | OK | Request completado exitosamente |
| **201** | Created | Se creó un recurso nuevo (nuevo usuario, nuevo post) |
| **301** | Moved Permanently | La página se movió para siempre → SEO y redirección |
| **302** | Found | Redirección temporal |
| **400** | Bad Request | El request está mal formado o le faltan datos |
| **401** | Unauthorised | Necesitas autenticarte primero (login) |
| **403** | Forbidden | Autenticado pero sin permisos para acceder |
| **404** | Not Found | La página/recurso no existe |
| **405** | Method Not Allowed | Usaste GET donde esperaban POST (o viceversa) |
| **500** | Internal Server Error | El servidor se rompió y no sabe cómo manejarlo |
| **503** | Service Unavailable | Servidor caído por mantenimiento o sobrecarga |

> [!tip] 401 vs 403
> - **401** → *"No sé quién eres"* — necesitas hacer login
> - **403** → *"Sé quién eres, pero no puedes"* — no tienes permisos aunque estés logueado

> [!warning] 500 en pentesting
> Un error 500 puede revelar información del servidor o de la aplicación. Los atacantes a veces lo provocan deliberadamente para obtener stack traces con rutas internas, versiones y tecnologías usadas.

---

## Headers HTTP

Datos adicionales que viajan junto al request o response para dar contexto.

### Request Headers (cliente → servidor)

| Header | Descripción |
|--------|-------------|
| **Host** | Qué dominio pide el cliente (crucial en servidores con múltiples sitios) |
| **User-Agent** | Navegador y versión del cliente |
| **Content-Length** | Tamaño del body enviado (en POST/PUT) |
| **Accept-Encoding** | Tipos de compresión que soporta el cliente (`gzip`, `br`) |
| **Cookie** | Datos almacenados previamente que se reenvían al servidor |

### Response Headers (servidor → cliente)

| Header | Descripción |
|--------|-------------|
| **Set-Cookie** | Instruye al navegador a guardar una cookie |
| **Cache-Control** | Cuánto tiempo el navegador puede cachear esta respuesta |
| **Content-Type** | Tipo de contenido devuelto (`text/html`, `application/json`, `image/png`) |
| **Content-Encoding** | Método de compresión usado en la respuesta |

> [!tip] En pentesting — headers que revelan info
> El header `Server: nginx/1.15.8` puede revelar software y versión. Si esa versión tiene vulnerabilidades conocidas, es una vía de ataque. Los servidores bien configurados lo ocultan o falsifican.

---

## Cookies

Una **cookie** es un pequeño dato que el servidor le pide al navegador que guarde y que se reenvía automáticamente en cada request posterior.

### ¿Por qué existen?

HTTP es **stateless** — cada request es independiente, el servidor no recuerda quién eres. Las cookies solucionan esto:

```
1. Haces login → servidor verifica tus credenciales
2. Servidor → Set-Cookie: session=abc123xyz
3. Tu navegador guarda esa cookie
4. Cada request siguiente incluye → Cookie: session=abc123xyz
5. El servidor reconoce tu sesión sin que hagas login otra vez
```

### Flujo de una cookie de sesión

```
CLIENTE                              SERVIDOR
  │── POST /login (user+pass) ──────→│
  │← ─ Set-Cookie: token=abc123 ─────│  (token cifrado, no la contraseña)
  │                                   │
  │── GET /dashboard                  │
  │   Cookie: token=abc123 ──────────→│  "Reconozco este token → usuario Andres ✅"
```

### Usos comunes de cookies

| Uso | Descripción |
|-----|-------------|
| **Autenticación** | Token de sesión para mantenerte logueado |
| **Preferencias** | Idioma, tema oscuro/claro, configuración |
| **Tracking** | Historial de navegación (cookies de terceros) |
| **Carrito de compras** | Guardar lo que has añadido |

### Ver cookies en el navegador

`F12` → pestaña **Network** → clic en cualquier request → pestaña **Cookies**

> [!warning] Seguridad de cookies
> - Las cookies de sesión son como **llaves de tu cuenta** — si alguien las roba (**Cookie Theft / Session Hijacking**), puede suplantar tu identidad sin necesitar tu contraseña
> - Flags importantes de seguridad:
>   - `HttpOnly` → la cookie no es accesible desde JavaScript (protege contra XSS)
>   - `Secure` → solo se envía por HTTPS
>   - `SameSite` → protege contra CSRF (Cross-Site Request Forgery)

---

## Flujo completo: desde URL hasta respuesta

```
Escribes: https://tryhackme.com/room/dns

1. [DNS] Resuelve tryhackme.com → 104.26.10.229  (ver [[DNS]])
2. [TCP] Three-Way Handshake con 104.26.10.229:443  (ver [[TCP-UDP-Puertos]])
3. [TLS] Handshake de cifrado (porque es HTTPS)
4. [HTTP] GET /room/dns HTTP/1.1 + headers → servidor
5. [HTTP] Servidor devuelve 200 OK + HTML
6. Navegador renderiza la página
7. Cookie de sesión se envía en el header Cookie: si ya estás logueado
```

---

## Notas relacionadas

- [[DNS]] — Cómo se resuelve el dominio antes del request HTTP
- [[TCP-UDP-Puertos]] — HTTP usa TCP puerto 80, HTTPS usa TCP puerto 443
- [[OSI-Model]] — HTTP vive en Layer 7 (Application), TLS en Layer 6 (Presentation)
- [[Seguridad-de-Red]] — HTTPS usa TLS (cifrado de capa de presentación)
- [[Packets-and-Frames]] — Los requests HTTP viajan dentro de packets TCP
