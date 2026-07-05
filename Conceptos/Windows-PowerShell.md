---
tags:
  - windows
  - powershell
  - scripting
  - administracion
  - blue-team
  - red-team
  - automatizacion
fecha: 2026-07-01
ruta: SEC0
fuente: TryHackMe — Windows Fundamentals / Cyber Security 101
---

# ⚡ Windows PowerShell

---

## ¿Qué es PowerShell?

PowerShell es una shell de comandos + lenguaje de scripting + framework de gestión de configuración, desarrollado por Microsoft sobre .NET. A diferencia de CMD que trabaja con **texto plano**, PowerShell trabaja con **objetos**.

```
CMD output:      "sshd.exe                2116   Services   6,992 K"
                 ← texto, hay que parsear para extraer el PID

PowerShell:      [Object] con propiedades: .Name, .Id, .CPU, .WorkingSet
                 ← accedes directamente a Get-Process | Where-Object {$_.Id -eq 2116}
```

**Creado por:** Jeffrey Snover (2006). Solución al problema de que Windows usa APIs y datos estructurados (no archivos de texto como Unix).

**PowerShell Core (2016):** versión open-source, multiplataforma (Windows, macOS, Linux).

### Lanzar PowerShell

```cmd
# Desde CMD:
powershell

# Desde GUI:
# Start Menu → "powershell"
# Win + R → "powershell"
# File Explorer → escribe "powershell" en la barra de ruta
```

El prompt cambia de `C:\>` a `PS C:\>`.

---

## Sintaxis Fundamental: Verb-Noun

Los comandos de PowerShell se llaman **cmdlets** (command-lets). Siguen el patrón `Verbo-Sustantivo`:

| Verbo | Significa | Ejemplo |
|-------|-----------|---------|
| Get | Obtener/leer | `Get-Content`, `Get-Process` |
| Set | Configurar/cambiar | `Set-Location`, `Set-Date` |
| New | Crear | `New-Item`, `New-Object` |
| Remove | Eliminar | `Remove-Item`, `Remove-Service` |
| Copy | Copiar | `Copy-Item` |
| Move | Mover | `Move-Item` |
| Start | Iniciar | `Start-Process` |
| Stop | Detener | `Stop-Process` |
| Invoke | Ejecutar | `Invoke-Command` |
| Find | Buscar | `Find-Module` |
| Install | Instalar | `Install-Module` |
| Select | Seleccionar/filtrar | `Select-Object`, `Select-String` |
| Sort | Ordenar | `Sort-Object` |
| Where | Filtrar condicional | `Where-Object` |

---

## Cmdlets de Descubrimiento

### `Get-Command` — Listar todos los cmdlets disponibles

```powershell
PS> Get-Command

CommandType  Name                     Version  Source
-----------  ----                     -------  ------
Alias        Add-AppPackage           2.0.1.0  Appx
Function     Add-BCDataCacheExtension 1.0.0.0  BranchCache
Cmdlet       Add-AppxPackage          2.0.1.0  Appx

# Filtrar por tipo
PS> Get-Command -CommandType "Cmdlet"
PS> Get-Command -CommandType "Function"
```

### `Get-Help` — Ayuda detallada de un cmdlet

```powershell
PS> Get-Help Get-Date               ← información básica
PS> Get-Help Get-Date -examples     ← ejemplos de uso
PS> Get-Help Get-Date -detailed     ← información detallada
PS> Get-Help Get-Date -full         ← información técnica completa
```

### `Get-Alias` — Ver alias disponibles

Los alias son atajos que hacen cmdlets accesibles con nombres familiares de CMD o Linux:

```powershell
PS> Get-Alias

Alias   cat   → Get-Content     ← equivalente a cat de Linux
Alias   cd    → Set-Location
Alias   dir   → Get-ChildItem
Alias   ls    → Get-ChildItem
Alias   rm    → Remove-Item
Alias   cp    → Copy-Item
Alias   mv    → Move-Item
Alias   ps    → Get-Process
```

---

## Gestión del Sistema de Archivos

| Cmdlet PowerShell | Alias | Equivalente CMD | Equivalente Linux |
|------------------|-------|-----------------|------------------|
| `Get-ChildItem` | `ls`, `dir` | `dir` | `ls` |
| `Set-Location` | `cd` | `cd` | `cd` |
| `New-Item` | — | `mkdir` / `echo >` | `mkdir` / `touch` |
| `Remove-Item` | `rm` | `rmdir` / `del` | `rm` |
| `Copy-Item` | `cp` | `copy` | `cp` |
| `Move-Item` | `mv` | `move` | `mv` |
| `Get-Content` | `cat` | `type` | `cat` |

### Ejemplos

```powershell
# Listar directorio
Get-ChildItem
Get-ChildItem -Path "C:\Windows"

# Navegar
Set-Location -Path ".\Documents"
Set-Location -Path "C:\Windows\System32"
cd ..                     ← alias funciona igual

# Crear
New-Item -Path ".\carpeta\subcarpeta" -ItemType "Directory"
New-Item -Path ".\archivo.txt" -ItemType "File"

# Eliminar
Remove-Item -Path ".\archivo.txt"
Remove-Item -Path ".\carpeta" -Recurse    ← borrar directorio con contenido

# Copiar y mover
Copy-Item -Path .\original.txt -Destination .\copia.txt
Move-Item -Path .\archivo.txt -Destination C:\backup\

# Leer contenido
Get-Content -Path ".\archivo.txt"
```

---

## Piping — El poder real de PowerShell

El pipe `|` pasa la **salida de un cmdlet como entrada al siguiente**. En PowerShell, lo que se pasa son **objetos completos** (con propiedades), no texto.

```powershell
Get-ChildItem | Sort-Object Length    ← listar archivos ordenados por tamaño
```

### `Sort-Object` — Ordenar objetos

```powershell
Get-ChildItem | Sort-Object Length        ← menor a mayor
Get-ChildItem | Sort-Object Length -Descending  ← mayor a menor
Get-ChildItem | Sort-Object Name          ← orden alfabético
Get-Process | Sort-Object CPU -Descending ← procesos que más CPU usan
```

### `Where-Object` — Filtrar por condición

```powershell
# Operadores de comparación:
-eq    ← igual a (equal)
-ne    ← no igual (not equal)
-gt    ← mayor que (greater than)  → estricto
-ge    ← mayor o igual (greater or equal)
-lt    ← menor que (less than)     → estricto
-le    ← menor o igual (less or equal)
-like  ← coincide con patrón (* como wildcard)
```

```powershell
# Solo archivos .txt
Get-ChildItem | Where-Object -Property "Extension" -eq ".txt"

# Archivos mayores de 1000 bytes
Get-ChildItem | Where-Object -Property "Length" -gt 1000

# Archivos cuyo nombre empieza con "ship"
Get-ChildItem | Where-Object -Property "Name" -like "ship*"

# Procesos que usan más de 100MB de RAM
Get-Process | Where-Object -Property "WorkingSet" -gt 104857600
```

### `Select-Object` — Seleccionar propiedades o limitar resultados

```powershell
# Mostrar solo Name y Length de los archivos
Get-ChildItem | Select-Object Name, Length

# Los 5 procesos con más CPU
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5

# El archivo más grande del directorio
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 1
```

### `Select-String` — Buscar texto en archivos (como grep)

```powershell
# Buscar "hat" en un archivo
Select-String -Path ".\archivo.txt" -Pattern "hat"

# Buscar en múltiples archivos con wildcard
Select-String -Path ".\*.log" -Pattern "ERROR"

# Con regex
Select-String -Path ".\access.log" -Pattern "192\.168\.\d+\.\d+"

# Equivalente Linux: grep "hat" archivo.txt
```

> [!tip] Cadena de cmdlets (pipeline complejo)
> ```powershell
> # Archivo más grande de un directorio:
> Get-ChildItem | Where-Object {$_.Extension -eq ".txt"} | Sort-Object Length -Descending | Select-Object -First 1
> ```

---

## Información del Sistema y Red

### `Get-ComputerInfo` — Información completa del sistema

```powershell
PS> Get-ComputerInfo
WindowsProductName : Windows Server 2022 Datacenter
WindowsEditionId   : ServerDatacenter
CsNumberOfProcessors : 2
CsPhysicallyInstalledMemory : 4194304  ← RAM en KB
```

Más completo que `systeminfo` de CMD.

### `Get-LocalUser` — Usuarios locales del sistema

```powershell
PS> Get-LocalUser

Name               Enabled  Description
----               -------  -----------
Administrator      True     Built-in account for administering
captain            True     The beloved captain of this pirate ship.
Guest              False    Built-in account for guest access
```

> [!tip] En Blue Team y pentesting
> `Get-LocalUser` muestra cuentas habilitadas/deshabilitadas. Busca cuentas con nombres inusuales o cuentas deshabilitadas que de repente están activas → posible compromiso.

### `Get-NetIPConfiguration` — Configuración de red

```powershell
PS> Get-NetIPConfiguration

InterfaceAlias     : Ethernet
IPv4Address        : 10.10.178.209
IPv4DefaultGateway : 10.10.0.1
DNSServer          : 10.0.0.2
```

### `Get-NetIPAddress` — Todas las IPs del sistema

```powershell
PS> Get-NetIPAddress
# Muestra IPv4 e IPv6 de todas las interfaces, incluyendo loopback
```

---

## Procesos, Servicios y Conexiones (Incident Response / Threat Hunting)

### `Get-Process` — Procesos en ejecución

```powershell
PS> Get-Process

Handles  NPM(K)  PM(K)  WS(K)  CPU(s)  Id  ProcessName
-------  ------  -----  -----  ------  --  -----------
    309      13  18312   1256    0.52  1524  amazon-ssm-agent
     78       6   4440    944    0.02   516  cmd
```

```powershell
# Procesos ordenados por CPU
Get-Process | Sort-Object CPU -Descending

# Proceso específico
Get-Process -Name "sshd"

# Propiedades de un proceso
Get-Process | Select-Object Name, Id, CPU, WorkingSet
```

### `Get-Service` — Servicios del sistema

```powershell
PS> Get-Service

Status   Name               DisplayName
------   ----               -----------
Running  AmazonSSMAgent     Amazon SSM Agent
Stopped  AppIDSvc           Application Identity
Running  BFE                Base Filtering Engine
```

```powershell
# Solo servicios en ejecución
Get-Service | Where-Object -Property "Status" -eq "Running"

# Servicio específico
Get-Service -Name "wuauserv"    ← Windows Update
```

### `Get-NetTCPConnection` — Conexiones TCP activas

```powershell
PS> Get-NetTCPConnection

LocalAddress     LocalPort  RemoteAddress    RemotePort  State        OwningProcess
------------     ---------  -------------    ----------  -----        -------------
::               22         ::               0           Listen       1444
0.0.0.0          3389       0.0.0.0          0           Listen       980
10.10.178.209    22         10.14.87.60      53523       Established  1444
```

```powershell
# Solo conexiones establecidas
Get-NetTCPConnection | Where-Object -Property "State" -eq "Established"

# Conexiones salientes a IPs externas
Get-NetTCPConnection | Where-Object -Property "RemoteAddress" -notlike "0.0.0.0"
```

> [!warning] Conexión establecida hacia IP desconocida = señal de alerta
> Durante incident response, `Get-NetTCPConnection` combinado con `Get-Process` puede revelar backdoors o conexiones C2 activas.

### `Get-FileHash` — Calcular hash de un archivo

```powershell
PS> Get-FileHash -Path .\archivo.exe

Algorithm  Hash                                      Path
---------  ----                                      ----
SHA256     54D2EC3C12BF3D...                         C:\...\archivo.exe
```

```powershell
# Comparar hashes para verificar integridad
Get-FileHash -Path .\archivo.exe -Algorithm MD5
Get-FileHash -Path .\archivo.exe -Algorithm SHA1
```

> [!tip] Uso en ciberseguridad
> El hash de un archivo es su "huella digital". Si un malware modifica un archivo del sistema, su hash cambia. Puedes comparar contra hashes conocidos en bases de datos como VirusTotal.

### Alternate Data Streams (ADS) — NTFS

Los ADS son flujos de datos ocultos que se pueden adjuntar a archivos NTFS. El malware los usa para esconder payloads.

```powershell
PS> Get-Item -Path "C:\archivo.txt" -Stream *

Stream     Length
------     ------
:$DATA     13      ← stream normal del archivo
housinginfo 21     ← ADS oculto "housinginfo"
```

`:$DATA` = contenido normal del archivo. Cualquier otro stream = ADS potencialmente sospechoso.

---

## Módulos — Extender PowerShell

```powershell
# Buscar módulos en PowerShell Gallery
Find-Module -Name "PowerShell*"

# Instalar módulo
Install-Module -Name "PowerShellGet"
```

> [!info] Requiere conexión a internet. En entornos aislados (laboratorios, servidores offline) puede no estar disponible.

---

## Scripting con PowerShell — Concepto

Los scripts de PowerShell son archivos `.ps1` con secuencias de cmdlets.

**Blue Team:**
- Automatizar análisis de logs
- Detectar anomalías
- Extraer IOCs de eventos del sistema

**Red Team:**
- Enumeración de sistemas
- Ejecutar comandos remotos
- Evasión de defensas con scripts ofuscados

### `Invoke-Command` — Ejecutar comandos en sistemas remotos

```powershell
# Ejecutar un script en un servidor remoto
Invoke-Command -FilePath c:\scripts\script.ps1 -ComputerName Servidor01

# Ejecutar un cmdlet en un servidor remoto
Invoke-Command -ComputerName Servidor01 -Credential Dominio\Usuario -ScriptBlock { Get-Process }

# ScriptBlock = bloque de código a ejecutar remotamente
```

> [!warning] `Invoke-Command` en pentesting
> Es uno de los cmdlets más utilizados por atacantes para movimiento lateral. Un defensor debe monitorear su ejecución, especialmente hacia múltiples hosts. En PowerShell logging habilitado, queda registrado en los event logs de Windows.

---

## Comparativa CMD vs PowerShell — Quick Reference

```
CMD                         PowerShell
─────────────────────────────────────────────────────
dir                    →    Get-ChildItem  (o ls, dir)
cd carpeta             →    Set-Location -Path carpeta
type archivo.txt       →    Get-Content -Path archivo.txt
copy orig dest         →    Copy-Item -Path orig -Destination dest
move orig dest         →    Move-Item -Path orig -Destination dest
del archivo            →    Remove-Item -Path archivo
mkdir nombre           →    New-Item -Path nombre -ItemType Directory
tasklist               →    Get-Process
taskkill /PID 1234     →    Stop-Process -Id 1234
netstat -abon          →    Get-NetTCPConnection
ipconfig /all          →    Get-NetIPConfiguration
systeminfo             →    Get-ComputerInfo
findstr "texto" file   →    Select-String -Path file -Pattern "texto"
```

---

## Notas relacionadas

- [[Windows-CMD]] — El CMD heredado: comandos básicos y networking
- [[Linux-Scripting-y-Shells]] — Equivalente Linux: bash scripting
- [[Defensiva-Blue-Team]] — Get-Process, Get-Service, Get-NetTCPConnection son herramientas de incident response
- [[Ofensiva-Pentesting]] — Invoke-Command y PowerShell ofensivo
- [[TCP-UDP-Puertos]] — Los puertos visibles en Get-NetTCPConnection
