# OWASP Top 10 (2025) — Vulnerabilidades web

Relacionado: [[HTTP-Peticiones-y-Respuestas]] · [[Bases-de-Datos-SQL]] · [[Herramientas-Burp-Suite]] · [[Hashing]] · [[Principios-de-Seguridad-CIA-DAD]]

## AAAA — el modelo de 4 niveles
No se puede saltar un nivel; cada uno depende del anterior:
1. **Identity** — cuenta única que representa a una persona/servicio.
2. **Authentication** — probar esa identidad (password, OTP, passkeys).
3. **Authorisation** — qué puede hacer esa identidad.
4. **Accountability** — registrar y alertar quién hizo qué, cuándo y desde dónde.

Las categorías del Top 10 relacionadas con AAAA (Broken Access Control, Authentication Failures, Logging Failures) apuntan a fallos en implementar este modelo.

## Broken Access Control
El servidor no aplica correctamente quién puede acceder a qué en cada request. Ejemplo clásico: **IDOR** (Insecure Direct Object Reference) — cambiar `?id=7` → `?id=6` y ver/editar datos de otro usuario.

- **Horizontal privilege escalation**: mismo rol, accedes a datos de **otro usuario**.
- **Vertical privilege escalation**: saltas a acciones **solo de admin**.

Causa raíz: la app confía demasiado en el cliente.

## Authentication Failures
Fallos al verificar/vincular la identidad de forma confiable:
- Enumeración de usuarios.
- Contraseñas débiles/adivinables (sin lockout/rate limit).
- Fallos de lógica en login/registro (ej. registrar `aDmiN` para colisionar con `admin` si la validación no es case-sensitive correctamente).
- Manejo inseguro de sesión/cookies.

## Logging Failures
Sin logs de eventos de seguridad, los defensores no pueden detectar ni investigar ataques. Fallas típicas: eventos de autenticación faltantes, logs vagos, sin alertas ante fuerza bruta o cambios de privilegio, retención corta, o logs almacenados donde el atacante puede manipularlos.

## Security Misconfigurations
No son bugs de código, sino errores de **cómo se despliega** el sistema/servidor/red: defaults inseguros, configuración incompleta, servicios expuestos.

**Ejemplo real**: en 2017, Uber expuso un bucket de backup con datos de usuarios porque el bucket era públicamente accesible — sin necesitar credenciales.

### Patrones comunes
Credenciales default/débiles sin cambiar · servicios/endpoints innecesarios expuestos a internet · buckets cloud mal configurados (S3, Azure Blob, GCP) · acceso API sin auth · mensajes de error verbosos (stack traces) · software/frameworks desactualizados · endpoints de CI/CD sin controles de acceso.

### Mitigación
Hardening de defaults · least privilege · segmentar red · parchear regularmente · ocultar stack traces · auditar configuraciones cloud · asegurar endpoints CI/CD · integrar chequeos de seguridad al pipeline de deploy.

## Software Supply Chain Failures
Depender de componentes, librerías, servicios o modelos de IA comprometidos, desactualizados o sin verificar.

**Ejemplo real**: SolarWinds (2021) — código malicioso insertado en un update **confiable** que miles de organizaciones instalaron automáticamente. El fallo no estaba en la lógica core, sino en el proceso de build/verificación/distribución del update.

### Patrones comunes
Librerías sin verificar/mantener · updates automáticos sin verificación · sobre-dependencia de modelos IA de terceros sin auditar · procesos de build/CI-CD inseguros · falta de tracking de licencias/procedencia.

### Mitigación
Verificar componentes/librerías/modelos antes de usarlos · monitorear y parchear dependencias · firmar/verificar/auditar updates · asegurar CI/CD · trackear procedencia y licencias · monitoreo en runtime de comportamiento anómalo.

## Cryptographic Failures
Cifrado usado incorrectamente o ausente: algoritmos débiles, claves hardcodeadas, mal manejo de claves, datos sensibles sin cifrar.

### Patrones comunes
Algoritmos deprecados (**MD5, SHA-1, ECB**) · secretos hardcodeados en código/config · mala rotación de claves · falta de cifrado en reposo/tránsito · certificados TLS self-signed o inválidos.

### Mitigación
Usar algoritmos modernos (**AES-GCM, ChaCha20-Poly1305, TLS 1.3** con certificados válidos) · gestores de secretos (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault) · rotar claves regularmente · inventario completo de certificados/claves.

> Regla de oro: **nunca crear tu propio algoritmo de cifrado** ("rolling your own crypto"). Usar librerías probadas de la industria. Para hashear passwords: **bcrypt, scrypt o Argon2** (funciones lentas por diseño).

## Insecure Design
Fallo arquitectónico desde el inicio: sin threat modeling, sin revisión de diseño, asunciones erróneas. Con IA se agrava: asumir que un modelo/agente es seguro, correcto o predecible sin validarlo.

**Ejemplo real**: Clubhouse asumía que solo la app móvil consultaba su backend, pero el backend no tenía autenticación real — cualquiera podía consultar datos de usuarios y conversaciones "privadas" directamente.

### Fallos de diseño en la era de IA
- **Prompt injection**: input de usuario mezclado con el prompt del sistema permite secuestrar el contexto o extraer datos ocultos.
- Confianza ciega en el output del modelo sin validación humana.
- Modelos "envenenados" de fuentes sin verificar, con backdoors ocultos.

### Mitigación
Tratar todo modelo como no confiable hasta probar lo contrario · validar inputs/outputs del modelo · separar prompts del sistema del contenido del usuario · requerir revisión humana en acciones de alto riesgo · threat modeling específico para IA (prompt attacks, inference risks, agent misuse) · least privilege · autenticación/autorización robusta.

## Injection
El input de usuario se pasa sin sanitizar a un sistema que puede ejecutar comandos o queries (base de datos, shell, motor de plantillas, IA).

Tipos clásicos: **SQL Injection** (ver [[Bases-de-Datos-SQL]]) · Command Injection · Prompt Injection · **SSTI** (Server Side Template Injection).

### Mitigación
Tratar todo input como no confiable · **prepared statements / parameterised queries** para SQL (nunca concatenar strings) · evitar funciones que pasan input directo al shell · validar y sanear input, escapar caracteres peligrosos, forzar tipos de dato estrictos.

## Ejercicios prácticos (referencia de puertos)
| Vulnerabilidad | Puerto/URL |
|-----------------|-----------|
| Broken Access Control (IDOR) | sitio estático — buscar `accountID` en la URL |
| Authentication Failures | sitio estático — registrar usuario `aDmiN` |
| Logging Failures | sitio estático — investigar logs de un ataque |
| Security Misconfigurations | `MACHINE_IP:5002` — User Management APIs |
| Software Supply Chain Failures | `MACHINE_IP:5003` — `lib/vulnerable_utils.py` desactualizado |
| Insecure Design | `MACHINE_IP:5005` — asume solo acceso móvil |
| Cryptographic Failures | `MACHINE_IP:8001` — note sharing con clave derivada débil |
| Injection (SSTI) | `MACHINE_IP:8000` — renderizado dinámico de contenido |
| Software/Data Integrity Failures | `MACHINE_IP:8002` — ataque de deserialización en Python |

## Software or Data Integrity Failures
Confiar en código/updates/datos sin verificar su autenticidad, integridad u origen: cargar scripts/configs de fuentes no confiables, no validar datos que afectan la lógica, aceptar binarios/plantillas/archivos sin confirmar si fueron alterados.

### Mitigación
Establecer **trust boundaries** claros · verificación de integridad (checksums criptográficos) para paquetes de update · solo fuentes confiables pueden modificar artefactos críticos · aplicar los mismos principios de integridad dentro de procesos CI/CD.
