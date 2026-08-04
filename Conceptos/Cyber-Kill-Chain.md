# Cyber Kill Chain (Lockheed Martin)

Relacionado: [[Ofensiva-Pentesting]] · [[Incident-Response]] · [[Herramientas-Metasploit]] · [[SOC-Centro-de-Operaciones]]

## Qué es
Framework de 2011 (inspirado en la cadena de mando militar) que divide un ciberataque en **7 etapas**. Si la organización entiende cada etapa, tiene mejor oportunidad de **romper la cadena** e interrumpir el ataque en progreso, en vez de esperar a la etapa final.

## Las 7 etapas

### 1. Reconnaissance
El atacante recolecta información sobre el objetivo (equivalente a la fase de enumeración de un pentest — ver [[Ofensiva-Pentesting]]).

### 2. Weaponisation
Con la info del reconocimiento, el atacante crea o modifica un payload adaptado a las vulnerabilidades descubiertas. Puede usar un **exploit kit** (plataforma automatizada con exploits para distintas vulnerabilidades) para empaquetar el exploit dentro de un archivo (ej. ejecutable, documento Office con macros maliciosas).

**Ejemplos**: documentos Office con macros maliciosas (se ejecutan si las macros están habilitadas) · obfuscación/cifrado del payload para evadir detección · ocultarlo en un archivo inocuo (Word, PDF).

**Contramedidas**: entrenamiento continuo a usuarios (inspeccionar adjuntos, revisar el origen del email) · deshabilitar macros por defecto o restringirlas a fuentes firmadas/confiables (via Group Policy) · deshabilitar features/software/plugins innecesarios para reducir superficie de ataque.

### 3. Delivery
Transmitir el payload preparado al entorno objetivo.

**Ejemplos**: **phishing emails** (adjuntos maliciosos, ej. `invoice.pdf.exe`) · **spear phishing** (dirigido, suplantando a alguien conocido/confiable) · enlaces maliciosos (domain spoofing, URL shortening) · plataformas de compartir archivos · **malvertising** (anuncios en sitios legítimos que redirigen a páginas maliciosas) · **smishing** (SMS con enlaces/instrucciones maliciosas) · ingeniería social · medios físicos (USB dejado en un lugar accesible, DVD por correo con pretexto convincente).

**Contramedidas**: entrenamiento de conciencia de seguridad · filtrado de email y web · **WAF** (ver [[Firewall-Fundamentos]]) · monitoreo de red · gestión de parches.

### 4. Exploitation
El payload entregado explota una vulnerabilidad en el sistema objetivo (esto es lo que cubren la mayoría de las salas técnicas de explotación, ver [[Metodologia-Pentest-Web-Chain]] e [[Infra-Pentest-Metasploit-Ejemplo]]).

### 5. Installation
Instalar malware/backdoor para mantener **persistencia** — poder volver al sistema sin repetir la explotación.

**Ejemplos**: tareas programadas (Windows) o cron jobs (Linux) · modificar scripts de arranque/config · instalar un nuevo servicio (Windows) o daemon (Linux) · **LOLBins** (Living-Off-the-Land Binaries — abusar de herramientas legítimas ya presentes en el sistema) · desplegar un **web shell** (script que ejecuta comandos del SO vía navegador; correrlo sobre HTTPS lo camufla dentro del tráfico normal).

**Contramedidas**: monitorear nuevos procesos/servicios (analizar el proceso padre y actividad asociada) · **EDR** (Endpoint Detection and Response) · auditorías regulares contra una baseline segura · **application allowlisting** (solo software aprobado puede ejecutarse).

### 6. Command and Control (C2)
Establecer un canal de comunicación encubierto entre el sistema comprometido y la infraestructura del atacante.

**Ejemplos**: usar protocolos de capa de aplicación comunes (HTTP, HTTPS, DNS, SMTP) para mezclarse con tráfico legítimo · cifrado (HTTPS) para ocultarse del monitoreo de red · **DNS tunnelling** (codificar datos dentro de consultas DNS para evadir firewalls) · usar plataformas de redes sociales o servicios cloud legítimos (Dropbox, Google Docs) para exfiltración · **DGA** (Domain Generation Algorithm — generar miles de dominios candidatos, registrar solo unos pocos; si uno se bloquea, el malware prueba el siguiente) · **Fast Flux** (asociar cientos/miles de IPs a un mismo dominio, rotándolas cada pocos minutos, usando dispositivos comprometidos como proxies).

**Contramedidas**: monitoreo de red vía firewall/IDS/IPS (ver [[IDS-Snort]]) · monitorear tráfico DNS (queries anormalmente largas o volumen alto a dominios sospechosos) · content filtering y monitoreo de tráfico web · inspección de tráfico cifrado · honeypots.

### 7. Actions on Objectives
El atacante ejecuta su objetivo final.

**Ejemplos**: ataque destructivo (borrar/corromper datos) · **ransomware** (cifrar archivos y exigir pago por la clave de descifrado) · fraude financiero (transferencias no autorizadas) · **data exfiltration** (robo de archivos sensibles — espionaje industrial/político) · **lateral movement** (comprometer otros sistemas de la red) · manipulación de sistemas ICS (control industrial).

**Contramedidas**: **DLP** (Data Loss Prevention) · plan de backup y recuperación sólido (mitiga ransomware) · **network segmentation** y **principio de mínimo privilegio** (limita el daño si un sistema se compromete — ver [[Principios-de-Seguridad-CIA-DAD]]) · monitoreo de actividad de usuario (ej. queries DNS a medianoche) · EDR en los endpoints.

## Por qué importa
- **Blue Team**: detectar y bloquear las acciones del atacante en **cualquier** etapa temprana rompe la cadena antes del daño mayor.
- **Red Team / pentesters**: seguir los pasos de adversarios reales garantiza que la simulación de ataque sea realista, y permite probar si las defensas y la capacidad de detección del equipo funcionan de verdad.

## Resumen visual
```
1. Reconnaissance   → recolectar info del objetivo
2. Weaponisation    → crear el payload malicioso
3. Delivery         → enviar el payload al objetivo
4. Exploitation     → el payload explota una vulnerabilidad
5. Installation     → instalar backdoor/malware (persistencia)
6. Command&Control  → canal encubierto atacante↔víctima
7. Actions on Objectives → exfiltración, ransomware, destrucción...
```
