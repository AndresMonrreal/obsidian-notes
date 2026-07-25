# REMnux — Análisis de malware (documentos, red, memoria)

Relacionado: [[Herramientas-CyberChef]] · [[Herramientas-CAPA-Malware-Analysis]] · [[Digital-Forensics]] · [[Linux-Herramientas-y-Admin]]

## Qué es
Distro especializada en análisis de malware. Incluye Volatility, YARA, Wireshark, oledump, INetSim y más — un lab de análisis listo para usar sin instalar nada manualmente.

## `oledump.py` — análisis de documentos OLE2 (Office malicioso)
Analiza archivos **OLE2** (Structured Storage / Compound File Binary Format) usados por Office (docx, xlsm, etc.) para detectar macros VBA maliciosas.

```bash
oledump.py agenttesla.xlsm
```
Salida: lista de **data streams** con índice (A1, A2...). Una **M** mayúscula junto al índice indica que hay una **Macro** ahí.
```
A: xl/vbaProject.bin
 A1:       468 'PROJECT'
 A4: M     688 'VBA/ThisWorkbook'   ← Macro aquí
```

Ver el contenido de un stream específico:
```bash
oledump.py agenttesla.xlsm -s 4          # -s = --select, número del stream
oledump.py agenttesla.xlsm -s 4 --vbadecompress   # descomprime y hace legible el VBA
```

### Flujo de análisis de un macro malicioso
1. `oledump.py archivo.xlsm` → identificar el stream con `M`.
2. `oledump.py archivo.xlsm -s N --vbadecompress` → ver el script VBA legible.
3. Si el script tiene un PowerShell ofuscado (con `*` y `^` como ruido), copiarlo a **CyberChef** ([[Herramientas-CyberChef]]) y usar **Find/Replace** dos veces: reemplazar `*` por nada, y `^` por nada.
4. Analizar el PowerShell resultante:
   - `-WindowStyle hidden` → oculta la ventana de PowerShell.
   - `-executionpolicy bypass` → ignora la política de ejecución de scripts.
   - `Invoke-WebRequest -Uri <URL> -OutFile $TempFile` → descarga un archivo.
   - `Start-Process $TempFile` → ejecuta el archivo descargado.

> Patrón típico: abrir el documento → corre la macro → PowerShell oculto descarga un `.exe` disfrazado de PDF → lo ejecuta. Técnica común para evadir detección temprana.

## INetSim — simular una red falsa
Suite de simulación de servicios de internet (DNS, HTTP, HTTPS, FTP, SMTP...) para observar el comportamiento de red del malware sin exponerlo a internet real.

### Configuración
```bash
ifconfig                              # obtener la IP de la máquina REMnux
sudo nano /etc/inetsim/inetsim.conf   # editar config
```
Buscar `#dns_default_ip 0.0.0.0`, quitar el `#` y poner la IP de la máquina:
```
dns_default_ip  MACHINE_IP
```
Guardar: `Ctrl+O` → Enter → salir: `Ctrl+X`.

Verificar el cambio:
```bash
cat /etc/inetsim/inetsim.conf | grep dns_default_ip
```

### Iniciar la simulación
```bash
sudo inetsim
```
Confirmar que dice **"Simulation running"** al final (ignorar el fallo de `http_80_tcp` si aparece).

### Probar desde otra máquina (ej. AttackBox)
```bash
sudo wget https://MACHINE_IP/second_payload.zip --no-check-certificate
sudo wget https://MACHINE_IP/second_payload.ps1 --no-check-certificate
```
Todos los archivos descargados son **falsos** (fake files) generados por INetSim — simula que el malware descarga un payload secundario.

### Reporte de conexiones
Al detener INetSim, genera un reporte de todo lo capturado:
```bash
sudo cat /var/log/inetsim/report/report.<PID>.txt
```
Muestra método, URL y archivo fake devuelto para cada conexión.

## Volatility 3 — análisis de imágenes de memoria
`vol3` corre plugins sobre un memory dump. Sintaxis general:
```bash
vol3 -f archivo.mem <plugin>
```

### Plugins Windows más usados
| Plugin | Qué hace |
|--------|----------|
| `windows.pstree.PsTree` | procesos en árbol (por parent PID) |
| `windows.pslist.PsList` | lista de procesos activos |
| `windows.psscan.PsScan` | escanea procesos (incluso ocultos/terminados) |
| `windows.cmdline.CmdLine` | argumentos de línea de comandos de cada proceso |
| `windows.filescan.FileScan` | escanea objetos de archivo en memoria |
| `windows.dlllist.DllList` | módulos/DLLs cargados por proceso |
| `windows.malfind.Malfind` | rangos de memoria con posible **código inyectado** |

```bash
vol3 -f wcry.mem windows.pstree.PsTree
vol3 -f wcry.mem windows.malfind.Malfind
```

### Preprocesar en bulk (loop) — guardar cada plugin a un .txt
```bash
for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do
  vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt
done
```
- `-q` → modo silencioso (sin barra de progreso)
- `-f` → archivo de memoria a leer
- `> archivo.txt` → guarda el output de cada plugin

## `strings` — extraer texto imprimible de la memoria
```bash
strings wcry.mem > wcry.strings.ascii.txt              # ASCII
strings -e l wcry.mem > wcry.strings.unicode_le.txt     # 16-bit little endian
strings -e b wcry.mem > wcry.strings.unicode_be.txt     # 16-bit big endian
```
Los tres formatos revelan información distinta (rutas, URLs, mensajes) útil para el analista.

> Buena práctica forense: **preprocesar** toda la evidencia (correr los plugins y guardar resultados) para que cualquier analista pueda buscar rápido después, sin re-correr las herramientas.
