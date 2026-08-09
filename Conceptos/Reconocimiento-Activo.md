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

> Flujo completo de testing manual (page source, Inspector, Debugger, Network, Storage) en [[Reconocimiento-Manual-Web-y-DevTools]].

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

## `nmap` — descubrimiento de hosts (host discovery)
Antes de escanear puertos (ver [[TCP-UDP-Puertos]]), Nmap puede usarse solo para averiguar qué hosts están vivos en un rango o subred — evita perder tiempo escaneando puertos de una IP que ni siquiera responde. Usa protocolos de distintas capas: **ARP** (enlace), **ICMP** (red), **TCP** y **UDP** (transporte). El flag `-sn` le dice a Nmap que se quede solo en el descubrimiento, sin pasar a escanear puertos después.

### Especificar objetivos
Lista (`nmap IP1 IP2 dominio.com`) · rango (`nmap 10.11.12.15-20`, 6 IPs) · subred (`nmap IP/30`, 4 IPs) · archivo (`nmap -iL lista.txt`). Para ver qué hosts va a tocar sin escanearlos de verdad: `nmap -sL TARGETS` (ojo: por default sí intenta resolución DNS inversa sobre todos — para evitarlo, sumar `-n`).

### Comportamiento por defecto (sin especificar técnica)
| Situación | Qué usa Nmap |
|---|---|
| Privilegiado + objetivo en la LAN | ARP requests |
| Privilegiado + objetivo fuera de la LAN | ICMP Echo + TCP ACK a 80 + TCP SYN a 443 + ICMP Timestamp |
| Sin privilegios + objetivo fuera de la LAN | 3-way handshake TCP (SYN) a 80 y 443 |

### ARP Scan — `-PR`
Solo funciona si estás en la **misma subred** que el objetivo (ARP no se enruta). Es la técnica más fiable en LAN — muchos firewalls no filtran capa 2, por eso también es clave en post-explotación / enumeración interna una vez dentro de una red.
```bash
nmap -PR -sn 10.65.84.234/24
```

### ICMP — Echo / Timestamp / Address Mask (`-PE` / `-PP` / `-PM`)
El ping clásico usa Echo Request/Reply (type 8/0), pero muchos firewalls lo bloquean (Windows lo bloquea por default). Si Echo falla, Nmap puede probar Timestamp (type 13/14) o Address Mask (type 17/18) — cada firewall filtra distinto, conviene tener las tres a la mano.
```bash
nmap -PE -sn 10.200.6.0/24   # ICMP Echo
nmap -PP -sn 10.200.6.0/24   # ICMP Timestamp
nmap -PM -sn 10.200.6.0/24   # ICMP Address Mask
```

### TCP SYN / ACK Ping — `-PS` / `-PA`
Mandan un paquete con la bandera SYN o ACK a un puerto (80 por default); cualquier respuesta (SYN/ACK o RST) confirma que el host vive — aquí no importa el estado real del puerto.
```bash
nmap -PS -sn TARGETS          # SYN, puerto 80 por default
nmap -PS21,80,443 -sn TARGETS # SYN a puertos específicos (ej. -PS23 para telnet)
nmap -PA -sn TARGETS          # ACK, puerto 80 por default
```
- `-PS` **no requiere privilegios**: sin ellos, Nmap completa el 3-way handshake normal por la pila del SO en vez de mandar el SYN "crudo" — el resultado del descubrimiento es igual.
- `-PA` **sí requiere `sudo`**: un ACK sin conexión previa no se puede armar con la API de sockets normal del SO — hace falta un raw socket, reservado a root/sudoers.

### UDP Ping — `-PU`
Manda un paquete UDP a un puerto (idealmente cerrado). Si el puerto está **cerrado**, el sistema responde con ICMP "port unreachable" — esa respuesta confirma que el host vive (un puerto UDP abierto normalmente no contesta nada).
```bash
nmap -PU53,161,162 -sn TARGETS
```

### DNS: `-n` vs `-R`
Por default Nmap solo intenta rDNS de los hosts que **confirmó vivos**. `-n` la desactiva por completo (más velocidad). `-R` la fuerza incluso en los que no respondieron — útil porque el registro DNS puede seguir revelando info (ej. `dc01.domain.com` delata un Domain Controller) aunque el host no conteste al ping.

### Masscan (alternativa más agresiva)
```bash
masscan 10.200.6.0/24 -p80,443
```
Mismo concepto que Nmap pero mucho más rápido/agresivo en la tasa de paquetes — pensado para escaneos masivos. No viene preinstalado en la AttackBox (`apt install masscan`).

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
| `nmap -PR -sn RANGO` | descubrimiento de hosts en la LAN (ARP) |
| `nmap -PE -sn RANGO` | descubrimiento de hosts fuera de la LAN (ICMP Echo) |

## Combinar reconocimiento pasivo + activo
Flujo típico: `ping` confirma que el host vive → `traceroute` entiende la ruta de red → `nc`/`telnet` prueban puertos específicos y confirman qué servicio corre → de ahí se pasa a escáneres más avanzados como Nmap ([[TCP-UDP-Puertos]]) o herramientas dedicadas por protocolo.
