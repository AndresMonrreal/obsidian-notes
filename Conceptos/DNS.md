---
tags:
  - pre-security
  - networks
  - DNS
  - web
  - protocolos
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🌐 DNS — Domain Name System

## ¿Qué es?

DNS traduce nombres de dominio legibles por humanos en direcciones IP que usan las máquinas.

```
tryhackme.com  ──── DNS ────→  104.26.10.229
   (fácil de recordar)         (lo que usa la red)
```

> [!info] Analogía
> DNS es como la libreta de contactos de Internet. En vez de marcar un número de 12 dígitos, marcas un nombre y el teléfono busca el número por ti.

---

## Estructura de un Dominio

```
jupiter.servers.tryhackme.com
   │        │        │      └── TLD (.com)
   │        │        └───────── Second-Level Domain (tryhackme)
   └────────┴────────────────── Subdomains (servers, jupiter)
```

### TLD — Top-Level Domain

La parte más a la **derecha** del dominio.

| Tipo | Significado | Ejemplos |
|------|-------------|---------|
| **gTLD** (Generic) | Propósito del sitio | `.com`, `.org`, `.edu`, `.gov`, `.net` |
| **ccTLD** (Country Code) | País de origen | `.uk`, `.ca`, `.mx`, `.es`, `.co` |
| **Nuevos gTLD** | Temáticos modernos | `.online`, `.club`, `.biz`, `.website` |

### Second-Level Domain

La parte justo a la **izquierda del TLD**. En `tryhackme.com` → `tryhackme`.

- Máximo **63 caracteres** + TLD
- Solo `a-z`, `0-9` y guiones `-`
- No puede empezar/terminar con guión ni tener guiones consecutivos

### Subdomain

Parte a la **izquierda del Second-Level Domain**, separada por punto.

- Mismas reglas que el Second-Level Domain (63 chars, `a-z 0-9 -`)
- Se pueden anidar múltiples: `jupiter.servers.tryhackme.com`
- Límite total del nombre: **253 caracteres**
- Sin límite de subdomains por dominio

---

## Tipos de Registros DNS

| Tipo | Resuelve a | Ejemplo |
|------|-----------|---------|
| **A** | Dirección IPv4 | `tryhackme.com → 104.26.10.229` |
| **AAAA** | Dirección IPv6 | `tryhackme.com → 2606:4700:20::681a:be5` |
| **CNAME** | Otro nombre de dominio | `store.tryhackme.com → shops.shopify.com` |
| **MX** | Servidor de email del dominio | `tryhackme.com → alt1.aspmx.l.google.com` |
| **TXT** | Texto libre | SPF, DMARC, verificación de dominio |

### Detalle de cada tipo

**A Record** — El más común. Apunta un dominio a una IP versión 4.

**AAAA Record** — Igual que A pero para IPv6 (128 bits, el futuro).

**CNAME Record** — Alias. Apunta a otro nombre de dominio en lugar de a una IP. Útil cuando un servicio externo (Shopify, GitHub Pages) aloja tu contenido. Se hacen dos consultas DNS en cadena.

**MX Record** — Define qué servidor recibe el correo del dominio. Tiene un campo **priority** (número): el servidor con **menor número** tiene mayor prioridad. Si el principal cae, el email va al de mayor número (backup).
```
tryhackme.com MX 10 alt1.aspmx.l.google.com   ← prioridad 1
tryhackme.com MX 20 alt2.aspmx.l.google.com   ← backup
```

**TXT Record** — Texto libre con múltiples usos:
```
# Autorizar servidores para enviar email (SPF)
@ TXT "v=spf1 ip4:192.0.2.0/24 include:_spf.google.com ~all"

# Política DMARC (anti-spoofing de email)
_dmarc.example.com TXT "v=DMARC1; p=reject; rua=mailto:reports@example.com"

# Verificar propiedad del dominio (Microsoft, Google, etc.)
@ TXT "MS=ms12345678"
```

> [!warning] TXT Records en ciberseguridad
> Los registros SPF, DKIM y DMARC en TXT son la primera línea de defensa contra **email spoofing** y **phishing**. Un dominio sin ellos es fácil de suplantar.

---

## Flujo de una Consulta DNS

¿Qué pasa cuando escribes `www.tryhackme.com` en el navegador?

```
TU PC
  │
  ├─ 1. Revisa la CACHÉ LOCAL
  │      ¿Ya pregunté esto recientemente? → SI: usa la IP guardada ✅
  │                                          NO: continúa ↓
  │
  ├─ 2. Pregunta al RECURSIVE DNS SERVER (tu ISP o 8.8.8.8)
  │      ¿Tienes esta IP en caché? → SI: devuelve resultado ✅
  │                                   NO: continúa ↓
  │
  ├─ 3. El Recursive DNS pregunta a un ROOT SERVER
  │      "¿Quién maneja .com?"
  │      Root Server → "Ve al TLD Server de .com"
  │
  ├─ 4. Pregunta al TLD SERVER (.com)
  │      "¿Quién es el nameserver de tryhackme.com?"
  │      TLD → "El nameserver es kip.ns.cloudflare.com"
  │
  ├─ 5. Pregunta al AUTHORITATIVE DNS SERVER (nameserver)
  │      "¿Cuál es la IP de www.tryhackme.com?"
  │      Authoritative → "Es 104.26.10.229" ✅
  │
  └─ 6. El Recursive DNS guarda en CACHÉ y devuelve la IP a tu PC
         Tu PC guarda en CACHÉ y se conecta al servidor web
```

### Servidores involucrados

| Servidor | Función |
|----------|---------|
| **Caché local** | Respuestas guardadas en tu PC para no repetir consultas |
| **Recursive DNS Server** | Tu "agente" — pregunta a otros servidores en tu nombre (ISP o `8.8.8.8`, `1.1.1.1`) |
| **Root Server** | La cima de la jerarquía. Sabe dónde está cada TLD Server |
| **TLD Server** | Sabe qué nameserver maneja cada dominio dentro de su TLD |
| **Authoritative DNS Server** | Tiene los registros reales del dominio. Respuesta definitiva |

### TTL en DNS

Cada registro DNS tiene un **TTL (Time To Live)** en segundos:
- Define cuánto tiempo el Recursive DNS puede usar la respuesta en caché
- TTL bajo (60s) → cambios se propagan rápido, más consultas al servidor
- TTL alto (86400s = 24h) → menos carga, pero cambios tardan en propagarse

> [!tip] En pentesting
> Herramientas como `nslookup`, `dig` y `host` permiten consultar registros DNS manualmente. Enumerar registros DNS es uno de los primeros pasos en **reconocimiento** de un objetivo.
> ```bash
> nslookup tryhackme.com
> dig tryhackme.com MX
> dig tryhackme.com TXT
> ```

---

## Notas relacionadas

- [[HTTP-Web]] — DNS resuelve el dominio antes de que salga el request HTTP
- [[TCP-UDP-Puertos]] — DNS usa el puerto 53 (UDP para consultas, TCP para transferencias de zona)
- [[Seguridad-de-Red]] — DNS sobre VPN, DNS poisoning como vector de ataque
- [[Protocolos-ARP-DHCP]] — DHCP entrega la dirección del DNS Server automáticamente
