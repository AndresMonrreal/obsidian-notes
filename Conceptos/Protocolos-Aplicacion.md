---
tags:
  - protocolos
  - aplicacion
  - TELNET
  - FTP
  - SMTP
  - POP3
  - IMAP
  - WHOIS
  - puertos
  - email
  - pre-security
fecha: 2026-07-01
ruta: SEC0
fuente: TryHackMe — Networking Concepts / Cyber Security 101
---

# 📡 Protocolos de Aplicación — TELNET, FTP, SMTP, POP3, IMAP

---

## Por qué usar `telnet` para aprender protocolos

`telnet` es un cliente de terminal remoto (TCP port 23 por defecto), pero su utilidad en aprendizaje y pentesting va más allá: **permite conectarte manualmente a cualquier servidor TCP y "hablar" el protocolo directamente**.

```bash
telnet IP_OBJETIVO PUERTO
```

Esto funciona con cualquier protocolo de texto plano (HTTP, FTP, SMTP, POP3...) porque puedes escribir los comandos del protocolo tú mismo y ver las respuestas raw del servidor.

> [!warning] TELNET en producción = peligro
> Todo lo que envías por TELNET (incluidas contraseñas) viaja en **texto plano**. Cualquiera que capture el tráfico lo ve. En producción se usa **SSH** (puerto 22) que cifra todo. TELNET solo es útil para pruebas en redes controladas.

### Ejemplos con telnet

```bash
# Echo server (puerto 7) — repite todo lo que envías
telnet 10.64.150.38 7
Hi         → respuesta: Hi
Ctrl+] → quit  (salir del modo telnet)

# Daytime server (puerto 13) — responde con fecha/hora y cierra
telnet 10.64.150.38 13
Thu Jun 20 12:36:32 PM UTC 2024
Connection closed by foreign host.

# HTTP (puerto 80) — "hablar HTTP" manualmente
telnet 10.64.150.38 80
GET / HTTP/1.1
Host: telnet.thm
[Enter dos veces]
HTTP/1.1 200 OK
Content-Type: text/html
[...]
```

---

## FTP — File Transfer Protocol

**Propósito:** Transferir archivos entre cliente y servidor. Muy eficiente para transferencias de archivos grandes.

**Puerto por defecto:** TCP **21** (control) + conexión separada para datos

### Comandos FTP del cliente

| Comando | Función |
|---------|---------|
| `USER <usuario>` | Autenticar con nombre de usuario |
| `PASS <contraseña>` | Enviar contraseña |
| `ls` / `LIST` | Listar archivos del servidor |
| `get <archivo>` / `RETR` | Descargar archivo del servidor |
| `put <archivo>` / `STOR` | Subir archivo al servidor |
| `type ascii` | Modo texto (para archivos .txt) |
| `type binary` | Modo binario (para ejecutables, imágenes) |
| `quit` | Cerrar la sesión |

### Sesión FTP real

```bash
$ ftp 10.64.150.38
Connected to 10.64.150.38
220 (vsFTPd 3.0.5)
Name: anonymous           ← login anónimo (sin password)
Password:                 ← Enter sin escribir nada
230 Login successful.

ftp> ls
150 Here comes the directory listing.
-rw-r--r--  coffee.txt
-rw-r--r--  flag.txt
-rw-r--r--  tea.txt

ftp> type ascii           ← para archivos de texto
ftp> get coffee.txt       ← descargar
226 Transfer complete.

ftp> quit
221 Goodbye.
```

> [!warning] FTP = texto plano
> Las credenciales y los archivos viajan sin cifrado. En la captura de Wireshark se ven el usuario, contraseña y contenido. Alternativas seguras: **SFTP** (SSH File Transfer Protocol, puerto 22) o **FTPS** (FTP sobre TLS, puerto 990).

> [!tip] Login anónimo en FTP
> Muchos servidores FTP públicos permiten login con usuario `anonymous` y cualquier contraseña (o vacía). Esto es intencional para distribución pública de archivos. En pentesting, siempre probar login anónimo primero.

---

## SMTP — Simple Mail Transfer Protocol

**Propósito:** Enviar email entre clientes y servidores de correo, y entre servidores de correo.

**Puerto por defecto:** TCP **25** (servidor a servidor) · también 587 (cliente a servidor con autenticación)

**Analogía:** El cartero que lleva tu carta a la oficina de correos y la envía.

### Comandos SMTP

| Comando | Función |
|---------|---------|
| `HELO` / `EHLO` | Iniciar sesión SMTP (EHLO = versión extendida) |
| `MAIL FROM: <email>` | Especificar el remitente |
| `RCPT TO: <email>` | Especificar el destinatario |
| `DATA` | Comenzar a escribir el contenido del email |
| `.` (un punto solo en una línea) | Indicar fin del email |
| `QUIT` | Cerrar la sesión |

### Sesión SMTP via telnet

```
$ telnet 10.64.150.38 25

220 example.thm ESMTP Exim 4.95
HELO client.thm
250 example.thm Hello client.thm
MAIL FROM: <user@client.thm>
250 OK
RCPT TO: <destinatario@server.thm>
250 Accepted
DATA
354 Enter message, ending with "." on a line by itself
From: user@client.thm
To: destinatario@server.thm
Subject: Prueba SMTP

Hola, esto es un email enviado con telnet.
.
250 OK id=1sMrpq-0001Ah-UT
QUIT
221 example.thm closing connection
```

> [!info] SMTP solo envía — no lee
> SMTP es el protocolo de **envío**. Para **recibir** y leer emails se usa POP3 o IMAP.

---

## POP3 — Post Office Protocol v3

**Propósito:** Descargar emails del servidor al cliente. Orientado a un solo dispositivo.

**Puerto por defecto:** TCP **110** (sin TLS) · 995 (con TLS/SSL)

**Analogía:** Tu buzón físico — sacas las cartas y las llevas a casa. El buzón queda vacío.

### Comportamiento de POP3

```
Servidor de correo          Tu cliente de correo (PC)
      │                              │
      │ Emails almacenados           │
      │ ────────────────────────────→│ RETR descarga
      │                              │
      │ Email eliminado del servidor │ Email guardado localmente
```

- Los emails se **descargan y eliminan** del servidor por defecto
- Funciona bien si solo usas **un dispositivo**
- No sincroniza entre múltiples dispositivos

### Comandos POP3

| Comando | Función |
|---------|---------|
| `USER <usuario>` | Identificar usuario |
| `PASS <contraseña>` | Autenticar |
| `STAT` | Número de mensajes y tamaño total |
| `LIST` | Listar todos los mensajes con tamaño |
| `RETR <número>` | Descargar mensaje específico |
| `DELE <número>` | Marcar mensaje para eliminación |
| `QUIT` | Cerrar sesión y aplicar eliminaciones |

### Sesión POP3 via telnet

```
$ telnet 10.64.150.38 110

+OK Dovecot ready.
USER linda
+OK
PASS Pa$$123
+OK Logged in.
STAT
+OK 3 1264              ← 3 mensajes, 1264 bytes total
LIST
+OK 3 messages:
1 407
2 412
3 445
.
RETR 3                  ← descargar mensaje número 3
+OK 445 octets
From: user@client.thm
To: strategos@server.thm
Subject: Telnet email

Hello. I am using telnet to send you an email!
.
QUIT
+OK Logging out.
```

---

## IMAP — Internet Message Access Protocol

**Propósito:** Acceder y sincronizar emails en el servidor. Orientado a múltiples dispositivos.

**Puerto por defecto:** TCP **143** (sin TLS) · 993 (con TLS/SSL)

**Analogía:** Tu correo en la nube — accedes desde cualquier dispositivo y los cambios se sincronizan en todos.

### Comportamiento de IMAP

```
Servidor de correo
      │
      ├──── PC (lee mensaje → se marca como leído en TODOS)
      ├──── Laptop (ve el mismo estado)
      └──── Smartphone (ve el mismo estado)

Los emails permanecen en el servidor → más almacenamiento usado
```

- Emails se **mantienen en el servidor**
- Cambios (leído, movido, eliminado) se **sincronizan** en todos los dispositivos
- Ideal si accedes al correo desde múltiples dispositivos

### Comandos IMAP

| Comando | Función |
|---------|---------|
| `LOGIN <usuario> <contraseña>` | Autenticar |
| `SELECT <bandeja>` | Seleccionar carpeta (ej: inbox) |
| `FETCH <número> body[]` | Descargar mensaje completo (cabecera + cuerpo) |
| `MOVE <rango> <carpeta>` | Mover mensajes a otra carpeta |
| `COPY <rango> <carpeta>` | Copiar mensajes |
| `LOGOUT` | Cerrar sesión |

> [!info] Sintaxis especial de IMAP
> Cada comando IMAP lleva un **prefijo de etiqueta** (A, B, C, D...) para que el servidor sepa a qué solicitud responde:
> ```
> A LOGIN strategos contraseña
> B SELECT inbox
> C FETCH 3 body[]
> D LOGOUT
> ```

### Sesión IMAP via telnet

```
$ telnet 10.10.41.192 143

* OK Dovecot ready.
A LOGIN strategos ContraseñaAqui
A OK Logged in
B SELECT inbox
* 4 EXISTS       ← 4 mensajes en inbox
B OK [READ-WRITE] Select completed.
C FETCH 3 body[]
* 3 FETCH (BODY[] {445}
[contenido del email]
)
C OK Fetch completed.
D LOGOUT
D OK Logout completed.
```

---

## POP3 vs IMAP — Comparación

| | **POP3** | **IMAP** |
|-|---------|---------|
| **Puerto** | 110 (143 con TLS: 995) | 143 (con TLS: 993) |
| **Emails en servidor** | Se descargan y eliminan | Se mantienen en servidor |
| **Sincronización** | ❌ No | ✅ Sí — todos los dispositivos |
| **Almacenamiento servidor** | 💚 Mínimo | 🔴 Alto |
| **Múltiples dispositivos** | ❌ Problemático | ✅ Diseñado para esto |
| **Sin internet** | ✅ Todo local | ❌ Necesitas conexión |
| **Uso típico** | Un PC, sin nube | Smartphone + PC + web |
| **Analogía** | Buzón físico → vacias las cartas | Bandeja en la nube → accedes desde cualquier lugar |

---

## WHOIS — Información de Registro de Dominios

**Propósito:** Consultar quién registró un dominio, cuándo, y con qué proveedor.

```bash
# Comando Linux
whois ejemplo.com

# Campos principales de un WHOIS record:
Domain Name:    EJEMPLO.COM
Creation Date:  1993-04-02T00:00:00Z
Updated Date:   2024-07-05T16:02:43Z
Registrar:      GoDaddy.com, LLC
Registrant Name: Registration Private   ← protección de privacidad activa
Registrant Organization: Domains By Proxy, LLC
```

**Información disponible:**
- Quién registró el dominio (nombre, email, dirección, teléfono)
- Cuándo se registró y cuándo expira
- Con qué registrar (GoDaddy, Namecheap, Cloudflare, etc.)
- Servidores DNS del dominio

> [!info] Privacidad WHOIS
> Si usas un servicio de privacidad al registrar, los datos del registrante aparecen como "Domains By Proxy" o similar — no se revela tu información real. Muchos registradores ofrecen esto.

> [!tip] WHOIS en pentesting (reconocimiento/OSINT)
> Durante reconocimiento, `whois` revela:
> - Email del administrador del dominio (posible vector de phishing o OSINT)
> - Historial de cambios del dominio
> - Registrar — puede revelar si el dominio es reciente (posible phishing si es muy nuevo)
> - Dominios registrados por la misma persona/empresa

---

## SSH — Secure Shell

**Propósito:** Acceso remoto seguro a sistemas mediante terminal. Sustituto cifrado de TELNET.

**Puerto por defecto:** TCP **22**

**Historia:**
- **1995** — Tatu Ylönen desarrolla SSH-1 como freeware (mismo año que SSL 2.0 de Netscape)
- **1996** — SSH-2, versión más segura y definida formalmente
- **1999** — Los desarrolladores de OpenBSD lanzan **OpenSSH**, implementación open source — base de casi todos los clientes SSH modernos

### Ventajas de SSH sobre TELNET

| Característica | TELNET | SSH |
|----------------|--------|-----|
| Cifrado | ❌ Texto plano | ✅ Cifrado de extremo a extremo |
| Autenticación | Solo usuario/contraseña | Contraseña, clave pública, 2FA |
| Integridad de datos | ❌ Sin verificación | ✅ HMAC protege contra modificación |
| Protección MitM | ❌ Ninguna | ✅ Notifica si la clave del servidor cambia |
| Tunneling | ❌ | ✅ Túnel para otros protocolos (VPN-like) |
| X11 Forwarding | ❌ | ✅ Ejecutar apps gráficas remotas |

### Comandos SSH básicos

```bash
# Conectar a un servidor SSH
ssh username@hostname
ssh username@192.168.1.100

# Si tu usuario local coincide con el remoto, omite el usuario
ssh hostname

# X11 Forwarding — ejecutar app gráfica remota (ej: Wireshark)
ssh -X username@192.168.1.100

# Puerto diferente al 22
ssh -p 2222 username@hostname

# Usar clave privada específica
ssh -i ~/.ssh/mi_clave.pem username@hostname
```

### Autenticación por clave pública

```bash
# Generar par de claves RSA (4096 bits = más seguro)
ssh-keygen -t rsa -b 4096

# Copiar tu clave pública al servidor
ssh-copy-id username@hostname

# Ahora puedes conectar sin contraseña (la clave privada autentica)
ssh username@hostname   # login inmediato ✅
```

> [!tip] SSH en pentesting
> SSH en puerto 22 = vector común. Estrategias:
> - Credenciales por defecto: `root/toor`, `admin/admin`, `pi/raspberry`
> - Fuerza bruta: `hydra -l root -P wordlist.txt ssh://10.10.10.10`
> - Buscar claves privadas expuestas: `~/.ssh/id_rsa`, `/home/*/.ssh/`

> [!warning] Fingerprint al conectar
> Primera conexión → SSH muestra la huella RSA del servidor. Si en una conexión posterior la huella cambió → posible **Man-in-the-Middle attack**. Verificar siempre.

---

## Protocolos de Email Seguros — TLS en el Correo

Los protocolos de email también pueden (y deben) usar TLS añadiendo "S" de Secure:

| Protocolo inseguro | Puerto | Protocolo seguro | Puerto TLS |
|--------------------|--------|-----------------|------------|
| SMTP | 25 | **SMTPS** | **465** (TLS) / **587** (STARTTLS) |
| POP3 | 110 | **POP3S** | **995** |
| IMAP | 143 | **IMAPS** | **993** |

- Sin TLS → credenciales y mensajes en texto plano, capturables con Wireshark
- Con TLS → tráfico cifrado, ilegible sin la clave de sesión

**STARTTLS vs TLS implícito:**

| Método | Puerto | Cómo funciona |
|--------|--------|---------------|
| **TLS implícito** | 465 | La conexión empieza cifrada desde el primer byte |
| **STARTTLS** | 587 | Conexión comienza en claro → cliente envía `STARTTLS` para upgradear |

STARTTLS en puerto 587 es el estándar moderno recomendado para cliente→servidor.

> [!warning] ¿Cuál expone credenciales?
> Si capturas tráfico en port **143 (IMAP)** o **110 (POP3)** sin TLS, extraes usuario y contraseña directamente. Port **993 (IMAPS)** y **995 (POP3S)** son ilegibles. En pentesting, capturar siempre puertos 25/110/143.

---

## Tabla Completa de Puertos — Referencia Rápida

| Protocolo | Capa | Transport | Puerto | Función |
|-----------|------|-----------|--------|---------|
| **TELNET** | Aplicación | TCP | **23** | Terminal remoto (texto plano) |
| **SSH** | Aplicación | TCP | **22** | Terminal remoto (cifrado) |
| **DNS** | Aplicación | UDP/TCP | **53** | Resolución de nombres |
| **HTTP** | Aplicación | TCP | **80** | Web sin cifrar |
| **HTTPS** | Aplicación | TCP | **443** | Web cifrada (TLS) |
| **FTP** | Aplicación | TCP | **21** | Transferencia de archivos |
| **SFTP** | Aplicación | TCP | **22** | FTP sobre SSH (seguro) |
| **SMTP** | Aplicación | TCP | **25** | Envío de email (servidor→servidor) |
| **SMTPS** | Aplicación | TCP | **465** | Envío de email cifrado (TLS implícito) |
| **SMTPS** | Aplicación | TCP | **587** | Envío de email (STARTTLS, cliente→servidor) |
| **POP3** | Aplicación | TCP | **110** | Recibir email (descarga) |
| **POP3S** | Aplicación | TCP | **995** | POP3 cifrado |
| **IMAP** | Aplicación | TCP | **143** | Recibir email (sincronizado) |
| **IMAPS** | Aplicación | TCP | **993** | IMAP cifrado |
| **SMB** | Aplicación | TCP | **445** | Compartir archivos (Windows) |
| **RDP** | Aplicación | TCP | **3389** | Escritorio remoto Windows |
| **DHCP** | Aplicación | UDP | **67** (servidor) / **68** (cliente) | Asignación dinámica de IPs |

### Solo los que debes memorizar para exámenes

```
Inseguro → Puerto   |   Seguro (TLS) → Puerto
──────────────────────────────────────────────
TELNET   →  23      |   SSH     →  22
FTP      →  21      |   SFTP    →  22  (FTP sobre SSH)
SMTP     →  25      |   SMTPS   →  465 / 587
POP3     → 110      |   POP3S   →  995
IMAP     → 143      |   IMAPS   →  993
HTTP     →  80      |   HTTPS   →  443

Otros:
53   → DNS     |  445 → SMB  |  3389 → RDP
67/68→ DHCP    |  853 → DoT (DNS over TLS)
```

---

## Flujo Completo de Email — SMTP + IMAP/POP3

```
Tú escribes el email en tu cliente (Outlook, Thunderbird, etc.)
            │
            ▼
        [SMTP :25/:587]
            │ Tu cliente envía el email a tu servidor de correo
            ▼
    Tu servidor de correo
            │
            │ [SMTP :25] Tu servidor envía al servidor del destinatario
            ▼
    Servidor del destinatario
            │
            │ Almacena el email
            ▼
        [POP3 :110 o IMAP :143]
            │ El destinatario descarga/accede al email
            ▼
    Cliente del destinatario
```

---

## Herramientas de Análisis de Protocolos

```bash
# Wireshark — GUI para capturar y analizar tráfico de red
# Muestra cada paquete con todos los campos del protocolo

# tshark — versión CLI de Wireshark
tshark -r captura.pcap -n          ← leer archivo pcap
tshark -r captura.pcap -n -Y "ftp" ← filtrar por protocolo

# tcpdump — captura de paquetes en CLI
tcpdump -r arp.pcapng -n -v        ← leer y mostrar verbosamente
tcpdump -i eth0 port 80            ← capturar en vivo en puerto 80
tcpdump -i eth0 host 10.10.10.5    ← capturar solo tráfico de esa IP

# netstat / ss — ver conexiones activas
netstat -tlnp    ← TCP, escuchando, numérico, con PID (Linux)
ss -tlnp         ← alternativa moderna
```

---

## Notas relacionadas

- [[TCP-UDP-Puertos]] — TCP/UDP, handshake, rango de puertos 1-65535
- [[DNS]] — Puerto 53, resolución de nombres, WHOIS complementa el reconocimiento
- [[HTTP-Web]] — HTTP/HTTPS en detalle (métodos, status codes, headers)
- [[Seguridad-de-Red]] — Firewall filtra puertos; estos protocolos son vectores de ataque
- [[Ofensiva-Pentesting]] — FTP anónimo, SMTP open relay, TELNET como vectores de explotación
- [[Protocolos-ARP-DHCP]] — DHCP (puerto 67/68 UDP), ARP, ICMP
