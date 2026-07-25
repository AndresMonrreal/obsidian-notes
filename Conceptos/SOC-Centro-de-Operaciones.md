# SOC — Security Operations Center

Relacionado: [[Defensiva-Blue-Team]] · [[Roles-Ciberseguridad]] · [[Logs-y-SIEM]] · [[Incident-Response]] · [[Seguridad-de-Red]]

## Qué es la seguridad defensiva
Proceso de **defender y asegurar** dispositivos y sistemas. A diferencia de la ofensiva, no atacas sistemas: los **monitoreas y proteges**. Objetivo principal: **detectar y responder** a los ataques antes de que causen daño.

## Qué es un SOC
Un **SOC (Security Operations Center)** es una instalación dedicada, operada por un equipo de seguridad especializado, que monitorea continuamente la red y los recursos de una organización para identificar actividad sospechosa y prevenir daños. Trabaja **24/7**. Su foco es **Detección y Respuesta**.

### Detección (Detect)
- **Vulnerabilidades**: debilidades que un atacante puede explotar (ej. equipos Windows sin parchear).
- **Actividad no autorizada**: uso de credenciales robadas (pistas: geolocalización de login).
- **Violaciones de políticas**: descarga de pirateo, envío inseguro de archivos confidenciales.
- **Intrusiones**: acceso no autorizado a sistemas/redes (web app explotada, sitio malicioso).

### Respuesta (Respond)
- Apoyar el **incident response**: minimizar impacto y hacer análisis de causa raíz (ver [[Incident-Response]]).

## Los 3 pilares del SOC
**People · Process · Technology.** Los tres coexisten; un SOC maduro combina profesionales + procesos adecuados + herramientas de última generación.

## People — roles del equipo SOC
| Rol | Responsabilidad |
|-----|-----------------|
| **SOC Analyst L1** | primer respondedor; **triage** básico de alertas y reporte |
| **SOC Analyst L2** | investigación más profunda; correlaciona múltiples fuentes |
| **SOC Analyst L3** | "threat hunter" experimentado; apoya IR (contención, erradicación, recuperación) |
| **Security Engineer** | despliega y configura las soluciones de seguridad |
| **Detection Engineer** | crea las reglas de detección (también L2/L3) |
| **SOC Manager** | gestiona procesos; reporta al **CISO** |

> Sin People, las soluciones generan mucho "ruido" (falsos positivos). El humano distingue lo realmente dañino.

## Process — procesos clave

### Alert Triage y las 5 Ws
Primer análisis de una alerta para determinar severidad y prioridad. Responde:

| W | Ejemplo (Alerta: malware en host "GEORGE PC") |
|---|----|
| **What?** | archivo malicioso detectado en un host |
| **When?** | 13:20 del 5 jun 2024 |
| **Where?** | directorio del host "GEORGE PC" |
| **Who?** | usuario George |
| **Why?** | descargó software pirata para usarlo gratis |

- **Reporting**: escalar alertas dañinas como **tickets** a analistas superiores, con las 5 Ws + capturas como evidencia.
- **Incident Response y Forensics**: para actividades críticas se inicia IR y, a veces, forense (ver [[Digital-Forensics]]).

## Technology — soluciones de seguridad
| Solución | Rol |
|----------|-----|
| **SIEM** (Security Information and Event Management) | recolecta y correlaciona logs de múltiples fuentes; **solo detección/alerta** (ver [[Logs-y-SIEM]]) |
| **EDR** (Endpoint Detection and Response) | visibilidad en tiempo real e histórica del endpoint; puede **responder** automáticamente |
| **Firewall** | barrera entre red interna y externa; filtra tráfico no autorizado (ver [[Firewall-Fundamentos]]) |

Otras: Antivirus, EPP, IDS/IPS (ver [[IDS-Snort]]), XDR, SOAR.

## Práctica: Level 1 Analyst (ejemplo de las 5 Ws)
Alerta de **port scan** desde un host. El SIEM muestra los logs asociados.
- **What**: Port Scan · **When**: 12 jun 2024 17:24 · **Where** (destino): 10.0.0.3 · **Who** (origen): Nessus · **Why**: Intended (lo corría el equipo de vulnerabilidades desde 10.0.0.8).

> Lección: no toda detección es maliciosa; el contexto (nota del equipo de vulnerabilidades) convierte un "port scan" en actividad **intended**.
