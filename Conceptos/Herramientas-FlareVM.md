# FlareVM — Toolkit de análisis de malware

Relacionado: [[Herramientas-REMnux]] · [[Herramientas-CAPA-Malware-Analysis]] · [[Digital-Forensics]] · [[Windows-PowerShell]]

## Qué es
"Forensics, Logic Analysis, and Reverse Engineering" — distro Windows con herramientas curadas por el equipo FLARE (Mandiant/FireEye) para reversing, análisis de malware, IR y pentesting.

## Categorías de herramientas
| Categoría | Herramientas destacadas |
|-----------|--------------------------|
| **Reverse Engineering & Debugging** | Ghidra, x64dbg, OllyDbg, Radare2, Binary Ninja, PEiD |
| **Disassemblers & Decompilers** | **CFF Explorer**, Hopper, RetDec |
| **Static & Dynamic Analysis** | **Process Hacker**, **PEview**, Dependency Walker, DIE |
| **Forensics & IR** | Volatility, Rekall, **FTK Imager** |
| **Network Analysis** | **Wireshark**, Nmap, Netcat |
| **File Analysis** | FileInsight, Hex Fiend, **HxD** |
| **Scripting & Automation** | Python, Empire |
| **Sysinternals Suite** | Autoruns, **Process Explorer**, **Process Monitor** |

## Herramientas core para investigación

### Process Monitor (Procmon)
Registra en tiempo real actividad de **archivos, registro y procesos/hilos**. Útil para malware research, troubleshooting y forense.

**Filtrar por proceso** (ej. investigar `lsass.exe`): `Ctrl+L` → configurar filtro.
```
Process Name → contains → cobalt → Include → Add → Apply
```
> LSASS maneja autenticación. Si ves accesos raros a `lsass.exe`, puede ser un intento de **credential dumping** (ej. Mimikatz).

### Process Explorer (Procexp)
Vista detallada de procesos activos: relación **padre-hijo**, usuario asociado, y qué archivos/carpetas está tocando cada proceso. Útil para ver qué proceso hijo lanza un Word/PDF/ISO malicioso.

### HxD — editor hexadecimal
Ver/editar archivos, memoria o discos en hex. Los primeros bytes `4D 5A` (little endian) = **firma de un ejecutable** (`MZ`, cabecera DOS de un PE). El **Data Inspector** muestra un byte en distintos tipos de dato.

### CFF Explorer — editor de PE
Genera hashes del archivo (MD5, SHA-1, SHA-256) para verificación de integridad y análisis de origen. Permite inspeccionar secciones del PE, incluyendo el **DOS Header** (campo `e_magic`).

### Wireshark
Analizador de tráfico de red — protocolo, IP origen/destino, puerto. Útil para detectar conexiones a C2, exfiltración, o tráfico cifrado sospechoso (TLS).

### PEStudio
Análisis estático de ejecutables **sin ejecutarlos**. Revisa:
- **Hashes** (MD5, SHA-1, SHA-256) para comparar en VirusTotal.
- **Entropy** (entropía): valores altos sugieren packing/cifrado.
- **Version info / metadata**: idioma, descripción — puede revelar origen sospechoso (ej. texto en ruso en un binario "legítimo" de Windows).
- **Manifest** (sección Administrator): `requestedExecutionLevel` indica si pide elevación de privilegios.
- **Imports (IAT)**: funciones importadas; el tab **blacklist** ordena las APIs peligrosas primero. Ej.:
  - `set_UseShellExecute` → lanza otros procesos vía el shell del SO.
  - `CryptoStream`, `RijndaelManaged`, `CipherMode`, `CreateDecryptor` → uso de cifrado AES (Rijndael); típico en ransomware o comunicación cifrada.

### FLOSS (FLARE Obfuscated String Solver)
Extrae y **de-ofusca** strings estáticas y dinámicamente decodificadas de un binario (mejora sobre `strings.exe`).
```powershell
floss .\cobaltstrike.exe
FLOSS.exe .\windows.exe > windows.txt
```
Revela rutas de archivo, URLs (C2), IPs, llamadas API, mensajes de error, claves de registro/cifrado ocultas en el binario.

## Flujo práctico de análisis (ejemplo)

### 1. Análisis estático inicial con PEStudio
Abrir el archivo sospechoso → revisar hashes (comparar en VirusTotal) → revisar entropía → revisar metadata/idioma → revisar imports sospechosos.

### 2. Confirmar strings con FLOSS
```powershell
FLOSS.exe .\windows.exe > windows.txt
```
Comparar contra lo visto en PEStudio.

### 3. Analizar comportamiento en ejecución
1. Ejecutar el binario (**solo en entorno aislado**).
2. Abrir **Process Explorer** → confirmar la relación padre-hijo (ej. `explorer.exe` → `cobaltstrike.exe`).
3. Clic derecho en el proceso → **Properties** → tab **TCP/IP** → ver destino de la conexión (IP:puerto).
4. **Verificar con una segunda herramienta**: reiniciar el proceso y abrir **Process Monitor**, filtrar por nombre del proceso (`Ctrl+L`), confirmar la misma conexión sospechosa.

> Regla de oro: no confiar en una sola herramienta — cruzar resultados (Process Explorer + Process Monitor, o PEStudio + FLOSS) para verificar hallazgos antes de reportar.

## Disclaimer
Las muestras en FlareVM son **maliciosas de verdad**. La VM no tiene acceso a internet. Nunca descargar, ejecutar (fuera de la VM) o distribuir estos archivos.
