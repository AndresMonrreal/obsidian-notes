# TCP, Banderas y Escaneo de Puertos — Fundamentos y Riesgo Real

Relacionado: [[Nmap-Escaneo-Puertos]] · [[Nmap-Escaneo-Avanzado-y-Evasion]] · [[Firewall-Fundamentos]] · [[IDS-Snort]] · [[Cyber-Kill-Chain]] · [[Defensiva-Blue-Team]] · [[Logs-y-SIEM]]

Síntesis de todo lo visto en las salas de Nmap/redes: qué pasa exactamente a nivel de protocolo, por qué existen tantas variantes de scan, y — lo más importante — **qué tan peligroso es cada cosa en el mundo real**, qué puede perder una empresa, y cómo se detecta que algo está mal.

## 1. ¿Qué es TCP y por qué importa tanto?
TCP (Transmission Control Protocol) es el protocolo que garantiza que los datos lleguen **completos, en orden y confirmados** entre dos sistemas — a diferencia de UDP, que manda datos "y ya", sin garantías. Casi todo lo que se usa a diario corre sobre TCP: HTTP/HTTPS (web), SSH, FTP, SMTP (correo), bases de datos, etc. Por ser tan universal, entender su funcionamiento interno es la base de:
- Cómo un pentester descubre qué hay corriendo en un servidor.
- Cómo un atacante real explota debilidades del protocolo mismo (no solo de las aplicaciones que corren encima).
- Cómo un defensor detecta que alguien está "tocando la puerta".

## 2. El header TCP y las 6 banderas — a fondo
Cada segmento TCP lleva 24 bytes de header antes de los datos. Dentro de ese header, 6 bits controlan el "estado" de la comunicación:

| Flag | Nombre completo | Uso normal | Qué implica si aparece "fuera de contexto" |
|---|---|---|---|
| **SYN** | Synchronize | Inicia una conexión nueva, sincroniza números de secuencia | Único flag legítimo para *empezar* algo — cualquier sistema que reciba un SYN sin conexión previa lo interpreta como "alguien quiere conectarse" |
| **ACK** | Acknowledge | Confirma que se recibió un segmento anterior | Un ACK sin conexión previa es "ilegal" — el sistema no tiene nada que reconocer, así que responde RST. Por eso `-sA`/`-sW` requieren privilegios: hay que forjar un paquete que rompe el protocolo a propósito |
| **PSH** | Push | Pide que los datos se entreguen a la aplicación de inmediato, sin esperar a llenar el buffer | Rara vez se usa sola en scanning; aparece combinada en el Xmas scan |
| **RST** | Reset | Tumba la conexión de golpe (error, puerto cerrado, o alguien cortó la comunicación) | Respuesta "default" del SO cuando llega algo que no puede procesar — la señal universal de "puerto cerrado" en casi todos los scans |
| **FIN** | Finish | El emisor avisa que ya no tiene más datos que mandar (cierre ordenado) | Igual que ACK, mandarlo sin conexión previa es "fuera de protocolo" — de ahí el FIN scan |
| **URG** | Urgent | Marca datos urgentes que deben procesarse antes que el resto | El menos usado en la práctica moderna; en scanning solo aparece en el combo del Xmas scan |

**El patrón que conecta todo:** cada técnica de scan (SYN, ACK, Window, Null, FIN, Xmas, Maimon — ver detalle completo en [[Nmap-Escaneo-Avanzado-y-Evasion]]) es simplemente **una combinación distinta de estas 6 banderas**, aprovechando que TCP (RFC 793) define un comportamiento esperado para paquetes "normales", pero deja zonas grises para combinaciones que nunca deberían pasar en tráfico real.

### Bonus: cómo Nmap detecta el sistema operativo (`-O`)
El mismo principio de "cada sistema reacciona distinto ante paquetes fuera de protocolo" es la base del OS fingerprinting: Nmap compara TTL inicial, tamaño de ventana TCP, orden/valores de opciones TCP y el comportamiento exacto ante paquetes imposibles contra una base de datos de miles de huellas conocidas — de ahí que Windows, Linux y sistemas BSD reaccionen distinto ante el mismo paquete (la nota sobre el Maimon scan y sistemas BSD es exactamente este principio en acción). Requiere privilegios y, idealmente, al menos un puerto abierto y uno cerrado para tener información suficiente que comparar. Comando: `sudo nmap -O IP`.

## 3. El 3-way handshake, paso a paso
```
Cliente                          Servidor
   |------ SYN (seq=x) ------------->|     Paso 1: "quiero conectar"
   |<--- SYN-ACK (seq=y, ack=x+1) ---|     Paso 2: "ok, aquí está mi secuencia también"
   |------ ACK (ack=y+1) ----------->|     Paso 3: "confirmado, empezamos"
```
- Cada lado elige un **número de secuencia inicial aleatorio** (por seguridad — si fuera predecible, alguien podría inyectar o secuestrar la conexión adivinando los números).
- La conexión no se considera "establecida" hasta completar los 3 pasos.
- **Base de las técnicas de escaneo:**
  - **TCP Connect scan (`-sT`)** completa los 3 pasos — es "honesto", pero por eso **siempre queda registrado** en los logs del sistema objetivo (el SO ve una conexión completa, como cualquier app real).
  - **TCP SYN scan (`-sS`)** se detiene después del paso 2 (manda RST en vez del ACK final) — nunca hay una conexión "oficialmente establecida" a nivel de aplicación, por lo que **muchos sistemas de logging a nivel de aplicación nunca se enteran** (aunque un firewall/IDS decente sí puede verlo a nivel de paquete).

## 4. Qué tan peligroso es cada tipo de scan (y para quién)
Separar dos perspectivas: **qué tan peligroso es *que te hagan* un scan**, y **qué tan peligroso es *hacer* un scan sin autorización**.

### 4.1 — El escaneo en sí mismo casi nunca "rompe" nada
Un SYN scan, ACK scan, etc. normalmente **no representan daño directo** — son solo paquetes de prueba, no explotan ninguna vulnerabilidad por sí mismos. El riesgo real no es el escaneo, es **lo que el atacante hace con la información que obtiene**. El escaneo es la fase de **reconocimiento** (la primera etapa de casi cualquier ataque real — ver [[Cyber-Kill-Chain]]) — el mapa antes de decidir por dónde entrar.

### 4.2 — Excepción real: SYN Flood (esto sí es un ataque, no reconocimiento)
Aquí el 3-way handshake se vuelve un arma en vez de solo información. Un **SYN flood** abusa exactamente del mismo mecanismo estudiado:
1. El atacante manda una avalancha de paquetes SYN al servidor, muchas veces con **IPs de origen falsificadas** (spoofed) — la misma técnica `-S` vista en [[Nmap-Escaneo-Avanzado-y-Evasion]], pero con intención maliciosa.
2. El servidor responde con SYN-ACK a cada uno y **reserva memoria/recursos** para cada conexión "medio abierta", esperando el ACK final que nunca llega (la IP de origen ni siquiera existe, o el atacante simplemente nunca lo manda).
3. Con suficiente volumen, la cola de conexiones medio-abiertas se llena por completo — el servidor se queda sin recursos (memoria, slots de conexión) para atender a usuarios reales.
4. Resultado: **denegación de servicio (DoS)** — el sitio/servicio se cae o deja de responder, sin que el atacante haya "hackeado" nada en el sentido tradicional, solo agotó recursos.

> Dato histórico: uno de los primeros SYN floods documentados públicamente fue el ataque de 1996 contra **Panix** (un proveedor de internet de Nueva York) — el incidente que popularizó la técnica y aceleró el diseño y adopción de **SYN cookies** como defensa estándar poco después.

**Cómo se defienden las empresas de esto:**
- **SYN cookies**: en vez de reservar memoria de inmediato al recibir un SYN, el servidor codifica la información de la conexión directamente en el número de secuencia del SYN-ACK que manda — si el ACK de vuelta nunca llega, no se desperdició memoria real reservada.
- **Rate limiting**: limitar cuántos SYN por segundo se aceptan desde una misma IP/red.
- **Firewalls/WAFs con detección de anomalías**: identifican patrones de tráfico "no humano" (miles de SYN sin ACKs de vuelta) y bloquean la fuente.

### 4.3 — Qué información se filtra con cada scan, y por qué le importa a un atacante
| Lo que descubres | Qué le permite hacer a un atacante |
|---|---|
| Puerto abierto + servicio (`-sV`) | Saber exactamente qué software/versión atacar — buscar CVEs conocidos para esa versión específica |
| Banner de versión vieja (ej. `wkhtmltopdf 0.12.5`, ver [[Web-Pentest-SSRF-Ejemplo]]) | Buscar exploits públicos ya documentados para esa versión exacta |
| Puertos filtrados vs cerrados vs abiertos | Mapear las reglas del firewall — saber por dónde "sí lo dejan pasar" (ACK/Window scan, ver [[Nmap-Escaneo-Avanzado-y-Evasion]]) |
| Sistema operativo (fingerprinting, TTL, comportamiento de flags — `-O`) | Elegir exploits específicos de ese SO |
| Puerto "unfiltered" sin servicio real detrás | Puede ser pista falsa, pero en la vida real a veces revela **infraestructura mal desmantelada** — un servicio apagado cuya regla de firewall nadie cerró, dejando una puerta "medio abierta" reactivable |

### 4.4 — Cómo se expone realmente una empresa (el panorama completo)
El escaneo de puertos casi nunca es el ataque final — es el primer paso de una cadena:
1. **Reconocimiento**: descubrir hosts vivos, puertos abiertos, servicios y versiones (también vía fuentes pasivas como Shodan, ver [[Reconocimiento-Pasivo]] — un atacante no necesita ni escanear directo si el dato ya está indexado ahí).
2. **Identificación de vulnerabilidad**: cruzar esa información con bases de datos de CVEs, o buscar configuraciones default/débiles (ej. una clave nunca cambiada, o un endpoint protegido solo por IP de origen — errores de configuración, no fallas del protocolo).
3. **Explotación**: usar esa vulnerabilidad para entrar (SSRF, LFI, credenciales default, servicio desactualizado con exploit público, etc.).
4. **Movimiento lateral / escalación**: una vez dentro, usar las mismas técnicas de descubrimiento (ej. ARP scan interno, ver [[Reconocimiento-Activo]]) para mapear el resto de la red interna — el atacante ahora "vive" dentro de un segmento y puede alcanzar todo lo que antes estaba fuera de su alcance.
5. **Impacto real**: robo de datos, ransomware, uso del servidor como pivote para atacar a otros, o como bot para más ataques.

**El punto clave para una empresa:** cada puerto abierto innecesariamente, cada servicio con versión vieja, cada regla de firewall mal configurada es **superficie de ataque** — no implica que vayan a ser hackeados mañana, pero es una puerta que no debería estar ahí, y tarde o temprano alguien la va a probar (los atacantes automatizados escanean **todo internet** constantemente, no hace falta ser un blanco específico para que te encuentren).

## 5. Cómo se sabe que "algo está mal" — señales concretas
- **Puertos abiertos que no deberían ser accesibles desde internet**: bases de datos (3306 MySQL, 5432 PostgreSQL, 27017 MongoDB), RDP (3389), SMB (445 — el puerto detrás de exploits históricos ampliamente documentados como EternalBlue/MS17-010), paneles de administración — casi nunca deberían estar expuestos públicamente, solo internamente o vía VPN.
- **Versiones de software desactualizadas** en los banners, o servidores de desarrollo corriendo directo en producción (ej. Flask en modo debug sin reverse proxy, puerto 5000 expuesto tal cual) — mala práctica en sí misma.
- **Puertos "unfiltered" sin servicio real detrás** — indica reglas de firewall desactualizadas respecto a lo que realmente corre.
- **Inconsistencia entre lo que un SYN scan encuentra y lo que un ACK/Window scan revela** — expone las reglas reales del firewall, información que en teoría no debería ser tan fácil de mapear desde afuera.
- **Servicios respondiendo con información de más** (banners detallados, mensajes de error con paths internos, stack traces) — cada dato de más es una pista gratis.
- **Ausencia de rate limiting**: si se pueden mandar miles de paquetes sin bloqueo ni alerta, no hay defensas activas, solo el servicio "desnudo".

## 6. Del lado defensivo — qué debería tener una empresa
- **Firewall con política default-deny** (ver [[Firewall-Fundamentos]]): bloquear todo por default y solo abrir excepciones explícitas.
- **IDS/IPS** (ej. Snort, ver [[IDS-Snort]]): inspeccionan tráfico buscando patrones de escaneo (muchos SYN a puertos distintos desde una misma IP en poco tiempo, combinaciones de flags "imposibles" como Null/Xmas) y pueden alertar o bloquear automáticamente.
- **Logging centralizado (SIEM)** (ver [[Logs-y-SIEM]]): para correlacionar "alguien escaneó" con "algo pasó después" — sin logs, un ataque es invisible en retrospectiva.
- **Segmentación de red** (ver [[Subnetting]]): si un atacante compromete un segmento, la segmentación (routers, VLANs, firewalls internos) limita qué más puede alcanzar.
- **Gestión de parches**: mantener servicios actualizados para que aunque los encuentren, no haya un exploit conocido esperando.
- **Escanearse a sí mismo primero**: las empresas serias corren sus propios pentests/escaneos periódicos para encontrar estos problemas antes que un atacante externo.

## 7. Nota legal/ética
Toda práctica de estas técnicas debe hacerse contra **máquinas de laboratorio autorizadas** (TryHackMe, HTB, labs propios). Escanear o explotar sistemas reales sin permiso — aunque sea "solo un ping" o "solo un scan, no exploté nada" — es **ilegal en la gran mayoría de jurisdicciones** (en México, cae dentro de los delitos informáticos del Código Penal Federal), sin importar la intención. Regla de oro: **nunca apuntar ninguna de estas técnicas a un sistema que no sea propio o para el que no exista autorización explícita por escrito** (contrato de pentest, programa de bug bounty con alcance definido, o un lab diseñado para esto).
