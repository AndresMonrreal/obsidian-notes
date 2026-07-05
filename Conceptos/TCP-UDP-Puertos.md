---
tags:
  - pre-security
  - networks
  - TCP
  - UDP
  - puertos
  - handshake
  - protocolos
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔌 TCP, UDP y Puertos — En Profundidad

> [!info] Nota previa
> Este documento amplía lo visto en [[OSI-Model#Layer 4 — Transport (Transporte)|OSI Model - Layer 4]]. Aquí entramos en detalle en los headers, el handshake y los puertos.

---

## El Modelo TCP/IP (4 Capas)

El modelo TCP/IP es una **versión simplificada del OSI** de 4 capas. Es el modelo que realmente usa Internet.

```
Modelo TCP/IP          Equivalente OSI
┌─────────────────┐    ┌─────────────────────┐
│  Application    │ ←→ │ 7. Application       │
│                 │    │ 6. Presentation      │
│                 │    │ 5. Session           │
├─────────────────┤    ├─────────────────────┤
│  Transport      │ ←→ │ 4. Transport         │
├─────────────────┤    ├─────────────────────┤
│  Internet       │ ←→ │ 3. Network           │
├─────────────────┤    ├─────────────────────┤
│ Network Interf. │ ←→ │ 2. Data Link         │
│                 │    │ 1. Physical          │
└─────────────────┘    └─────────────────────┘
```

La **encapsulación** funciona igual: información se añade capa por capa al bajar, y se quita al subir en el destino. Ver: [[Packets-and-Frames#Encapsulamiento|Encapsulamiento]].

---

## TCP — En Profundidad

### Headers de un Packet TCP

| Header | Descripción |
|--------|-------------|
| **Source Port** | Puerto de origen — elegido **al azar** por el cliente (rango 0–65535) |
| **Destination Port** | Puerto de destino — número fijo del servicio (ej. 80 para HTTP) |
| **Source IP** | IP del dispositivo que envía |
| **Destination IP** | IP del dispositivo que recibe |
| **Sequence Number** | Número aleatorio asignado al primer dato. Permite reensamblar en orden |
| **Acknowledgement Number** | Sequence Number + 1. Confirma qué dato se espera a continuación |
| **Checksum** | Cálculo matemático para verificar integridad. Si no coincide → datos corruptos |
| **Data** | Los bytes del archivo/mensaje que se transmite |
| **Flag** | Controla el comportamiento de la conexión (SYN, ACK, FIN, RST...) |

---

### Three-Way Handshake — Establecer Conexión

TCP **requiere establecer conexión** antes de enviar datos. Esto se hace con el **Three-Way Handshake**:

```
CLIENTE (Alice)                    SERVIDOR (Bob)
      |                                  |
      |──── SYN ────────────────────────→|   Paso 1: "Quiero conectar, mi ISN es 0"
      |                                  |
      |←─── SYN/ACK ────────────────────|   Paso 2: "OK, mi ISN es 5000, acepto tu 0"
      |                                  |
      |──── ACK ────────────────────────→|   Paso 3: "Acepto tu ISN 5000, empezamos"
      |                                  |
      |════════════ DATA ════════════════|   Datos viajan aquí
```

**Mensajes del Handshake:**

| Paso | Mensaje | Quién lo envía | Descripción |
|------|---------|----------------|-------------|
| 1 | **SYN** | Cliente | Inicia la conexión. Envía su ISN (Initial Sequence Number) |
| 2 | **SYN/ACK** | Servidor | Confirma el SYN del cliente y envía su propio ISN |
| 3 | **ACK** | Cliente | Confirma el ISN del servidor. Conexión establecida ✅ |
| 4 | **DATA** | Ambos | Los datos reales viajan aquí |
| 5 | **FIN** | Ambos | Cierra la conexión limpiamente |
| — | **RST** | Ambos | Aborta la conexión abruptamente (error o problema) |

### Números de Secuencia en acción

Los sequence numbers garantizan que los datos se **reensamblen en orden**:

| Dispositivo | ISN inicial | Siguiente |
|-------------|------------|-----------|
| Cliente | 0 | 0 + 1 = **1** |
| Cliente | 1 | 1 + 1 = **2** |
| Cliente | 2 | 2 + 1 = **3** |

> [!tip] Por qué importa el ISN aleatorio
> Si el ISN fuera predecible, un atacante podría **secuestrar la sesión TCP** (TCP Session Hijacking). Por eso se elige aleatoriamente.

---

### Cerrar una Conexión TCP (FIN Handshake)

Una vez enviados todos los datos, TCP cierra la conexión con un proceso de 4 pasos:

```
ALICE                              BOB
  |                                 |
  |──── FIN ───────────────────────→|   "Terminé, quiero cerrar"
  |                                 |
  |←─── ACK ────────────────────────|   "Ok, recibido"
  |                                 |
  |←─── FIN ────────────────────────|   "Yo también quiero cerrar"
  |                                 |
  |──── ACK ───────────────────────→|   "Confirmado, cerramos ✅"
  |                                 |
```

> [!warning] Importancia del cierre correcto
> TCP reserva recursos del sistema durante la conexión. Cerrar con FIN libera esos recursos. Un RST es un cierre brusco que indica que algo salió mal.

---

## UDP — En Profundidad

### Headers de un Packet UDP

Mucho más simple que TCP — **sin conexión, sin garantías**:

| Header | Descripción |
|--------|-------------|
| **Time to Live (TTL)** | Timer de expiración del packet (igual que en IP). Ver [[Packets-and-Frames#TTL|TTL]] |
| **Source Address** | IP de origen |
| **Destination Address** | IP de destino |
| **Source Port** | Puerto de origen (elegido al azar) |
| **Destination Port** | Puerto de destino del servicio |
| **Data** | Los bytes transmitidos |

> [!note] Lo que UDP NO tiene vs TCP
> UDP **no tiene**: Sequence Number, Acknowledgement Number, Checksum de datos, ni Flags de control. Es intencionalmente simple para ser rápido.

### Flujo UDP — Sin Handshake

```
ALICE                              BOB
  |                                 |
  |──── DATA ──────────────────────→|   Packet 1 ✅
  |──── DATA ──────────────────────→|   Packet 2 ✅
  |──── DATA ──────────────────────→|   Packet 3 ✗ (perdido, nadie avisa)
  |──── DATA ──────────────────────→|   Packet 4 ✅
  |                                 |
     (No hay confirmación de nada)
```

---

## TCP vs UDP — Cuadro comparativo completo

| Característica | TCP | UDP |
|----------------|-----|-----|
| Tipo de conexión | Orientado a conexión | Sin conexión (stateless) |
| Handshake | ✅ Three-Way Handshake | ❌ No existe |
| Garantía de entrega | ✅ Sí | ❌ No |
| Orden de packets | ✅ Garantizado (seq numbers) | ❌ Pueden llegar desordenados |
| Verificación de errores | ✅ Checksum | ❌ No |
| Velocidad | 🐢 Más lento | 🐇 Más rápido |
| Recursos del sistema | 🔴 Reserva conexión constante | 🟢 Sin reserva |
| Casos de uso | Email, web, archivos, SSH | Streaming, gaming, DNS, VoIP |

---

## Puertos

### ¿Qué es un puerto?

Un **puerto** es un punto de entrada/salida numérico (0–65535) que permite que múltiples servicios corran en el mismo dispositivo simultáneamente. Es como las diferentes puertas de un edificio: cada una lleva a un departamento distinto.

```
                    ┌─────────────────────┐
                    │      SERVIDOR       │
  → Puerto 80  ────→│  Servicio HTTP      │
  → Puerto 443 ────→│  Servicio HTTPS     │
  → Puerto 22  ────→│  Servicio SSH       │
  → Puerto 21  ────→│  Servicio FTP       │
                    └─────────────────────┘
```

### Rango de Puertos

Un número de puerto usa **2 octetos (16 bits)** → rango: **0 a 65.535**

`65.535 = 2^16 - 1` (el puerto 0 está reservado, no se usa en práctica)

| Rango | Nombre | Descripción |
|-------|--------|-------------|
| 0 | Reservado | No se usa |
| **1 – 1.023** | Well-known ports | Servicios estándar (HTTP=80, SSH=22, FTP=21...) Requieren privilegios de root para abrir |
| **1.024 – 49.151** | Registered ports | Servicios de aplicaciones registradas (RDP=3389, MSSQL=1433...) |
| **49.152 – 65.535** | Dynamic/ephemeral ports | Puerto de origen elegido al azar por el cliente para cada conexión |

### Puertos Comunes — Los que debes memorizar

| Protocolo | Puerto | Transport | Descripción |
|-----------|--------|-----------|-------------|
| **FTP** | 21 | TCP | Transferencia de archivos (sin cifrar) |
| **SSH** | 22 | TCP | Acceso remoto seguro por terminal |
| **Telnet** | 23 | TCP | Acceso remoto (sin cifrar, obsoleto) |
| **SMTP** | 25 | TCP | Envío de correos electrónicos |
| **DNS** | 53 | UDP/TCP | Resolución de nombres de dominio |
| **DHCP** | 67/68 | UDP | Asignación dinámica de IPs (servidor/cliente) |
| **HTTP** | 80 | TCP | Navegación web sin cifrar |
| **POP3** | 110 | TCP | Descargar correo electrónico |
| **IMAP** | 143 | TCP | Sincronizar correo electrónico |
| **HTTPS** | 443 | TCP | Navegación web cifrada (TLS) |
| **SMB** | 445 | TCP | Compartir archivos e impresoras en redes Windows |
| **RDP** | 3389 | TCP | Escritorio remoto gráfico (Windows) |

> [!warning] Puertos en pentesting
> Estos puertos son los primeros que se escanean en un **reconocimiento de red** (ej. con `nmap`). Como analista SOC, tráfico inesperado en estos puertos es una señal de alerta.

### Puertos no estándar

Las aplicaciones **pueden** correr en puertos distintos al estándar (ej. un servidor web en el puerto **8080** en vez de 80). En ese caso hay que especificarlo:

```
http://192.168.1.1:8080    ← puerto no estándar, hay que indicarlo
https://192.168.1.1        ← puerto 443 implícito, no hace falta indicarlo
```

---

## La Vida de un Paquete — Escenario Completo

¿Qué pasa exactamente cuando escribes `tryhackme.com` en el navegador y presionas Enter? Aquí el viaje completo del paquete, capa por capa.

```
Escenario: Tu PC (192.168.1.10) busca "ethical hacking" en TryHackMe (104.22.3.4)
```

### Paso 1 — Aplicación prepara la solicitud

El navegador crea una solicitud HTTP GET para `tryhackme.com/search?q=ethical+hacking`. Necesita TLS (HTTPS) → inicia negociación.

### Paso 2 — Transport Layer (TCP) establece conexión

Antes de enviar datos, TCP hace el **Three-Way Handshake** con el servidor:

```
Tu PC ──── SYN ──────────────────────────→ TryHackMe :443
Tu PC ←─── SYN/ACK ──────────────────────  TryHackMe :443
Tu PC ──── ACK ──────────────────────────→ TryHackMe :443
          ↑ Conexión TCP establecida ✅
```

TCP agrega header con: Source Port (aleatorio, ej. 52413), Destination Port (443), Sequence Numbers, Flags.

### Paso 3 — Network Layer (IP) añade direcciones

La capa de red (IP) añade:
- **Source IP:** 192.168.1.10 (tu IP privada)
- **Destination IP:** 104.22.3.4 (IP pública de TryHackMe)
- **TTL:** 64 (se decrementa en cada router)

### Paso 4 — Link Layer (Ethernet) añade MACs

Para salir de tu red local, necesita la MAC del router (obtenida vía ARP):

```
[Frame Ethernet]
Header: MAC_origen → MAC_router_local
        ← IP Packet (con IPs) →
Trailer: FCS (checksum del frame)
```

### Paso 5 — Viaje por la red (router a router)

Cada router en el camino:

```
Router 1 recibe el frame → quita el header Ethernet (L2)
         → lee la IP destino en el IP header (L3)
         → consulta su tabla de routing
         → decrementa TTL en 1
         → añade NUEVO header Ethernet con su propia MAC
         → reenvía al siguiente salto

Router 2 → mismo proceso
Router 3 → mismo proceso
...
```

> [!info] Por qué el header Ethernet cambia en cada salto
> Las MACs solo son válidas dentro de un segmento de red. En cada router se quita el header L2 y se pone uno nuevo para el siguiente segmento. Las IPs permanecen iguales durante todo el viaje.

### Paso 6 — NAT en tu router

Antes de salir a internet, tu router hace NAT:

```
192.168.1.10:52413  ─── [NAT] ───→  85.123.45.67:61234
(IP privada)                         (IP pública de tu casa)
```

El servidor de TryHackMe ve que la solicitud viene de `85.123.45.67`, no de `192.168.1.10`.

### Paso 7 — Respuesta regresa

TryHackMe envía la respuesta a `85.123.45.67:61234`. Tu router consulta la tabla NAT, traduce de vuelta a `192.168.1.10:52413`, y entrega el frame a tu PC. TCP confirma recepción con ACK, el navegador recibe los datos y muestra la página.

```
Resumen del viaje:
[Navegador] → TCP → IP → Ethernet → [Router/NAT] → Internet → [Routers ISP] → [TryHackMe]
[TryHackMe] → Internet → [Routers ISP] → [Router/NAT] → IP → TCP → [Navegador]
```

> [!tip] Por qué importa esto en ciberseguridad
> - Un atacante que captura tráfico en la red local con ARP Spoofing puede interceptar los frames antes de que salgan
> - Un firewall trabaja en L3/L4 — analiza IPs y puertos
> - Un IDS/IPS analiza el payload de L7 (aplicación)
> - Wireshark puede mostrar exactamente cada capa de este proceso

---

## Mapa mental completo

```
DATOS
  └── Se dividen en → PACKETS (L3, lleva IPs) → ver [[Packets-and-Frames]]
        └── Dentro del → FRAME (L2, lleva MACs)
              └── Enviados por → PROTOCOLO DE TRANSPORTE
                    ├── TCP → Confiable, con handshake, usa PUERTOS
                    │     └── Three-Way Handshake: SYN → SYN/ACK → ACK
                    └── UDP → Rápido, sin conexión, usa PUERTOS
                          └── Solo envía, no verifica
```

---

## Notas relacionadas

- [[OSI-Model]] — Contexto completo de las 7 capas
- [[Packets-and-Frames]] — Qué hay dentro de un packet (headers IP)
- [[Protocolos-ARP-DHCP]] — ARP y DHCP: protocolos que usan UDP
- [[Subnetting]] — Cómo se organizan las IPs que viajan en los packets
