---
tags:
  - pre-security
  - networks
  - router
  - switch
  - VLAN
  - port-forwarding
  - infraestructura
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🖧 Infraestructura de Red — Router, Switch, VLAN y Port Forwarding

---

## Router

**Función:** Conectar **diferentes redes** entre sí y dirigir los datos por el mejor camino posible.

- Opera en **Layer 3 (Network)** del [[OSI-Model]]
- Trabaja con **[[Packets-and-Frames#Packet vs Frame|Packets]]** (direcciones IP)
- Suele tener una interfaz web o consola para configuración (port forwarding, firewall, etc.)

### Routing — Cómo decide el camino

El **routing** es el proceso de crear la ruta óptima entre redes. Los routers evalúan:

| Factor | Descripción |
|--------|-------------|
| **Distancia** | ¿Cuál es el camino con menos saltos (hops)? |
| **Fiabilidad** | ¿Cuál ruta tiene menor tasa de pérdida de packets? |
| **Velocidad del medio** | ¿Fibra óptica vs cable de cobre? |
| **Carga actual** | ¿Qué enlace está menos saturado? |

```
RED A ──→ [Router 1] ──→ [Router 2] ──→ RED B
               ↓                ↑
           [Router 3] ──────────┘
           (ruta alternativa si la principal falla)
```

Los protocolos de enrutamiento dinámico (OSPF, BGP, RIP) calculan estas rutas automáticamente.

> [!info] Router vs Switch
> - **Router** → une **redes distintas**, trabaja con IPs (Layer 3)
> - **Switch** → conecta **dispositivos en la misma red**, trabaja con MACs (Layer 2)

---

## Port Forwarding

**Función:** Redirigir tráfico entrante de Internet hacia un dispositivo específico dentro de la red local.

Se configura **en el router** de la red.

### El problema que resuelve

Sin port forwarding, un servidor web interno solo es accesible desde la **intranet** (red local):

```
Internet ✗         RED INTERNA
                   [PC-1]
                   [Servidor Web: 192.168.1.10:80]  ← solo accesible internamente
                   [PC-2]
```

### Con port forwarding activado

```
Internet ✅        ROUTER              RED INTERNA
[Usuario externo]──→ 82.62.51.70:80 ──→ 192.168.1.10:80 (Servidor Web)
```

El router recibe tráfico en su IP pública (82.62.51.70) en el puerto 80 y lo **reenvía** al servidor interno.

### Port Forwarding ≠ Firewall

| | Port Forwarding | Firewall |
|-|----------------|---------|
| **Qué hace** | Abre y redirige puertos | Decide si el tráfico puede pasar |
| **Dónde** | Router | Router, dispositivo dedicado o software |
| **Analogía** | Abrir una puerta | El guardia que decide quién entra |

> [!warning] Seguridad
> Port forwarding expone servicios internos a Internet. Cada puerto abierto es una superficie de ataque potencial. Siempre combinar con [[Seguridad-de-Red#Firewall|Firewall]] para filtrar quién puede conectarse.

---

## Switch

**Función:** Conectar múltiples dispositivos **dentro de la misma red** y dirigir el tráfico al dispositivo correcto.

Un switch puede conectar desde 3 hasta 63+ dispositivos mediante cables Ethernet.

### Layer 2 Switch

- Opera en **Layer 2 (Data Link)** del [[OSI-Model]]
- Usa **direcciones MAC** para entregar **Frames** al dispositivo correcto
- Mantiene una **tabla MAC** (CAM table): mapea qué MAC está en qué puerto
- **No puede** enrutar tráfico entre redes distintas

```
[PC-1: MAC AA:BB:CC]──┐
[PC-2: MAC DD:EE:FF]──┤ SWITCH L2 ──→ entrega frames por MAC
[PC-3: MAC 11:22:33]──┘
```

### Layer 3 Switch

- Opera en **Layer 2 Y Layer 3** simultáneamente
- Puede hacer todo lo que hace un L2 switch + **enrutar packets** entre subredes usando IP
- Más sofisticado y costoso que L2
- Permite crear **VLANs** con enrutamiento entre ellas

```
Subred 192.168.1.0/24 ──┐
                          ├── SWITCH L3 ──→ enruta por IP entre subredes
Subred 192.168.2.0/24 ──┘
```

### Switch L2 vs L3

| | Switch L2 | Switch L3 |
|-|-----------|-----------|
| **Capa OSI** | Layer 2 | Layer 2 + Layer 3 |
| **Identificador** | MAC address | MAC + IP address |
| **Unidad de datos** | Frame | Frame + Packet |
| **Enrutamiento** | ❌ No puede | ✅ Sí puede |
| **VLANs** | ✅ Sí (sin enrutamiento entre ellas) | ✅ Sí (con enrutamiento entre ellas) |
| **Costo** | 💲 Menor | 💲💲 Mayor |

> [!info] Switch vs Hub
> Un **Hub** (Layer 1) manda el tráfico a **todos** los puertos sin importar el destino — genera colisiones. Un **Switch** es inteligente: solo envía el frame al puerto donde está el destinatario.

---

## VLAN — Virtual Local Area Network

**Función:** Dividir **virtualmente** una red física en múltiples redes lógicas aisladas dentro del mismo switch.

### ¿Por qué usar VLANs?

Sin VLAN, todos los dispositivos conectados al mismo switch pueden comunicarse entre sí, lo que es un riesgo de seguridad.

Con VLAN, aunque todos compartan el mismo switch físico, están **segmentados lógicamente**:

```
                    ┌─────────────────────┐
[PC Ventas 1] ──────┤                     ├── VLAN 10: Ventas
[PC Ventas 2] ──────┤    SWITCH L3        │   (192.168.10.0/24)
                    │                     │
[PC Contab. 1] ─────┤                     ├── VLAN 20: Contabilidad
[PC Contab. 2] ─────┤                     │   (192.168.20.0/24)
                    └─────────────────────┘
         Ventas y Contabilidad NO se pueden comunicar entre sí
         pero ambas tienen acceso a Internet
```

### Beneficios de VLANs

| Beneficio | Descripción |
|-----------|-------------|
| **Seguridad** | Departamentos aislados — si un dispositivo es comprometido, no alcanza a otros |
| **Rendimiento** | Reduce el broadcast domain (menos tráfico innecesario) |
| **Flexibilidad** | Reorganizar la red sin mover cables físicos |
| **Gestión** | Aplicar políticas distintas por departamento |

> [!warning] Relevancia en ciberseguridad
> La segmentación con VLANs es una técnica defensiva clave (**defense in depth**). Si un atacante compromete un dispositivo en la VLAN de usuarios, no puede alcanzar directamente los servidores en otra VLAN.

---

## Resumen: ¿Qué dispositivo uso para qué?

| Necesidad | Dispositivo | Capa OSI |
|-----------|-------------|----------|
| Conectar dispositivos en la misma red | Switch L2 | Layer 2 |
| Segmentar la red con VLANs | Switch L3 | Layer 2/3 |
| Conectar redes distintas | Router | Layer 3 |
| Exponer un servicio interno a Internet | Port Forwarding en Router | Layer 3/4 |
| Controlar quién puede entrar/salir | Firewall | Layer 3/4 |
| Conectar redes de forma segura por Internet | VPN | Layer 3 |

---

## Notas relacionadas

- [[OSI-Model]] — Contexto de layers donde opera cada dispositivo
- [[Redes-Fundamentos]] — IP, MAC y topologías base
- [[Subnetting]] — Las subredes que los routers y VLANs manejan
- [[TCP-UDP-Puertos]] — Los puertos que el port forwarding redirige
- [[Seguridad-de-Red]] — Firewall y VPN como complemento a esta infraestructura
