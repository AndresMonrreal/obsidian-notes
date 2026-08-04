# Reconocimiento Activo

Relacionado: [[Reconocimiento-Pasivo]] · [[Protocolos-Aplicacion]] · [[TCP-UDP-Puertos]] · [[Ofensiva-Pentesting]] · [[Herramientas-Burp-Suite]]

## Qué es
A diferencia del reconocimiento pasivo (ver [[Reconocimiento-Pasivo]]), aquí hay **interacción directa** con el objetivo: se envían paquetes, se abren conexiones, se prueban servicios. Deja rastro (logs, alertas de IDS/WAF) y **siempre requiere autorización legal explícita** antes de hacerlo.

## Navegador + DevTools como herramienta de recon
El navegador es de las herramientas más útiles y menos sospechosas: su tráfico se mezcla con el de usuarios normales. Abrir DevTools: `Ctrl+Shift+I` (Windows/Linux) o `Option+Cmd+I` (macOS).

| Pestaña | Qué revela |
|---------|-----------|
| **Network** | headers de request/response (`Server`, `X-Powered-By`, `Content-Security-Policy`), timing, status codes, cookies |
| **Console** | ejecutar JS en el contexto de la página, ver errores |
| **Sources** | archivos JS/CSS/HTML cargados — **frecuentemente tienen endpoints de API hardcodeados, rutas internas, comentarios de desarrolladores** que nunca debían ser públicos |
| **Application → Storage** | cookies, Local/Session Storage — a veces exponen tokens de sesión o API keys expuestas del lado cliente |
| **Security** | detalles del certificado TLS (issuer, validez, **SANs** — pueden revelar subdominios adicionales) |

### Extensiones útiles
**Wappalyzer** (identifica CMS, frameworks JS, servidores, analytics, CDN de forma pasiva mientras navegas) · **FoxyProxy** (cambiar entre proxies como Burp Suite/ZAP) · **User-Agent Switcher** (emular otro navegador/dispositivo — pero cambios rápidos de UA pueden disparar WAFs modernos).

## `ping` — verificar si el objetivo está vivo
Usa **ICMP**: envía Echo Request (type 8), espera Echo Reply (type 0).

```bash
ping -c 5 MACHINE_IP          # Linux/macOS, 5 paquetes
ping -n 5 MACHINE_IP          # Windows, equivalente
ping -4 -c 5 MACHINE_IP       # forzar IPv4
ping -6 -c 5 MACHINE_IPV6     # forzar IPv6
```

### TTL como pista de fingerprinting del SO
El **TTL** (Time To Live) no es tiempo — es el número máximo de saltos (routers) que puede atravesar el paquete. Cada router lo decrementa en 1. El SO fija un TTL inicial:

| SO | TTL inicial típico |
|----|---------------------|
| **Linux** | 64 |
| **Windows** | 128 |

> Ojo: el TTL que ves en la respuesta ya fue decrementado por los routers intermedios. Un TTL de 58 probablemente es Linux (64) a 6 saltos de distancia, **no** un SO distinto.

### Interpretar "sin respuesta"
`100% packet loss` o `Destination Host Unreachable` puede significar: la máquina está apagada/reiniciando · un firewall bloquea ICMP · el objetivo está detrás de NAT que descarta ICMP · **Windows Firewall bloquea ping por defecto** en la mayoría de versiones · WAFs/CDNs/cloud providers (AWS, Azure, GCP) suelen bloquear ICMP por completo.

**Header ICMP: 8 bytes.** Flag para ajustar tamaño del payload: `-s` (ej. `ping -s 1000 MACHINE_IP`).

## `traceroute` / `tracert` / `mtr` — mapear la ruta de red
```bash
traceroute MACHINE_IP     # Linux/macOS
tracert MACHINE_IP        # Windows
traceroute -6 MACHINE_IPV6   # IPv6
mtr MACHINE_IP            # vista en tiempo real, combina ping + traceroute
```

### Cómo funciona
Explota el campo TTL: envía paquetes con TTL creciente (1, 2, 3...). El primer router con TTL=1 lo descarta y responde con **ICMP Time-to-Live Exceeded**, revelando su IP. Así, salto por salto, se reconstruye la ruta completa hasta el destino.

- Routers que no responden (por configuración de seguridad) aparecen como `*` en el output.
- La ruta **no es fija** — enrutamiento dinámico (BGP/OSPF), balanceo de carga y anycast (típico en CDNs como Cloudflare) hacen que corridas consecutivas den rutas distintas.
- `-T` fuerza modo TCP (para saltar filtros de UDP) · `-I` fuerza modo ICMP.

## `telnet` — banner grabbing de servicios en texto plano
Telnet como **servidor de administración remota** está obsoleto (reemplazado por SSH), pero el **cliente** sigue siendo útil para conectar a cualquier puerto TCP y leer la respuesta cruda del servicio ("banner grabbing").

```bash
telnet MACHINE_IP 80
GET / HTTP/1.1
host: telnet
[Enter dos veces]

HTTP/1.1 200 OK
Server: nginx/1.6.2
...
```
El header `Server: nginx/1.6.2` revela software y versión — se puede cruzar contra bases de CVE/Exploit-DB. Para servicios cifrados (HTTPS 443, SMTPS 465) telnet no sirve: usar `curl --head https://IP` u `openssl s_client -connect IP:443`.

## `netcat` (`nc`) — banner grabbing y listener
Más versátil que telnet: soporta TCP y UDP, y puede actuar como **cliente o servidor**.

### Como cliente (banner grabbing)
```bash
nc MACHINE_IP 80
GET / HTTP/1.1
host: netcat
[Enter/Shift+Enter]
```
Funciona igual contra FTP (puerto 21, banner inmediato sin comandos), SMTP (puerto 25), etc.

### Como servidor (listener)
```bash
nc -vnlp 1234       # escuchar en el puerto 1234
```
| Flag | Significado |
|------|-------------|
| `-l` | modo escucha (listen) |
| `-p` | número de puerto (debe ir justo antes del número) |
| `-n` | sin resolución DNS |
| `-v` / `-vv` | verbose / muy verbose |
| `-k` | seguir escuchando tras desconectar un cliente |
| `-6` | IPv6 |

> Puertos < 1024 requieren privilegios root para escuchar. Para cifrado, usar `ncat --ssl` (de Nmap) en vez de `nc`.

## Referencia rápida
| Comando | Propósito |
|---------|-----------|
| `ping -c 10 MACHINE_IP` | verificar si el host responde |
| `traceroute MACHINE_IP` | mapear ruta de red |
| `mtr MACHINE_IP` | traceroute en tiempo real con estadísticas |
| `telnet MACHINE_IP PUERTO` | banner grabbing (legacy) |
| `nc MACHINE_IP PUERTO` | banner grabbing (cliente) |
| `nc -lvnp PUERTO` | listener (servidor) |
| `curl -I http://MACHINE_IP` | banner grabbing HTTP (más seguro que telnet) |

## Combinar reconocimiento pasivo + activo
Flujo típico: `ping` confirma que el host vive → `traceroute` entiende la ruta de red → `nc`/`telnet` prueban puertos específicos y confirman qué servicio corre → de ahí se pasa a escáneres más avanzados como Nmap ([[TCP-UDP-Puertos]]) o herramientas dedicadas por protocolo.
