---
tags:
  - pre-security
  - networks
  - packets
  - frames
  - encapsulation
  - IP-headers
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 📦 Packets y Frames

## ¿Qué son?

Son piezas pequeñas de datos que, al unirse, forman un mensaje o archivo completo. Existen porque enviar datos en trozos pequeños es más eficiente: menor riesgo de cuellos de botella y mejor uso del ancho de banda.

> [!example] Analogía del correo
> Enviar datos es como mandar una carta por correo:
> - El **sobre** = **Frame** → mueve el contenido de un lugar a otro (usa MACs)
> - La **carta dentro** = **Packet** → contiene la información real (usa IPs)
> Cuando el destinatario abre el sobre (frame), sabe cómo reenviar la carta (packet).

---

## Packet vs Frame — La diferencia clave

| | **Packet** | **Frame** |
|-|-----------|-----------|
| **Capa OSI** | Layer 3 — Network | Layer 2 — Data Link |
| **Identificador** | Dirección IP | Dirección MAC |
| **Contiene** | IP header + payload (datos) | Encapsula el packet + añade MACs |
| **Relación** | Va dentro del Frame | Envuelve al Packet |

> [!info] Regla rápida
> - ¿Hablas de **IP addresses**? → Estás hablando de **Packets**
> - ¿Le quitaron la info de encapsulamiento? → Estás hablando del **Frame**

---

## Encapsulamiento — Cómo se construye un Packet/Frame

Al bajar por las capas del [[OSI-Model]], cada capa **añade su propia cabecera (header)**:

```
[Datos de usuario]
      ↓  Layer 7-5: Datos de aplicación
[Datos de aplicación]
      ↓  Layer 4: Se añade header TCP/UDP → Segmento
[Segmento TCP/UDP]
      ↓  Layer 3: Se añade header IP → PACKET ← aquí nace el "packet"
[Packet IP]
      ↓  Layer 2: Se añade header Ethernet (MAC) → FRAME ← aquí nace el "frame"
[Frame Ethernet]
      ↓  Layer 1: Se envía como bits por el cable
```

Al llegar al destino, el proceso se **invierte (desencapsulamiento)**: cada capa quita su header hasta llegar a los datos originales.

---

## Headers de un Packet IP

Un packet usando el **Internet Protocol (IP)** tiene un conjunto de headers con información adicional:

| Header | Descripción |
|--------|-------------|
| **Time to Live (TTL)** | Contador de saltos (hops). Se reduce en 1 en cada router. Cuando llega a 0, el packet se descarta. Evita que packets perdidos circulen indefinidamente. |
| **Checksum** | Valor de integridad. Si cualquier dato cambia en tránsito, el checksum no coincidirá y el packet se considera **corrupto**. |
| **Source Address** | IP del dispositivo que **envía** el packet. Para que el destino sepa a dónde responder. |
| **Destination Address** | IP del dispositivo al que va **dirigido** el packet. Para que los routers sepan hacia dónde dirigirlo. |
| **Protocol** | Indica qué protocolo de capa 4 lleva (TCP = 6, UDP = 17, ICMP = 1). |
| **Header Length** | Tamaño del header IP en bytes. |
| **Version** | IPv4 o IPv6. |

### Diagrama del Packet IP

```
┌──────────────────────────────────────────┐
│  IP Header                               │
│  ┌──────────┬──────────┬───────────────┐ │
│  │ Version  │  TTL     │   Protocol    │ │
│  ├──────────┴──────────┴───────────────┤ │
│  │         Source IP Address           │ │
│  ├─────────────────────────────────────┤ │
│  │       Destination IP Address        │ │
│  ├─────────────────────────────────────┤ │
│  │            Checksum                 │ │
│  └─────────────────────────────────────┘ │
│  Payload (Datos: segmento TCP/UDP)       │
└──────────────────────────────────────────┘
```

---

## ¿Por qué dividir en packets?

Si se enviara toda la imagen/archivo de golpe:
- Una sola interrupción = **perder todo**
- Monopoliza el canal → **cuello de botella**

Con packets:
- Solo se reenvían los packets perdidos, no todo
- Múltiples dispositivos pueden **compartir el canal** simultáneamente
- El receptor **reensambla** los packets en orden usando los números de secuencia

> [!example] Ejemplo real
> Al cargar una imagen desde una web, llega en decenas o cientos de packets pequeños. Tu navegador los reensambla para mostrarte la imagen completa.

---

## TTL — Time to Live en la práctica

El TTL es muy útil en ciberseguridad para identificar el sistema operativo de un host:

| Sistema Operativo | TTL inicial típico |
|------------------|--------------------|
| Windows | 128 |
| Linux / Unix | 64 |
| Cisco / Routers | 255 |

Cuando haces `ping` a un host y ves el TTL en la respuesta, puedes estimar qué SO corre.

---

## Notas relacionadas

- [[OSI-Model]] — Layers donde viven packets (L3) y frames (L2)
- [[TCP-UDP-Puertos]] — Headers específicos de TCP y UDP dentro de los packets
- [[Protocolos-ARP-DHCP]] — ARP trabaja con frames (L2) para resolver IPs a MACs
- [[Redes-Fundamentos]] — Conceptos base de IP y MAC
