# Incident Response (IR)

Relacionado: [[Defensiva-Blue-Team]] · [[SOC-Centro-de-Operaciones]] · [[Digital-Forensics]] · [[Logs-y-SIEM]]

## Qué es
El **Incident Response** maneja un incidente de principio a fin: desde desplegar seguridad para prevenirlos, hasta combatirlos y minimizar su impacto.

## Eventos, alertas e incidentes
- **Evento**: cualquier acción registrada de un proceso (interactivo o en background). Se generan en cantidades enormes.
- **Alerta**: se dispara cuando un evento (o grupo) apunta a una posible actividad dañina.
- **Falso positivo**: alerta que parece peligrosa pero no lo es (ej. backup a la nube que dispara alerta de exfiltración).
- **Verdadero positivo (True Positive)**: alerta que apunta a algo dañino y realmente lo es (ej. email de phishing confirmado).
- **Incidente**: una alerta de true positive confirmada.

> Analogía: alarma de incendio disparada por humo de cocina = **falso positivo**.

## Severidad de incidentes
Se prioriza por impacto: **Low → Medium → High → Critical**. Los **Critical** se atienden primero.

## Tipos de incidentes
| Tipo | Descripción |
|------|-------------|
| **Malware Infection** | programa malicioso daña sistema/red/app (la mayoría de incidentes) |
| **Security Breach** | acceso no autorizado a datos confidenciales |
| **Data Leak** | exposición de info confidencial (a veces por error humano/misconfig) |
| **Insider Attack** | ataque desde dentro (ej. empleado descontento con USB) |
| **Denial of Service (DoS)** | inunda el sistema con peticiones falsas → agota recursos → indisponibilidad |

> El impacto varía por organización: un data leak puede ser menor para una empresa y un DoS devastador para otra que depende de su web.

## Frameworks de IR

### SANS — 6 fases (mnemónico PICERL)
| Fase | Qué se hace | Ejemplo |
|------|-------------|---------|
| **P**reparation | crear equipos, plan y soluciones de seguridad | training de phishing a empleados |
| **I**dentification | buscar comportamiento anómalo que indique incidente | detectan exfiltración desde un host comprometido |
| **C**ontainment | minimizar el impacto | aislar el host de la red / deshabilitar cuentas |
| **E**radication | eliminar la amenaza del entorno | scan profundo de malware para limpiar el host |
| **R**ecovery | recuperar/reconstruir sistemas desde backup y probarlos | restaurar datos exfiltrados desde backup |
| **L**essons Learned | documentar brechas de detección y mejorar | reunión post-incidente de causa raíz |

### NIST — 4 fases
Similar a SANS pero condensado:
1. **Preparation**
2. **Detection and Analysis** (≈ Identification de SANS)
3. **Containment, Eradication & Recovery**
4. **Post-Incident Activity** (≈ Lessons Learned de SANS)

## Incident Response Plan
Documento formal aprobado por dirección con los procedimientos antes/durante/después de un incidente. Componentes clave:
- Roles y responsabilidades
- Metodología de IR
- Plan de comunicación con stakeholders (incl. autoridades)
- Ruta de escalado (escalation path)

## Soluciones para detección y respuesta
- **SIEM**: centraliza y correlaciona logs (ver [[Logs-y-SIEM]]).
- **AV (Antivirus)**: detecta malware conocido con escaneos regulares.
- **EDR**: desplegado en cada sistema; puede **contener y erradicar** amenazas avanzadas.

## Playbooks vs Runbooks
- **Playbook**: guía/lineamientos comprehensivos para responder a un tipo de incidente.
  - Ejemplo (Phishing Email): notificar stakeholders → analizar cabecera y cuerpo del correo → analizar adjuntos → ver quién los abrió → aislar sistemas infectados → bloquear al remitente.
- **Runbook**: ejecución **paso a paso** y detallada de acciones específicas durante un incidente (varía según los recursos disponibles).

> Práctica IR (phishing en varios hosts): identificar remitente malicioso, vector de amenaza (email attachment), cuántos equipos descargaron el adjunto y cuántos lo ejecutaron.
