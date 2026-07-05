---
tags:
  - pre-security
  - networks
  - ARP
  - DHCP
  - protocolos
  - ICMP
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# ⚙️ Protocolos Esenciales: ARP, DHCP e ICMP

## Encapsulamiento (Encapsulation)

Cuando envías datos por la red, estos **bajan por capas** del modelo de red (OSI/TCP-IP). En cada capa se añade una **cabecera (header)** con información técnica.

```
[Datos originales]
      ↓  + Header de Aplicación
[Segmento TCP/UDP]
      ↓  + Header de Red (IP)
[Paquete IP]
      ↓  + Header de Enlace (MAC)
[Frame Ethernet]
      ↓
   📡 Red física
```

> [!info] Encapsulamiento
> Es el proceso de **envolver datos** con información adicional (headers) a medida que bajan por las capas. En el destino, el proceso inverso se llama **desencapsulamiento**.

---

## ARP — Address Resolution Protocol

**Propósito:** Traducir (mapear) una **dirección IP conocida** → **dirección MAC física**.

### ¿Por qué existe ARP?

En una red local, los frames Ethernet se envían usando **MACs**, no IPs. ARP es el "directorio telefónico" que conecta ambas.

### Flujo de ARP

```
PC1 quiere hablar con 192.168.1.5 pero no sabe su MAC

1. [ARP Request] → Broadcast a TODA la red
   "¿Quién tiene la IP 192.168.1.5? Respóndeme a mí (MAC: AA:BB:CC:...)"

2. [ARP Reply] ← Solo responde el dueño de esa IP
   "Yo soy 192.168.1.5, mi MAC es DD:EE:FF:..."

3. PC1 guarda esto en su ARP Cache para no preguntar de nuevo
```

### Tipos de paquetes ARP

| Tipo | Dirección | Descripción |
|------|-----------|-------------|
| **ARP Request** | Broadcast (a todos) | "¿Quién tiene esta IP?" |
| **ARP Reply** | Unicast (al que preguntó) | "Yo tengo esa IP, mi MAC es..." |

### Ver la tabla ARP en tu PC

```bash
# Windows
arp -a

# Linux
arp -n
# o
ip neigh show
```

> [!warning] Amenaza: ARP Spoofing / ARP Poisoning
> Un atacante puede enviar ARP Replies falsos para asociar su MAC con la IP de otro dispositivo (ej. el gateway). Esto permite ataques **Man-in-the-Middle (MitM)**.

---

## DHCP — Dynamic Host Configuration Protocol

**Propósito:** Asignar **automáticamente** una dirección IP y configuración de red a un dispositivo al conectarse.

### El proceso DORA

```
CLIENTE                              SERVIDOR DHCP
   |                                      |
   |── DISCOVER ──────────────────────── →|   "¿Hay algún servidor DHCP?"
   |                                      |   (Broadcast)
   |← ─────────────────────────── OFFER ──|   "Sí, te ofrezco 192.168.1.50"
   |                                      |
   |── REQUEST ─────────────────────────→ |   "Acepto la IP 192.168.1.50"
   |                                      |
   |← ────────────────────────────── ACK ─|   "Confirmado ✅ Es tuya por [lease time]"
   |                                      |
```

| Paso | Nombre | Quién lo envía | Descripción |
|------|--------|----------------|-------------|
| **D** | DHCP Discover | Cliente | Busca servidores DHCP disponibles (broadcast) |
| **O** | DHCP Offer | Servidor | Ofrece una IP disponible |
| **R** | DHCP Request | Cliente | Acepta formalmente la IP ofrecida |
| **A** | DHCP ACK | Servidor | Confirma y entrega la configuración completa |

### ¿Qué entrega el servidor DHCP?

- ✅ Dirección **IP** asignada
- ✅ **Subnet Mask** — para saber el tamaño de la red
- ✅ **Default Gateway** — la puerta de salida hacia otras redes
- ✅ **DNS Server** — para resolver nombres de dominio
- ✅ **Lease time** — tiempo que puedes usar esa IP

---

## DHCP — Puertos y Comportamiento en Red

DHCP usa **UDP** (no TCP) porque el cliente todavía no tiene IP cuando inicia el proceso:

| Lado | Puerto UDP |
|------|-----------|
| Servidor DHCP | **67** |
| Cliente DHCP | **68** |

**El cliente sin IP envía desde `0.0.0.0` al broadcast `255.255.255.255`:**
```
tshark -r DHCP.pcap -n
1   0.000000    0.0.0.0 → 255.255.255.255  DHCP Discover
2   0.013904 192.168.66.1 → 192.168.66.133 DHCP Offer
3   4.115318    0.0.0.0 → 255.255.255.255  DHCP Request
4   4.228117 192.168.66.1 → 192.168.66.133 DHCP ACK
```

> [!info] ¿Por qué `0.0.0.0` en Discover y Request?
> El cliente aún no tiene IP asignada cuando envía DHCP Discover y DHCP Request. Por eso usa la IP especial `0.0.0.0`. Solo después del ACK puede usar la IP que le asignaron.

---

## ICMP — Internet Control Message Protocol

**Propósito:** Diagnóstico de red y reporte de errores. No transfiere datos de usuario.

### Tipos de mensajes ICMP

| Tipo | Nombre | Cuándo se usa |
|------|--------|--------------|
| **Type 0** | Echo Reply | Respuesta al ping |
| **Type 8** | Echo Request | El ping que tú envías |
| **Type 11** | Time Exceeded | Un router descartó el paquete porque el TTL llegó a 0 |
| **Type 3** | Destination Unreachable | No hay ruta hacia el destino |

### Ping — Verificar conectividad

```bash
ping 10.10.10.10           # ping continuo (Linux)
ping -c 4 10.10.10.10     # 4 paquetes (Linux)
ping -n 4 10.10.10.10     # 4 paquetes (Windows)
```

**Flujo:**
```
[Tu PC] ──── ICMP Type 8 (Echo Request) ────→ [Destino]
[Tu PC] ←─── ICMP Type 0 (Echo Reply) ──────  [Destino]
```

**Salida real:**
```
64 bytes from 192.168.11.1: icmp_seq=1 ttl=63 time=11.2 ms
                                              ↑ TTL restante    ↑ Round-Trip Time
Packets: Sent=4, Received=4, Lost=0 (0% loss)
rtt min/avg/max/mdev = 3.8/10.6/23.4/7.9 ms
```

| Resultado | Significado |
|-----------|-------------|
| Respuesta normal | Host activo y alcanzable |
| Request timeout | Firewall bloqueando ICMP, host apagado, o inalcanzable |
| Destination unreachable | No existe ruta hacia el destino |

### Traceroute / Tracert — Descubrir la ruta

`traceroute` (Linux) / `tracert` (Windows) revela **cada router** por el que pasa el paquete hasta el destino.

**Mecanismo — TTL:**
```
El IP header tiene un campo TTL (Time-To-Live):
  - Cada router que reenvía el paquete decrementa el TTL en 1
  - Cuando TTL llega a 0, el router descarta el paquete
  - Y envía un ICMP Type 11 (Time Exceeded) de vuelta al origen
```

**Cómo traceroute usa esto:**
```
traceroute envía paquetes con TTL=1, 2, 3, 4...

TTL=1 → primer router lo descarta → envía ICMP Type 11 → reveals su IP
TTL=2 → segundo router lo descarta → envía ICMP Type 11 → reveals su IP
TTL=3 → tercer router...
...y así hasta llegar al destino
```

```
$ traceroute example.com

 1  _gateway (192.168.66.1)       4ms    ← tu router
 2  192.168.11.1 (192.168.11.1)   5ms    ← router del ISP
 3  100.104.0.1                   11ms
 4  10.149.1.45                   6ms
 5  * * *                               ← router que no responde a ICMP (normal)
 8  172.16.48.1 (172.16.48.1)     5ms
 9  ae81.edge4.Marseille1.Level3.net  50ms  ← router de backbone (ISP grande)
16  93.184.215.14 (93.184.215.14) 140ms  ← destino (example.com)
```

**`* * *`** = ese router no devuelve ICMP Time Exceeded — no es un error, es configuración del router.

> [!tip] ¿Qué revela traceroute en un pentest?
> - La ruta real que siguen los paquetes (puede cambiar entre ejecuciones)
> - IPs privadas de routers del ISP (si responden)
> - Geolocalización aproximada de los saltos (por el nombre DNS del router)
> - Latencia por tramo — útil para identificar cuellos de botella
> - Los routers que bloquean ICMP = están detrás de firewall

> [!tip] Uso en reconocimiento
> `ping` es la primera herramienta para saber si un host está activo. Muchos firewalls bloquean ICMP entrante precisamente para evitar reconocimiento.

---

## Notas relacionadas

- [[Redes-Fundamentos]] — IP, MAC, topologías, NAT, protocolos de routing
- [[Subnetting]] — La subnet mask que DHCP entrega
- [[Protocolos-Aplicacion]] — Puertos de DHCP (67/68), tabla completa de puertos
- [[TCP-UDP-Puertos]] — TTL en headers IP; ICMP usa IP directamente (sin TCP/UDP)
- [[Roles-Ciberseguridad]] — ARP Spoofing como vector de ataque que un analista debe detectar
