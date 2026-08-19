# Fundamentos de Ciberseguridad — Conceptos Generales (repaso tipo examen)

Relacionado: [[CIA-Triad]] · [[Principios-de-Seguridad-CIA-DAD]] · [[Roles-Ciberseguridad]] · [[Ofensiva-Pentesting]] · [[TCP-UDP-Puertos]] · [[SQL-Injection-Fundamentos-y-Explotacion]]

## Qué es "ciberseguridad"
Es un esfuerzo **continuo** para proteger los sistemas conectados a Internet y los datos asociados contra daño o uso no autorizado — no se limita a una herramienta específica (antivirus, firewall) ni a un documento estático de políticas; abarca personas, procesos y tecnología trabajando de forma constante, porque las amenazas y vectores de ataque evolucionan sin parar.

## Categorías de controles/medidas de seguridad (3 pilares)
- **Tecnología** (controles técnicos/lógicos): herramientas, software, hardware, configuraciones.
- **Sensibilización, formación y educación** (personas): capacitación continua para reducir el riesgo del factor humano (ingeniería social, malas prácticas).
- **Políticas y procedimientos** (procesos/administrativos): directrices, normas, planes de respuesta.

Ejemplos concretos (firewall, cámara) son **herramientas**, no categorías — un firewall es un control técnico, una cámara es un control físico.

## Malware y prevención

### Spyware — mejor prevención
**Instalar software únicamente de sitios web confiables.** A diferencia de malware que explota vulnerabilidades sin interacción del usuario (ahí sí ayudan los parches de SO/navegador), el spyware suele instalarse por engaño: empaquetado con software gratuito/pirateado, descargadores de terceros falsos, extensiones dudosas. Si el propio usuario autoriza la instalación desde una fuente no confiable, ni el antivirus ni las actualizaciones alcanzan a bloquearlo a tiempo — cortar el vector de entrada (la fuente de descarga) es lo que realmente previene.

### Stuxnet
Diseñado para **provocar daño físico a equipos controlados por computadora** — ciberarma dirigida al programa nuclear de Irán, atacando sistemas ICS/SCADA (Siemens) y PLCs. Modificaba la velocidad de rotación de las centrifugadoras de enriquecimiento de uranio en Natanz hasta desgastarlas y destruirlas físicamente, mientras enviaba lecturas falsas a los operadores para simular normalidad.

## Vulnerabilidades — distinguir tipos
| Vulnerabilidad | Qué es |
|---|---|
| **Entrada no validada** | la app procesa datos externos sin verificar formato/longitud/contenido — causa raíz de SQLi, XSS, inyección de comandos (ver [[SQL-Injection-Fundamentos-y-Explotacion]]) |
| **Desbordamiento de búfer** | se escribe más información en un bloque de memoria de la que puede contener |
| **Condición de carrera (race condition)** | el resultado depende del orden/tiempo de ejecución entre procesos/hilos que acceden a un recurso compartido — base de vulnerabilidades TOCTOU (Time-of-Check to Time-of-Use) |
| **Problemas de control de acceso** | fallos en permisos/privilegios que dejan a usuarios no autorizados acceder a recursos — distinto de recibir datos maliciosos que alteran la lógica del programa |

> El desbordamiento de búfer y las condiciones de carrera a veces se confunden: buffer overflow es un problema de **gestión de memoria** (escribir de más); condición de carrera es un problema de **sincronización/tiempo** (el orden de ejecución importa).

## Análisis basado en comportamiento
Funciona estableciendo primero una **línea base (baseline)** de qué es "normal" para una red/usuario/sistema — cualquier desviación significativa de ese patrón se detecta como **anomalía**. Distinto de: vulnerabilidades (se detectan con escaneos/análisis estático, no comparando contra una baseline), backdoors (pueden causar comportamiento anómalo, pero lo que el análisis detecta directamente es la desviación en sí, no la puerta trasera), riesgo (una métrica calculada de impacto × probabilidad, no algo que se "detecte" por monitoreo).

## Integridad de los datos (pilar de la tríada CIA)
Dos objetivos centrales: **los datos no se alteran durante el tránsito**, y **las entidades no autorizadas no pueden modificar los datos**. No confundir con:
- "Datos disponibles todo el tiempo" → pilar de **Disponibilidad**.
- "Datos cifrados en tránsito y en reposo" → pilar de **Confidencialidad**.
- "Acceso autenticado" → apoya la seguridad general, pero no define integridad en sí.

## Destrucción segura de datos
La **única** forma de garantizar que archivos eliminados sean 100% irrecuperables es la **destrucción física del disco** (trituración, desmagnetización/degaussing, incineración — estándar NIST SP 800-88). Herramientas de software (SDelete, Secure Empty Trash) sobrescriben sectores lógicos, pero en SSDs modernos el wear leveling y los bloques reasignados pueden dejar copias residuales recuperables con técnicas forenses avanzadas. Vaciar la papelera de reciclaje solo borra los punteros de la tabla de archivos (ej. MFT) — los datos crudos siguen intactos hasta que se sobrescriben.

## Autenticación multifactor (MFA)
Requiere combinar **al menos dos categorías distintas**:
| Factor | Categoría | Ejemplo |
|---|---|---|
| Algo que **eres** | Inherencia/biometría | huella, rostro, iris |
| Algo que **sabes** | Conocimiento | contraseña, PIN, respuesta de seguridad |
| Algo que **tienes** | Posesión | token OTP, tarjeta física, smartphone |

Huella + contraseña = 2FA real (dos categorías distintas). Contraseña + respuesta de seguridad = **un solo factor en dos pasos** (ambos son "algo que sabes"), no MFA real.

## Identificadores en la comunicación de red
- **Dirección MAC**: identifica la interfaz física en la red local (Capa 2).
- **Dirección IP**: identifica el dispositivo/host a nivel de red (Capa 3).
- **Número de puerto**: identifica la **aplicación/proceso específico** dentro de un host (Capa de Transporte) — la combinación IP + puerto (socket) es lo que garantiza que los datos lleguen no solo al dispositivo correcto, sino a la aplicación exacta en ambos extremos.
- **Número de secuencia**: usado en TCP para ordenar segmentos y asegurar entrega confiable — no identifica la aplicación destino.

## Tipos de atacante (perfiles)
| Perfil | Motivación/método |
|---|---|
| **Hacktivistas** | hacen declaraciones políticas para concientizar sobre causas que les importan |
| **Patrocinados por el estado** | recopilan inteligencia o atacan objetivos específicos en nombre de su gobierno |
| **Script kiddies** | usan herramientas ya existentes en internet para lanzar ataques, sin desarrollarlas ellos mismos |

## Cisco ISE + TrustSec
Implementan control de acceso basado en identidad y contexto (**RBAC**) — clasifican usuarios/dispositivos con etiquetas de grupo de seguridad (**SGT**, Security Group Tags), permitiendo definir políticas de segmentación y privilegios según el **rol** de la entidad, sin importar su IP física o topología de red. No son lo mismo que:
- **DLP** (Data Loss Prevention): evita que datos confidenciales se filtren/roben.
- **IPS/IDS o firewalls basados en firmas**: bloquean tráfico según reglas positivas o coincidencia de firma.

## Equipo de respuesta a incidentes (CSIRT / IR team)
Función principal: **proteger la empresa, el sistema, y la preservación de los datos** — gestionar, contener, mitigar e investigar brechas para minimizar el impacto, salvaguardando confidencialidad, integridad y disponibilidad de los datos corporativos/de clientes. No diseñan hardware/firmware, no crean estándares de cifrado, y definitivamente no diseñan malware.

## Ética en el trabajo (ejemplo)
Compartir características de un producto nuevo en una reunión interna entre departamentos de la misma empresa (ej. TI ↔ Marketing) es **ético** — colaboración interdepartamental legítima y necesaria. Sería **poco ético** solo si esa información se divulgara a entidades externas, competidores, o al público sin autorización (violación de NDA/confidencialidad).

## Dónde "viven" las criptomonedas
No existen como archivos locales — son registros de transacciones/saldos en la **blockchain** (libro mayor público, distribuido e inmutable). Las **billeteras (wallets)** no guardan las monedas en sí, sino las claves privada/pública para firmar transacciones y demostrar propiedad. Los **exchanges** custodian esas claves privadas en sus propias direcciones y llevan un registro contable interno de los saldos de cada usuario.

## Certificaciones de ciberseguridad — comparativa rápida
| Certificación | Enfoque | Nivel |
|---|---|---|
| **CompTIA Security+** | conceptos generales de seguridad, redes, operaciones defensivas — vendor-neutral | Entrada / inicial |
| **EC-Council CEH** (Certified Ethical Hacker) | metodología ofensiva: encontrar vulnerabilidades con las mismas técnicas/herramientas que un atacante malicioso, pero de forma legal/ética | Intermedio / especializado (pentesting) |
| **ISC2 CISSP** | gestión estratégica, arquitectura, gobernanza y liderazgo en seguridad — requiere ~5 años de experiencia en 2+ dominios | Avanzado / gerencial |
| **MTA Security Fundamentals / Palo Alto PCCSA / ISACA CSX Fundamentals** | introductorias o específicas de un producto/fabricante (vendor-specific) | Introductorio |

**Para empezar la carrera**: CompTIA Security+ es el estándar de referencia como certificación de entrada. **Para validar habilidades ofensivas/pentesting específicamente**: CEH es la que evalúa exactamente esa metodología (encontrar debilidades con las mismas herramientas que un atacante, de forma legal). Ver también [[Roles-Ciberseguridad]] para el panorama de roles y valor del training.
