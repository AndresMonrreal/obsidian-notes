# Reconocimiento Pasivo (OSINT)

Relacionado: [[Protocolos-Aplicacion]] · [[DNS]] · [[Reconocimiento-Activo]] · [[Ofensiva-Pentesting]] · [[Herramientas-CyberChef]]

## Passive vs Active Recon
- **Passive**: solo información **públicamente disponible**, sin enviar tráfico al objetivo. No detectable. Ej.: consultar DNS/WHOIS en servidores públicos, revisar certificate transparency logs, LinkedIn, Shodan/Censys, repos públicos de GitHub.
- **Active**: interacción **directa** con el objetivo — probes que pueden ser detectados/logueados/bloqueados (ver [[Reconocimiento-Activo]]). Cualquier contacto directo con una **persona** de la organización (ej. conversación en un evento social) también cuenta como activo, aunque no se envíen paquetes.

> Consultar servidores WHOIS/DNS públicos es **pasivo** porque hablas con el servidor público del registrador/resolver, no con el objetivo directamente.

## WHOIS y su sucesor RDAP
WHOIS (RFC 3912, puerto TCP 43) da datos de registro de un dominio. Desde el **28 de enero de 2025**, ICANN retiró oficialmente WHOIS para gTLDs a favor de **RDAP** (Registration Data Access Protocol) — usa HTTPS, devuelve JSON estructurado, mejor privacidad. Ver también [[Protocolos-Aplicacion]] para el detalle completo de WHOIS.

```bash
whois tryhackme.com

# RDAP (formato JSON moderno)
curl -s https://rdap.verisign.com/com/v1/domain/tryhackme.com | jq .
```
Qué buscar: fechas (creación/expiración — estiman antigüedad de la empresa) · registrar (patrones de phishing) · name servers (posibles objetivos adicionales) · status codes (`clientTransferProhibited` = dominio bloqueado contra transferencia no autorizada).

## DNS — `dig` vs `nslookup`
`dig` (Domain Information Groper) es la herramienta **moderna preferida**: output más limpio, muestra TTL por defecto, más confiable para scripting. `nslookup` sigue apareciendo en documentación vieja y Windows.

```bash
# nslookup (legacy)
nslookup -type=A tryhackme.com 1.1.1.1
nslookup -type=MX tryhackme.com
nslookup -type=TXT tryhackme.com

# dig (recomendado)
dig tryhackme.com A
dig @1.1.1.1 tryhackme.com MX
dig tryhackme.com TXT
```

| Tipo | Qué revela |
|------|-----------|
| **A** | IPv4 del dominio |
| **AAAA** | IPv6 del dominio |
| **CNAME** | alias hacia otro dominio |
| **MX** | servidores de correo (número = prioridad, menor = más prioritario) |
| **SOA** | name server primario, email admin, número de serie de zona |
| **TXT** | texto libre — SPF, DKIM, DMARC, verificación de dominio |

> Tip de privacidad: usar resolvers públicos como `1.1.1.1` (soporta DoH/DoT) para que tu ISP no registre tus consultas.

## Enumeración pasiva de subdominios
Un `dig`/`nslookup` normal solo resuelve nombres que **ya conoces**. No revela subdominios olvidados como `dev.empresa.com` o `blog.empresa.com`.

### DNSDumpster
Agrega datos DNS públicos (caché de buscadores, bases de zone transfer, certificados) — **sin fuerza bruta**, por eso sigue siendo pasivo. Muestra subdominios/hosts, IPs con geolocalización, registros MX/TXT/CNAME, y un mapa visual de relaciones.

### Certificate Transparency (CT) Logs — crt.sh
El método **más efectivo** hoy en día. Desde ~2015 es obligatorio que las CAs publiquen cada certificado TLS emitido en logs públicos. Cada certificado tiene un campo **SAN** (Subject Alternative Name) que lista los (sub)dominios que cubre.

```
https://crt.sh/?q=%.tryhackme.com
```
El `%` es wildcard — busca certificados de **cualquier** subdominio de `tryhackme.com`. Suele revelar entre 10x y 100x más subdominios que DNSDumpster solo. Alternativas: SecurityTrails, o herramientas CLI como **Subfinder** (agregan varias fuentes pasivas).

> Perspectiva defensiva: las organizaciones monitorean sus propios logs CT y listas de subdominios para detectar **dangling DNS records** (riesgo de subdomain takeover) o subdominios no autorizados.

### Wayback Machine — línea de tiempo de exposición pública
Además de subdominios, el **CDX API** de Wayback Machine (`web.archive.org`) sirve para reconstruir **cuándo** un recurso específico estuvo expuesto públicamente — útil para fechar hallazgos como un `.git/HEAD` filtrado o un endpoint que ya no existe.

```bash
curl "http://web.archive.org/cdx/search/cdx?url=SITIO/.git/HEAD&output=json"
```
Devuelve cada snapshot registrado de esa URL exacta, con timestamp — si aparece un resultado, confirma que el recurso fue público (indexable) desde al menos esa fecha, dato útil para el reporte de un hallazgo de exposición.

> Dato secundario para lo mismo: la fecha de emisión del certificado SSL en `crt.sh` (`https://crt.sh/?q=SITIO&output=json`) acota cuándo un (sub)dominio empezó a existir, aunque no confirma que un recurso específico estuviera expuesto.

## Shodan — motor de búsqueda de dispositivos
A diferencia de Google (indexa páginas web), **Shodan** indexa **dispositivos conectados a internet**: servidores, IoT, cámaras, routers, sistemas de control industrial. Escanea continuamente y guarda banners/respuestas de puertos abiertos.

```
https://www.shodan.io
```
Buscar por dominio o IP directamente. La ficha de cada host muestra: IP y ASN, proveedor de hosting/organización, ubicación geográfica, puertos abiertos con banners/versiones, y tags (ej. `vuln` si coincide con una CVE conocida).

### Filtros de búsqueda útiles
```
hostname:tryhackme.com
org:"TryHackMe"
port:443 country:US
http.component:"wordpress"
```
Alternativa/complemento: **Censys.io** (datos similares de hosts y certificados).

## Referencia rápida de comandos
| Propósito | Comando |
|-----------|---------|
| WHOIS | `whois tryhackme.com` |
| A record (legacy) | `nslookup -type=A tryhackme.com` |
| MX en servidor específico (legacy) | `nslookup -type=MX tryhackme.com 1.1.1.1` |
| A record (recomendado) | `dig tryhackme.com A` |
| MX en servidor específico (recomendado) | `dig @1.1.1.1 tryhackme.com MX` |
| Subdominios pasivos | ir a `crt.sh` y buscar `%.tryhackme.com` |

> Todos estos métodos son fully passive: no generan alertas, riesgo legal mínimo (si están dentro del scope autorizado), pero a menudo destapan subdominios olvidados, servicios desactualizados o misconfiguraciones.
