---
tags:
  - pre-security
  - networks
  - firewall
  - VPN
  - seguridad
  - cifrado
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔒 Seguridad de Red — Firewall y VPN

---

## Firewall

**Función:** Dispositivo (o software) que controla qué tráfico puede **entrar y salir** de una red, basándose en reglas configuradas por el administrador.

> [!summary] Analogía
> El firewall es la **seguridad de fronteras** de una red. Inspecciona cada "vehículo" (packet) antes de dejarlo pasar.

### ¿Qué puede filtrar un firewall?

| Criterio de filtrado | Ejemplo de regla |
|---------------------|------------------|
| **Origen del tráfico** | "Bloquear todo tráfico de la IP 45.33.32.156" |
| **Destino del tráfico** | "Solo permitir tráfico hacia 192.168.1.10" |
| **Puerto** | "Solo permitir tráfico en el puerto 443 (HTTPS)" |
| **Protocolo** | "Bloquear todo UDP, solo permitir TCP" |
| **Estado de la conexión** | "Solo permitir tráfico de conexiones ya establecidas" |

Esto se decide mediante **packet inspection** — el firewall analiza los headers de cada packet.

### Formatos de Firewall

| Tipo | Descripción | Usado en |
|------|-------------|----------|
| **Hardware dedicado** | Dispositivo físico de alto rendimiento | Empresas y data centers |
| **Router con firewall** | Router doméstico con funciones básicas de filtrado | Hogares y PYMEs |
| **Software (host-based)** | Programa en el propio dispositivo | PCs individuales |

Ejemplos de software firewall: **Windows Defender Firewall**, **iptables** (Linux), **Snort**, **pfSense**.

---

### Stateful vs Stateless — Las 2 Categorías Principales

#### Stateful Firewall

- Analiza la **conexión completa**, no solo packets individuales
- Mantiene una **tabla de estados** de todas las conexiones activas
- Decisiones **dinámicas**: puede permitir el inicio del [[TCP-UDP-Puertos#Three-Way Handshake|handshake TCP]] pero bloquear si algo falla en pasos siguientes
- Si detecta comportamiento malicioso → **bloquea el dispositivo entero**
- **Consume más recursos** (CPU y RAM) por el seguimiento del estado

```
Conexión entrante:
  SYN → [Firewall registra: "Nueva conexión de IP X al puerto 80"]
  SYN/ACK → [Firewall verifica: "¿Esta respuesta corresponde a una conexión válida?"]
  ACK → [Firewall decide si la conexión continúa o se bloquea]
```

#### Stateless Firewall

- Evalúa cada **packet de forma independiente** con reglas estáticas
- Si el packet no viola ninguna regla → pasa. Si viola una → se descarta
- Un packet malo **no bloquea al dispositivo** — los siguientes packets se evalúan por separado
- **Consume menos recursos** — ideal para alto volumen de tráfico
- Menos inteligente: solo es tan bueno como las reglas definidas. Si una amenaza no tiene una regla que la cubra, pasa sin problema

```
Packet de IP maliciosa:
  Packet 1: BLOQUEADO (IP está en lista negra)
  Packet 2 del mismo host: evaluado de nuevo desde cero
```

#### Comparación

| | Stateful | Stateless |
|-|----------|-----------|
| **Analiza** | Conexión completa | Packet individual |
| **Decisiones** | Dinámicas | Estáticas (reglas fijas) |
| **Recursos** | 🔴 Alto consumo | 🟢 Bajo consumo |
| **Inteligencia** | Alta | Limitada a las reglas |
| **Bloqueo** | Todo el dispositivo | Solo el packet específico |
| **Ideal para** | Protección general de red | DDoS, alto volumen de tráfico |
| **Capas OSI** | Layer 3 & 4 (y hasta L7 con NGFW) | Layer 3 & 4 |

> [!tip] En ciberseguridad ofensiva
> Los atacantes intentan **bypassear firewalls stateless** con técnicas como fragmentar packets o usar puertos permitidos (ej. puerto 80/443). Los firewalls stateful son mucho más difíciles de evadir.

---

## VPN — Virtual Private Network

**Función:** Crear un **túnel cifrado** a través de Internet que conecta dispositivos de diferentes redes como si estuvieran en la misma red privada.

```
[Oficina A]──────────────────────────────[Oficina B]
192.168.1.0/24    INTERNET    192.168.2.0/24
      │          (túnel VPN cifrado)         │
      └─────────────── 🔒 ──────────────────┘
           Red privada virtual (Network #3)
```

Ambas oficinas forman una **red privada compartida** sobre Internet. Los dispositivos de ambas pueden comunicarse como si estuvieran en la misma LAN.

### Beneficios de una VPN

| Beneficio | Descripción |
|-----------|-------------|
| **Conectividad geográfica** | Une oficinas o usuarios en distintas partes del mundo |
| **Privacidad** | El tráfico va cifrado — ni el ISP ni intermediarios pueden leerlo |
| **Anonimato** | Tu IP real queda oculta detrás de la IP del servidor VPN |
| **Seguridad en WiFi público** | Protege tu tráfico en redes abiertas (hoteles, aeropuertos) |
| **Acceso a recursos internos** | Trabajar desde casa accediendo a servidores corporativos |

> [!warning] Límite del anonimato
> Una VPN que **guarda logs** de tu actividad es casi tan mala como no usar VPN. El nivel real de anonimato depende de la política del proveedor.

> [!info] TryHackMe y VPN
> THM usa VPN para conectarte a sus máquinas vulnerables. Así las máquinas no están expuestas en Internet y tu ISP no ve que estás "atacando" IPs externas.

---

### Tipos de VPN por Escenario

#### Site-to-Site VPN (Empresa multi-sede)

Conecta **redes enteras** entre ubicaciones geográficas. Los routers/firewalls en cada sede establecen el túnel VPN:

```
Sede Madrid (192.168.1.0/24)
      │
   [Router VPN] ──────── túnel cifrado ──────── [Router VPN]
                                                        │
                                              Sede Barcelona (192.168.2.0/24)
```

- Todos los dispositivos de cada sede pueden verse mutuamente como si estuvieran en la misma LAN
- Los empleados no necesitan instalar nada — el router maneja el túnel

#### Remote Access VPN (Usuario remoto)

Un **dispositivo individual** (laptop del empleado, teléfono) conecta al VPN server de la empresa:

```
Empleado en casa
  [Laptop con VPN client] ──── túnel cifrado ────→ [VPN Server empresa]
                                                           │
                                                    Red interna (192.168.1.0/24)
                                                    ├── Servidor archivos
                                                    ├── Servidor correo
                                                    └── ERP interno
```

El empleado trabaja desde casa con acceso a todos los recursos internos como si estuviera en la oficina.

#### VPN Comercial (Privacidad personal / Geo-bypass)

El usuario conecta a un VPN server comercial para:
- Ocultar su IP real a los servidores que visita → ven la IP del VPN server
- Circunvalar restricciones geográficas → si el servidor VPN está en Japón, los servicios creen que el usuario está en Japón
- Cifrar tráfico ante el ISP local → el ISP solo ve datos cifrados, no puede saber qué sitios visita

```
Usuario en España
  [VPN client] ──── túnel cifrado ────→ [VPN server en Japón]
                                                │
                                         Servidor Google, Netflix, etc.
                                         (ven IP de Japón, no de España)
```

> [!warning] El ISP y el cifrado VPN
> Con VPN activa, tu ISP solo ve que te conectas al VPN server. No puede ver qué haces dentro del túnel. Sin embargo, el **proveedor del VPN** sí puede ver todo tu tráfico — por eso la política de "no-logs" del proveedor es crucial.

### Tecnologías VPN

#### PPP (Point-to-Point Protocol)

- Base de **autenticación y cifrado** usada por PPTP
- Funciona con una **clave privada + certificado público** (similar a SSH)
- Ambos deben coincidir para conectarse
- **No puede salir de la red por sí solo** — necesita ser encapsulado por PPTP

#### PPTP (Point-to-Point Tunneling Protocol)

- Permite que los datos de PPP **viajen fuera de la red** (le da la capacidad de "tunelizar")
- ✅ Muy fácil de configurar y compatible con casi todos los dispositivos
- ❌ **Cifrado débil** comparado con alternativas modernas
- Considerado obsoleto para usos de alta seguridad

#### IPSec (Internet Protocol Security)

- Cifra datos usando directamente el **marco del protocolo IP**
- ✅ **Cifrado fuerte** — estándar de la industria para VPNs empresariales
- ✅ Soportado en la mayoría de dispositivos modernos
- ❌ **Difícil de configurar** correctamente
- Usado en VPNs site-to-site (oficina a oficina) y en protocolos como IKEv2

#### Comparación de tecnologías VPN

| Tecnología | Cifrado | Facilidad | Uso recomendado |
|------------|---------|-----------|-----------------|
| **PPP** | Básico | — | Base de PPTP, no se usa solo |
| **PPTP** | ❌ Débil | ✅ Muy fácil | Redes domésticas simples (no crítico) |
| **IPSec** | ✅ Fuerte | ❌ Complejo | Empresas, site-to-site |
| **OpenVPN** | ✅ Fuerte | Moderada | Uso general, open source |
| **WireGuard** | ✅ Fuerte | ✅ Fácil | Estándar moderno emergente |

---

### Consideraciones Importantes al usar VPN

**DNS Leak:**
Incluso con VPN activa, las solicitudes DNS pueden salir fuera del túnel (por la conexión normal) revelando qué sitios visitas. Verificar con herramientas como `dnsleaktest.com`. Una VPN correctamente configurada ruteará también el DNS por el túnel.

**Split tunneling vs Full tunnel:**

| Modo | Comportamiento |
|------|---------------|
| **Full tunnel** | Todo el tráfico del dispositivo va por el túnel VPN |
| **Split tunneling** | Solo el tráfico a la red corporativa va por VPN; el resto va directo a internet |

**Legalidad:**
El uso de VPNs varía según el país. En algunos países (China, Rusia, Irán, EAU) el uso de VPNs no autorizadas es ilegal y puede conllevar multas. Siempre verificar las leyes locales antes de usar VPN, especialmente al viajar.

---

## Firewall + VPN + Port Forwarding — Cómo trabajan juntos

```
INTERNET
    │
    ↓
[Router] ← Port Forwarding: redirige puerto 443 al servidor interno
    │
[Firewall] ← Filtra: solo permite HTTPS (443) y VPN (51820)
    │
    ├── [Servidor Web: 192.168.1.10]
    │
    ├── [VPN Server: 192.168.1.20] ← empleados remotos se conectan aquí
    │
    └── [Red interna: 192.168.1.0/24]
```

| Tecnología | Rol en la arquitectura |
|------------|----------------------|
| **Port Forwarding** | Abre la "puerta" en el router para tráfico específico |
| **Firewall** | Decide qué tráfico puede entrar/salir por esa puerta |
| **VPN** | Crea un canal privado cifrado para acceso seguro desde fuera |

---

## Notas relacionadas

- [[OSI-Model]] — Los firewalls operan principalmente en Layer 3 & 4
- [[TCP-UDP-Puertos]] — Los puertos y protocolos que el firewall filtra
- [[Packets-and-Frames]] — Los packets que el firewall inspecciona
- [[Infraestructura-de-Red]] — Router, Switch y Port Forwarding como contexto
- [[Criptografia]] — TLS, IPSec, cifrado asimétrico/simétrico que usa VPN
- [[Protocolos-Aplicacion]] — SSH (puerto 22) como protocolo seguro para acceso remoto; SMTPS/IMAPS sobre TLS
