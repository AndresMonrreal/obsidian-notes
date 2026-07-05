---
tags:
  - pre-security
  - web
  - HTTP
  - cliente-servidor
  - protocolos
  - GET
  - request
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🍕 Modelo Cliente-Servidor y HTTP en Profundidad

---

## El Modelo Cliente-Servidor

Todo servicio en red sigue el mismo patrón: un **cliente** hace una petición y un **servidor** la responde.

> [!example] Analogía: Pedir pizza
> - **Alice** = usuaria final (decide qué quiere)
> - **Bob (el carro + GPS)** = cliente + protocolo (lleva la petición al lugar correcto)
> - **Luigi's Pizza** = servidor (recibe, procesa y responde)
> - **La pizza** = el recurso solicitado (respuesta)

En términos de computación:

| Pizza | Computación |
|-------|------------|
| Alice pide pizza | Usuario abre el navegador |
| Bob lleva el pedido a Luigi's | El navegador (cliente) envía request HTTP al servidor |
| Luigi recibe y hace la pizza | El servidor procesa el request |
| Bob trae la pizza de vuelta | El servidor devuelve la respuesta HTTP |
| Alice come la pizza | El navegador renderiza la página |

> [!info] Regla fundamental
> El **cliente siempre inicia la comunicación**. El servidor espera (escucha) y responde. Nunca al revés en el modelo HTTP estándar.

---

## Los 5 Conceptos Clave del Modelo

### 1. Service (Servicio)
El servidor ofrece un servicio específico. Luigi's ofrece pizzas; un servidor web ofrece páginas web. Un mismo servidor puede ofrecer múltiples servicios a la vez.

### 2. Client (Cliente)
El que **inicia la request**. Puede ser un navegador, una app móvil, `curl`, `wget`, o cualquier programa que hable HTTP. El cliente formatea la petición según el protocolo.

### 3. Request y Response
- **Request mal formado o recurso no disponible** → el servidor devuelve un error (ej. 404, 400)
- **Request correcto** → el servidor devuelve el recurso solicitado (ej. 200 OK + HTML)

### 4. Protocolo
El protocolo define **el idioma** que cliente y servidor comparten:
- Qué **comandos/métodos** existen (GET, POST, etc.)
- Cómo se **estructura** un request (método + ruta + versión + headers)
- Qué **sintaxis** usar
- Qué **respuesta** corresponde a cada tipo de request
- Cómo responder a requests **erróneos**

Sin protocolo común, cliente y servidor no pueden comunicarse — como intentar pedir pizza en un idioma que el empleado no entiende.

### 5. Port (Puerto)
Identifica **qué servicio** dentro del servidor recibe la petición. Como las diferentes puertas de Luigi's (puerta A = para llevar, puerta B = comer aquí, puerta C = delivery).

```
Un servidor puede correr múltiples servicios:
  Puerto 80  → Servicio HTTP
  Puerto 443 → Servicio HTTPS
  Puerto 22  → Servicio SSH
  Puerto 3306 → Servicio MySQL
```

Ver [[TCP-UDP-Puertos#Puertos Comunes|tabla de puertos comunes]].

---

## HTTP — Stateless por diseño

HTTP es un protocolo **stateless**: cada request es completamente independiente. El servidor no recuerda requests anteriores.

```
Request 1: GET /login      → servidor procesa, responde, OLVIDA
Request 2: GET /dashboard  → servidor NO sabe quién eres
```

### ¿Cómo se añade estado entonces?

Las aplicaciones web implementan estado a nivel de **aplicación**, no de protocolo:

| Mecanismo | Cómo funciona |
|-----------|--------------|
| **Cookies de sesión** | Servidor crea un `session_id` al hacer login, lo guarda en cookie. El navegador lo reenvía en cada request. |
| **Tokens JWT** | Similar a cookies pero el token contiene información cifrada del usuario |
| **Local Storage** | El navegador guarda datos entre sesiones (no enviados automáticamente) |

Sin estos mecanismos, tendrías que hacer login en **cada request individual**.

> [!tip] Conexión con [[HTTP-Web#Cookies|Cookies]]
> El header `Set-Cookie` es exactamente el mecanismo que los servidores usan para "recordar" al usuario en un protocolo que por diseño no recuerda nada.

---

## Los 9 Métodos HTTP

Las especificaciones oficiales de HTTP (RFC documents) definen 9 métodos. En la práctica, la mayoría de las apps solo usan los primeros 4:

| Método | Acción | ¿Tiene body? | Uso |
|--------|--------|-------------|-----|
| **GET** | Obtener un recurso | ❌ No | Cargar páginas, obtener datos |
| **POST** | Crear un recurso nuevo | ✅ Sí | Login, registros, formularios |
| **PUT** | Reemplazar un recurso completo | ✅ Sí | Actualizar todo un perfil |
| **DELETE** | Eliminar un recurso | ❌ Generalmente no | Borrar cuenta, eliminar post |
| **PATCH** | Modificar parte de un recurso | ✅ Sí | Actualizar solo el nombre de un perfil |
| **HEAD** | Igual que GET pero sin body de respuesta | ❌ No | Verificar si un recurso existe sin descargarlo |
| **OPTIONS** | Preguntar qué métodos acepta el servidor | ❌ No | CORS preflight, descubrimiento de API |
| **CONNECT** | Establece un túnel TCP al servidor | — | Proxies HTTPS |
| **TRACE** | Diagnóstico — el servidor devuelve el request recibido | ❌ No | Debugging (usualmente desactivado por seguridad) |

### GET vs POST — Diferencia crítica

```
GET  → datos en la URL:  /search?q=hacking&sort=asc
                          ↑ visible en historial, logs, barra del navegador

POST → datos en el body: (no visibles en la URL)
                          username=andres&password=1234
```

> [!warning] GET no es seguro para datos sensibles
> Las URLs con parámetros GET quedan en logs de servidores, historial del navegador y headers de Referer. **Nunca enviar passwords ni datos sensibles por GET**.

---

## GET Request — Anatomía Completa

Cuando escribes `http://example.com` en el navegador, el navegador construye y envía esto:

### El Request

```http
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 Firefox/87.0
Accept: text/html,application/xhtml+xml
Accept-Language: es-ES,es;q=0.9
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

| Campo | Descripción |
|-------|-------------|
| `GET /` | Método + ruta del recurso (`/` = página principal = `index.html`) |
| `HTTP/1.1` | Versión del protocolo |
| `Host` | Dominio que se solicita (crucial para Virtual Hosts) |
| `User-Agent` | Navegador y versión (el servidor adapta la respuesta) |
| `Accept` | Tipos de contenido que el cliente puede procesar |
| `Accept-Encoding` | Métodos de compresión soportados |
| `Connection: keep-alive` | Mantener la conexión TCP abierta para múltiples requests |

### La Response

```http
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Cache-Control: max-age=3600

<!DOCTYPE html>
<html>
  <head><title>Example</title></head>
  <body>Hello World</body>
</html>
```

La respuesta se divide en dos partes:

```
┌─────────────────────────────────────────┐
│         RESPONSE HEADERS                │
│  Status code, Content-Type, Date, etc.  │
├─────────────────────────────────────────┤
│            (línea en blanco)            │
├─────────────────────────────────────────┤
│           RESPONSE BODY                 │
│   El contenido real (HTML, JSON, etc.)  │
└─────────────────────────────────────────┘
```

### Campos del request en el Developer Tools (F12 → Network)

| Campo en Dev Tools | Descripción |
|-------------------|-------------|
| **Scheme** | Protocolo usado: `http` o `https` |
| **Host** | Nombre del servidor al que se hace el request |
| **Filename** | Ruta del recurso pedido (`/` → `index.html`) |
| **Address** | IP del servidor (`127.0.0.1` si es local) |
| **Status** | Código de respuesta (`200 OK`, `404 Not Found`, etc.) |

> [!tip] Cómo ver requests reales
> En cualquier navegador: `F12` → pestaña **Network** → recarga la página. Puedes inspeccionar cada request: headers enviados, headers recibidos, body de respuesta, tiempo de carga y más. Esto es fundamental en pentesting web.

---

## Flujo completo: DNS + Cliente-Servidor + HTTP

```
Escribes: http://tryhackme.com/dashboard

1. [DNS] Resuelve tryhackme.com → 104.26.10.229        (ver [[DNS]])
           └─ Caché local → Recursive DNS → Root → TLD → Authoritative

2. [TCP] Conexión al puerto 80 de 104.26.10.229         (ver [[TCP-UDP-Puertos]])
           └─ SYN → SYN/ACK → ACK (Three-Way Handshake)

3. [HTTP] Cliente envía:
           GET /dashboard HTTP/1.1
           Host: tryhackme.com
           Cookie: session=abc123xyz                    (ver [[HTTP-Web#Cookies]])

4. [Servidor] Procesa el request:
           └─ Verifica la cookie de sesión
           └─ Consulta la base de datos                 (ver [[Arquitectura-Web]])
           └─ Genera el HTML dinámico

5. [HTTP] Servidor responde:
           HTTP/1.1 200 OK
           Content-Type: text/html
           [HTML del dashboard]

6. [Navegador] Renderiza el Frontend                    (ver [[Arquitectura-Web#Front End vs Back End]])
```

---

## Notas relacionadas

- [[HTTP-Web]] — HTTP/HTTPS básico, URLs, status codes, headers, cookies
- [[DNS]] — Cómo se resuelve el dominio antes del request
- [[TCP-UDP-Puertos]] — TCP subyacente + puertos donde escucha el servidor
- [[Arquitectura-Web]] — El backend que procesa los requests GET/POST
- [[OSI-Model]] — HTTP en Layer 7, TCP en Layer 4
