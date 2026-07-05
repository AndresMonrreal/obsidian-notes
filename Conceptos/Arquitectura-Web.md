---
tags:
  - pre-security
  - web
  - arquitectura
  - backend
  - frontend
  - servidores
  - vulnerabilidades-web
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🏗️ Arquitectura Web — Cómo funcionan los sitios web

---

## Front End vs Back End

Todo sitio web tiene dos lados:

```
USUARIO
  │
  │  Navegador (Chrome, Firefox, Safari)
  │       ↓
  │  ┌─────────────────────────────────────┐
  │  │         FRONT END (Client-Side)     │
  │  │  HTML · CSS · JavaScript            │
  │  │  Lo que VES y con lo que interactúas│
  │  └─────────────────────────────────────┘
  │                    ↕ HTTP Request/Response
  │  ┌─────────────────────────────────────┐
  │  │         BACK END (Server-Side)      │
  │  │  PHP · Python · NodeJS · Ruby       │
  │  │  Lógica, base de datos, seguridad   │
  │  │  No lo ves directamente             │
  │  └─────────────────────────────────────┘
```

| | Front End | Back End |
|-|-----------|----------|
| **Dónde corre** | En tu navegador | En el servidor remoto |
| **Visible para el usuario** | ✅ Sí (View Source) | ❌ No |
| **Tecnologías** | HTML, CSS, JavaScript | PHP, Python, Ruby, NodeJS |
| **Qué hace** | Mostrar la página, manejar la UI | Procesar datos, consultar BD, lógica de negocio |

> [!info] Importante para ciberseguridad
> El código Backend **nunca llega al navegador del usuario** — solo el resultado de su ejecución. Sin embargo, si está mal escrito, puede ser explotado a través de los inputs que sí son accesibles desde el Frontend.

---

## Web Server (Servidor Web)

Un **web server** es el software que escucha conexiones entrantes HTTP/HTTPS y entrega contenido al cliente.

### Software más común

| Software | Sistema Operativo | Root Directory por defecto |
|----------|------------------|---------------------------|
| **Apache** | Linux | `/var/www/html` |
| **Nginx** | Linux | `/var/www/html` |
| **IIS** | Windows | `C:\inetpub\wwwroot` |
| **NodeJS** | Ambos | Configurable |

### ¿Cómo sirve archivos?

```
Request: http://example.com/images/foto.jpg
                                    ↓
Servidor busca: /var/www/html/images/foto.jpg
                                    ↓
Si existe → lo envía | Si no → 404 Not Found
```

### Virtual Hosts

Un solo servidor físico puede alojar **múltiples sitios web** con diferentes dominios. Usa el header `Host:` del request HTTP para decidir cuál mostrar.

```
Request llega con Host: one.com  →  sirve /var/www/website_one
Request llega con Host: two.com  →  sirve /var/www/website_two
Request sin coincidencia          →  sirve el sitio por defecto
```

Los virtual hosts son **archivos de configuración de texto** dentro del servidor web. No hay límite de cuántos sitios puede alojar un servidor.

> [!tip] Relevancia en pentesting
> La técnica de **virtual host enumeration** (con herramientas como `gobuster vhost`) busca subdominios o hosts adicionales en el mismo servidor que no están públicamente anunciados pero siguen activos.

---

## Contenido Estático vs Dinámico

| | Estático | Dinámico |
|-|---------|----------|
| **Qué es** | Archivo fijo que no cambia | Contenido generado en tiempo real |
| **Ejemplos** | Imágenes, CSS, JS, HTML fijo | Feed de noticias, resultados de búsqueda, perfil de usuario |
| **Quién lo genera** | El archivo tal como está en disco | El Backend lo construye en cada request |
| **Velocidad** | ⚡ Muy rápido (puede servirse desde CDN) | 🐢 Más lento (requiere procesamiento) |

### Ejemplo de contenido dinámico con PHP

Request: `http://example.com/index.php?name=andres`

Código en el servidor (nunca visible para el usuario):
```php
<html><body>Hello <?php echo $_GET["name"]; ?></body></html>
```

Lo que recibe el navegador:
```html
<html><body>Hello andres</body></html>
```

El PHP se ejecutó en el servidor, el usuario solo ve HTML.

---

## Lenguajes de Backend

| Lenguaje | Uso común |
|----------|-----------|
| **PHP** | WordPress, aplicaciones web tradicionales |
| **Python** | Django, Flask — APIs, machine learning integrado |
| **Ruby** | Ruby on Rails — startups, prototipado rápido |
| **NodeJS** | JavaScript en el servidor — apps en tiempo real |
| **Perl** | Scripts de sistema, aplicaciones legacy |
| **Java / C#** | Aplicaciones empresariales grandes |

> [!warning] Seguridad en el Backend
> Los lenguajes de Backend tienen acceso a la base de datos, al sistema de archivos y a recursos internos. Un input del usuario que llegue sin validación al Backend puede derivar en **SQL Injection, RCE, Path Traversal**, etc. La seguridad de una app web depende casi completamente de qué tan bien está escrito el Backend.

---

## Arquitectura Completa de una Web App Moderna

```
INTERNET
    │
    ▼
┌──────────────────┐
│   WAF            │  ← Filtra ataques antes de que lleguen
└────────┬─────────┘
         │
┌────────▼─────────┐
│  Load Balancer   │  ← Distribuye tráfico entre servidores
└──┬───────────┬───┘
   │           │
┌──▼──┐     ┌──▼──┐
│ Web │     │ Web │  ← Múltiples servidores web (Apache/Nginx)
│ Srv │     │ Srv │
└──┬──┘     └──┬──┘
   └─────┬─────┘
         │
┌────────▼─────────┐
│    Base de Datos │  ← MySQL, PostgreSQL, MongoDB...
└──────────────────┘
         +
┌──────────────────┐
│      CDN         │  ← Archivos estáticos distribuidos globalmente
└──────────────────┘
```

---

## Load Balancer

**Función:** Distribuir el tráfico entrante entre múltiples servidores para garantizar **alta disponibilidad** y evitar que un solo servidor se sature.

### Algoritmos de distribución

| Algoritmo | Cómo funciona |
|-----------|---------------|
| **Round-Robin** | Envía requests a cada servidor por turno (1→2→3→1→2→3...) |
| **Weighted** | Envía más tráfico al servidor menos ocupado en ese momento |
| **IP Hash** | El mismo usuario siempre va al mismo servidor (sesiones consistentes) |

### Health Checks

El load balancer hace **verificaciones periódicas** a cada servidor:
- Si el servidor responde correctamente → sigue recibiendo tráfico ✅
- Si el servidor no responde o devuelve error → **se saca de rotación** hasta que se recupere ❌

> [!info] Alta disponibilidad
> Con load balancer, si un servidor se cae, los demás absorben el tráfico sin que el usuario note nada. Esto se llama **failover**.

---

## CDN — Content Delivery Network

**Función:** Distribuir archivos **estáticos** (imágenes, JS, CSS, videos) en servidores ubicados en múltiples partes del mundo para que el usuario descargue desde el servidor **más cercano** geográficamente.

```
Sin CDN:
[Usuario en México] ──── 200ms ────→ [Servidor en Alemania]

Con CDN:
[Usuario en México] ── 15ms ──→ [Nodo CDN en Dallas]
                                  (copia del archivo)
```

### Beneficios

| Beneficio | Descripción |
|-----------|-------------|
| **Velocidad** | Menor latencia al servir desde un nodo cercano |
| **Reduce carga** | El servidor principal no tiene que servir archivos estáticos |
| **Disponibilidad** | Si un nodo cae, otro asume el tráfico |
| **DDoS Protection** | Los CDNs grandes absorben ataques volumétricos |

Proveedores comunes: **Cloudflare, AWS CloudFront, Akamai, Fastly**.

---

## Databases (Bases de Datos)

Los servidores web se comunican con bases de datos para **almacenar y recuperar información** de los usuarios.

| Base de Datos | Tipo | Uso típico |
|---------------|------|-----------|
| **MySQL** | Relacional (SQL) | Aplicaciones web generales, WordPress |
| **PostgreSQL** | Relacional (SQL) | Apps que requieren consistencia fuerte |
| **MSSQL** | Relacional (SQL) | Entornos Windows/empresariales |
| **MongoDB** | No relacional (NoSQL) | APIs, datos no estructurados, flexibilidad |
| **SQLite** | Relacional (SQL) | Apps pequeñas, desarrollo local |
| **Redis** | Clave-valor (NoSQL) | Caché, sesiones, tiempo real |

> [!warning] SQL Injection
> Cuando el Backend construye consultas SQL usando directamente el input del usuario sin sanitizar, un atacante puede **inyectar código SQL** para leer, modificar o eliminar datos de la base de datos. Es una de las vulnerabilidades más críticas y antiguas (OWASP Top 10 #3).

---

## WAF — Web Application Firewall

**Función:** Capa de seguridad que se interpone entre el usuario y el servidor web, analizando cada request HTTP en busca de patrones maliciosos.

### ¿Qué analiza un WAF?

| Análisis | Descripción |
|---------|-------------|
| **Técnicas de ataque** | SQL Injection, XSS, Path Traversal, Command Injection |
| **Bots vs humanos** | Verifica si la request viene de un navegador real o automatización |
| **Rate limiting** | Limita el número de requests por IP por segundo → protege contra DDoS y brute force |
| **Anomalías** | Payloads inusuales, tamaños de request fuera de lo normal |

### WAF vs Firewall tradicional

| | Firewall (red) | WAF |
|-|---------------|-----|
| **Trabaja con** | IPs, puertos, protocolos | Contenido HTTP (URLs, body, headers) |
| **Capa OSI** | L3 / L4 | L7 (Application) |
| **Ve** | Quién se conecta | Qué se está pidiendo |
| **Protege contra** | Acceso no autorizado a la red | Ataques a la aplicación web |

> [!info] Bypass de WAF
> En pentesting avanzado existe la técnica de **WAF bypass** — modificar los payloads de ataque para que el WAF no los reconozca (encoding, case variation, fragmentación). Por eso un WAF solo no es suficiente: el código debe estar bien escrito.

---

## 🔴 HTML Injection — Primera Vulnerabilidad Web

**HTML Injection** ocurre cuando el sitio web toma **input del usuario y lo incluye directamente en la página** sin sanitizar, permitiendo que el atacante inyecte código HTML que el navegador ejecuta.

### Ejemplo básico

El sitio tiene un campo que muestra el nombre del usuario:
```html
<!-- Código vulnerable en el servidor -->
<p>Bienvenido, <?= $_GET['name'] ?></p>
```

Request normal:
```
http://example.com/welcome?name=Andres
→ <p>Bienvenido, Andres</p>
```

Request con HTML injection:
```
http://example.com/welcome?name=<h1>HACKEADO</h1>
→ <p>Bienvenido, <h1>HACKEADO</h1></p>
```

El navegador renderiza el `<h1>` como un encabezado real — el atacante controló el HTML de la página.

### ¿Qué puede hacer un atacante con HTML Injection?

| Ataque | Descripción |
|--------|-------------|
| **Defacement** | Cambiar visualmente el contenido de la página |
| **Phishing** | Inyectar un formulario de login falso en la página real |
| **Redirección** | `<meta http-equiv="refresh" content="0;url=http://evil.com">` |
| **Escalada a XSS** | Si acepta `<script>`, la vulnerabilidad se convierte en **Cross-Site Scripting** |

### Cómo se previene

```php
// ❌ Vulnerable
echo "Bienvenido, " . $_GET['name'];

// ✅ Seguro — convierte < > en entidades HTML (&lt; &gt;)
echo "Bienvenido, " . htmlspecialchars($_GET['name']);
```

> [!warning] HTML Injection → XSS
> HTML Injection es el precursor de **XSS (Cross-Site Scripting)**. Si el sitio también acepta la etiqueta `<script>`, el atacante puede ejecutar JavaScript en el navegador de otras personas — robando cookies de sesión, redirigiendo usuarios o realizando acciones en su nombre.

---

## Flujo completo de un request web moderno

```
1. [Usuario] escribe: https://tryhackme.com/dashboard
2. [DNS] resuelve tryhackme.com → IP  (ver [[DNS]])
3. [TCP] conexión + TLS handshake (ver [[TCP-UDP-Puertos]])
4. [WAF] analiza el request HTTP — ¿es legítimo?
5. [Load Balancer] decide a qué servidor web enviarlo
6. [Web Server] (Nginx/Apache) recibe el request
7. [Backend] PHP/Python consulta la base de datos
8. [DB] devuelve los datos del usuario
9. [Backend] construye el HTML dinámico
10. [CDN] los archivos estáticos (CSS/JS/imágenes) vienen del nodo más cercano
11. [Navegador] renderiza el Frontend con todo lo recibido ✅
```

---

## Notas relacionadas

- [[HTTP-Web]] — El protocolo que usan todos estos componentes
- [[DNS]] — Paso previo a llegar al servidor
- [[TCP-UDP-Puertos]] — Puertos 80/443 donde escucha el web server
- [[Seguridad-de-Red]] — Firewall de red como complemento al WAF
- [[Roles-Ciberseguridad]] — El analista SOC monitorea logs de web servers y WAF
