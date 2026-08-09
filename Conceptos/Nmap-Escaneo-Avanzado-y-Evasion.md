# Nmap — Escaneo Avanzado y Evasión de Firewalls/IDS

Relacionado: [[Nmap-Escaneo-Puertos]] · [[Reconocimiento-Activo]] · [[Firewall-Fundamentos]] · [[IDS-Snort]] · [[TCP-Escaneo-Riesgos-y-Defensa]]

Tercera sala de la serie de Nmap. Con los flags TCP y el 3-way handshake ya entendidos (ver [[Nmap-Escaneo-Puertos]]), esta sala explora qué pasa al mandar un paquete TCP que **no pertenece a ninguna conexión real**, con combinaciones de banderas "fuera de protocolo" — y cómo usar eso tanto para escanear con sigilo como para mapear las reglas de un firewall.

## Escaneos basados en ausencia de respuesta (Null / FIN / Xmas)
Comparten la misma lógica: un puerto **abierto** simplemente ignora el paquete (silencio → Nmap reporta `open|filtered`, no puede tener certeza); un puerto **cerrado** sí responde con RST (comportamiento estándar del kernel ante un paquete que no puede procesar).

| Scan | Opción | Banderas activas | Analogía |
|---|---|---|---|
| Null | `-sN` | 0 (ninguna) | mandar un sobre vacío |
| FIN | `-sF` | 1 (FIN) | avisar que cierras algo que nunca abriste |
| Xmas | `-sX` | 3 (FIN+PSH+URG) | árbol de navidad — todo iluminado a la vez |

```bash
sudo nmap -sN 10.67.155.119
sudo nmap -sF 10.67.155.119
sudo nmap -sX 10.67.155.119
```
`sudo` es obligatorio en los tres: arman paquetes TCP "a mano" con banderas no estándar (raw sockets), algo que la API de sockets normal del SO no permite sin privilegios.

**Útiles contra firewalls stateless** — un firewall stateless solo revisa si el paquete trae el flag SYN para detectar "intento de conexión"; como estos tres nunca lo llevan, se cuelan sin activar esa regla. Un firewall **stateful**, en cambio, bloquea prácticamente cualquier combinación fuera de lo normal, dejando estas técnicas inútiles contra él.

## TCP Maimon Scan — `-sM`
Descrito por Uriel Maimon en 1996. Activa **2 banderas** (FIN + ACK) — combinación igual de "ilegal" que las anteriores (ACK sin conexión que reconocer, FIN sin conexión que cerrar).

```bash
sudo nmap -sM 10.67.155.119
```
Explota un comportamiento específico de sistemas viejos derivados de BSD, que **descartaban el paquete silenciosamente si el puerto estaba abierto** (revelando los abiertos por ausencia de respuesta). Los sistemas modernos responden RST sin importar el estado del puerto, así que hoy este scan casi nunca aporta información real — se estudia por valor histórico/conceptual, no como herramienta práctica actual.

## TCP ACK Scan — `-sA` (mapea firewalls, no puertos)
Manda solo el flag ACK. El objetivo **siempre** responde RST, sin importar si el puerto está abierto o cerrado — un ACK "de la nada" rompe el protocolo y el kernel solo sabe reaccionar con RST. Por eso este scan **no sirve para saber si un puerto está abierto**; sirve para algo distinto: **mapear las reglas de un firewall**.

```bash
sudo nmap -sA 10.67.155.119
```
`unfiltered` → el firewall **deja pasar** el paquete hacia ese puerto (no implica que haya un servicio ahí). `filtered` → el firewall lo está bloqueando. Comparar resultados con/sin firewall activo confirma exactamente qué reglas existen.

## TCP Window Scan — `-sW` (como ACK, pero más informativo)
Casi idéntico al ACK scan (mismo flag, mismo RST universal de respuesta), pero examina el **campo TCP Window** del RST recibido — en ciertos sistemas ese detalle sí distingue entre puerto abierto y cerrado, algo que el ACK scan puro no puede ver.

```bash
sudo nmap -sW 10.67.155.119
```
Contra un firewall activo, donde el ACK scan solo dice "unfiltered" para varios puertos, el Window scan puede refinar esa misma lista en `open`/`closed` — más información sobre el mismo conjunto de puertos que el firewall deja pasar.

## Custom Scan — `--scanflags`
Para combinaciones de banderas fuera de los tipos predefinidos:
```bash
sudo nmap --scanflags RSTSYNFIN 10.67.155.119
sudo nmap --scanflags URGACKPSHRSTSYNFIN 10.67.155.119   # todas las banderas activas
```
Útil para experimentar, pero exige entender de antemano cómo va a reaccionar cada estado de puerto ante la combinación elegida — sin esa base, el resultado es ruido, no información.

## Spoofing y evasión

### IP spoofeada — `-S`
```bash
nmap -e NET_INTERFACE -Pn -S SPOOFED_IP 10.67.155.119
```
`-S SPOOFED_IP` fuerza esa IP como origen de todos los paquetes · `-e` especifica la interfaz de red a usar (Nmap ya no puede autodetectarla a partir de una IP de origen falsa) · `-Pn` salta el descubrimiento de host (ver [[Nmap-Escaneo-Puertos]]) porque no se espera ver directamente la respuesta del ping. **Solo tiene sentido si puedes monitorear el tráfico de red para capturar las respuestas** — si el objetivo responde a `SPOOFED_IP` y esa IP no es tuya ni la puedes ver, el scan queda ciego.

### MAC spoofeada — `--spoof-mac`
```bash
nmap --spoof-mac SPOOFED_MAC 10.67.155.119
```
Solo funciona si atacante y objetivo están en el **mismo segmento Ethernet/WiFi** (una MAC no se enruta más allá de la LAN local).

### Puerto de origen forzado — `--source-port`
```bash
nmap --source-port 53 10.67.155.119
```
Fuerza el puerto de origen de los paquetes del scan. Técnica histórica de evasión: algunos firewalls viejos confían ciegamente en tráfico que dice venir del puerto 53 (DNS) o 20 (FTP-data), asumiendo que es tráfico legítimo de esos servicios.

### Decoy scan — `-D`
```bash
nmap -D 10.10.0.1,10.10.0.2,ME 10.67.155.119
nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME 10.67.155.119
```
Hace que el scan parezca originarse desde varias IPs simultáneas (decoys), escondiendo la real entre ellas. `ME` marca en qué posición del orden aparece tu IP real · `RND` genera una IP decoy aleatoria distinta en cada ejecución. No oculta el origen por completo (un analista cuidadoso puede correlacionar), pero dificulta señalar con certeza cuál IP es la real.

### Fragmentación de paquetes — `-f` / `-ff`
```bash
sudo nmap -sS -p80 -f 10.67.155.119     # fragmentos de 8 bytes
sudo nmap -sS -p80 -ff 10.67.155.119    # fragmentos de 16 bytes
nmap --mtu 32 10.67.155.119             # tamaño de fragmento personalizado (múltiplo de 8)
nmap --data-length 200 10.67.155.119    # agrega bytes extra para que el paquete parezca más "normal"
```
`-f` divide los 24 bytes del header TCP en fragmentos IP de 8 bytes o menos (repetir → `-ff`/`-f -f` da fragmentos de 16). Algunos firewalls/IDS simples solo inspeccionan el primer fragmento o no reensamblan correctamente antes de aplicar sus reglas — fragmentar puede colar tráfico que de otra forma sería bloqueado. `--data-length` es la táctica opuesta: en vez de esconder el paquete, hacerlo parecer más grande/normal (los paquetes de scan suelen ser muy pequeños, un patrón reconocible por sí solo).

### Idle / Zombie Scan — `-sI`
```bash
nmap -sI ZOMBIE_IP 10.67.155.119
```
La técnica más elaborada: usa un host **inactivo** (zombie) de la red para que el scan parezca venir de él, sin revelar la IP propia en ningún momento.

Mecanismo en 3 pasos, basado en el campo **IP ID** (se incrementa en 1 con cada paquete que el zombie envía):
1. Provocar al zombie (ej. SYN/ACK) y anotar su IP ID actual.
2. Mandar un SYN **spoofeado como si viniera del zombie** hacia el puerto del objetivo real.
3. Provocar al zombie de nuevo y comparar el nuevo IP ID:
   - Diferencia de **+1** → el objetivo nunca le respondió al zombie (puerto cerrado o filtrado).
   - Diferencia de **+2** → el objetivo mandó SYN/ACK al zombie y este respondió con RST sin querer (puerto **abierto**).

Requiere que el zombie esté realmente inactivo (poco tráfico propio) — si está ocupado, sus IP ID cambian por su propio tráfico y el resultado queda inservible.

## Verbosidad y diagnóstico
```bash
nmap --reason -sS 10.67.155.119    # por qué Nmap concluyó cada estado (ej. "syn-ack" para open)
nmap -v 10.67.155.119              # verbose — progreso en tiempo real
nmap -vv 10.67.155.119             # muy verbose
nmap -d 10.67.155.119              # debugging del propio Nmap
nmap -dd 10.67.155.119             # debugging aún más detallado
```
`--reason` es el más útil para aprender/auditar: agrega la columna `REASON`, mostrando exactamente qué paquete de respuesta llevó a cada veredicto (`syn-ack` para abiertos en un SYN scan, `reset` para cerrados) — convierte a Nmap de "caja negra" a explicable.

## Referencia rápida
| Scan | Comando |
|---|---|
| Null | `sudo nmap -sN 10.67.155.119` |
| FIN | `sudo nmap -sF 10.67.155.119` |
| Xmas | `sudo nmap -sX 10.67.155.119` |
| Maimon | `sudo nmap -sM 10.67.155.119` |
| ACK | `sudo nmap -sA 10.67.155.119` |
| Window | `sudo nmap -sW 10.67.155.119` |
| Custom | `sudo nmap --scanflags URGACKPSHRSTSYNFIN 10.67.155.119` |
| Spoofed IP | `sudo nmap -S SPOOFED_IP 10.67.155.119` |
| Spoofed MAC | `--spoof-mac SPOOFED_MAC` |
| Decoy | `nmap -D DECOY_IP,ME 10.67.155.119` |
| Idle/Zombie | `sudo nmap -sI ZOMBIE_IP 10.67.155.119` |
| Fragmentar en 8 bytes | `-f` |
| Fragmentar en 16 bytes | `-ff` |

| Opción | Propósito |
|---|---|
| `--source-port PORT_NUM` | fuerza el puerto de origen |
| `--data-length NUM` | agrega bytes extra para disimular el paquete |
| `--reason` | explica cómo Nmap llegó a esa conclusión |
| `-v` / `-vv` | verbose / muy verbose |
| `-d` / `-dd` | debugging / debugging detallado |

## Nota: OS fingerprinting con Nmap — `-O`
Los escaneos de esta sala revelan, de paso, algo más que estados de puerto: distintos sistemas operativos **reaccionan distinto** ante paquetes "imposibles" (por eso el Maimon scan solo funcionaba en ciertos BSD viejos). Nmap explota exactamente ese principio con:
```bash
sudo nmap -O 10.67.155.119
```
Compara TTL inicial, tamaño de ventana TCP, orden/valores de las opciones TCP, y el comportamiento ante paquetes fuera de protocolo, contra una base de datos de miles de huellas conocidas, para adivinar el sistema operativo del objetivo. Requiere privilegios y, idealmente, al menos un puerto abierto y uno cerrado detectados de antemano para tener suficiente información que comparar.
