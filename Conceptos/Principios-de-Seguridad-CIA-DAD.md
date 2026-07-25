# Principios de Seguridad — CIA, DAD y Modelos

Relacionado: [[CIA-Triad]] · [[Seguridad-de-Red]] · [[Roles-Ciberseguridad]] · [[OWASP-Top-10-2025]]

## La tríada CIA
| Elemento | Significa |
|----------|-----------|
| **Confidentiality** | solo los destinatarios previstos acceden a los datos |
| **Integrity** | los datos no pueden alterarse; si se alteran, se **detecta** |
| **Availability** | el sistema/servicio está disponible cuando se necesita |

No siempre pesan igual: un anuncio universitario no es confidencial, pero su **integridad** es crítica (que nadie lo altere).

## Más allá de CIA
- **Authenticity** (autenticidad): el dato/archivo es genuino, viene de la fuente que dice.
- **Nonrepudiation** (no repudio): la fuente no puede negar haber sido el origen. Crítico en compras, banca, diagnósticos médicos.

## Parkerian Hexad (Donn Parker, 1998)
Extiende CIA a 6 elementos: **Availability, Utility, Integrity, Authenticity, Confidentiality, Possession**.

- **Utility**: qué tan útil es la información. Ej.: perder la clave de descifrado de un disco cifrado → el dato sigue disponible pero **no útil**.
- **Possession**: proteger contra toma/copia/control no autorizado. Ej.: un atacante roba el disco de backup (pierdes *posesión*) o cifra tus datos con ransomware.

## La tríada DAD (opuesto de CIA)
| DAD | Ataca a |
|-----|---------|
| **Disclosure** (divulgación) | Confidentiality |
| **Alteration** (alteración) | Integrity |
| **Destruction / Denial** (destrucción/negación) | Availability |

> Proteger CIA al extremo puede sacrificar el otro lado: proteger confidencialidad/integridad al máximo puede restringir disponibilidad, y viceversa. Se necesita **balance**.

## Modelos de seguridad

### Bell-LaPadula (confidencialidad)
- **Simple Security Property** ("no read up"): un sujeto de nivel bajo NO puede leer un objeto de nivel alto.
- **Star Security Property** ("no write down"): un sujeto de nivel alto NO puede escribir a un objeto de nivel bajo.
- **Discretionary-Security Property**: matriz de acceso (read/write por sujeto/objeto).
- Resumen: **"write up, read down"**.
- Limitación: no maneja bien file-sharing.

### Biba (integridad)
- **Simple Integrity Property** ("no read down"): sujeto alto no lee de objeto bajo.
- **Star Integrity Property** ("no write up"): sujeto bajo no escribe a objeto alto.
- Resumen: **"read up, write down"** — inverso de Bell-LaPadula.
- Limitación: no maneja bien insider threats.

### Clark-Wilson (integridad)
Conceptos: **CDI** (Constrained Data Item, dato a proteger) · **UDI** (Unconstrained Data Item, input externo) · **TPs** (Transformation Procedures, mantienen integridad) · **IVPs** (Verification Procedures, validan CDIs).

Otros modelos (para explorar): Brewer and Nash, Goguen-Meseguer, Sutherland, Graham-Denning, Harrison-Ruzzo-Ullman.

## Defence-in-Depth (Multi-Level Security)
Múltiples capas de seguridad en vez de un solo control. Analogía: cajón con llave + habitación con llave + puerta del apartamento + portón del edificio + cámaras. Ninguna capa detiene todo, pero juntas bloquean/ralentizan a la mayoría de atacantes.

## ISO/IEC 19249 — Principios arquitectónicos y de diseño

### 5 principios arquitectónicos
| Principio | Idea |
|-----------|------|
| **Domain Separation** | agrupar componentes relacionados con su propio dominio/atributos (ej. rings de privilegio x86) |
| **Layering** | estructurar en capas abstractas, aplicando políticas por capa (ej. modelo OSI) |
| **Encapsulation** | ocultar implementación interna, exponer solo métodos/API definidos (OOP) |
| **Redundancy** | garantiza disponibilidad/integridad (ej. doble fuente de poder, RAID 5) |
| **Virtualization** | compartir hardware entre SOs; mejora aislamiento (sandboxing) |

### 5 principios de diseño
| Principio | Idea |
|-----------|------|
| **Least Privilege** | dar solo el permiso mínimo necesario ("need-to-know") |
| **Attack Surface Minimisation** | reducir vulnerabilidades expuestas (ej. deshabilitar servicios no usados al hardening) |
| **Centralized Parameter Validation** | validar todo input desde una librería/sistema central |
| **Centralized General Security Services** | centralizar servicios de seguridad (ej. servidor de autenticación central) |
| **Preparing for Error and Exception Handling** | diseñar para **fallar seguro** (ej. si un firewall crashea, debe bloquear todo, no permitir todo); no filtrar info sensible en mensajes de error |

## Confianza: Trust but Verify vs Zero Trust
- **Trust but Verify**: confías en una entidad, pero siempre verificas (logging, IDS/IPS automatizados) — no es factible revisar todo manualmente.
- **Zero Trust**: la confianza es tratada como **vulnerabilidad**; "never trust, always verify". No importa ubicación ni propiedad del dispositivo — se requiere autenticación/autorización antes de acceder a cualquier recurso. Contiene mejor el daño ante una brecha.
  - **Microsegmentation**: implementación de Zero Trust donde un segmento de red puede ser tan pequeño como un solo host; la comunicación entre segmentos requiere autenticación y controles de acceso.

## Vulnerabilidad, Amenaza y Riesgo
| Término | Definición |
|---------|-----------|
| **Vulnerability** | debilidad explotable |
| **Threat** | peligro potencial asociado a esa debilidad |
| **Risk** | probabilidad de que un actor explote la vulnerabilidad × impacto en el negocio |

> Ejemplo: una vitrina de vidrio estándar (vulnerabilidad) puede romperse (amenaza); el riesgo depende de qué tan probable es y qué tanto afecta al negocio si pasa.
