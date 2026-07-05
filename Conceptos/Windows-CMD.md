---
tags:
  - windows
  - cmd
  - redes
  - networking
  - procesos
  - filesystem
  - administracion
fecha: 2026-07-01
ruta: SEC0
fuente: TryHackMe — Windows Fundamentals / Cyber Security 101
---

# 🪟 Windows — Command Prompt (CMD)

---

## ¿Qué es CMD?

La línea de comandos de Windows (`cmd.exe`) es el intérprete de comandos heredado de MS-DOS. Ejecuta comandos que se encuentren dentro del **PATH del sistema**.

```cmd
C:\>set
Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;...
```

> [!tip] Para ver el PATH completo: `set` o `echo %PATH%`

---

## Información del Sistema

| Comando | Qué hace |
|---------|----------|
| `ver` | Versión del OS |
| `systeminfo` | Información completa: OS, CPU, RAM, hostname, dominio |
| `hostname` | Nombre del equipo |
| `whoami` | Usuario actual |
| `set` | Variables de entorno (incluye PATH) |
| `driverquery` | Lista todos los drivers instalados |

```cmd
C:\>ver
Microsoft Windows [Version 10.0.17763.1821]

C:\>systeminfo
Host Name:    WIN-SRV-2019
OS Name:      Microsoft Windows Server 2019 Datacenter
OS Version:   10.0.17763 N/A Build 17763
```

**Truco:** si la salida es muy larga, pásala por `more`:
```cmd
driverquery | more        ← avanza página a página con Spacebar, salir con Ctrl+C
systeminfo | more
```

**Otros atajos útiles:**
```cmd
help comando    ← muestra ayuda del comando
cls             ← limpia la pantalla
```

---

## Configuración de Red

### `ipconfig` — Información básica de red

```cmd
C:\>ipconfig

Ethernet adapter Ethernet:
   IPv4 Address. . . : 10.10.230.237
   Subnet Mask . . . : 255.255.0.0
   Default Gateway . : 10.10.0.1
```

### `ipconfig /all` — Información completa

Añade: MAC address, DHCP habilitado/deshabilitado, DNS servers, lease del DHCP.

```cmd
C:\>ipconfig /all

   Physical Address. . . : 02-B7-DF-1D-0D-99   ← MAC address
   DHCP Enabled. . . . . : Yes
   DHCP Server . . . . . : 10.10.0.1
   DNS Servers . . . . . : 10.0.0.2
```

> [!tip] ¿Cómo ver la MAC address desde CMD?
> `ipconfig /all` → busca la línea **Physical Address**

---

## Herramientas de Diagnóstico de Red

### `ping` — Verificar conectividad

Envía paquetes ICMP y espera respuesta. Confirma si el destino es alcanzable.

```cmd
C:\>ping example.com

Reply from 93.184.215.14: bytes=32 time=78ms TTL=52   ← respuesta recibida
Reply from 93.184.215.14: bytes=32 time=78ms TTL=52
Reply from 93.184.215.14: bytes=32 time=78ms TTL=52
Reply from 93.184.215.14: bytes=32 time=78ms TTL=52

Packets: Sent=4, Received=4, Lost=0 (0% loss)
Average = 78ms
```

**Ping sin respuesta = host caído, firewall bloqueando ICMP, o red inaccesible.**

---

### `tracert` — Trazar la ruta (Trace Route)

Muestra cada salto (router) por el que pasa el paquete hasta llegar al destino.

```cmd
C:\>tracert example.com

  1   42ms   ec2-3-248-240-3.eu-west-1.compute.amazonaws.com [3.248.240.3]
  2    *       *       *     Request timed out.   ← router que no responde ICMP
  ...
 16   78ms   93.184.215.14
```

**`*` = el router no responde a TTL-exceeded (normal, no significa fallo).**

> [!tip] Útil para diagnosticar:
> - Dónde se interrumpe la ruta hacia un destino
> - Latencia acumulada por salto
> - Identificar routers intermedios

---

### `nslookup` — Resolución de nombres DNS

```cmd
C:\>nslookup example.com
Server:  ip-10-0-0-2.eu-west-1.compute.internal   ← DNS server usado
Address: 10.0.0.2
Name:    example.com
Addresses: 2606:2800:21f:cb07:6820:80da:af6b:8b2c   ← IPv6
           93.184.215.14                              ← IPv4

C:\>nslookup example.com 1.1.1.1    ← usar DNS server específico (Cloudflare)
Server: one.one.one.one
Address: 1.1.1.1
```

---

### `netstat` — Conexiones de red activas

```cmd
C:\>netstat           ← conexiones establecidas

Active Connections
  TCP  10.10.230.237:22   ip-10-11-81:53486   ESTABLISHED
```

**Flags más útiles:**

| Flag | Descripción |
|------|-------------|
| `-a` | Todas las conexiones + puertos en escucha |
| `-b` | Ejecutable asociado a cada conexión |
| `-o` | PID (Process ID) de cada conexión |
| `-n` | Direcciones en formato numérico (sin DNS reverse lookup) |

```cmd
C:\>netstat -abon

TCP   0.0.0.0:22      0.0.0.0:0   LISTENING   2116
[sshd.exe]
TCP   0.0.0.0:3389    0.0.0.0:0   LISTENING   980
[RDP]
TCP   10.10.230.237:22  10.11.81.126:53486   ESTABLISHED   2116
[sshd.exe]
```

> [!tip] Uso en pentesting y Blue Team
> `netstat -abon` muestra todas las conexiones con el proceso responsable. Útil para detectar:
> - Backdoors escuchando en puertos inusuales
> - Conexiones salientes hacia IPs desconocidas (posible C2)
> - Servicios expuestos que no deberían estarlo

---

## Navegación por Directorios

```cmd
cd                  ← muestra directorio actual (equivalente a pwd en Linux)
cd Users            ← entrar a subdirectorio
cd ..               ← subir un nivel
cd C:\Windows\System32   ← ir a ruta absoluta

dir                 ← listar contenido del directorio
dir /a              ← incluir archivos ocultos y del sistema
dir /s              ← listar este directorio y todos los subdirectorios
tree                ← árbol visual de subdirectorios
```

**Salida de `dir`:**
```
05/01/2024  02:40 PM    <DIR>    Documents
05/01/2024  02:40 PM    <DIR>    Downloads
05/02/2024  07:57 AM          17 test.txt
              1 File(s)       17 bytes
              2 Dir(s)  14,984 bytes free
```

**Crear y eliminar directorios:**
```cmd
mkdir nombre_carpeta    ← crear directorio
rmdir nombre_carpeta    ← eliminar directorio (debe estar vacío)
```

---

## Gestión de Archivos

| Comando | Función | Equivalente Linux |
|---------|---------|------------------|
| `type archivo.txt` | Mostrar contenido | `cat` |
| `more archivo.txt` | Mostrar paginado (lento) | `less` |
| `copy origen destino` | Copiar archivo | `cp` |
| `move origen destino` | Mover archivo | `mv` |
| `del archivo.txt` | Eliminar archivo | `rm` |
| `erase archivo.txt` | Eliminar archivo (igual que del) | `rm` |

```cmd
copy test.txt test2.txt              ← copiar en mismo directorio
copy test.txt C:\backup\test.txt    ← copiar a otra ruta
move test2.txt ..                   ← mover al directorio padre
copy *.txt C:\Backups\             ← copiar todos los .txt (wildcard *)
del test2.txt
erase test2.txt                     ← equivalente a del
```

> [!tip] El wildcard `*` en Windows CMD
> `*` = cualquier nombre/extensión. Ejemplos:
> - `copy *.md C:\Markdown` → todos los .md
> - `del *.tmp` → todos los .tmp
> - `dir *.exe` → ver ejecutables en el directorio

---

## Gestión de Procesos

### `tasklist` — Ver procesos en ejecución

```cmd
C:\>tasklist

Image Name          PID  Session     Mem Usage
============== ======== =========== ============
System               4  Services         88 K
svchost.exe        704  Services     23,432 K
sshd.exe          2116  Services      6,992 K
```

**Filtrar por nombre:**
```cmd
tasklist /FI "imagename eq sshd.exe"

Image Name     PID  Session    Mem Usage
=========== ====== ========== ==========
sshd.exe    2116   Services    6,992 K
sshd.exe    2712   Services    7,668 K
```

### `taskkill` — Terminar proceso

```cmd
taskkill /PID 4567           ← matar proceso por PID
taskkill /IM chrome.exe      ← matar proceso por nombre
taskkill /F /PID 4567        ← forzar terminación (/F = force)
```

---

## CMD vs PowerShell — Comparación rápida

| | CMD | PowerShell |
|-|-----|------------|
| **Salida** | Texto plano | Objetos con propiedades |
| **Scripting** | Básico (batch .bat) | Avanzado (.ps1) |
| **Potencia** | Comandos simples | Completo (ver [[Windows-PowerShell]]) |
| **Compatibilidad** | Máxima (legacy) | Recomendado para administración moderna |
| **Iniciar PS desde CMD** | Escribir `powershell` |  |

---

## Comandos de referencia rápida

```cmd
:: Información del sistema
ver | systeminfo | hostname | whoami | set

:: Red
ipconfig | ipconfig /all
ping ejemplo.com
tracert ejemplo.com
nslookup dominio.com
netstat -abon

:: Navegación
cd | cd carpeta | cd .. | dir | dir /a | dir /s | tree

:: Archivos
type archivo.txt | more archivo.txt
copy orig dest | move orig dest | del archivo | erase archivo

:: Procesos
tasklist | tasklist /FI "imagename eq nombre.exe"
taskkill /PID 1234 | taskkill /IM nombre.exe
```

---

## Notas relacionadas

- [[Windows-PowerShell]] — La evolución moderna de CMD con objetos y cmdlets
- [[Redes-Fundamentos]] — IP, MAC, subnetting
- [[Protocolos-ARP-DHCP]] — DHCP y ARP (visible en ipconfig /all)
- [[DNS]] — nslookup y resolución de nombres
- [[TCP-UDP-Puertos]] — netstat muestra puertos TCP en uso
- [[Linux-Permisos-y-Sistema]] — Equivalentes Linux de los comandos CMD
