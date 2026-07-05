---
tags:
  - índice
  - THM
  - TryHackMe
  - roadmap
  - certificaciones
fecha-inicio: 2026-06-27
ultima-actualizacion: 2026-06-27
estado: en-progreso
---

# 🗺️ TryHackMe — Base de Estudio y Certificaciones

Este vault es tu **Second Brain de Ciberseguridad**. Todo lo que aprendes en TryHackMe queda documentado aquí para que puedas repasar antes de exámenes, laboratorios o entrevistas técnicas.

> [!tip] Cómo sacarle el máximo
> - `Ctrl+Shift+F` → buscar cualquier término en todo el vault
> - **Graph View** → ver cómo se conectan los conceptos visualmente
> - Filtra por tags para estudiar por área (`#networks`, `#blue-team`, etc.)
> - Los `[[enlaces]]` te llevan directamente al concepto relacionado

---

## 📁 Estructura del Vault

```
THM/
├── README.md                     ← Estás aquí (índice + guía de estudio)
└── Conceptos/
    ├── Roles-Ciberseguridad.md
    ├── Sistemas-Hardware.md
    ├── OSI-Model.md
    ├── Redes-Fundamentos.md
    ├── Protocolos-ARP-DHCP.md
    ├── Subnetting.md
    ├── Packets-and-Frames.md
    ├── TCP-UDP-Puertos.md
    ├── Infraestructura-de-Red.md
    ├── Seguridad-de-Red.md
    ├── DNS.md
    ├── HTTP-Web.md
    ├── Cliente-Servidor-HTTP.md
    ├── Arquitectura-Web.md
    ├── Virtualizacion.md
    ├── Cloud-Computing.md
    ├── Sistemas-Operativos.md
    ├── Representacion-de-Datos.md
    ├── CIA-Triad.md
    ├── Criptografia.md
    ├── Ofensiva-Pentesting.md
    ├── Defensiva-Blue-Team.md
    ├── Linux-Permisos-y-Sistema.md
    ├── Linux-Herramientas-y-Admin.md
    ├── Linux-Scripting-y-Shells.md
    ├── Windows-CMD.md
    ├── Windows-PowerShell.md
    └── Protocolos-Aplicacion.md
```

---

## 📚 Notas por Módulo

### 🔐 Pre-Security — SEC0

#### Roles, Equipos y Hardware

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Roles-Ciberseguridad]] | Security Analyst, Blue/Red/Purple Team, SOC L1-L3, Threat Hunting, Malware Analysis | ✅ |
| [[Sistemas-Hardware]] | Boot process (UEFI/BIOS, POST, Bootloader), Laptop/Desktop/Workstation/Server, IoT vs Embedded | ✅ |

#### Modelo OSI y Fundamentos de Red

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[OSI-Model]] | 7 capas, TCP/IP de 4 capas, encapsulamiento, TCP vs UDP, puertos comunes | ✅ |
| [[Redes-Fundamentos]] | IP vs MAC, IPv4, octetos, topologías Bus/Star/Mesh | ✅ |
| [[Protocolos-ARP-DHCP]] | ARP request/reply, DHCP DORA, ICMP/ping, encapsulamiento | ✅ |
| [[Subnetting]] | Subnet mask, CIDR, Network/Host/Broadcast/Gateway, cálculo de hosts | ✅ |

#### Packets, Protocolos de Transporte y Puertos

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Packets-and-Frames]] | Packet vs Frame, headers IP (TTL, Checksum, Source/Dest), TTL por SO | ✅ |
| [[TCP-UDP-Puertos]] | TCP/IP model, headers TCP y UDP, Three-Way Handshake, cierre FIN, rango de puertos (2 octetos = 65535), vida de un paquete (7 pasos end-to-end) | ✅ |

#### Infraestructura y Seguridad de Red

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Infraestructura-de-Red]] | Router, routing, port forwarding, Switch L2 vs L3, VLAN | ✅ |
| [[Seguridad-de-Red]] | Firewall stateful vs stateless, VPN (site-to-site vs remote-access vs comercial), geo-bypass, DNS leak, split tunneling, legalidad, PPP/PPTP/IPSec/OpenVPN/WireGuard | ✅ |

#### Web y DNS

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[DNS]] | TLD/SLD/Subdomain, registros A/AAAA/CNAME/MX/TXT, flujo de resolución DNS | ✅ |
| [[HTTP-Web]] | HTTP vs HTTPS, URL, métodos GET/POST/PUT/DELETE, status codes, headers, cookies | ✅ |
| [[Arquitectura-Web]] | Front/Back end, Web Servers, Virtual Hosts, Load Balancer, CDN, Databases, WAF, HTML Injection | ✅ |
| [[Cliente-Servidor-HTTP]] | Modelo cliente-servidor, los 9 métodos HTTP, GET request/response en detalle, stateless + cookies | ✅ |
| [[Virtualizacion]] | Por qué existe, Hypervisor Type 1 vs 2, VMs, Containers vs VMs, Docker | ✅ |
| [[Cloud-Computing]] | Beneficios, Public/Private/Hybrid, IaaS/PaaS/SaaS, vendors AWS/Azure/GCP, responsabilidad compartida | ✅ |
| [[Sistemas-Operativos]] | Qué es un OS, kernel/user space, system calls, responsabilidades (process/memory/file/user/device mgmt), GUI vs CLI, tipos y familias de OS | ✅ |
| [[Representacion-de-Datos]] | Bits/bytes, RGB y colores, binario, hexadecimal, octal, ASCII, Unicode, UTF-8/16/32 | ✅ |
| [[CIA-Triad]] | Confidentiality, Integrity, Availability — definiciones, técnicas de protección, ataques que afectan cada pilar, tips de examen | ✅ |
| [[Criptografia]] | Plaintext/ciphertext/key/algorithm, cifrado simétrico (César, AES), asimétrico (RSA, clave pública/privada), historia SSL/TLS (1995→2018), CSR→CA→certificado, self-signed certs, protocolos TLS (HTTP→HTTPS, SMTP→SMTPS, etc.), HTTPS 3-pasos vs HTTP 2-pasos | ✅ |

#### Seguridad Ofensiva y Defensiva

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Ofensiva-Pentesting]] | Red Team vs Pentesting, scope, mentalidad del atacante, Gobuster (directory enumeration), Hydra (dictionary attack), cadena de dominós, fases de un pentest | ✅ |
| [[Defensiva-Blue-Team]] | 5 pilares (Prevention/Detection/Mitigation/Analysis/Response), infraestructura del cliente, defensa en profundidad, SIEM/IDS/EDR, SOC L1-L3, Threat Intelligence, DFIR, 6 fases de IR | ✅ |

#### Linux Fundamentals

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Linux-Permisos-y-Sistema]] | Permisos simbólicos (ls -l), formato numérico (chmod 755/644/700), usuarios y grupos, `su` vs `su -l`, directorios /etc /var /root /tmp | ✅ |
| [[Linux-Herramientas-y-Admin]] | Nano y VIM (editores), wget, SCP, Python HTTP Server, crontab (formato MIN HOUR DOM MON DOW CMD), apt y repositorios, GPG keys, logs (/var/log, Apache access.log, auth.log) | ✅ |
| [[Linux-Scripting-y-Shells]] | Bash vs Fish vs Zsh (comparativa), comandos básicos (pwd/cd/ls/cat/grep), scripting: shebang, variables, `read`, loops `for`, condicionales `if/elif/else/fi`, comentarios, `chmod +x`, `./` | ✅ |

#### Windows Fundamentals

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Windows-CMD]] | `ver`/`systeminfo`, `ipconfig`/`ipconfig /all` (MAC address), `ping`, `tracert`, `nslookup`, `netstat -abon`, `dir`/`tree`/`mkdir`/`rmdir`, `type`/`copy`/`move`/`del`, `tasklist`/`taskkill`, pipe con `more` | ✅ |
| [[Windows-PowerShell]] | Verb-Noun cmdlets, Get-Command/Help/Alias, sistema de archivos (Get-ChildItem/Set-Location/New-Item), piping con Sort-Object/Where-Object (-eq/-ne/-gt/-like)/Select-Object/Select-String, Get-Process/Get-Service/Get-NetTCPConnection/Get-FileHash, ADS, Invoke-Command | ✅ |

#### Protocolos de Aplicación

| Nota | Temas clave | Estado |
|------|-------------|--------|
| [[Protocolos-Aplicacion]] | TELNET (port 23, test de protocolos), FTP (port 21, login anónimo, SFTP/FTPS), SMTP (port 25, sesión manual), POP3 (port 110), IMAP (port 143), WHOIS, **SSH** (Tatu Ylönen 1995, OpenSSH, port 22, clave pública, X11, hydra), **SMTPS/POP3S/IMAPS** (puertos TLS: 465/587/995/993, STARTTLS), tabla completa, flujo email | ✅ |

---

## 🎓 Guía de Estudio por Certificación

Las notas de este vault cubren contenido que aparece directamente en los siguientes exámenes. Úsalo como checklist de repaso.

### CompTIA Network+ (N10-009)

| Dominio | Notas relevantes | Peso en el examen |
|---------|-----------------|-------------------|
| Networking Concepts | [[OSI-Model]], [[Redes-Fundamentos]], [[Packets-and-Frames]] | ~23% |
| Network Implementation | [[Infraestructura-de-Red]], [[Subnetting]], [[TCP-UDP-Puertos]] | ~19% |
| Network Operations | [[DNS]], [[Protocolos-ARP-DHCP]], [[HTTP-Web]] | ~17% |
| Network Security | [[Seguridad-de-Red]], [[Roles-Ciberseguridad]], [[Defensiva-Blue-Team]] | ~20% |
| Network Troubleshooting | [[Protocolos-ARP-DHCP]] (ping/ICMP), [[TCP-UDP-Puertos]] | ~21% |

**Conceptos clave a dominar para Network+:**
- Modelo OSI de memoria (los 7 layers, qué va en cada uno)
- Subnetting rápido con CIDR (calcular hosts, network/broadcast address)
- Diferencia TCP vs UDP y sus casos de uso
- Puertos comunes: 21, 22, 23, 25, 53, 80, 443, 445, 3389
- Tipos de registros DNS y para qué sirve cada uno
- Switch L2 vs L3, VLANs, port forwarding

---

### CompTIA Security+ (SY0-701)

| Dominio | Notas relevantes | Peso en el examen |
|---------|-----------------|-------------------|
| General Security Concepts | [[Roles-Ciberseguridad]], [[CIA-Triad]], [[Criptografia]] | ~12% |
| Threats, Vulnerabilities & Mitigations | [[Seguridad-de-Red]], [[Ofensiva-Pentesting]], [[Protocolos-ARP-DHCP]] | ~22% |
| Security Architecture | [[Infraestructura-de-Red]], [[Seguridad-de-Red]], [[Defensiva-Blue-Team]] | ~18% |
| Security Operations | [[Roles-Ciberseguridad]], [[Defensiva-Blue-Team]], [[HTTP-Web]] | ~28% |
| Security Program Management | [[Roles-Ciberseguridad]], [[CIA-Triad]] | ~20% |

**Conceptos clave a dominar para Security+:**
- Firewall stateful vs stateless y cuándo usar cada uno
- VPN: diferencias entre PPTP, IPSec, OpenVPN, WireGuard
- ARP Spoofing / ARP Poisoning como vector de ataque MitM
- Flags de seguridad en cookies (HttpOnly, Secure, SameSite)
- HTTP vs HTTPS y por qué importa el cifrado TLS
- Segmentación de red con VLANs como técnica defensiva

---

### CEH (Certified Ethical Hacker) / eJPT

| Área | Notas relevantes |
|------|-----------------|
| Reconocimiento de red | [[DNS]], [[TCP-UDP-Puertos]], [[Redes-Fundamentos]], [[Ofensiva-Pentesting]] (Gobuster), [[Windows-CMD]] (nslookup/netstat) |
| Escaneo y enumeración | [[Packets-and-Frames]], [[TCP-UDP-Puertos]], [[Infraestructura-de-Red]], [[Windows-PowerShell]] (Get-NetTCPConnection) |
| Ataques de credenciales | [[Ofensiva-Pentesting]] (Hydra, dictionary attack) |
| Ataques de red | [[Protocolos-ARP-DHCP]] (ARP Spoofing), [[Seguridad-de-Red]] |
| Ataques web | [[HTTP-Web]], [[DNS]], [[Arquitectura-Web]] (HTML Injection) |
| Post-explotación / Scripting ofensivo | [[Linux-Scripting-y-Shells]], [[Windows-PowerShell]] (Invoke-Command) |
| Evasión de defensas | [[Seguridad-de-Red]] (Firewall bypass), [[OSI-Model]] |

---

## 🔥 Cheatsheet Rápido — Repaso Express

Antes de un examen, repasa estos puntos en orden:

### OSI Model de memoria
```
7 - Application  → HTTP, DNS, FTP, SMTP
6 - Presentation → TLS/SSL, cifrado, compresión
5 - Session      → Sesiones, checkpoints
4 - Transport    → TCP (fiable) / UDP (rápido), puertos
3 - Network      → IP, ICMP, enrutamiento — PACKETS
2 - Data Link    → MAC, ARP, switches — FRAMES
1 - Physical     → Cables, señales, bits
```
Mnemónico: **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

### Puertos que sí o sí debes saber
```
21   → FTP          80   → HTTP
22   → SSH          443  → HTTPS
23   → Telnet       445  → SMB
25   → SMTP         3389 → RDP
53   → DNS
```

### Subnetting rápido
```
/24 → 254 hosts    (255.255.255.0)
/25 → 126 hosts    (255.255.255.128)
/26 → 62 hosts     (255.255.255.192)
/27 → 30 hosts     (255.255.255.224)
Fórmula: 2^(32 - CIDR) - 2
```

### TCP Three-Way Handshake
```
Cliente → SYN      → Servidor
Cliente ← SYN/ACK  ← Servidor
Cliente → ACK      → Servidor
[Datos viajan aquí]
Cliente → FIN      → Servidor  (cierre)
```

### Status Codes HTTP
```
200 OK · 201 Created
301 Moved Permanently · 302 Found
400 Bad Request · 401 Unauthorized · 403 Forbidden · 404 Not Found
500 Internal Server Error · 503 Service Unavailable
```

### DNS Records
```
A     → IPv4          AAAA → IPv6
CNAME → Alias/dominio  MX   → Email server (con prioridad)
TXT   → SPF, DMARC, verificación de dominio
```

### Arquitectura Web
```
WAF → Load Balancer → Web Server → Backend → Base de Datos
                                ↑
                              CDN (archivos estáticos)

Web Servers:  Apache · Nginx (/var/www/html) | IIS (C:\inetpub\wwwroot)
Backend:      PHP · Python · Ruby · NodeJS
Bases datos:  MySQL · PostgreSQL · MongoDB · Redis
Estático:     HTML/CSS/JS/imágenes que no cambian
Dinámico:     Contenido generado en tiempo real por el Backend
HTML Inject:  Input sin sanitizar → htmlspecialchars() para prevenir
```

### CIA Triad + Criptografía
```
CIA Triad:
  Confidentiality → solo autorizados ven los datos (cifrado, control de acceso)
  Integrity       → datos no modificados sin permiso (hashing, firmas digitales)
  Availability    → servicio accesible cuando se necesita (redundancia, anti-DDoS)

Criptografía simétrica: 1 clave cifra y descifra (rápido) → AES, ChaCha20
Criptografía asimétrica: pública cifra, privada descifra (lento) → RSA, ECC
HTTPS híbrido: Asimétrico para negociar clave → Simétrico para los datos
Certificado: public key + identidad + firma de una CA de confianza
```

### Representación de Datos
```
1 bit = 0 o 1 | 1 byte = 8 bits = 256 valores
Binario (base 2): potencias de 2 de derecha a izquierda
Hex (base 16): 0-9 y A-F | 4 bits = 1 dígito hex | byte = 2 dígitos hex
ASCII: 7 bits, 128 chars, solo inglés | 'A'=0x41=65 | 'a'=0x61=97
Unicode: code points universales (U+XXXX) para todos los idiomas + emoji
UTF-8: 1-4 bytes dinámicos, compatible con ASCII, dominante en la web
```

### Sistemas Operativos
```
Capas: Usuario → Apps (User Space) → OS → Kernel Space → Hardware
Kernel space: acceso total al hardware | User space: apps restringidas, usan system calls
GUI: visual, clics | CLI: texto, comandos exactos — esencial en ciberseguridad
Tipos: Desktop · Server (headless) · Mobile · Embedded · Virtual/Cloud
Linux domina en servidores, cloud y herramientas de seguridad (Kali, Ubuntu)
Seguridad del OS: Authentication · Permissions · Isolation · System Protection
```

### Virtualización y Cloud
```
Hypervisor Type 1: corre en hardware → producción (VMware ESXi, Proxmox)
Hypervisor Type 2: corre en OS → lab/testing (VirtualBox, VMware Workstation)
VM vs Container: VM = OS completo aislado | Container = solo la app, comparte kernel
Docker: Image (plantilla) → Container (instancia en ejecución)

Cloud deployment: Public (todos) | Private (datos sensibles) | Hybrid (ambos)
IaaS → tú gestionas OS y app    (AWS EC2)
PaaS → tú gestionas solo el código (Heroku)
SaaS → solo usas la app          (Gmail, Zoom)
Responsabilidad compartida: proveedor = hardware | cliente = datos y configuración
```

### Boot Process
```
Power Button → UEFI/BIOS → POST → Boot Device → Bootloader → OS

UEFI vs BIOS: UEFI es el estándar moderno (Secure Boot, GPT, interfaz gráfica)
POST:         Verifica RAM · CPU · Storage · GPU → beep codes si algo falla
Boot Order:   USB > SSD > DVD > Red (PXE) — asegurar con contraseña UEFI
Bootloader:   GRUB (Linux) · Windows Boot Manager · iBoot (macOS)
Tipos PC:     Laptop (portátil) · Desktop (fijo) · Workstation (precisión) · Server (sin pantalla)
IoT vs Emb:   IoT = conectado a red | Embedded = dentro del dispositivo, sin red
```

### Los 9 métodos HTTP
```
GET     → obtener recurso (datos en URL — nunca passwords)
POST    → crear recurso  (datos en body)
PUT     → reemplazar recurso completo
PATCH   → modificar parte del recurso
DELETE  → eliminar recurso
HEAD    → como GET pero sin body de respuesta
OPTIONS → qué métodos acepta el servidor (CORS preflight)
CONNECT → túnel TCP (proxies HTTPS)
TRACE   → diagnóstico (usualmente desactivado)
```

### Firewall
```
Stateful  → analiza la CONEXIÓN completa, bloquea el dispositivo entero
Stateless → analiza PACKETS individuales, consume menos recursos
```

### Ofensiva — Herramientas y Conceptos Clave
```
Gobuster: gobuster dir --url http://target/ -w /wordlist.txt
  → Descubre páginas/dirs ocultos probando cada nombre del wordlist
  → HTTP 200 = existe ✅  |  HTTP 404 = no existe ❌

Hydra:    hydra -l admin -P passlist.txt target http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect" -V
  → Automatiza intentos de login con wordlist de contraseñas
  → Dictionary Attack: prueba lista predefinida (vs Brute Force: todas las combinaciones)

Fases de un Pentest: Recon → Escaneo → Explotación → Post-Explotación → Reporte
Cadena de dominós: fallo pequeño + fallo pequeño = compromiso total
SIEMPRE necesitas permiso escrito + scope definido (sin esto = delito)
```

### IP Pública vs Privada — RFC 1918
```
Privada Clase A: 10.0.0.0  – 10.255.255.255  (/8)   → redes corporativas
Privada Clase B: 172.16.0.0 – 172.31.255.255 (/12)  → redes medianas
Privada Clase C: 192.168.0.0 – 192.168.255.255 (/16) → redes domésticas
Loopback:        127.0.0.1                           → propia máquina

IP pública: asignada por ISP, única en internet, alcanzable desde afuera
IP privada: solo válida en LAN, necesita NAT para salir a internet
NAT:        router convierte IPs privadas ↔ IP pública del router

Ver IP en Windows: ipconfig | ipconfig /all (también MAC)
Ver IP en Linux:   ifconfig | ip a s
```

### Windows CMD — Quick Reference
```
Información:
  ver | systeminfo | hostname | whoami

Red:
  ipconfig              → IP, gateway, máscara
  ipconfig /all         → + MAC, DHCP, DNS
  ping ejemplo.com      → conectividad ICMP
  tracert ejemplo.com   → ruta (hops) hasta destino
  nslookup dominio.com  → resolución DNS
  netstat -abon         → conexiones TCP activas + PID + proceso

Archivos:
  dir | dir /a | dir /s | tree
  type archivo | more archivo
  copy orig dest | move orig dest | del archivo
  mkdir nombre | rmdir nombre
  comando | more    ← paginación con Spacebar

Procesos:
  tasklist
  tasklist /FI "imagename eq proceso.exe"
  taskkill /PID 1234
```

### PowerShell — Sintaxis y Cmdlets Clave
```
Verbo-Sustantivo: Get-Process, Set-Location, New-Item, Remove-Item...

Sistema de archivos:
  Get-ChildItem (ls/dir)  Set-Location (cd)
  New-Item -ItemType File/Directory
  Remove-Item | Copy-Item | Move-Item | Get-Content (cat)

Piping y filtrado:
  | Sort-Object Propiedad [-Descending]
  | Where-Object -Property "Ext" -eq ".txt"   (-ne/-gt/-ge/-lt/-le/-like)
  | Select-Object Name,Length [-First N]
  Select-String -Path *.log -Pattern "ERROR"    (≈ grep)

Información del sistema (incident response):
  Get-ComputerInfo    Get-LocalUser
  Get-NetIPConfiguration  Get-NetIPAddress
  Get-Process         Get-Service
  Get-NetTCPConnection    ← conexiones TCP activas (detectar C2)
  Get-FileHash -Path archivo   ← SHA256 para verificar integridad
  Get-Item -Path archivo -Stream *  ← ver ADS (Alternate Data Streams)
  Invoke-Command -ComputerName server -ScriptBlock { ... }
```

### Linux — Permisos y Comandos Esenciales
```
Formato ls -l:   -rw-r--r-- 1 owner group tamaño fecha nombre
Grupos:          owner (2-4) | group (5-7) | others (8-10)

Numérico:   r=4  w=2  x=1  → sumar por grupo
  rwxr-xr-x = 755  |  rw-r--r-- = 644  |  rwx------ = 700

chmod 755 archivo     # dar permisos
chown user:group archivo  # cambiar propietario
su -l usuario         # cambiar de usuario (con su entorno completo)

Directorios clave:
  /etc   → configuración (passwd, shadow, sudoers)
  /var   → datos variables (logs en /var/log)
  /root  → home del superusuario root
  /tmp   → temporal, se borra al reiniciar, escribible por todos

Cron: MIN HOUR DOM MON DOW CMD
  0 */12 * * * comando    → cada 12 horas
  crontab -e / crontab -l → editar / listar

Transferencia:
  wget URL                              → descargar de web
  scp local.txt user@IP:/ruta/         → subir a remoto via SSH
  scp user@IP:/ruta/file.txt local/    → bajar de remoto via SSH
  python3 -m http.server               → servidor HTTP en puerto 8000

Logs:
  /var/log/apache2/access.log   → requests HTTP (IP, URL, status)
  /var/log/auth.log              → logins SSH, uso de sudo
  tail -f /var/log/auth.log      → monitoreo en tiempo real
  grep "Failed" /var/log/auth.log
```

### TLS/SSL + SSH — Seguridad de Comunicaciones
```
Historia TLS:
  1995 → SSL 2.0 (Netscape)     ← mismo año que SSH-1 (Tatu Ylönen)
  1999 → TLS 1.0 (IETF)         ← sucesor de SSL 3.0
  2018 → TLS 1.3                ← versión actual, más rápida y segura

Proceso de certificado:
  Servidor crea CSR → envía a CA → CA firma → instala certificado
  Cliente conecta → verifica firma CA → 🔒 si CA es de confianza
  Self-signed = sin CA de confianza → no prueba autenticidad → evitar en prod

HTTP  vs HTTPS:
  HTTP:  TCP handshake → HTTP data (en claro)
  HTTPS: TCP handshake → TLS handshake → HTTP data (cifrado)

Protocolos con TLS (la S = Secure):
  HTTP→HTTPS(443)  SMTP→SMTPS(465/587)  POP3→POP3S(995)
  IMAP→IMAPS(993)  FTP→FTPS(990)        DNS→DoT(853)

SSH (puerto 22) — sustituto seguro de TELNET (puerto 23):
  ssh user@host                  ← conectar con contraseña
  ssh -X user@host               ← X11 Forwarding (apps gráficas)
  ssh-keygen -t rsa -b 4096      ← generar par de claves
  ssh-copy-id user@host          ← copiar clave pública al servidor
  ssh user@host                  ← login sin contraseña ✅
  OpenSSH 1999, open source, base de todos los clientes SSH modernos
```

### Protocolos de Aplicación — Puertos y Email
```
Protocolo  Puerto  Transport  Función
─────────────────────────────────────────────────────────
TELNET      23     TCP        Terminal remoto (texto plano — NO usar en prod)
SSH         22     TCP        Terminal remoto (cifrado)
FTP         21     TCP        Transferencia de archivos (anon: usuario "anonymous")
SMTP        25     TCP        Envío de email (servidor↔servidor)
SMTP sub.  587     TCP        Envío de email (cliente → servidor)
DNS         53     UDP/TCP    Resolución de nombres
DHCP        67/68  UDP        Asignación IP (servidor/cliente)
HTTP        80     TCP        Web sin cifrar
POP3       110     TCP        Email entrante (descarga, un dispositivo)
IMAP       143     TCP        Email entrante (sincronizado, multi-dispositivo)
HTTPS      443     TCP        Web cifrada (TLS)
SMB        445     TCP        Compartir archivos Windows
RDP       3389     TCP        Escritorio remoto Windows

Email flow: Tu cliente → [SMTP:587] → Tu servidor → [SMTP:25] → Servidor destino
         Destinatario ← [POP3:110 o IMAP:143] ← Servidor destino

WHOIS: whois dominio.com → registrante, fechas, registrar, DNS — OSINT en recon
FTP anónimo: usuario "anonymous", contraseña vacía → probar siempre en pentesting

Rango de puertos: 2 octetos → 0-65535 (puerto 0 reservado)
  Well-Known: 1-1023  |  Registered: 1024-49151  |  Ephemeral: 49152-65535
```

### Defensiva — Blue Team Quick Reference
```
5 Pilares: Prevention → Detection → Mitigation → Analysis → Response
Infraestructura a proteger: Dispositivos · Web Server · Mail Server · Firewall
Defense in Depth: Firewall → WAF/IPS → Server → Antivirus/EDR → Datos cifrados
IOC: Indicador de Compromiso (IP sospechosa, hash de malware, comportamiento anómalo)

SOC L1: triage de alertas | SOC L2: investigación | SOC L3: threat hunting
Threat Intelligence: TTPs, APTs, IOCs, MITRE ATT&CK
DFIR: 6 fases → Preparación · Identificación · Contención · Erradicación · Recuperación · Lecciones

Señales de alerta:
  Logins fallidos repetidos → brute force
  Login nocturno desde otro país → cuenta comprometida
  Tráfico saliente a IP desconocida → posible C2
```

---

## 🗓️ Registro de Progreso

| Fecha | Sala / Módulo | Temas cubiertos | Notas creadas |
|-------|---------------|-----------------|---------------|
| 2026-06-27 | Pre-Security — Intro to Cybersecurity | Roles, Blue Team, SOC | [[Roles-Ciberseguridad]] |
| 2026-06-27 | Pre-Security — Intro to Networking (1) | IP, MAC, topologías, ARP, DHCP, subnetting | [[Redes-Fundamentos]], [[Protocolos-ARP-DHCP]], [[Subnetting]] |
| 2026-06-27 | Pre-Security — Intro to Networking (2) | OSI model completo, TCP vs UDP | [[OSI-Model]] |
| 2026-06-27 | Pre-Security — Intro to Networking (3) | Packets vs Frames, TCP/IP model, headers, handshake, puertos | [[Packets-and-Frames]], [[TCP-UDP-Puertos]] |
| 2026-06-27 | Pre-Security — Intro to Networking (4) | Router, Switch, VLAN, Port Forwarding, Firewall, VPN | [[Infraestructura-de-Red]], [[Seguridad-de-Red]] |
| 2026-06-27 | Pre-Security — How the Web Works (1) | DNS, HTTP/HTTPS, URLs, métodos, status codes, headers, cookies | [[DNS]], [[HTTP-Web]] |
| 2026-06-27 | Pre-Security — How the Web Works (2) | Front/Back end, Web Servers, Load Balancer, CDN, Databases, WAF, HTML Injection | [[Arquitectura-Web]] |
| 2026-06-27 | Pre-Security — Intro to Computing | Boot process, UEFI/BIOS, POST, tipos de computadoras, IoT vs Embedded | [[Sistemas-Hardware]] |
| 2026-06-27 | Pre-Security — HTTP in Depth | Modelo cliente-servidor, 9 métodos HTTP, GET request/response, stateless | [[Cliente-Servidor-HTTP]] |
| 2026-06-27 | Pre-Security — Virtualization | Hypervisors T1/T2, VMs, Containers, Docker | [[Virtualizacion]] |
| 2026-06-27 | Pre-Security — Cloud Computing | Public/Private/Hybrid, IaaS/PaaS/SaaS, AWS/Azure/GCP | [[Cloud-Computing]] |
| 2026-06-27 | Pre-Security — Operating Systems | OS, kernel/user space, responsabilidades, GUI vs CLI, tipos y familias | [[Sistemas-Operativos]] |
| 2026-06-27 | Pre-Security — Data Representation | Bits, binario, hex, ASCII, Unicode, UTF-8 | [[Representacion-de-Datos]] |
| 2026-06-27 | Pre-Security — Cybersecurity Fundamentals | CIA Triad, cifrado simétrico/asimétrico, certificados, HTTPS | [[CIA-Triad]], [[Criptografia]] |
| 2026-06-28 | Pre-Security — Intro to Offensive Security | Red Teaming, Pentesting, scope, Gobuster, Hydra, dictionary attack, mentalidad del atacante | [[Ofensiva-Pentesting]] |
| 2026-06-28 | Pre-Security — Intro to Defensive Security | Blue Team, 5 pilares, infraestructura del cliente, defender mindset, SOC, Threat Intel, DFIR | [[Defensiva-Blue-Team]] |
| 2026-06-29 | Linux Fundamentals Part 2 | Permisos (ls -l, chmod simbólico y numérico), su/-l, /etc /var /root /tmp | [[Linux-Permisos-y-Sistema]] |
| 2026-06-29 | Linux Fundamentals Part 3 | Nano, VIM, wget, SCP, Python HTTP server, crontab, apt, logs | [[Linux-Herramientas-y-Admin]] |
| 2026-07-01 | Cyber Security 101 — Linux Shells | Bash vs Fish vs Zsh, comandos básicos, scripting bash (variables, loops, condicionales) | [[Linux-Scripting-y-Shells]] |
| 2026-07-01 | Cyber Security 101 — Windows CMD | ver/systeminfo, ipconfig/ping/tracert/nslookup/netstat, dir/copy/move/del, tasklist/taskkill | [[Windows-CMD]] |
| 2026-07-01 | Cyber Security 101 — PowerShell | Verb-Noun, Get-Command/Help/Alias, piping, Where-Object, Get-Process/Service/NetTCPConnection/FileHash, Invoke-Command | [[Windows-PowerShell]] |
| 2026-07-01 | Cyber Security 101 — IP Addresses | IP pública vs privada, RFC 1918 rangos, NAT, ifconfig/ip a s | [[Redes-Fundamentos]] (extendido) |
| 2026-07-01 | Cyber Security 101 — Networking Concepts | NAT translation table, OSPF/EIGRP/BGP/RIP, DHCP ports 67/68, ICMP types 0/8/11/3, traceroute TTL mechanism | [[Redes-Fundamentos]] (extendido), [[Protocolos-ARP-DHCP]] (extendido) |
| 2026-07-01 | Cyber Security 101 — Application Protocols | TELNET, FTP (anon login), SMTP, POP3, IMAP, WHOIS, tabla completa de puertos, flujo email | [[Protocolos-Aplicacion]] (nuevo) |
| 2026-07-01 | Cyber Security 101 — TCP/UDP deep dive | Rango de puertos (2^16), vida de un paquete end-to-end (7 pasos) | [[TCP-UDP-Puertos]] (extendido) |
| 2026-07-01 | Cyber Security 101 — TLS & HTTPS | Historia SSL/TLS (1995→2018), CSR→CA→cert, self-signed, HTTPS 3-pasos, protocolos con TLS | [[Criptografia]] (extendido) |
| 2026-07-01 | Cyber Security 101 — SSH | OpenSSH, port 22, auth por clave pública, X11 Forwarding, SSH vs TELNET | [[Protocolos-Aplicacion]] (extendido) |
| 2026-07-01 | Cyber Security 101 — SMTPS/POP3S/IMAPS | Puertos seguros email: 465/587/995/993, STARTTLS vs TLS implícito | [[Protocolos-Aplicacion]] (extendido) |
| 2026-07-01 | Cyber Security 101 — VPN deep dive | Site-to-site vs remote-access vs comercial, geo-bypass, DNS leak, split tunneling, legalidad | [[Seguridad-de-Red]] (extendido) |

> [!note] Cómo mantener este registro actualizado
> Cada vez que termines una sala en THM, agrega una fila con la fecha, el nombre de la sala y enlaza las notas que creaste. Así puedes ver de un vistazo todo tu recorrido.

---

## 🛤️ Próximas Salas (Ruta Sugerida)

### Pre-Security (completar la ruta) ✅ COMPLETADA
- [x] Intro to Cybersecurity · Networking · Web · Computing · Virtualization · Cloud · OS · Crypto
- [x] Intro to Offensive Security (Gobuster, Hydra, pentesting mindset)
- [x] Intro to Defensive Security (Blue Team, SOC, DFIR, defender mindset)

### Linux Fundamentals (en progreso)
- [x] **Linux Fundamentals Part 2** → Permisos, su, /etc /var /root /tmp
- [x] **Linux Fundamentals Part 3** → Nano, wget, SCP, cron, apt, logs
- [ ] **Linux Fundamentals Part 1** → Comandos básicos, navegación, permisos
- [ ] **Linux Fundamentals Part 2** → Flags, operadores, permisos avanzados
- [ ] **Linux Fundamentals Part 3** → Automatización, cron, procesos
- [ ] **Windows Fundamentals Part 1** → Sistema de archivos, usuarios, registros
- [ ] **Windows Fundamentals Part 2** → Active Directory, UAC, PowerShell

### Jr Penetration Tester (siguiente ruta)
- [ ] **Intro to Offensive Security** → Primeros pasos como pentester
- [ ] **Metasploit** → Framework de explotación
- [ ] **Burp Suite Basics** → Interceptar y manipular tráfico HTTP
- [ ] **OWASP Top 10** → Las 10 vulnerabilidades web más críticas
- [ ] **Nmap** → Escaneo y enumeración de redes

### Blue Team / SOC Analyst
- [ ] **SOC Level 1** → Análisis de logs, alertas, triaje
- [ ] **Splunk** → SIEM, búsquedas SPL, dashboards
- [ ] **Wireshark** → Análisis de tráfico de red

---

## 🏷️ Tags del Vault

| Tag | Contenido |
|-----|-----------|
| `#pre-security` | Ruta SEC0 completa |
| `#networks` | Redes, IPs, protocolos |
| `#OSI` | Modelo OSI y sus capas |
| `#blue-team` | Defensa, SOC, análisis |
| `#ARP` | Protocolo ARP y ARP Spoofing |
| `#DHCP` | Asignación dinámica de IPs |
| `#subnetting` | División de redes, CIDR |
| `#TCP` | Protocolo TCP, handshake |
| `#UDP` | Protocolo UDP |
| `#puertos` | Puertos comunes, port forwarding |
| `#firewall` | Stateful, stateless |
| `#VPN` | Tecnologías VPN |
| `#DNS` | Registros DNS, resolución |
| `#HTTP` | HTTP, HTTPS, requests, cookies |
| `#web` | Todo lo relacionado con web |
| `#arquitectura` | Web servers, Load Balancer, CDN, WAF |
| `#backend` | Lenguajes server-side, bases de datos |
| `#vulnerabilidades-web` | HTML Injection, XSS y derivados |
| `#hardware` | Componentes físicos, tipos de computadoras |
| `#boot` | UEFI, BIOS, POST, bootloader, proceso de arranque |
| `#IoT` | Dispositivos IoT y embedded computing |
| `#cliente-servidor` | Modelo request-response, protocolos |
| `#roles` | Carreras en ciberseguridad |
| `#certificaciones` | Contenido para exámenes |
| `#ofensiva` | Pentesting, Red Team, explotación |
| `#pentesting` | Metodología y herramientas ofensivas |
| `#gobuster` | Directory enumeration |
| `#hydra` | Ataques a credenciales |
| `#vulnerabilidades` | Tipos de debilidades y exploits |
| `#defensiva` | Blue Team, prevención, detección |
| `#blue-team` | Defensores, SOC, respuesta a incidentes |
| `#SOC` | Security Operations Center |
| `#DFIR` | Digital Forensics & Incident Response |
| `#threat-intelligence` | IOCs, APTs, TTPs |
| `#incident-response` | Ciclo de respuesta a incidentes |
| `#linux` | Comandos, sistema de archivos, administración |
| `#permisos` | chmod, chown, usuarios, grupos |
| `#filesystem` | Directorios del sistema Linux |
| `#administracion` | apt, cron, logs, transferencia de archivos |
| `#logs` | /var/log, Apache, auth.log, análisis forense |
| `#windows` | CMD, PowerShell, administración Windows |
| `#cmd` | Comandos CMD de Windows |
| `#powershell` | Cmdlets PowerShell, scripting, administración |
| `#bash` | Scripting bash, variables, loops, condicionales |
| `#scripting` | Automatización con scripts (bash, PowerShell) |
| `#shells` | Bash, Fish, Zsh — tipos de shells |
| `#automatizacion` | Scripts y tareas automatizadas |
| `#TELNET` | Protocolo TELNET, uso para pruebas de protocolos |
| `#FTP` | File Transfer Protocol, login anónimo, SFTP/FTPS |
| `#SMTP` | Envío de correo, comandos EHLO/MAIL FROM/RCPT TO |
| `#POP3` | Descarga de correo, un dispositivo |
| `#IMAP` | Sincronización de correo, multi-dispositivo |
| `#WHOIS` | Información de registro de dominios, OSINT |
| `#email` | Flujo completo de email, protocolos de correo |
| `#TLS` | Transport Layer Security, historia SSL/TLS, certificados, CSR, CA |
| `#SSL` | SSL como predecesor de TLS |
| `#SSH` | Secure Shell, OpenSSH, acceso remoto cifrado, clave pública |
| `#certificados` | Certificados digitales, CA, self-signed, PKI |
| `#HTTPS` | HTTP sobre TLS, 3-pasos, cifrado de tráfico web |
| `#VPN` | Virtual Private Network, site-to-site, remote-access, geo-bypass |

---

## 🔗 Recursos Externos

| Recurso | Uso |
|---------|-----|
| [TryHackMe](https://tryhackme.com) | Plataforma de aprendizaje principal |
| [CyberChef](https://gchq.github.io/CyberChef/) | Codificar/decodificar/analizar datos |
| [ExplainShell](https://explainshell.com) | Entender cualquier comando de Linux |
| [Subnet Calculator](https://www.subnet-calculator.com) | Calcular subredes visualmente |
| [Shodan](https://www.shodan.io) | Buscador de dispositivos expuestos en Internet |
| [GTFOBins](https://gtfobins.github.io) | Escalada de privilegios en Linux |
| [HackTricks](https://book.hacktricks.xyz) | Guía de técnicas de pentesting |
| [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Las 10 vulnerabilidades web más críticas |
| [CompTIA Study Guide](https://www.comptia.org/certifications) | Objetivos oficiales de examen |
