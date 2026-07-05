---
tags:
  - linux
  - pre-security
  - nano
  - vim
  - wget
  - scp
  - cron
  - apt
  - logs
  - administracion
fecha: 2026-06-29
ruta: SEC0
fuente: TryHackMe — Linux Fundamentals
---

# 🛠️ Linux — Herramientas y Administración del Sistema

---

## Editores de Texto en la Terminal

### Nano — El editor amigable

Nano es el editor más fácil de usar en la terminal. Ideal para editar archivos de configuración o scripts rápidos.

```bash
nano archivo.txt        # Abrir/crear archivo
nano /etc/hosts         # Editar archivo del sistema (necesita sudo)
```

**Interfaz de Nano:**
```
  GNU nano 4.8                 myfile

Hello TryHackMe
I can write things into "myfile"

^G Get Help  ^O Write Out  ^X Exit  ^W Where Is  ^K Cut  ^U Paste
```

**Atajos clave (Ctrl = ^):**

| Atajo | Acción |
|-------|--------|
| `Ctrl + X` | Salir (pide guardar si hay cambios) |
| `Ctrl + O` | Guardar (Write Out) |
| `Ctrl + W` | Buscar texto |
| `Ctrl + K` | Cortar línea |
| `Ctrl + U` | Pegar línea |
| `Ctrl + C` | Ver número de línea actual |
| `Ctrl + _` | Ir a número de línea específico |
| `Ctrl + \` | Buscar y reemplazar |

> [!tip] Para guardar y salir rápido
> `Ctrl + X` → presionar `Y` para confirmar → `Enter` para confirmar el nombre del archivo.

---

### VIM — El editor avanzado

VIM tiene una curva de aprendizaje alta pero es extremadamente poderoso. Está disponible en prácticamente cualquier sistema Linux (a diferencia de nano que puede no estar instalado).

```bash
vim archivo.txt
```

**Ventajas de VIM sobre Nano:**

| Característica | Descripción |
|----------------|-------------|
| **Syntax Highlighting** | Colorea código fuente — muy útil para scripts y programas |
| **Atajos personalizables** | Puedes redefinir cualquier combinación de teclas |
| **Disponibilidad** | Está en casi cualquier sistema Unix/Linux |
| **Modo visual** | Seleccionar y manipular texto con precisión |

**Modos de VIM:**
```
NORMAL (por defecto) → navegar, copiar, eliminar
INSERT (tecla i)     → escribir texto
VISUAL (tecla v)     → seleccionar texto
COMMAND (:)          → guardar, salir, buscar
```

**Comandos básicos de VIM:**
```
i         → entrar en modo INSERT
Esc       → volver a modo NORMAL
:w        → guardar
:q        → salir
:wq       → guardar y salir
:q!       → salir sin guardar (forzado)
/término  → buscar
dd        → eliminar línea
yy        → copiar línea
p         → pegar
```

> [!info] ¿Cuándo usar VIM?
> - Al editar código (bash scripts, Python, configs complejas)
> - En sistemas minimalistas donde nano no está disponible (servidores, contenedores)
> - Al trabajar en ciberseguridad con shells limitados (reverse shells, etc.)

---

## Transferencia de Archivos

### Wget — Descargar desde la web

`wget` descarga archivos desde internet vía HTTP/HTTPS, como hacerlo desde el navegador pero en la terminal.

```bash
# Descargar un archivo
wget https://example.com/archivo.txt

# Descargar en un nombre diferente
wget -O mi_archivo.txt https://example.com/original.txt

# Descargar de un servidor Python (puerto específico)
wget http://10.64.187.89:8000/myfile
```

---

### SCP — Transferencia segura entre máquinas

SCP (Secure Copy) transfiere archivos entre sistemas usando SSH — con autenticación y cifrado.

**Modelo: ORIGEN → DESTINO**

```bash
# Subir: de mi máquina → máquina remota
scp archivo.txt usuario@192.168.1.30:/home/usuario/destino.txt

# Descargar: de máquina remota → mi máquina
scp usuario@192.168.1.30:/home/usuario/archivo.txt copia_local.txt

# Copiar directorio completo (-r = recursivo)
scp -r /carpeta/ usuario@192.168.1.30:/home/usuario/
```

**Ejemplo desglosado:**

| Variable | Valor | Parte del comando |
|----------|-------|------------------|
| IP remota | 192.168.1.30 | `usuario@192.168.1.30` |
| Usuario remoto | ubuntu | `ubuntu@192.168.1.30` |
| Archivo local | important.txt | primer argumento |
| Nombre en destino | transferred.txt | `:ruta/transferred.txt` |

```bash
scp important.txt ubuntu@192.168.1.30:/home/ubuntu/transferred.txt
```

> [!tip] SCP vs FTP
> SCP es preferible a FTP porque usa SSH (cifrado). FTP transmite datos en texto plano — cualquiera en la red puede interceptarlo. Usa siempre SCP o SFTP.

---

### Python HTTP Server — Servidor web rápido para compartir archivos

Python3 incluye un servidor HTTP ligero que sirve archivos del directorio actual. Útil para pasar archivos entre máquinas en la misma red.

```bash
# Terminal 1: iniciar el servidor (en el directorio con los archivos)
python3 -m http.server
# Serving HTTP on 0.0.0.0 port 8000 ...

# Terminal 2: descargar desde otra máquina
wget http://IP_DEL_SERVIDOR:8000/nombre_del_archivo
curl http://IP_DEL_SERVIDOR:8000/nombre_del_archivo -o archivo_descargado
```

```
Máquina A (servidor)        Máquina B (cliente)
/webserver/
  └── archivo.sh    ←──── wget http://10.64.187.89:8000/archivo.sh
```

> [!warning] Limitación del Python HTTP Server
> No tiene índice de archivos — debes conocer el nombre exacto del archivo que quieres descargar. Alternativa más avanzada: **Updog** (mismo concepto, con listado de directorio).

> [!info] Uso en pentesting
> Al obtener acceso a una máquina víctima, puedes usar el servidor Python en tu Kali/AttackBox para servir herramientas de enumeración (LinPEAS, etc.) y descargarlas en la máquina víctima con wget.

---

## Cron Jobs — Programar Tareas Automáticas

El proceso `cron` ejecuta comandos automáticamente según un horario definido en un **crontab** (cron table).

### Formato del crontab

```
MIN  HOUR  DOM  MON  DOW  CMD
 │    │     │    │    │    └── Comando a ejecutar
 │    │     │    │    └─────── Day of Week (0-7, donde 0 y 7 = domingo)
 │    │     │    └──────────── Month (1-12)
 │    │     └───────────────── Day of Month (1-31)
 │    └─────────────────────── Hour (0-23)
 └──────────────────────────── Minute (0-59)
```

**El asterisco `*` = cualquier valor (no importa)**

### Ejemplos

```bash
# Ejecutar backup cada 12 horas
0 */12 * * * cp -R /home/cmnatic/Documents /var/backups/

# Ejecutar script todos los días a las 3am
0 3 * * * /home/user/script.sh

# Ejecutar cada minuto
* * * * * /ruta/al/script.sh

# Ejecutar cada lunes a las 9am
0 9 * * 1 /ruta/script.sh

# Ejecutar el primer día de cada mes a medianoche
0 0 1 * * /ruta/backup.sh
```

### Gestionar crontabs

```bash
crontab -e      # Editar el crontab del usuario actual
crontab -l      # Listar los crons del usuario actual
crontab -r      # Eliminar el crontab del usuario actual
sudo crontab -e # Editar el crontab de root
```

> [!tip] Herramientas para crear crontabs
> - **[crontab.guru](https://crontab.guru)** — editor visual online que explica el formato en inglés
> - **Crontab Generator** — genera el formato automáticamente desde opciones visuales

> [!warning] Cron jobs peligrosos en ciberseguridad
> Los cron jobs que corren como root son un vector clásico de escalada de privilegios. Si un script ejecutado por root tiene permisos de escritura para usuarios normales, un atacante puede modificarlo y ejecutar código arbitrario como root.
>
> Revisar siempre: `cat /etc/crontab` y `crontab -l` durante enumeración.

---

## Gestión de Paquetes — apt

`apt` (Advanced Package Tool) gestiona la instalación, actualización y eliminación de software en sistemas basados en Debian/Ubuntu.

### Comandos esenciales

```bash
# Instalar software
sudo apt install nombre-paquete

# Actualizar lista de repositorios disponibles (siempre antes de instalar)
sudo apt update

# Actualizar todos los paquetes instalados
sudo apt upgrade

# Instalar + actualizar en un paso
sudo apt update && sudo apt upgrade -y

# Eliminar paquete
sudo apt remove nombre-paquete

# Buscar paquete disponible
apt search nombre-paquete

# Ver información de un paquete
apt show nombre-paquete
```

### Repositorios y GPG Keys

Los paquetes vienen de **repositorios** (servidores con software verificado). Para agregar un repositorio de terceros:

```bash
# 1. Añadir la GPG key del proveedor (verifica la autenticidad)
wget -qO - https://proveedor.com/clave.gpg | sudo apt-key add -

# 2. Agregar el repositorio a las fuentes de apt
sudo nano /etc/apt/sources.list.d/nuevo-repo.list
# Añadir la línea de repositorio dentro del archivo

# 3. Actualizar apt para que reconozca el nuevo repositorio
sudo apt update

# 4. Instalar el software
sudo apt install nombre-paquete
```

```bash
# Eliminar un repositorio PPA
sudo add-apt-repository --remove ppa:NOMBRE/ppa
sudo apt remove nombre-paquete
```

> [!info] ¿Por qué GPG keys?
> Las GPG keys garantizan que el software descargado viene realmente del desarrollador y no fue modificado. Si la clave no coincide, apt rechaza la instalación. Es la cadena de confianza del software en Linux.

---

## Archivos de Log en Linux

Los logs registran la actividad de servicios, aplicaciones y el sistema. Se encuentran en `/var/log`.

```bash
ls /var/log
# apache2/  auth.log  fail2ban.log  syslog  kern.log ...
```

### Logs importantes

| Archivo | Servicio | Qué registra |
|---------|----------|-------------|
| `/var/log/apache2/access.log` | Apache (web server) | Cada request HTTP (IP, método, URL, status, tamaño) |
| `/var/log/apache2/error.log` | Apache | Errores del servidor web |
| `/var/log/auth.log` | Sistema | Intentos de login SSH, uso de sudo, cambios de usuario |
| `/var/log/fail2ban.log` | Fail2ban | IPs bloqueadas por intentos de brute force |
| `/var/log/syslog` | Sistema | Actividad general del kernel y servicios |
| `/var/log/kern.log` | Kernel | Mensajes del kernel de Linux |

### Ver logs en la terminal

```bash
# Ver las últimas líneas
tail /var/log/apache2/access.log
tail -n 50 /var/log/apache2/access.log     # últimas 50 líneas

# Ver en tiempo real (monitoreo activo)
tail -f /var/log/auth.log

# Buscar dentro del log
grep "192.168.1.100" /var/log/apache2/access.log
grep "Failed password" /var/log/auth.log
grep "BANNED" /var/log/fail2ban.log

# Ver el log completo (paginado)
cat /var/log/syslog | less
```

### Formato de una línea en access.log de Apache

```
10.9.232.11 - - [04/May/2021:12:26:16 +0000] "GET /catsanddogs.jpg HTTP/1.1" 200 51095
│           │   │                              │    │                        │   │
IP cliente  │   Timestamp                     Método URL                  Status Tamaño
            Identidad/Auth (- = vacío)
```

**Qué revela este log:**
- Quién (IP) visitó el sitio
- Qué recurso accedieron (URL)
- Cuándo (timestamp)
- Resultado (HTTP status code)
- Tamaño de la respuesta

> [!tip] Los logs son clave en Blue Team
> Durante un incidente, los logs de Apache son la primera fuente de evidencia para identificar ataques web (SQLi, directory traversal, fuzzing). Combinar `access.log` con `auth.log` permite reconstruir el timeline completo de una intrusión.

> [!info] Log Rotation
> El OS gestiona automáticamente los logs para que no llenen el disco. El proceso se llama **log rotation** — archiva logs viejos (comprimiéndolos en `.gz`) y crea nuevos archivos vacíos.

---

## Resumen de Comandos Clave

```bash
# Editores
nano archivo.txt          # Editor amigable
vim archivo.txt           # Editor avanzado

# Descargar / Transferir
wget URL                  # Descargar desde web
scp local.txt user@IP:/ruta/  # Subir a máquina remota
scp user@IP:/ruta/file.txt local/  # Descargar de máquina remota
python3 -m http.server    # Servidor HTTP en puerto 8000

# Cron
crontab -e                # Editar tareas programadas
crontab -l                # Ver tareas actuales

# apt
sudo apt update           # Actualizar lista de repos
sudo apt install pkg      # Instalar paquete
sudo apt remove pkg       # Eliminar paquete

# Logs
tail -f /var/log/auth.log       # Monitoreo en tiempo real
grep "ERROR" /var/log/syslog    # Buscar errores
cat /var/log/apache2/access.log # Ver accesos web
```

---

## Notas relacionadas

- [[Linux-Permisos-y-Sistema]] — chmod, su, /etc, /var, /tmp
- [[Sistemas-Operativos]] — Tipos de OS, kernel, CLI vs GUI
- [[Defensiva-Blue-Team]] — Los logs de /var/log son herramienta de detección del Blue Team
- [[Ofensiva-Pentesting]] — Python HTTP server y SCP para exfiltración y entrega de herramientas
- [[TCP-UDP-Puertos]] — SCP usa puerto 22 (SSH); HTTP server usa puerto 8000
