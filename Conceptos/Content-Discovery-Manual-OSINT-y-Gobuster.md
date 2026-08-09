# Content Discovery — Manual, OSINT y Automatizado (robots.txt, Gobuster)

Relacionado: [[Reconocimiento-Pasivo]] · [[Reconocimiento-Manual-Web-y-DevTools]] · [[Reconocimiento-Subdominios-Takeover-Ejemplo]] · [[Git-Expuesto-y-Fuga-de-Secretos]] · [[HTTP-Peticiones-y-Respuestas]]

Content discovery combina tres enfoques que se complementan: revisiones **manuales** rápidas, **OSINT** (información que el objetivo ya expuso públicamente sin querer), y herramientas **automatizadas** para cubrir la amplitud que ni lo manual ni el OSINT alcanzan solos. Flujo recomendado: correr los tres contra un objetivo antes de pasar a explotación — lo que se encuentra aquí alimenta directamente las siguientes fases del pentest.

## 1. Manual — archivos que los servidores exponen por convención

### robots.txt
```bash
curl http://SITIO/robots.txt
```
Le dice a los crawlers de buscadores qué **no** indexar. Los dueños de sitios a veces listan ahí directorios sensibles para que no aparezcan en resultados de búsqueda — sin querer, terminan dando una lista lista para usar de rutas interesantes. **No es un control de seguridad, es solo una convención que los bots "bien portados" respetan** — cualquier ruta restringida ahí sigue siendo accesible visitándola directo.

### sitemap.xml
```bash
curl http://SITIO/sitemap.xml
```
Al revés de `robots.txt` (que restringe), `sitemap.xml` **lista** las páginas que el dueño sí quiere indexadas — a veces incluye páginas de staging, contenido viejo, o URLs difíciles de alcanzar navegando normal. Vale la pena revisar cada endpoint listado, especialmente los que traen parámetros (`?id=1`) — son puntos de entrada candidatos para probar.

### HTTP Headers y fingerprinting del framework
```bash
curl http://SITIO -v
```
`-v` (verbose) muestra los headers completos de la respuesta. Buscar especialmente:
- **`Server`**: software del servidor web y versión (`nginx/1.18.0`).
- **`X-Powered-By`**: lenguaje/framework de backend.
- Headers personalizados no estándar (`X-FLAG`, etc.) — a veces exponen información de debug que nunca debió llegar a producción.

Una vez identificado el framework (por headers, favicon, o comentarios en el código fuente — ver [[Reconocimiento-Manual-Web-y-DevTools]]), visitar el sitio oficial del framework: la documentación suele describir la estructura de directorios default, rutas de panel de administración, y credenciales por default. **Probar `admin`/`admin` u otras credenciales default en el panel encontrado es, sorprendentemente, todavía efectivo en aplicaciones mal configuradas.**

## 2. OSINT — información ya expuesta públicamente

### Google Dorking / Google Hacking
Operadores de búsqueda avanzada de Google para filtrar resultados ya indexados del objetivo:

| Operador | Ejemplo | Qué hace |
|---|---|---|
| `site` | `site:sitio.com` | solo resultados de ese dominio |
| `inurl` | `inurl:admin` | resultados con esa palabra en la URL |
| `filetype` | `filetype:pdf` | resultados de un tipo de archivo específico |
| `intitle` | `intitle:admin` | resultados con esa palabra en el título |
| `intext` | `intext:password` | resultados con esa palabra en el cuerpo |
| `cache` | `cache:sitio.com` | versión cacheada de Google de esa página |

Se pueden combinar: `site:sitio.com filetype:pdf` devuelve todos los PDFs indexados de ese dominio.

### Wappalyzer
Extensión de navegador / herramienta online que identifica el stack tecnológico de un sitio (frameworks, CMS, CDN, analytics, pasarelas de pago) con sus versiones — directo desde el ícono en la barra del navegador, sin correr nada.

### Wayback Machine
Detalle completo en [[Reconocimiento-Pasivo]] (`web.archive.org`, CDX API). Aquí sirve específicamente para encontrar páginas **ya removidas** del sitio en vivo pero que siguen archivadas: formularios de login viejos, endpoints de API olvidados, contenido publicado brevemente y luego retirado.

### GitHub
Buscar el nombre de la empresa/dominio directo en GitHub. Cuando aparece un repo relevante, **revisar el historial de commits, no solo los archivos actuales** — es común que datos sensibles (API keys, credenciales, `.env`) se suban por error y luego se "eliminen" en un commit posterior, pero sigan vivos en el historial (mismo principio que [[Git-Expuesto-y-Fuga-de-Secretos]], aplicado aquí a repos públicos en vez de un `.git/` expuesto en el propio servidor).

### S3 Buckets
Formato de URL: `https://{nombre}.s3.amazonaws.com`. Los permisos los define el dueño del bucket, pero las malas configuraciones son comunes — un bucket público expone archivos que nunca debieron ser visibles. Patrones de nombre típicos a probar: `{empresa}-assets`, `{empresa}-backup`, `{empresa}-www`, `{empresa}-dev`. También vale la pena buscar URLs de buckets ya referenciadas en el código fuente del sitio o en repos de GitHub.

## 3. Automatizado — Gobuster
Gobuster (Go, preinstalado en Kali/AttackBox) cubre la amplitud que manual + OSINT no alcanzan: cientos o miles de requests contra un wordlist. **Requiere un buen wordlist** — SecLists (`/usr/share/wordlists/SecLists/`) es la colección estándar; para directorios, `Discovery/Web-Content/common.txt` o `directory-list-2.3-medium.txt` cubren la mayoría de casos.

### Flags globales
| Flag | Significado |
|---|---|
| `-t` / `--threads` | hilos concurrentes (default 10) — subir para acelerar |
| `-w` / `--wordlist` | ruta al wordlist (obligatorio en todos los modos) |
| `-o` / `--output` | guarda resultados en archivo en vez de stdout |
| `--delay` | tiempo de espera entre requests — útil contra servidores con rate-limiting |

### Modo `dir` — directorios y archivos
```bash
gobuster dir -u http://SITIO -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```
`-u` (obligatorio) → URL objetivo · `-w` (obligatorio) → wordlist. Sin alguno de los dos, Gobuster no corre.

Flags adicionales útiles:
| Flag | Significado |
|---|---|
| `-x` / `--extensions` | extensiones a probar (ej. `-x .php,.txt,.js`) |
| `-r` / `--followredirect` | seguir redirecciones HTTP |
| `-k` / `--no-tls-validation` | saltar verificación TLS (útil en labs) |
| `-s` / `--status-codes` | mostrar solo ciertos códigos (ej. `-s 200,301`) |

Resultado típico: directorios (`/assets`, `/private`), redirecciones a login (`/customers` → 302), y **archivos sueltos como `/development.log`** — señal clásica de higiene de despliegue descuidada; un log de desarrollo/debug expuesto en producción puede contener trazas de error, rutas internas del servidor, o credenciales que un desarrollador dejó ahí durante pruebas.

### Subdominios vs Virtual Hosts — la distinción antes de usar `dns`/`vhost`
- **Subdominio**: se resuelve por **DNS** — un registro que apunta a una IP (`blog.sitio.thm`).
- **Virtual host (vhost)**: lo resuelve el **servidor web**, no el DNS — varios sitios corren en la misma IP y el servidor decide cuál servir según el header `Host:` de la petición. Por eso puede haber vhosts que nunca aparecen en un DNS público.

### Modo `dns` — fuerza bruta de subdominios
```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```
`-d` / `--domain` (obligatorio junto con `-w`) → dominio base contra el que se prueba cada palabra del wordlist como subdominio · `--wildcard` → fuerza que la enumeración siga aunque se detecte DNS wildcard (una regla que resuelve *cualquier* subdominio, real o no, a la misma IP) — sin este flag, Gobuster se detendría asumiendo que todos los resultados serían falsos positivos, lo cual no siempre es correcto.

Otros flags: `-i`/`--show-ips` (mostrar las IPs a las que resuelve cada subdominio) · `-r`/`--resolver` (usar un servidor DNS específico para las consultas, en vez del configurado en el sistema).

Justificación de por qué vale la pena: algo parchado en el dominio principal **no implica** que también esté parchado en un subdominio — puede haber una vulnerabilidad presente solo ahí.

### Modo `vhost` — fuerza bruta de virtual hosts
```bash
gobuster vhost -u "http://SITIO" --domain example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain --exclude-length 250-320
```
No usa DNS — manda peticiones HTTP directo a la IP del objetivo, ciclando cada palabra del wordlist como valor del header `Host:`. Encuentra vhosts que **nunca están registrados en DNS público** (ver el caso práctico completo en [[Reconocimiento-Subdominios-Takeover-Ejemplo]]).

`--append-domain` → arma cada intento como `PALABRA.example.thm` en vez de mandar la palabra sola · `--exclude-length 250-320` → filtra respuestas de ese tamaño de bytes (los falsos positivos suelen compartir el mismo tamaño de respuesta — la página "no encontrado" default del servidor).

## Preparar el entorno para labs con DNS local (nota práctica de THM)
Algunos labs corren su propio DNS de prueba y requieren apuntar el resolver local ahí antes de que `gobuster dns` funcione:
```bash
sudo nano /etc/resolv-dnsmasq
# agregar como primera línea: nameserver IP_DEL_LAB
/etc/init.d/dnsmasq restart

sudo nano /etc/hosts
# agregar al final: IP_DEL_LAB dominio.thm
ping dominio.thm   # verificar que resuelve
```

## Referencia rápida
| Método | Técnicas |
|---|---|
| Manual | `robots.txt`, `sitemap.xml`, fingerprinting por favicon, headers HTTP, stack del framework |
| OSINT | Google dorking, Wappalyzer, Wayback Machine, GitHub, S3 buckets |
| Automatizado | Gobuster `dir`, `dns`, `vhost` |
