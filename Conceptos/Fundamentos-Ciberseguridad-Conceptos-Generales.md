# Fundamentos de Ciberseguridad — Conceptos Generales (repaso tipo examen)

Relacionado: [[CIA-Triad]] · [[Principios-de-Seguridad-CIA-DAD]] · [[Roles-Ciberseguridad]] · [[Ofensiva-Pentesting]] · [[TCP-UDP-Puertos]] · [[SQL-Injection-Fundamentos-y-Explotacion]] · [[IDS-Snort]] · [[Reconocimiento-Pasivo]] · [[Reconocimiento-Activo]] · [[Nmap-Escaneo-Puertos]]

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

## Privacidad y protección de datos personales
- **Completar un perfil profesional (redes/plataformas de trabajo)**: llenar solo campos "seguros" — **foto de perfil** y **nombre de la organización**. Evitar llenar correo/celular/fecha de nacimiento/datos de colegas o del gerente si no son obligatorios — son canales directos de contacto o datos de terceros que aumentan el riesgo de spam, robo de identidad o ingeniería social dirigida.
- **Wifi pública/abierta (ej. tren, café) para trabajo**: conectarse siempre a través de la **VPN corporativa** del dispositivo de trabajo — cifra todo el tráfico en un túnel, así que aunque la red sea abierta, un atacante en esa misma red no puede leer los datos dentro del túnel.
- **Verificar si una app legítima necesita actualización**: ir directo al **sitio web oficial** del desarrollador a comprobar, en vez de confiar en un botón/enlace de actualización que podría ser phishing.
- **Cerrar una ventana emergente (popup) de forma segura**: clic en la **"X"** de la ventana — minimizar la deja corriendo en segundo plano, y "recordarme más tarde" solo pospone el mensaje sin cerrarla.
- **Analizar un sistema que aún no muestra daño**: correr un análisis con el **antivirus ya instalado y de confianza** — sin descargar nada nuevo ni de fuente dudosa.
- **Ocultar el historial de navegación en una computadora compartida**: usar el **modo privado/incógnito** del navegador — no guarda historial, cookies ni datos de formularios localmente (a diferencia de HTTPS, que solo protege el tránsito, no lo que queda guardado en disco).
- **Evitar que otras personas que viven contigo accedan a tus datos** (mismo dispositivo/red, no amenaza externa): **configurar protección por contraseña** en el dispositivo/cuenta — firewall, antivirus o privacidad del navegador no aplican a una amenaza de acceso físico compartido.

## Contraseñas y autenticación

### Administradores de contraseñas
Aplicación que almacena y cifra las contraseñas en una bóveda protegida por una única contraseña maestra — genera claves largas y aleatorias por cuenta, las guarda cifradas, las autocompleta al iniciar sesión, y puede sincronizarlas entre dispositivos. Beneficio: evita memorizar o reutilizar contraseñas débiles, reduce el riesgo de que una sola filtración comprometa varias cuentas a la vez, y facilita usar una clave única y fuerte por sitio. Es la solución recomendada cuando cuesta trabajo recordar contraseñas de muchas cuentas — nunca anotar las contraseñas en texto plano, reutilizar una sola clave en todos lados, ni compartirlas con soporte técnico.

### Técnicas de descifrado de contraseñas (computacionales, sobre el hash)
- **Pulverización (password spraying)**: probar una sola contraseña común contra muchas cuentas distintas (evita bloqueos por intentos fallidos en una sola cuenta).
- **Mesas arcoíris (rainbow tables)**: tablas precomputadas de hashes para revertir un hash a su valor original sin calcular cada intento en tiempo real.
- **Ataque de diccionario**: probar una lista de palabras/contraseñas comunes.
- **Fuerza bruta**: probar sistemáticamente todas las combinaciones posibles de caracteres.

> Ingeniería social e intimidación no son técnicas de "descifrado" — son formas de conseguir que la persona **revele** la contraseña directamente, no de calcular el valor a partir del hash.

### OAuth (Autorización Abierta)
Tecnología que genera un **token de autorización** para iniciar sesión en una app/servicio usando las credenciales de otra plataforma (Google, Facebook, etc.), sin compartir la contraseña original con el segundo servicio. Ejemplo típico: iniciar sesión en un servicio de almacenamiento en la nube y, al querer imprimir fotos vía un servicio de impresión de terceros, obtener acceso automático sin volver a loguearse — esto ocurre porque el servicio de impresión ya está **aprobado/integrado** con el de almacenamiento vía un token OAuth intercambiado entre ambos.

## Dispositivos y arquitectura de seguridad (ejemplos Cisco)
| Dispositivo | Categoría | Función |
|---|---|---|
| ISR 4000 | Router | enrutamiento, filtrado y cifrado en una sola plataforma |
| Firepower 4100 | Firewall | visibilidad de lo que pasa en la red para reaccionar rápido ante un ataque |
| AnyConnect Secure Mobility Client | VPN | acceso remoto seguro desde cualquier dispositivo/lugar |
| AMP (Advanced Malware Protection) | Protección contra malware | protección de terminales, análisis y monitoreo constante de archivos |

## Firewalls — tipos y cuándo aplica cada uno
Definición general: dispositivo que **controla/filtra el tráfico que entra o sale de una red**. Un IPS es más especializado (firmas de ataque específicas); un router enruta, no filtra por seguridad; una VPN cifra túneles, no filtra tráfico en general.

| Tipo de firewall | Escenario típico |
|---|---|
| **NAT** | una LAN pequeña necesita salida a Internet compartiendo una sola IP pública entre varios dispositivos internos |
| **Basado en host** | controlar qué apps de ese equipo específico aceptan conexiones entrantes (ej. el firewall de Windows bloqueando apps que reciben conexiones de otros equipos de la red) |
| **Servidor proxy** | filtrar contenido según URL/dominio (ej. bloquear sitios de apuestas para empleados) |
| **Aplicaciones sensibles al contexto** | filtra combinando usuario, dispositivo, rol, tipo de aplicación y perfil de amenaza a la vez — el más granular |
| **Capa de aplicación** | filtra por tipo de app/protocolo, pero sin combinar todas las variables anteriores |
| **Capa de red** | el más básico — solo IP/puerto |

> **"Hoy existe un dispositivo único que resuelve todas las necesidades de seguridad de red"** → **Falso**. La seguridad efectiva se basa en **defensa en profundidad**: varios dispositivos/controles especializados trabajando en capas, nunca uno solo.

## IDS / IPS / DLP / SIEM — quién hace qué
| Sigla | Qué hace |
|---|---|
| **IPS** | **bloquea o niega** tráfico activamente, según regla positiva o coincidencia de firma |
| **IDS** | analiza tráfico contra una base de reglas/firmas, **solo detecta y alerta** al admin — no bloquea (ver [[IDS-Snort]] para el detalle práctico con Snort) |
| **DLP** (Data Loss Prevention) | sistema para evitar el robo/fuga de datos confidenciales |
| **SIEM** | recopila y correlaciona alertas, logs y datos históricos/en tiempo real de otros dispositivos de seguridad — no genera las alertas originales, las agrega |

**Diferencia clave IPS vs IDS**: el IPS actúa (bloquea), el IDS solo observa y avisa.

## Seguridad de red inalámbrica y Bluetooth
- **Ocultar el SSID de un router NO es una medida de seguridad real** — es "seguridad por oscuridad"; el SSID sigue siendo detectable con herramientas de escaneo inalámbrico. Medidas que sí cuentan: habilitar cifrado inalámbrico (WPA2+), cambiar las credenciales admin por default.
- **Mejor defensa contra ataques Bluetooth** (bluejacking, bluesnarfing, bluebugging): **desactivar Bluetooth siempre que no se esté usando activamente** — elimina la superficie de ataque en vez de solo mitigarla. Una VPN no interviene en conexiones Bluetooth.

## NetFlow — protocolo de visibilidad de tráfico
Protocolo (originado en Cisco) diseñado específicamente para recopilar **metadatos de flujo** de red: quién habló con quién, cuánto volumen, cuándo, con qué protocolo — sin capturar el contenido real del tráfico. Distinto de HTTPS (cifrado del tráfico web en sí), Telnet (acceso remoto sin cifrar) y NAT (traducción de direcciones).

## Riesgos de dispositivos IoT (IdC)
El mayor riesgo: **la mayoría de los dispositivos IoT no reciben actualizaciones de software frecuentes** — quedan con vulnerabilidades sin parchear durante mucho tiempo, siendo el eslábón más débil de la red (ej. la botnet Mirai). No es que no usen Internet (sí lo requieren), ni un tema de cifrado inalámbrico inherente — aislarlos en su propia red/VLAN es una **recomendación** de mitigación, no una limitación técnica del dispositivo.

## Backups en la nube — beneficio clave
Elimina los costos de mantenimiento y equipo propio: el proveedor mantiene toda la infraestructura física, el usuario solo paga por el uso. Un disco externo, NAS, o cinta magnética requieren compra, mantenimiento y reemplazo directos por parte del usuario.

## DDoS y "zombis"
- **DDoS**: ataque desde **múltiples orígenes distintos simultáneamente** (distribuido) — se distingue de un DoS simple (un solo origen) por eso mismo. Señal típica: un servidor recibiendo una cantidad anormalmente alta de solicitudes desde ubicaciones diferentes al mismo tiempo.
- **Zombi**: equipo infectado y controlado remotamente sin que su dueño lo sepa. Un conjunto de zombis forma una **botnet**, típicamente usada para lanzar ataques DDoS.

## Cifrado de disco — proteger datos en reposo
Para proteger datos almacenados localmente contra acceso no autorizado (incluso si roban el disco físico), el método correcto es el **cifrado de datos** — sin la clave, los datos son ilegibles aunque el atacante tenga el hardware en la mano. Duplicar el disco es solo backup (no protege contra lectura no autorizada); eliminar datos implica perderlos; 2FA protege el login de cuentas, no datos ya extraídos físicamente del disco.

## Gestión de riesgos
**Definición**: el proceso de **identificar y evaluar riesgos para reducir el impacto** de amenazas y vulnerabilidades — incluye el propósito final (reducir impacto), no solo detectar/evaluar. Transferencia y aceptación son solo dos estrategias específicas de respuesta al riesgo, no la definición completa del proceso.

> **"Con planificación cuidadosa, algunos riesgos se pueden eliminar por completo"** → **Verdadero**. Una de las estrategias de gestión de riesgo es **evitarlo** (dejar de hacer la actividad que lo genera por completo) — para esa actividad específica, el riesgo asociado desaparece del todo.

## Security Playbook (manual de estrategias)
Colección de consultas/informes **repetibles** que describen un proceso **estandarizado** para la detección y respuesta a incidentes — los procedimientos predefinidos que sigue un equipo de seguridad (SOC). No es documentación genérica de TI, ni los datos/alertas en sí (eso es lo que recopila un SIEM) — es el "libro de jugadas" que dice qué hacer paso a paso ante cada tipo de incidente.

## Ética y marco legal en ciberseguridad
- **Diferencia entre un hacker y un profesional de ciberseguridad**: no está en las habilidades técnicas (pueden ser idénticas) — está en el **marco legal/ético**: autorización explícita, alcance definido, propósito defensivo.
- **Para qué sirven las habilidades de un profesional de ciberseguridad**: son de **"doble uso"** — se pueden usar para hacer el bien o para hacer daño, dependiendo de la intención y autorización de quien las aplica, no de la habilidad en sí misma.
- **Responsabilidad vicaria/corporativa**: si un empleado hace algo ilegal actuando **como representante de la organización** (dentro de su función), la organización puede ser **legalmente responsable**, no solo el individuo.
- **Compartir propiedad intelectual de un ex-empleador**: un empleado que, tras dejar una empresa, comparte documentos/ideas que propuso mientras trabajaba ahí con su nuevo empleador actúa de forma **poco ética** — esa información se desarrolló dentro y para la organización original (propiedad intelectual/confidencial de la empresa), sin importar quién la propuso individualmente; compartirla rompe confidencialidad y lealtad aunque el empleado ya no trabaje ahí. (Distinto del ejemplo de ética de arriba con la reunión interna TI↔Marketing — la diferencia es divulgar información **fuera** de la organización dueña de esa información, no dentro de ella.)

## Metodología de pruebas de penetración — las 5 fases
1. **Planificación**: recopilar la mayor cantidad de información posible sobre el objetivo — reconocimiento pasivo o activo (footprinting), ver [[Reconocimiento-Pasivo]] y [[Reconocimiento-Activo]].
2. **Escaneo**: identificar posibles debilidades explotables, ver [[Nmap-Escaneo-Puertos]].
3. **Obtener acceso**: aprovechar una vulnerabilidad identificada, simulando un ataque real.
4. **Mantener el acceso**: sin ser detectado, seguir recopilando información sobre las vulnerabilidades del objetivo.
5. **Reporte / análisis e informes**: comunicar los hallazgos a la organización para que se implementen mejoras de seguridad — siempre la última etapa.

> La frase "sin que lo detecten" corresponde a **Mantener el acceso**, no a Planificación — es la pista que distingue ambas fases entre sí en un examen.

## Certificaciones de ciberseguridad — comparativa rápida
| Certificación | Enfoque | Nivel |
|---|---|---|
| **CompTIA Security+** | conceptos generales de seguridad, redes, operaciones defensivas — vendor-neutral | Entrada / inicial |
| **EC-Council CEH** (Certified Ethical Hacker) | metodología ofensiva: encontrar vulnerabilidades con las mismas técnicas/herramientas que un atacante malicioso, pero de forma legal/ética | Intermedio / especializado (pentesting) |
| **ISC2 CISSP** | gestión estratégica, arquitectura, gobernanza y liderazgo en seguridad — requiere ~5 años de experiencia en 2+ dominios | Avanzado / gerencial |
| **MTA Security Fundamentals / Palo Alto PCCSA / ISACA CSX Fundamentals** | introductorias o específicas de un producto/fabricante (vendor-specific) | Introductorio |

**Para empezar la carrera**: CompTIA Security+ es el estándar de referencia como certificación de entrada. **Para validar habilidades ofensivas/pentesting específicamente**: CEH es la que evalúa exactamente esa metodología (encontrar debilidades con las mismas herramientas que un atacante, de forma legal). Ver también [[Roles-Ciberseguridad]] para el panorama de roles y valor del training.
