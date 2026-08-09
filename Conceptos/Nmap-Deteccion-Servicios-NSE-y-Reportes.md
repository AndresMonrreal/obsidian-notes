# Nmap — Detección de Servicios/OS, NSE y Formatos de Reporte

Relacionado: [[Nmap-Escaneo-Puertos]] · [[Nmap-Escaneo-Avanzado-y-Evasion]] · [[TCP-Escaneo-Riesgos-y-Defensa]] · [[Reconocimiento-Activo]]

Cuarta y última sala de la serie de Nmap. Con los puertos abiertos ya identificados, esta sala cubre los pasos que siguen: qué corre exactamente en cada puerto, qué sistema operativo hay detrás, cómo extender Nmap con scripts, y cómo guardar los resultados para un reporte.

## Detección de servicio y versión — `-sV`
```bash
sudo nmap -sV 10.201.118.127
sudo nmap -sV --version-light 10.201.118.127   # intensidad 2 (rápido, probes más probables)
sudo nmap -sV --version-all 10.201.118.127     # intensidad 9 (todos los probes disponibles, más lento)
sudo nmap -sV --version-intensity LEVEL 10.201.118.127   # nivel manual 0-9
```
El nombre del **servicio** (`ssh`, `http`, etc.) es una suposición basada en el número de puerto — no requiere conexión. La **versión** (`OpenSSH 9.2p1`, `nginx 1.22.1`) sí requiere que Nmap complete la conexión y hable con el servicio para capturar el banner. Por eso **`-sV` obliga a completar el 3-way handshake real, incluso pidiendo un SYN scan (`-sS`)**: no existe forma sigilosa de obtener la versión, hay que conversar con el servicio.

## Detección de sistema operativo — `-O`
```bash
sudo nmap -sS -O 10.65.146.116
```
Compara TTL inicial, comportamiento de números de secuencia TCP, tamaño de ventana, y reacción ante paquetes "imposibles" (ver [[Nmap-Escaneo-Avanzado-y-Evasion]]) contra una base de datos de miles de huellas conocidas.

- **TTL**: decrece 1 por cada router que atraviesa un paquete. Linux normalmente arranca en **64**, Windows en **128** — una pista rápida incluso sin match exacto.
- Es común que Nmap responda **"No exact OS matches for host"** — virtualización, filtrado de firewall, capas de red de nube, kernels personalizados o middleboxes alteran el fingerprint lo suficiente para que no calce con ninguna firma exacta de la base de datos. Aun sin match exacto, los datos parciales (TTL, comportamiento de secuencia) siguen siendo pistas útiles.

> **`-O` nunca es 100% definitivo** — complementar siempre con otra técnica de recon (banners de `-sV`, comportamiento observado en `-sC`) antes de asumir el sistema operativo real de un objetivo.

## Traceroute — `--traceroute`
```bash
nmap -sS --traceroute 10.201.118.127
```
Agrega la ruta de routers entre el atacante y el objetivo al final del resultado. El traceroute de Nmap funciona **al revés** que el `traceroute`/`tracert` estándar de Linux/macOS/Windows: en vez de empezar con un TTL bajo e ir subiendo, Nmap empieza con un TTL **alto** y lo va bajando. Muchos routers están configurados para **no** mandar mensajes ICMP "Time-to-Live exceeded" — eso deja huecos (hops sin IP) en la ruta reconstruida.

## Nmap Scripting Engine (NSE)
Nmap trae ~600 scripts Lua preinstalados en `/usr/share/nmap/scripts/`, organizados por el protocolo que atacan (todos los `http-*.nse`, `ftp-*.nse`, etc. — más de 130 solo para HTTP). No hace falta saber Lua para usarlos.

```bash
ls /usr/share/nmap/scripts | grep ^http     # ver todos los scripts de un protocolo
less /usr/share/nmap/scripts/ftp-brute.nse  # leer qué hace un script antes de correrlo
```

### Categorías de script
| Categoría | Qué hace |
|---|---|
| `auth` | scripts relacionados con autenticación |
| `broadcast` | descubre hosts mandando mensajes broadcast |
| `brute` | fuerza bruta de contraseñas contra logins |
| `default` | set de scripts "seguros y útiles" que corren con `-sC` |
| `discovery` | recupera info accesible (tablas de BD, nombres DNS) |
| `dos` | detecta servidores vulnerables a Denial of Service |
| `exploit` | intenta explotar vulnerabilidades directamente |
| `external` | consulta un servicio de terceros (ej. VirusTotal) |
| `fuzzer` | lanza ataques de fuzzing |
| `intrusive` | scripts agresivos (brute-force, explotación) — riesgo de tumbar el servicio |
| `malware` | busca backdoors |
| `safe` | scripts que no deberían crashear el objetivo |
| `version` | obtiene versiones de servicio |
| `vuln` | revisa vulnerabilidades/exploits conocidos |

Un mismo script puede pertenecer a varias categorías. ⚠️ Categorías como `brute`, `dos`, `exploit` e `intrusive` pueden tumbar o dañar el servicio objetivo — usar con cuidado y solo con autorización clara.

### Correr scripts
```bash
sudo nmap -sS -sC 10.201.118.127                # -sC = scripts de la categoría "default"
sudo nmap -sS --script=default 10.201.118.127   # equivalente explícito a -sC

sudo nmap -sS --script "http-date" 10.201.118.127    # un script específico por nombre
sudo nmap -sS --script "ftp*" 10.201.118.127         # patrón — corre todos los que empiecen con "ftp"
```
`-sC` reveló, por ejemplo, todas las llaves públicas SSH del servidor (`ssh-hostkey`) y el título por default de la página en el puerto 80 (`http-title: Welcome to nginx on Debian!` — señal de que el sitio quedó sin configurar).

> ⚠️ Descargar e instalar scripts de terceros amplía la funcionalidad de Nmap, pero correr código de un autor en el que no confías es un riesgo real — revisar el script (`less archivo.nse`) antes de correrlo, igual que con cualquier código ajeno.

## Guardar resultados — formatos de salida
| Formato | Flag | Para qué sirve |
|---|---|---|
| Normal | `-oN archivo` | igual a lo que se ve en pantalla — el más legible para humanos |
| Grepable | `-oG archivo` | una línea larga y completa por host — pensado para `grep`, no para leer a simple vista |
| XML | `-oX archivo` | el más conveniente para procesar el resultado con otros programas/scripts |
| Los tres a la vez | `-oA archivo` | genera `.nmap` + `.gnmap` + `.xml` en un solo comando |
| Script Kiddie | `-oS archivo` | mismo contenido que el normal pero en texto "1337speak" — sin utilidad real, solo curiosidad |

```bash
nmap -oN scan.nmap 10.201.118.127
nmap -oA MACHINE_IP_scan 10.201.118.127   # genera scan.nmap, scan.gnmap, scan.xml
grep http MACHINE_IP_scan.gnmap           # el formato grepable SÍ trae la IP en cada línea — el normal no
```
La razón de que el grepable exista pese a ser feo de leer: cada línea es **autocontenida** (incluye IP, puertos, servicio, OS detectado). Buscar con `grep` en el output normal (`-oN`) da líneas sueltas como `80/tcp open http nginx 1.6.2` sin decir de qué host — inútil al comparar muchos targets a la vez. El grepable resuelve justo ese problema.

## `-A` — el atajo "todo junto"
```bash
sudo nmap -A 10.201.118.127
```
Equivale a `-sV -O -sC --traceroute` en un solo flag: versión de servicios, detección de OS, scripts default, y traceroute — el escaneo "completo" típico para un primer vistazo a fondo de un objetivo. Ya no es sigiloso (`-O` y `-sV` requieren interactuar de más con el objetivo), así que en un engagement donde el sigilo importa, conviene correr las piezas por separado y con más control.

## Referencia rápida
| Opción | Significado |
|---|---|
| `-sV` | detecta servicio/versión en puertos abiertos |
| `-sV --version-light` | probes más probables (intensidad 2) |
| `-sV --version-all` | todos los probes disponibles (intensidad 9) |
| `-O` | detecta el sistema operativo |
| `--traceroute` | traceroute hacia el objetivo |
| `--script=SCRIPTS` | scripts de Nmap a correr |
| `-sC` / `--script=default` | corre los scripts de la categoría default |
| `-A` | equivale a `-sV -O -sC --traceroute` |
| `-oN` | guarda en formato normal |
| `-oG` | guarda en formato grepable |
| `-oX` | guarda en formato XML |
| `-oA` | guarda en normal + XML + grepable a la vez |

Con esto se cierra la serie completa de Nmap: [[Reconocimiento-Activo]] (host discovery) → [[Nmap-Escaneo-Puertos]] (scans básicos) → [[Nmap-Escaneo-Avanzado-y-Evasion]] (scans avanzados/evasión) → esta nota (servicios, OS, NSE, reportes).
