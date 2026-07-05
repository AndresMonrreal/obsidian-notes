---
tags:
  - pre-security
  - roles
  - blue-team
  - SOC
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🛡️ Roles en Ciberseguridad

## Security Analyst

El **Security Analyst** (Analista de Seguridad) es el defensor digital de una organización. Forma parte del **[[Blue Team]]** — el equipo encargado de la defensa.

> [!summary] Función principal
> Monitorear, investigar y responder a alertas de actividad inusual en dispositivos y la red.

### Tareas del día a día

- Monitorear **tráfico de red** y **logs** del sistema
- Investigar accesos sospechosos (ej. empleado en Londres que aparece logueado desde otro país)
- Colaborar con otros equipos para reforzar la postura de seguridad
- Documentar incidentes y crear reportes

### Herramientas comunes

| Herramienta | Uso |
|-------------|-----|
| SIEM (Splunk, Elastic) | Centralizar y analizar logs |
| IDS/IPS | Detectar intrusiones |
| Wireshark | Análisis de tráfico de red |
| Threat Intel Platforms | Investigar amenazas conocidas |

---

## Blue Team vs Red Team

| Equipo | Rol | Enfoque |
|--------|-----|---------|
| 🔵 **Blue Team** | Defensivo | Detectar, responder, proteger |
| 🔴 **Red Team** | Ofensivo | Atacar, explotar, reportar |
| 🟣 **Purple Team** | Híbrido | Mejorar defensa usando técnicas ofensivas |

---

## Especializaciones futuras

Desde el rol de Analista puedes especializarte en:

- **Incident Response (IR)** — Respuesta a incidentes activos
- **Threat Hunting** — Búsqueda proactiva de amenazas ocultas en la red
- **Malware Analysis** — Análisis de código malicioso (reversing, sandboxing)
- **SOC Analyst (L1/L2/L3)** — Centro de Operaciones de Seguridad

> [!tip] Consejo de carrera
> El camino típico es: **SOC Analyst L1 → L2 → Threat Hunter / IR Specialist**. Cada nivel requiere más autonomía e investigación profunda.

---

## Notas relacionadas

- [[Redes-Fundamentos]] — La base técnica que necesita un analista
- [[Protocolos-ARP-DHCP]] — Protocolos que aparecen constantemente en los logs
- [[Subnetting]] — Esencial para entender la segmentación de redes
