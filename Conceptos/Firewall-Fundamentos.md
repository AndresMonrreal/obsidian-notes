# Firewall — Fundamentos

Relacionado: [[Seguridad-de-Red]] · [[OSI-Model]] · [[TCP-UDP-Puertos]] · [[IDS-Snort]] · [[Linux-Herramientas-y-Admin]] · [[Windows-PowerShell]]

## Qué es
Un firewall inspecciona el tráfico **entrante y saliente** de una red o dispositivo y lo permite o deniega según **reglas**. Es el "guardia de seguridad" en la frontera de la red. Todo lo que entra/sale pasa primero por él.

## Tipos de firewall (y capa OSI)
| Tipo | Capa OSI | Características |
|------|----------|----------------|
| **Stateless** | L3–L4 | filtra por reglas fijas, **sin recordar** conexiones previas; rápido pero sin políticas complejas |
| **Stateful** | L3–L4 | mantiene una **state table** de conexiones; decide según el historial |
| **Proxy** (application-level gateway) | L7 | intermediario; **inspecciona el contenido** de los paquetes; enmascara IPs internas; content filtering; descifra SSL/TLS |
| **Next-Generation (NGFW)** | L3–L7 | deep packet inspection, **IPS** integrado, análisis heurístico, descifrado SSL/TLS + threat intel |

- **Stateful** = "recognize traffic by patterns", monitorea conexiones.
- **NGFW** = análisis heurístico y protección avanzada.
- **Proxy** = inspecciona el tráfico que llega a una aplicación.

## Componentes de una regla
- **Source address** — IP origen del tráfico.
- **Destination address** — IP destino.
- **Port** — número de puerto.
- **Protocol** — protocolo de la comunicación.
- **Action** — qué hacer con ese tráfico.
- **Direction** — aplica a tráfico entrante o saliente.

### Tipos de acción
| Acción | Efecto |
|--------|--------|
| **Allow** | permite el tráfico |
| **Deny** | bloquea el tráfico |
| **Forward** | redirige el tráfico a otro segmento (firewalls con routing) |

Ejemplos:
```
Allow   192.168.1.0/24  →  Any            TCP  80   Outbound   (permitir HTTP saliente)
Deny    Any             →  192.168.1.0/24 TCP  22   Inbound    (bloquear SSH entrante)
Forward Any             →  192.168.1.8    TCP  80   Inbound    (reenviar HTTP al web server)
```

### Direccionalidad
- **Inbound rules** — tráfico entrante (ej. permitir HTTP 80 al web server).
- **Outbound rules** — tráfico saliente (ej. bloquear SMTP 25 salvo el mail server).
- **Forward rules** — reenvío interno.

## Windows Defender Firewall
Firewall integrado de Windows. Abrir: buscar "Windows Defender Firewall".

### Network Profiles (NLA elige el perfil)
- **Private**: red de casa (confiable).
- **Guest/Public**: redes no confiables (cafés, etc.) → suele bloquear todo lo entrante.

### Crear regla personalizada (bloquear HTTP/HTTPS saliente)
`Advanced Settings` → **Outbound Rules** → **New Rule**:
1. Tipo: **Custom** → Next.
2. Programas: **All programs** → Next.
3. Protocolo: **TCP**; Remote port: **Specific ports** → `80,443` (separados por coma, **sin espacios**).
4. Scope: dejar IPs por defecto → Next.
5. Action: **Block the connection** → Next.
6. Profile: dejar los tres marcados → nombre y Finish.
> Tras aplicarla, no se puede navegar (80/443 bloqueados).

## Linux — Netfilter y utilidades
**Netfilter** es el framework del kernel (packet filtering, NAT, connection tracking). Utilidades encima:
| Utilidad | Nota |
|----------|------|
| **iptables** | la más usada tradicionalmente |
| **nftables** | **sucesor de iptables**, filtrado y NAT mejorados |
| **firewalld** | usa zonas pre-configuradas |
| **ufw** | *Uncomplicated Firewall*, interfaz fácil que configura iptables por detrás |

## Comandos ufw (Uncomplicated Firewall)
```bash
sudo ufw status                    # ver estado (inactive/active)
sudo ufw enable                    # activar (y en el arranque)
sudo ufw disable                   # desactivar

sudo ufw default allow outgoing    # política por defecto: permitir salida
sudo ufw default deny outgoing     # política por defecto: denegar salida
sudo ufw deny 22/tcp               # bloquear SSH entrante (acción + puerto/protocolo)

sudo ufw status numbered           # listar reglas numeradas
sudo ufw delete 2                  # borrar la regla nº 2
```

Ejemplo de salida numerada:
```
[ 1] 22/tcp        DENY IN     Anywhere
[ 2] 22/tcp (v6)   DENY IN     Anywhere (v6)
```
