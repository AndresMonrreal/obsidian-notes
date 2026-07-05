---
tags:
  - pre-security
  - networks
  - subnetting
  - subnet-mask
  - CIDR
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔢 Subnetting (Subneteado)

## ¿Qué es?

**Subnetting** es el proceso de **dividir una red grande en redes más pequeñas** (subredes). Permite organizar mejor los dispositivos, mejorar el rendimiento y aislar segmentos por seguridad.

> [!example] Analogía
> Imagina un edificio de oficinas. En vez de que todos estén en un solo piso enorme, divides el edificio en **departamentos** (subredes). Contabilidad no necesita acceder directamente a los servidores de IT.

---

## Subnet Mask (Máscara de Subred)

- Tiene exactamente **32 bits** (igual que una IPv4)
- Se escribe en el mismo formato de 4 octetos: `255.255.255.0`
- Define **cuánta parte de la IP es la red** y **cuánta es para hosts**

### Cómo leerla

```
IP:          192  . 168  .  1  .  50
Subnet Mask: 255  . 255  . 255 .   0
             ─────────────────   ────
               PARTE DE RED     PARTE HOST
```

Los octetos en **255** = pertenecen a la red (fijos)
El octeto en **0** = disponible para hosts

---

## Notación CIDR

En vez de escribir la máscara completa, se usa la notación **CIDR** (`/n`) que indica cuántos bits son de red:

| Subnet Mask | CIDR | Hosts disponibles |
|-------------|------|-------------------|
| 255.0.0.0 | /8 | 16,777,214 |
| 255.255.0.0 | /16 | 65,534 |
| 255.255.255.0 | /24 | 254 |
| 255.255.255.128 | /25 | 126 |
| 255.255.255.192 | /26 | 62 |
| 255.255.255.224 | /27 | 30 |

> [!info] Fórmula
> Hosts disponibles = **2^(32 - CIDR) - 2**
> Se restan 2 porque una IP es la **Network Address** y otra es el **Broadcast**.

---

## Las 3 Direcciones Especiales de una Subred

Para la red `192.168.1.0/24`:

### 1. Network Address (Dirección de Red)

```
192.168.1.0
```

- **Identifica la subred** como tal
- **No se asigna** a ningún dispositivo
- Es siempre la **primera IP** del rango

### 2. Host Address (Rango de Hosts)

```
192.168.1.1  →  192.168.1.254
```

- Todas las IPs **entre** la Network Address y el Broadcast
- Se asignan a dispositivos (PCs, impresoras, servidores, etc.)
- En este ejemplo: **254 hosts disponibles**

### 3. Default Gateway

```
Usualmente: 192.168.1.1  (convención, no regla)
```

- Dirección del **router** que conecta esta red con otras
- Es el "portero" que decide si el tráfico sale de la subred
- Configurado automáticamente por [[Protocolos-ARP-DHCP#DHCP|DHCP]]

### 4. Broadcast Address

```
192.168.1.255
```

- Un paquete enviado aquí llega a **todos** los dispositivos de la subred
- **No se asigna** a ningún dispositivo
- Es siempre la **última IP** del rango

---

## Ejemplo completo: `192.168.10.0/24`

| Dirección | Valor | Descripción |
|-----------|-------|-------------|
| Network Address | 192.168.10.**0** | Identifica la red |
| Primer host | 192.168.10.**1** | Usualmente el Default Gateway |
| Último host | 192.168.10.**254** | Último dispositivo asignable |
| Broadcast | 192.168.10.**255** | Envío masivo a todos |
| Total hosts | **254** | (256 - 2) |

---

## ¿Por qué importa en ciberseguridad?

> [!warning] Relevancia para el Blue Team
> - Entender subnetting te permite **identificar si una IP pertenece a tu red o es externa**
> - La segmentación de red (**VLANs y subredes**) es una técnica defensiva clave para **contener brechas**
> - En análisis de logs, saber el rango de una subred te dice si el tráfico es interno o potencialmente malicioso

---

## Comandos útiles

```bash
# Ver tu IP y máscara (Linux)
ip addr show
# o
ifconfig

# Ver tu IP y máscara (Windows)
ipconfig /all

# Ver tu default gateway (Linux)
ip route show
ip route | grep default
```

---

## Notas relacionadas

- [[Redes-Fundamentos]] — Conceptos base de IP y redes
- [[Protocolos-ARP-DHCP]] — DHCP entrega la subnet mask automáticamente
- [[Roles-Ciberseguridad]] — Segmentación como técnica defensiva
