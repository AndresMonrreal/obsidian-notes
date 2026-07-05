---
tags:
  - pre-security
  - networks
  - OSI
  - protocolos
  - TCP
  - UDP
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🧱 Modelo OSI — Los 7 Layers

El **Modelo OSI** (Open Systems Interconnection) es un marco de referencia que estandariza cómo los dispositivos se comunican en una red. Divide el proceso en **7 capas**, cada una con una función específica.

> [!tip] Mnemónico para recordar el orden (de abajo hacia arriba)
> **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way
> Physical → Data Link → Network → Transport → Session → Presentation → Application

---

## Vista General

```
┌─────────────────────────────────────────┐
│  Layer 7 — Application   (Aplicación)   │  ← El usuario interactúa aquí
├─────────────────────────────────────────┤
│  Layer 6 — Presentation  (Presentación) │  ← Traducción y cifrado
├─────────────────────────────────────────┤
│  Layer 5 — Session       (Sesión)       │  ← Gestión de conexiones
├─────────────────────────────────────────┤
│  Layer 4 — Transport     (Transporte)   │  ← TCP / UDP
├─────────────────────────────────────────┤
│  Layer 3 — Network       (Red)          │  ← IP, enrutamiento
├─────────────────────────────────────────┤
│  Layer 2 — Data Link     (Enlace)       │  ← MAC, switches
├─────────────────────────────────────────┤
│  Layer 1 — Physical      (Física)       │  ← Cables, señales, bits
└─────────────────────────────────────────┘
         Los datos bajan al enviar ↓
         Los datos suben al recibir ↑
```

---

## Layer 1 — Physical (Física)

**¿Qué hace?** Transmite los datos como **señales eléctricas, ópticas o de radio** a través del medio físico.

- Trabaja con **bits** (0s y 1s)
- No interpreta los datos, solo los transporta
- Incluye el hardware de transmisión: cables, conectores, señales

**Dispositivos de esta capa:**

| Dispositivo | Función |
|-------------|---------|
| Hub | Repite señales a todos los puertos (sin inteligencia) |
| Repetidor | Amplifica la señal para cubrir mayor distancia |
| Cables (Ethernet, Fibra) | El medio físico de transmisión |
| NIC (Network Interface Card) | Tarjeta de red del dispositivo |

**Medios de transmisión:**

| Medio | Velocidad | Uso típico |
|-------|-----------|------------|
| Cable de cobre (UTP/STP) | Hasta 10 Gbps | Redes de oficina |
| Fibra óptica | Hasta 100 Gbps+ | Backbone, larga distancia |
| Wi-Fi (ondas de radio) | Variable | Conexiones inalámbricas |

> [!info] Dato clave
> En esta capa **no existe el concepto de dirección** (ni IP ni MAC). Solo hay señales brutas.

---

## Layer 2 — Data Link (Enlace de Datos)

**¿Qué hace?** Se encarga de la **transmisión confiable entre dos dispositivos en la misma red local**. Añade las **direcciones MAC** a los datos.

- Los datos en esta capa se llaman **Frames (tramas)**
- Detecta y corrige errores que ocurren en la capa física
- Controla el acceso al medio (quién puede transmitir y cuándo)

**Dispositivos de esta capa:**

| Dispositivo | Función |
|-------------|---------|
| Switch | Envía frames al dispositivo correcto usando la tabla MAC |
| Bridge | Une dos segmentos de red |
| NIC | Genera y procesa los frames |

**Subcapas del Data Link:**

| Subcapa | Nombre | Función |
|---------|--------|---------|
| LLC | Logical Link Control | Control de flujo y errores |
| MAC | Media Access Control | Direccionamiento y acceso al medio |

> [!info] Dato clave
> Aquí aparece la **dirección MAC** como identificador. El [[Protocolos-ARP-DHCP#ARP — Address Resolution Protocol|protocolo ARP]] opera entre esta capa y la de red para traducir IPs a MACs.

---

## Layer 3 — Network (Red)

**¿Qué hace?** Se encarga del **enrutamiento** — decidir el mejor camino para que los datos lleguen de una red a otra.

- Los datos en esta capa se llaman **Packets (paquetes)**
- Usa **direcciones IP** para identificar origen y destino
- Los routers operan en esta capa

**Dispositivos de esta capa:**

| Dispositivo | Función |
|-------------|---------|
| Router | Decide la ruta óptima entre redes |
| Firewall (L3) | Filtra tráfico por IP de origen/destino |

**Protocolos importantes:**

| Protocolo | Función |
|-----------|---------|
| IP (IPv4 / IPv6) | Direccionamiento lógico |
| ICMP | Mensajes de error y diagnóstico (ping) |
| OSPF / BGP | Protocolos de enrutamiento dinámico |

> [!info] Dato clave
> Esta capa **no garantiza** que los paquetes lleguen en orden ni que lleguen. Eso lo maneja la capa 4 (Transport).

---

## Layer 4 — Transport (Transporte)

**¿Qué hace?** Controla **cómo se transmiten los datos** de extremo a extremo. Elige entre **fiabilidad (TCP)** o **velocidad (UDP)**.

Los datos se dividen en **Segments (segmentos)**.

---

### TCP — Transmission Control Protocol

Diseñado con **fiabilidad y garantía**. Establece una conexión antes de enviar cualquier dato.

**Handshake de 3 pasos (Three-Way Handshake):**

```
CLIENTE          SERVIDOR
   |── SYN ──────────→ |   "¿Podemos conectar?"
   |← ── SYN-ACK ─────|   "Sí, adelante"
   |── ACK ──────────→ |   "Perfecto, empezamos"
   |                   |
   |===== DATOS ======|
```

**Ventajas y desventajas de TCP:**

| ✅ Ventajas | ❌ Desventajas |
|------------|---------------|
| Garantiza la precisión de los datos | Requiere conexión estable — si falta un chunk, nada llega |
| Sincroniza dispositivos para evitar saturación | Conexión lenta puede bloquear al receptor |
| Realiza verificación de errores | Es significativamente más lento que UDP |

**¿Cuándo se usa TCP?**
- 📧 Email (SMTP, IMAP, POP3)
- 🌐 Navegación web (HTTP/HTTPS)
- 📁 Transferencia de archivos (FTP, SFTP)

---

### UDP — User Datagram Protocol

**No establece conexión**. Envía los datos y no verifica si llegaron. Filosofía: *"best effort"*.

**Ventajas y desventajas de UDP:**

| ✅ Ventajas | ❌ Desventajas |
|------------|---------------|
| Mucho más rápido que TCP | No verifica si los datos fueron recibidos |
| No reserva conexión continua | Conexiones inestables dan mala experiencia |
| Flexible para los desarrolladores | Sin control de orden de llegada |

**¿Cuándo se usa UDP?**
- 🎮 Videojuegos online (latencia mínima > precisión)
- 📹 Streaming de video/audio (Zoom, YouTube)
- 📡 Descubrimiento de dispositivos ([[Protocolos-ARP-DHCP#ARP|ARP]], [[Protocolos-ARP-DHCP#DHCP|DHCP]])
- 🌐 DNS (consultas rápidas de resolución de nombres)

---

### TCP vs UDP — Comparación rápida

| Característica | TCP | UDP |
|----------------|-----|-----|
| Conexión | ✅ Orientado a conexión | ❌ Sin conexión |
| Fiabilidad | ✅ Garantiza entrega | ❌ No garantiza nada |
| Orden | ✅ Datos en orden | ❌ Pueden llegar desordenados |
| Velocidad | 🐢 Más lento | 🐇 Más rápido |
| Uso | Email, web, archivos | Streaming, gaming, DNS |

---

## Layer 5 — Session (Sesión)

**¿Qué hace?** Crea, gestiona y cierra las **sesiones** de comunicación entre dispositivos.

- Una **sesión** se crea cuando se establece una conexión exitosa
- La sesión se mantiene activa mientras dure la comunicación
- Cierra la conexión automáticamente si está inactiva demasiado tiempo o si se pierde

**Características clave:**

| Característica | Descripción |
|----------------|-------------|
| Checkpoints | Si se pierde la conexión, solo se reenvían los datos desde el último checkpoint |
| Unicidad | Los datos de una sesión **no pueden** viajar por otra sesión |
| Sincronización | Coordina el diálogo entre aplicaciones |

> [!example] Analogía
> Imagina una llamada telefónica. La "sesión" empieza cuando alguien contesta y termina cuando alguien cuelga. Si la llamada se cae, no empiezas desde el principio de la conversación gracias a los checkpoints.

> [!info] Términos clave
> - **Session** = conexión establecida activa
> - La sesión se crea al conectarse y se destruye al desconectarse o por timeout

---

## Layer 6 — Presentation (Presentación)

**¿Qué hace?** Es el **traductor** del modelo OSI. Estandariza el formato de los datos para que cualquier software pueda entenderlos, sin importar cómo fue escrito.

- Traduce datos entre la capa de aplicación y el resto de la red
- Garantiza que los datos se vean igual en el receptor aunque use otro software

**Funciones principales:**

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| Traducción | Convierte datos a un formato común | ASCII ↔ Unicode |
| Cifrado / Descifrado | Protege los datos en tránsito | TLS/HTTPS |
| Compresión | Reduce el tamaño de los datos | gzip, JPEG, MP3 |

> [!example] Ejemplo real
> Tú usas Gmail y tu amigo usa Outlook. Los datos del email se formatean en esta capa para que ambos clientes puedan leer el mismo contenido correctamente.

> [!warning] Seguridad
> El **cifrado HTTPS** (TLS) ocurre en esta capa. Es por eso que ver 🔒 en tu navegador significa que los datos van cifrados desde aquí.

---

## Layer 7 — Application (Aplicación)

**¿Qué hace?** Es la capa con la que **interactúas directamente**. Define los protocolos y reglas para que el usuario acceda a los datos de red.

- Proporciona la interfaz entre el software del usuario y la red
- Aquí operan los protocolos que conoces en el día a día

**Protocolos de esta capa:**

| Protocolo | Puerto | Función |
|-----------|--------|---------|
| HTTP | 80 | Navegación web (sin cifrar) |
| HTTPS | 443 | Navegación web (cifrada) |
| FTP | 21 | Transferencia de archivos |
| SMTP | 25 | Envío de correos |
| DNS | 53 | Resolución de nombres de dominio |
| SSH | 22 | Acceso remoto seguro |
| IMAP/POP3 | 143/110 | Recepción de correos |

**Interfaz con el usuario:**
Los usuarios interactúan con esta capa a través de una **GUI (Graphical User Interface)** — navegadores, clientes de email, exploradores de archivos como FileZilla.

> [!info] Dato clave
> El **DNS** (Domain Name System) opera aquí: traduce nombres como `google.com` en direcciones IP como `142.250.80.46`.

---

## Resumen Total: Los 7 Layers

| # | Nombre | Unidad de datos | Dispositivos | Protocolos clave |
|---|--------|-----------------|--------------|-----------------|
| 7 | Application | Datos | — | HTTP, DNS, FTP, SMTP |
| 6 | Presentation | Datos | — | TLS/SSL, JPEG, ASCII |
| 5 | Session | Datos | — | NetBIOS, RPC |
| 4 | Transport | Segmentos | Firewall (L4) | TCP, UDP |
| 3 | Network | Paquetes | Router | IP, ICMP, OSPF |
| 2 | Data Link | Frames | Switch, Bridge | Ethernet, ARP, MAC |
| 1 | Physical | Bits | Hub, Cables, NIC | — |

---

## Flujo de datos: enviar un email

```
ENVIAR                              RECIBIR
  │                                    │
  │ [7] Application  → Email client    │
  │ [6] Presentation → Cifrar (TLS)    │
  │ [5] Session      → Crear sesión    │
  │ [4] Transport    → Segmentar (TCP) │
  │ [3] Network      → Añadir IP       │
  │ [2] Data Link    → Añadir MAC      │
  │ [1] Physical     → Enviar bits ──→ │ [1] Physical → Recibir bits
  │                                    │ [2] Data Link → Quitar MAC
  │                                    │ [3] Network   → Quitar IP
  │                                    │ [4] Transport → Reensamblar
  │                                    │ [5] Session   → Mantener sesión
  │                                    │ [6] Presentation → Descifrar
  │                                    │ [7] Application → Leer email ✅
```

---

## Notas relacionadas

- [[Redes-Fundamentos]] — IP y MAC (capas 3 y 2)
- [[Protocolos-ARP-DHCP]] — ARP (capa 2/3), DHCP (capa 7), ICMP (capa 3)
- [[Subnetting]] — Direccionamiento IP (capa 3)
- [[Roles-Ciberseguridad]] — Un analista SOC analiza tráfico en todas estas capas
