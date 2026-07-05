---
tags:
  - pre-security
  - ciberseguridad
  - CIA-triad
  - confidencialidad
  - integridad
  - disponibilidad
  - certificaciones
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔐 CIA Triad — Los 3 Pilares de la Ciberseguridad

La **CIA Triad** es el marco fundamental de la ciberseguridad. Prácticamente todo lo que hagas en seguridad — ya sea defender o atacar — gira en torno a estos tres principios.

```
         Confidentiality
              /\
             /  \
            /    \
           /  CIA \
          /  TRIAD \
         /──────────\
    Integrity    Availability
```

> [!info] Pregunta clave ante cualquier incidente
> 1. ¿Fue expuesta información sensible a personas no autorizadas? → **Confidentiality**
> 2. ¿Fueron modificados datos sin permiso? → **Integrity**
> 3. ¿Quedaron sistemas o servicios inaccesibles? → **Availability**

---

## C — Confidentiality (Confidencialidad)

**Definición:** Solo las personas autorizadas pueden acceder a la información.

**Amenaza:** Acceso no autorizado a datos sensibles.

**Consecuencias de una brecha:** pérdidas financieras, violaciones de privacidad, consecuencias legales.

> [!example] Analogía
> Estás en una conversación privada y alguien escucha sin permiso y luego usa esa información para manipularte.

> [!example] Ejemplo digital
> Estás en el WiFi de una cafetería. Un atacante intercepta tus credenciales cuando haces login en redes sociales (ataque MitM). Tu sesión es comprometida.

### ¿Cómo se protege la confidencialidad?

| Técnica | Descripción |
|---------|-------------|
| **Cifrado** | Los datos viajan o se almacenan en forma ilegible sin la clave |
| **Control de acceso** | Solo usuarios autorizados pueden ver ciertos recursos (RBAC, ACLs) |
| **Autenticación fuerte** | MFA, contraseñas robustas, certificados |
| **HTTPS / TLS** | Cifrado del tráfico web |
| **VPN** | Túnel cifrado para tráfico de red |

### Ejemplos rápidos

| Situación | ¿Confidencialidad OK? |
|-----------|----------------------|
| Credenciales de Gmail en post-its sobre el escritorio | ❌ No |
| Documentos internos accesibles solo a empleados que los necesitan | ✅ Sí |
| Documento personal publicado en Internet | ❌ No |

---

## I — Integrity (Integridad)

**Definición:** Los datos solo pueden ser modificados por personas autorizadas y a través de procesos autorizados.

**Amenaza:** Modificación no autorizada de datos.

**Consecuencias:** Datos en los que no se puede confiar, decisiones erróneas, consecuencias peligrosas.

> [!example] Analogía
> Tu profesor te pone una buena nota. Alguien la modifica antes de que llegue a la institución — tu expediente refleja algo que no ocurrió.

> [!example] Ejemplo digital
> Inicias una transferencia bancaria. Un atacante intercepta la transacción y cambia el número de cuenta destino. El dinero llega a otro lugar.

### ¿Cómo se protege la integridad?

| Técnica | Descripción |
|---------|-------------|
| **Hashing** | Genera una "huella digital" del archivo — cualquier cambio altera el hash |
| **Firmas digitales** | Verifican que el mensaje no fue alterado y viene del remitente correcto |
| **Control de versiones** | Registra quién modificó qué y cuándo |
| **Checksums** | Valor de verificación en transmisión de datos (ej. en headers TCP) |
| **Permisos de escritura** | Solo usuarios autorizados pueden modificar archivos |

### Ejemplos rápidos

| Situación | ¿Integridad OK? |
|-----------|----------------|
| Datos modificados con aprobación autorizada | ✅ Sí |
| Registros de asistencia cambiados después de ser cerrados por el profesor | ❌ No |
| Precio de un pedido modificado antes del checkout | ❌ No |

---

## A — Availability (Disponibilidad)

**Definición:** Los datos y servicios están accesibles para los usuarios autorizados cuando los necesitan.

**Amenaza:** Interrupción del acceso a sistemas o servicios.

**Consecuencias:** Pérdida de negocio, daño a la reputación, impacto operacional grave.

> [!example] Analogía
> Depositaste dinero en un banco que lo guarda muy seguro. Pero el día que lo necesitas el banco está cerrado por falla eléctrica. Aunque tu dinero es seguro (confidencialidad ✅, integridad ✅), no puedes acceder a él.

> [!example] Ejemplo digital
> Un **DDoS (Distributed Denial of Service)** envía millones de requests a un servidor. El servidor se satura y cae. No se filtraron ni modificaron datos, pero el servicio está inaccesible — la disponibilidad está comprometida.

### ¿Cómo se protege la disponibilidad?

| Técnica | Descripción |
|---------|-------------|
| **Load Balancers** | Distribuyen el tráfico para evitar saturación (ver [[Arquitectura-Web]]) |
| **Redundancia** | Servidores y conexiones duplicadas — si uno falla, otro toma el relevo |
| **Backups** | Copias de seguridad para recuperar datos en caso de pérdida |
| **Rate Limiting** | Limita requests por IP para prevenir DDoS (implementado en WAF) |
| **DDoS Protection** | Servicios como Cloudflare absorben el tráfico malicioso |
| **Generadores de respaldo** | En el mundo físico, garantizan energía ante cortes |

### Ejemplos rápidos

| Situación | ¿Disponibilidad OK? |
|-----------|---------------------|
| Servicios críticos interrumpidos por instalación de software | ❌ No |
| El sitio web de la empresa cayó durante horas de trabajo | ❌ No |
| Todos los sistemas accesibles para los empleados en horario laboral | ✅ Sí |

---

## CIA Triad vs Tipos de Ataque

| Tipo de Ataque | CIA afectado |
|----------------|-------------|
| **Phishing / Robo de credenciales** | Confidentiality |
| **Interceptación de tráfico (MitM)** | Confidentiality + Integrity |
| **Ransomware (datos cifrados, inaccesibles)** | Availability (+Confidentiality si hay exfiltración) |
| **DDoS** | Availability |
| **SQL Injection (lectura de BD)** | Confidentiality |
| **Modificación de transacciones** | Integrity |
| **Defacement de web** | Integrity |
| **Exfiltración de datos** | Confidentiality |

---

## Para Certificaciones

La CIA Triad aparece directamente en:

- **CompTIA Security+ (SY0-701)** — Dominio 1: General Security Concepts
- **CompTIA Network+** — Fundamentos de seguridad de red
- **CEH** — Marco teórico de cualquier análisis de seguridad
- **CISSP** — CIA es la base de todos los dominios

> [!tip] Tip de examen
> En preguntas de escenario, primero identifica **qué pasó**:
> - ¿Alguien VIO algo que no debía? → **Confidentiality**
> - ¿Alguien CAMBIÓ algo sin permiso? → **Integrity**
> - ¿Algo NO ESTABA DISPONIBLE cuando se necesitaba? → **Availability**
> Un solo incidente puede afectar más de un pilar simultáneamente.

---

## Notas relacionadas

- [[Seguridad-de-Red]] — Firewall y VPN protegen Confidentiality y Availability
- [[Criptografia]] — El cifrado es la herramienta principal de Confidentiality e Integrity
- [[Arquitectura-Web]] — WAF y Load Balancers protegen Availability
- [[Roles-Ciberseguridad]] — El SOC Analyst defiende los 3 pilares constantemente
