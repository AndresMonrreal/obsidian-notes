---
tags:
  - pre-security
  - networks
  - IP
  - MAC
  - topologias
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🌐 Fundamentos de Redes

## Identificadores de Dispositivos

Todo dispositivo en una red necesita **dos identificadores** para comunicarse:

### Dirección IP (Internet Protocol)

- Dirección **lógica** asignada dinámicamente
- Puede **cambiar** al conectarte a otra red
- Asignada por [[Protocolos-ARP-DHCP#DHCP|DHCP]] automáticamente

#### IPv4

```
192  .  168  .  1  .  1
 ^        ^      ^    ^
Oct 1   Oct 2  Oct 3  Oct 4
```

> [!info] Octeto (Octet)
> Cada una de las 4 secciones de una IP se llama **octeto** — representa **8 bits**. Rango por octeto: **0 – 255**.

**Tipos de IP:**

| Tipo | Rango | Uso |
|------|-------|-----|
| Pública | Asignada por ISP | Identificarte en Internet |
| Privada (A) | 10.0.0.0 – 10.255.255.255 | Redes internas grandes |
| Privada (B) | 172.16.0.0 – 172.31.255.255 | Redes medianas |
| Privada (C) | 192.168.0.0 – 192.168.255.255 | Redes domésticas |
| Loopback | 127.0.0.1 | Probar tu propia máquina |

---

## IP Pública vs IP Privada — Diferencias Clave

Esta distinción es fundamental en redes y en ciberseguridad.

### IP Pública

- Asignada por tu **ISP (proveedor de internet)**
- **Única en internet** — ningún otro dispositivo tiene la misma IP en el mundo simultáneamente
- Permite que tu red sea alcanzada **desde internet**
- Análogo a tu **dirección postal real** — cualquiera puede enviarte algo si la conoce
- Visible desde el exterior de tu red

### IP Privada

- Asignada por tu router dentro de tu red local (LAN)
- **No es única globalmente** — millones de dispositivos tienen `192.168.1.1` en sus redes internas sin conflicto
- **No puede comunicarse directamente con internet** — necesita NAT
- Solo válida dentro de su red local
- Análogo a un **número de apartamento dentro de un edificio privado** — solo tiene sentido dentro de ese edificio

```
Internet (IPs públicas)
         │
    [Router / NAT]  ← tiene IP pública (ej: 85.123.45.67)
         │
  Red privada (LAN)
  ├── PC1: 192.168.1.10
  ├── PC2: 192.168.1.11
  └── Móvil: 192.168.1.12
```

### NAT — Network Address Translation

El router convierte IPs privadas → IP pública cuando el tráfico sale a internet, y viceversa cuando regresa. Así, todos los dispositivos de la LAN "comparten" la misma IP pública.

```
PC1 (192.168.1.10) → Router (NAT) → Internet (85.123.45.67)
Internet → Router (NAT) → PC1 (192.168.1.10)
```

> [!tip] ¿Por qué importa en ciberseguridad?
> - Desde internet, **no puedes alcanzar directamente** una IP privada (como 10.1.33.7 o 192.168.1.5)
> - Si ves una IP como `10.x.x.x` o `172.16-31.x.x` en un pentest, es tráfico de red interna — no accesible desde fuera sin VPN/Port Forwarding
> - Los atacantes necesitan explotar el NAT o comprometer primero el router para llegar a dispositivos internos

### RFC 1918 — Los 3 Rangos Privados Oficiales

Definidos por el estándar **RFC 1918**:

| Clase | Rango | Notación CIDR | Hosts disponibles | Uso típico |
|-------|-------|---------------|------------------|------------|
| **A** | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | ~16 millones | Redes corporativas grandes |
| **B** | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | ~1 millón | Redes medianas |
| **C** | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | ~65,000 | Redes domésticas y pequeñas |

**Loopback:** `127.0.0.1` → referencia a la propia máquina (no sale a la red)

> [!warning] Memoriza los rangos privados
> Si ves `10.1.33.7`, `172.31.33.7` o `192.168.5.40` → son IPs privadas. No puedes accederlas directamente desde internet. Intentar hacerlo desde una IP pública no funcionará sin configuración de red especial.

### ¿Cómo ver tu IP? — Comandos

**Windows:**
```cmd
ipconfig              ← IP, máscara de subred, gateway
ipconfig /all         ← también MAC address, DHCP, DNS
```

**Linux:**
```bash
ifconfig              ← muestra configuración de red
ip a s                ← versión moderna de ifconfig (ip address show)
ip addr show          ← equivalente completo
```

**Ejemplo de `ifconfig`:**
```
wlo1: inet 192.168.66.89  netmask 255.255.255.0  broadcast 192.168.66.255
      ether cc:5e:f8:02:21:a7   ← MAC address (ether)
```

**Ejemplo de `ip a s`:**
```
inet 192.168.66.89/24 brd 192.168.66.255   ← /24 = 255.255.255.0
```

> [!info] `255.255.255.0` = `/24`
> La máscara de subred puede escribirse en decimal (255.255.255.0) o en notación CIDR (/24). Los 24 bits más a la izquierda de la IP identifican la red, los 8 restantes identifican el host. Ver [[Subnetting]] para el cálculo completo.

---

### Dirección MAC (Media Access Control)

- Dirección **física**, grabada de fábrica en la **NIC** (Network Interface Card)
- Formato: `AA:BB:CC:DD:EE:FF` (6 pares hexadecimales)
- Los primeros 3 pares identifican al **fabricante (OUI)**
- Los últimos 3 pares son únicos por dispositivo

> [!warning] MAC Spoofing
> Aunque la MAC viene grabada en el hardware, **puede suplantarse por software**. Por eso no es un método de autenticación 100% confiable.

---

## Topologías de Red

La **topología** define cómo están físicamente conectados los dispositivos.

### Bus

```
[PC1]---[PC2]---[PC3]---[PC4]
         (cable backbone)
```

- ✅ **Barata y simple** de implementar
- ❌ Si dos equipos transmiten a la vez → **colisión**
- ❌ Si el cable central falla → **toda la red cae**

---

### Star (Estrella) ⭐ — La más común

```
       [PC1]
         |
[PC4]--[Switch]--[PC2]
         |
       [PC3]
```

- ✅ Fallo en un cable **no afecta** a los demás
- ✅ Fácil de escalar (solo agregar al switch)
- ❌ Si el **switch central** falla → toda la red cae

---

### Mesh (Malla)

```
[PC1]---[PC2]
  |  \ /  |
  |   X   |
  |  / \  |
[PC3]---[PC4]
```

- ✅ **Alta redundancia** — múltiples caminos alternativos
- ✅ Si un nodo cae, el tráfico se redirige
- ❌ **Muy cara** — cableado masivo
- ❌ Difícil de mantener a escala

---

### Comparación rápida

| Topología | Costo | Mantenimiento | Redundancia | Uso típico |
|-----------|-------|---------------|-------------|------------|
| Bus | 💲 Bajo | Fácil | ❌ Ninguna | Redes legacy |
| Star | 💲💲 Medio | Moderado | ⚠️ Parcial | Oficinas, hogares |
| Mesh | 💲💲💲 Alto | Complejo | ✅ Alta | Data centers, ISPs |

---

---

## NAT — Network Address Translation (detalle técnico)

El router con NAT mantiene una **tabla de traducción** que mapea IP:puerto interno → IP:puerto externo.

```
Dispositivo interno          Router NAT (IP pública: 212.3.4.5)
192.168.0.129:15401   ─────→  212.3.4.5:19273   ─────→ Servidor web

Tabla NAT del router:
┌─────────────────────┬──────────────────────┐
│ Interno             │ Externo              │
├─────────────────────┼──────────────────────┤
│ 192.168.0.129:15401 │ 212.3.4.5:19273      │
│ 192.168.0.130:8877  │ 212.3.4.5:19274      │
│ 192.168.0.131:44321 │ 212.3.4.5:19275      │
└─────────────────────┴──────────────────────┘
```

El router hace esta traducción **de forma transparente** — ni el dispositivo interno ni el servidor externo saben que NAT está ocurriendo.

**Beneficios de NAT:**
- Extiende la vida útil de IPv4 (millones de dispositivos con una sola IP pública)
- Oculta la estructura interna de la red (seguridad)
- Ahorra IPs públicas (costosas)

**Limitación de NAT:** la comunicación siempre debe iniciarse desde adentro hacia afuera. Para que un servidor interno sea accesible desde internet se necesita [[Infraestructura-de-Red#Port Forwarding|Port Forwarding]].

---

## Protocolos de Enrutamiento (Routing Protocols)

Los routers necesitan saber cómo llegar a otras redes. Los protocolos de routing permiten que los routers intercambien información y calculen las mejores rutas automáticamente.

```
Internet = millones de routers que se comunican entre sí
           para saber cómo enrutar paquetes

Red 1 ─── Router A ─┬─ Router B ─── Red 2
                    └─ Router C ─── Red 3
                              └─ Router D ─── Red 2 (ruta alternativa)
```

### OSPF — Open Shortest Path First

- **Tipo:** Interior (dentro de una misma organización/AS)
- **Cómo funciona:** Cada router comparte el estado de sus enlaces con todos los demás. Cada router tiene el **mapa completo** de la red y calcula la ruta más eficiente.
- **Uso:** Redes empresariales medianas y grandes.
- **Algoritmo:** Dijkstra (shortest path)

### EIGRP — Enhanced Interior Gateway Routing Protocol

- **Tipo:** Interior · **Propietario de Cisco**
- **Cómo funciona:** Comparte información sobre redes alcanzables y el "costo" (ancho de banda, delay) de cada ruta. Elige la ruta más eficiente.
- **Uso:** Redes Cisco, entornos empresariales con infraestructura Cisco.

### BGP — Border Gateway Protocol

- **Tipo:** Exterior — el protocolo de routing de **internet**
- **Cómo funciona:** Permite que diferentes redes/ISPs intercambien información de routing. Define cómo los datos viajan entre distintas organizaciones a nivel global.
- **Uso:** ISPs, grandes organizaciones, núcleo de internet.
- **Conocido como:** "el protocolo que hace funcionar internet"

### RIP — Routing Information Protocol

- **Tipo:** Interior · Protocolo simple y antiguo
- **Cómo funciona:** Cada router comparte cuántos saltos (hops) necesita para llegar a cada red. Elige la ruta con **menos saltos** (máximo 15 hops).
- **Uso:** Redes pequeñas. En desuso en redes modernas.
- **Limitación:** Solo considera saltos — ignora velocidad o confiabilidad de los enlaces.

### Comparativa rápida

| Protocolo | Ámbito | Propietario | Métrica | Uso típico |
|-----------|--------|-------------|---------|------------|
| **OSPF** | Interior | Open (estándar) | Costo del enlace (ancho de banda) | Empresas medianas/grandes |
| **EIGRP** | Interior | Cisco | Ancho de banda + delay | Redes Cisco |
| **BGP** | Exterior | Open (estándar) | Políticas de red | Internet, ISPs |
| **RIP** | Interior | Open (estándar) | Número de hops (máx 15) | Redes pequeñas, legacy |

> [!tip] Para certificaciones
> - **Network+**: conocer los 4 protocolos, sus diferencias y casos de uso
> - **Security+**: BGP hijacking es un ataque donde se manipulan rutas BGP para redirigir tráfico
> - **CEH/pentesting**: el análisis de rutas (traceroute) ayuda a entender la topología de red del objetivo

---

## Notas relacionadas

- [[Protocolos-ARP-DHCP]] — Cómo los dispositivos obtienen y resuelven sus IPs (DHCP, ARP, ICMP)
- [[Subnetting]] — Cómo dividir redes en subredes (CIDR, /24, /25...)
- [[Infraestructura-de-Red]] — Router, Switch, VLAN, Port Forwarding
- [[Windows-CMD]] — `ipconfig` y `ipconfig /all` para ver IPs en Windows
- [[Protocolos-Aplicacion]] — Tabla completa de puertos; FTP, SMTP, POP3, IMAP
- [[Roles-Ciberseguridad]] — Por qué un analista necesita saber esto
