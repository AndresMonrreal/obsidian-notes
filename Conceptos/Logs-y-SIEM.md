# Logs y análisis + SIEM

Relacionado: [[SOC-Centro-de-Operaciones]] · [[Incident-Response]] · [[HTTP-Peticiones-y-Respuestas]] · [[Windows-PowerShell]] · [[Linux-Herramientas-y-Admin]] · [[Seguridad-de-Red]]

## Qué son los logs
Los logs son las **huellas digitales** de cualquier actividad (normal o maliciosa). La mayoría de los rastros de un ataque están en los logs.

### Casos de uso
Security Events Monitoring · Incident Investigation & Forensics · Troubleshooting · Performance Monitoring · Auditing & Compliance.

## Tipos de logs
| Tipo | Usa para | Ejemplos de eventos |
|------|----------|---------------------|
| **System** | troubleshooting del SO | arranque/apagado, carga de drivers, errores de sistema, hardware |
| **Security** | detectar/investigar incidentes | autenticación, autorización, cambios de política, cambios de cuentas |
| **Application** | eventos de una app | interacción de usuario, cambios/updates, errores |
| **Audit** | compliance y monitoreo | acceso a datos, cambios de sistema, actividad de usuario, enforcement de políticas |
| **Network** | tráfico de red | tráfico entrante/saliente, conexiones, firewall |
| **Access** | acceso a recursos | web server, base de datos, aplicación, API |

## Fuentes de logs (log sources)
- **Host-Centric**: eventos dentro del host → acceso a archivo, autenticación, ejecución de proceso, cambios en el **registro**, ejecución de PowerShell. (Generan Windows, Linux, servidores.)
- **Network-Centric**: comunicación entre hosts / a internet → conexión SSH, FTP, tráfico web, VPN, compartición de archivos. (Generan firewalls, IDS/IPS, routers.)

## Windows — Event Viewer y Event IDs
Windows registra eventos vistos en **Event Viewer** (GUI). Campos clave de un evento: **Description**, **Log Name**, **Logged** (hora), **Event ID** (identificador único de la actividad).

### Event IDs importantes
| Event ID | Significado |
|----------|-------------|
| **4624** | login de cuenta **exitoso** |
| **4625** | login **fallido** |
| **4634** | logoff exitoso |
| **4720** | cuenta **creada** |
| **4722** | cuenta habilitada |
| **4725** | cuenta deshabilitada |
| **4724** | intento de reset de contraseña |
| **4726** | cuenta **eliminada** |
| **4688** | ejecución de proceso (New Process) |
| **104** | **event log limpiado/borrado** (típico de anti-forense) |

En Event Viewer → **Filter Current Log** para filtrar por Event ID (ej. 4624 para ver todos los logins exitosos).

## Linux — ubicaciones de logs
| Ruta | Contenido |
|------|-----------|
| `/var/log/httpd` o `/var/log/apache` | logs HTTP request/response y errores |
| `/var/log/cron` | eventos de cron jobs |
| `/var/log/auth.log` y `/var/log/secure` | autenticación |
| `/var/log/kern` | eventos del kernel |
| `/var/log/apache2/access.log` | accesos al web server Apache |

## Análisis manual de logs (comandos Linux)

### `cat` — mostrar y combinar
```bash
cat access.log                              # mostrar contenido
cat access1.log access2.log > combined.log  # combinar varios logs (útil tras rotación)
```

### `grep` — buscar patrones
```bash
grep "192.168.1.1" access.log     # todas las líneas con esa IP
```

### `less` — ver página por página
```bash
less access.log
# espacio = siguiente página · b = anterior
# /patrón = buscar · n = siguiente coincidencia · N = anterior
```

## Anatomía de un log de acceso Apache
```
172.16.0.1 - - [06/Jun/2024:13:58:44] "GET /products HTTP/1.1" 404 "-" "Mozilla/5.0 ..."
```
- **IP Address** `172.16.0.1` — quién hizo la petición
- **Timestamp** `[06/Jun/2024:13:58:44]`
- **HTTP Method** `GET` — acción (ver [[HTTP-Peticiones-y-Respuestas]])
- **URL** `/products` — recurso solicitado
- **Status Code** `404` — respuesta del servidor
- **User-Agent** — SO/navegador del cliente

---

# SIEM — Security Information and Event Management

## Qué es
Solución que recolecta logs de muchas fuentes, **estandariza su formato**, los correlaciona y detecta actividad maliciosa con reglas de detección. Resuelve los problemas del análisis manual: numerosas fuentes, falta de centralización, contexto limitado, análisis limitado y formatos distintos.

## Features clave
- **Centralized Log Collection**: todos los logs en un solo lugar (vía agentes/APIs).
- **Normalization**:
  - **Parsing** = dividir un log en campos.
  - **Normalization** = convertir logs de todas las fuentes a un formato consistente.
- **Correlation**: relaciona logs de distintas fuentes para revelar patrones (ej. VPN desde IP nueva + acceso a documentos + PowerShell + conexión saliente = posible exfiltración con credenciales comprometidas).
- **Real-time Alerting**: dispara alertas cuando se cumplen las reglas.
- **Dashboards & Reporting**: insights accionables (login fallidos, reglas disparadas, dominios top, etc.). Ej. Splunk.

## Métodos de ingesta de logs
| Método | Descripción |
|--------|-------------|
| **Agent / Forwarder** | herramienta ligera instalada en el endpoint (Splunk lo llama *forwarder*) |
| **Syslog** | protocolo para enviar datos en tiempo real a un destino central |
| **Manual Upload** | subir datos offline para análisis rápido (Splunk, ELK) |
| **Port-Forwarding** | el SIEM escucha en un puerto y los endpoints le reenvían datos |

## Reglas de detección
Expresiones lógicas que disparan alertas. Ejemplos:
- 5 logins fallidos en 10 s → "Multiple Failed Login Attempts".
- Login exitoso tras varios fallidos → "Successful Login After Multiple Attempts".
- Tráfico saliente > 25 MB → posible exfiltración.

### Ejemplos con Event Logs
```
# Borrado de logs (anti-forense):
Rule: si LogSource = WinEventLog AND EventID = 104 → alerta "Event Log Cleared"

# Ejecución de whoami (post-explotación):
Rule: si LogSource = WinEventLog AND EventCode = 4688
       AND NewProcessName contiene "whoami" → alerta "WHOAMI Execution Detected"
```
Por esto es importante tener logs **normalizados**: las reglas vigilan pares campo-valor.

## Investigación de alertas
Tras dispararse una alerta, el analista revisa qué condiciones se cumplieron y decide:
- **Falso positivo** → puede requerir **tuning** de la regla.
- **Verdadero positivo** → investigar más: contactar al dueño del activo, aislar host, bloquear IP.
