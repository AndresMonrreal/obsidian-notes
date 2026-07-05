---
tags:
  - pre-security
  - defensiva
  - blue-team
  - SOC
  - DFIR
  - threat-intelligence
  - incident-response
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔵 Seguridad Defensiva y Blue Team

---

## ¿Qué es la Seguridad Defensiva?

La seguridad defensiva se enfoca en **proteger sistemas, detectar amenazas y responder a incidentes**. A diferencia de la ofensiva (que busca vulnerabilidades), la defensiva trabaja para que esas vulnerabilidades no sean explotadas.

> [!example] Analogía de la ciudad
> Imagina la infraestructura de tu cliente como una ciudad amurallada:
> - Los **guardias** = el equipo de seguridad
> - Las **murallas y puertas** = firewalls y controles de acceso
> - Las **cámaras y patrullas** = logs, monitoreo y alertas
> - Los **edificios y habitantes** = servidores, datos y usuarios

---

## Los 5 Pilares de la Seguridad Defensiva

| Pilar | Qué significa | Ejemplo práctico |
|-------|--------------|-----------------|
| **Prevention** | Controles para detener ataques antes de que ocurran | Firewalls, antivirus, parches de seguridad |
| **Detection** | Monitoreo para identificar actividad sospechosa | SIEM, logs, alertas, IDS/IPS |
| **Mitigation** | Acciones para limitar el daño durante un incidente | Bloquear IP, aislar sistema, deshabilitar cuenta |
| **Analysis** | Investigar qué pasó, cómo y qué sistemas afectó | Revisar logs, correlacionar eventos, forensia |
| **Response & Improvement** | Recuperarse y mejorar defensas para evitar que repita | Parchear, actualizar reglas, plan de mejora |

---

## La Mentalidad del Defensor

Para proteger bien, el defensor debe entender al atacante:

| Principio | Descripción |
|-----------|-------------|
| **Threat Anticipation** | Revisar sistemas y preguntar "¿qué pasaría si...?" — imaginar los caminos que usaría un atacante |
| **Attack Awareness** | Los ataques siguen etapas reconocibles. Conocer el MITRE ATT&CK Framework ayuda a anticiparlos |
| **Risk Prioritization** | No todo tiene el mismo riesgo. Los sistemas de alto valor merecen más protección |
| **Continuous Adaptation** | Las amenazas evolucionan — la defensa no es una configuración única, es un proceso continuo |

### La cadena de ataque — Vista del defensor

```
Email malicioso → Workstation comprometida → Robo de credenciales
      → Acceso al servidor de correo → Base de datos sensible

Cada eslabón es una oportunidad de DETECCIÓN y BLOQUEO para el defensor.
```

---

## Infraestructura del Cliente — Lo que proteges

Antes de defender, debes entender qué existe en el entorno. El defensor mapea todos los activos:

| Componente | Propósito | Analogía de ciudad | Amenaza principal |
|-----------|-----------|-------------------|-----------------|
| **Employee Devices** | Donde los usuarios trabajan y acceden a recursos | Casas | Phishing, malware |
| **Web Server** | Aloja el sitio web o la aplicación | Tienda / Edificio público | Ataques web, defacement |
| **Mail Server** | Envía y recibe email | Oficina de correos | Phishing, spam, exfiltración |
| **Firewall** | Controla qué tráfico entra y sale | Puerta de la ciudad | Acceso no autorizado, DDoS |
| **Internet** | Redes externas fuera del control de la organización | Todo fuera de la muralla | Origen de la mayoría de amenazas |

> [!tip] Visibilidad antes de proteger
> No puedes proteger lo que no puedes ver. El primer paso es siempre tener un **inventario completo** de los activos: dispositivos, servidores, aplicaciones, usuarios.

---

## Defensas por Componente

| Componente | Qué puede salir mal | Defensas disponibles |
|-----------|--------------------|--------------------|
| **Employee Devices** | Clic en enlace malicioso, descarga de malware | Antivirus, EDR, actualizaciones regulares, MFA |
| **Web Server** | Ataques web (SQLi, XSS, brute force) | WAF, HTTPS, updates, control de acceso |
| **Mail Server** | Phishing, spam, adjuntos maliciosos | Filtros de spam, escaneo de adjuntos, SPF/DKIM/DMARC |
| **Firewall** | Intentos de intrusión desde internet | Reglas de firewall, bloqueo de IPs conocidas maliciosas |
| **Internet (tráfico entrante)** | Amenazas externas, DDoS, reconocimiento | Rate limiting, IPS, monitoreo de tráfico |

### Defensa en profundidad (Defense in Depth)

```
Internet
   ↓
[Firewall]          ← Primera línea
   ↓
[WAF / IPS]         ← Segunda línea
   ↓
[Web/Mail Server]   ← Tercera línea
   ↓
[Antivirus / EDR]   ← Cuarta línea (endpoint)
   ↓
[Datos cifrados]    ← Última línea
```

Ninguna defensa es perfecta sola. Las capas garantizan que si una falla, las otras aguantan.

---

## Herramientas del Defensor

| Herramienta | Categoría | Función |
|-------------|-----------|---------|
| **SIEM** (Splunk, Elastic) | Detection | Centralizar y correlacionar logs para detectar amenazas |
| **IDS/IPS** (Snort, Suricata) | Detection/Mitigation | Detectar y bloquear tráfico malicioso |
| **EDR** (CrowdStrike, Defender) | Detection/Response | Monitorear endpoints en tiempo real |
| **Firewall** | Prevention | Filtrar tráfico por reglas |
| **Antivirus** | Prevention/Detection | Detectar y eliminar malware |
| **WAF** | Prevention | Filtrar ataques a aplicaciones web (ver [[Arquitectura-Web]]) |
| **VPN** | Prevention | Cifrar comunicaciones y controlar acceso (ver [[Seguridad-de-Red]]) |

---

## Terminología Clave del Blue Team

| Término | Definición |
|---------|-----------|
| **Blue Team** | Equipo de defensores responsables de proteger sistemas y responder a amenazas |
| **Client Infrastructure** | Redes, servidores, dispositivos y aplicaciones de una organización que necesitan protección |
| **Visibility** | Capacidad de ver y monitorear actividad en los sistemas para detectar problemas |
| **Threat** | Peligro potencial (hacker, malware, insider) que podría dañar sistemas o datos |
| **Risk** | La probabilidad e impacto potencial de que una amenaza dañe a la organización |
| **IOC (Indicator of Compromise)** | Evidencia de que un sistema ha sido comprometido (IP sospechosa, hash de malware, etc.) |
| **Alert** | Notificación generada por un sistema de seguridad cuando detecta actividad sospechosa |

---

## Comportamiento sospechoso — ¿Qué busca el defensor?

```
🚩 Inicios de sesión repetidos fallidos  → Posible brute force
🚩 Login exitoso a las 3am desde otro país → Posible cuenta comprometida
🚩 Transferencia masiva de datos          → Posible exfiltración
🚩 Nuevo proceso desconocido en servidor  → Posible malware
🚩 Tráfico saliente a IP desconocida      → Posible C2 (Command & Control)
🚩 Cambios en archivos del sistema        → Posible modificación maliciosa
```

---

## Carreras en Seguridad Defensiva

### SOC — Security Operations Center

**Qué hace:** Monitorea redes y sistemas 24/7 para detectar e investigar actividad sospechosa o alertas de seguridad.

**Niveles:**
```
SOC L1 → Triage de alertas, primera respuesta, escalada
SOC L2 → Investigación profunda, correlación de eventos
SOC L3 → Threat Hunting, respuesta a incidentes complejos, mejora de detecciones
```

**Habilidades clave:** SIEM, análisis de logs, conocimiento de ataques comunes, redes.

---

### Threat Intelligence

**Qué hace:** Investiga amenazas actuales, actores maliciosos y tendencias para ayudar a la organización a prepararse y defenderse.

- Analiza malware, TTPs (Tácticas, Técnicas y Procedimientos) de atacantes
- Produce informes sobre grupos de amenaza (APTs, ransomware gangs)
- Alimenta al SOC con indicadores de compromiso (IOCs)

**Habilidades clave:** OSINT, análisis de malware, MITRE ATT&CK, geopolítica de amenazas.

---

### DFIR — Digital Forensics & Incident Response

**Qué hace:** Investiga incidentes de seguridad para entender cómo ocurrió el ataque, contener la amenaza y restaurar los sistemas afectados.

| Sub-área | Enfoque |
|----------|---------|
| **Digital Forensics** | Preservar y analizar evidencia digital (memoria, disco, logs, red) |
| **Incident Response** | Contener el incidente, erradicar la amenaza, recuperar sistemas |

**Fases de respuesta a incidentes:**
```
1. Preparación    → Planes, herramientas y equipos listos antes del incidente
2. Identificación → Detectar y confirmar que hay un incidente real
3. Contención     → Aislar sistemas comprometidos para detener la propagación
4. Erradicación   → Eliminar el malware/acceso del atacante
5. Recuperación   → Restaurar sistemas a operación normal
6. Lecciones      → Qué salió mal y cómo mejorar
```

**Habilidades clave:** Forensia de memoria y disco, análisis de red, timeline de ataques.

---

## Red Team vs Blue Team — La relación

```
RED TEAM                           BLUE TEAM
(Ofensiva)                        (Defensiva)
    │                                  │
    │── Simula ataques reales ────────→│── Detecta y responde
    │                                  │
    │←─ Elude defensas ───────────────│── Mejora sus detecciones
    │                                  │
    └──────────────────────────────────┘
              PURPLE TEAM
         (Colaboración activa entre ambos)
```

El Red Team mejora el Blue Team al demostrar qué falló. El Blue Team mejora el Red Team al cerrar los vectores que usaron.

---

## Pre-Security Path — Completado ✅

Completar la ruta Pre-Security de TryHackMe cubre los fundamentos necesarios para continuar en cualquier dirección de ciberseguridad.

**Siguientes rutas sugeridas:**
- **Cyber Security 101** → Fundamentos más profundos de ofensiva y defensiva
- **Jr. Penetration Tester** → Ruta ofensiva práctica
- **SOC Level 1** → Ruta defensiva práctica

---

## Notas relacionadas

- [[Roles-Ciberseguridad]] — Descripción de roles: analista SOC, especialidades del Blue Team
- [[CIA-Triad]] — Los 3 pilares que el Blue Team defiende
- [[Ofensiva-Pentesting]] — La perspectiva del atacante que el defensor debe conocer
- [[Seguridad-de-Red]] — Firewall y VPN como herramientas defensivas de red
- [[Arquitectura-Web]] — WAF como herramienta defensiva de aplicaciones web
