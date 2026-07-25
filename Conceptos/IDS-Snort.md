# IDS / IPS y Snort

Relacionado: [[Firewall-Fundamentos]] · [[Seguridad-de-Red]] · [[Defensiva-Blue-Team]] · [[Packets-and-Frames]] · [[Logs-y-SIEM]]

## Qué es un IDS
Un **Intrusion Detection System (IDS)** monitorea el tráfico de red **después** del firewall. Si un atacante pasa el firewall con una conexión de apariencia legítima y hace algo malicioso dentro, el IDS lo detecta. Analogía: el firewall es el guardia de la entrada; el IDS son las **cámaras de vigilancia** dentro del edificio.

> Importante: el IDS **solo detecta y alerta**, NO actúa/previene. El que previene es el **IPS** (Intrusion Prevention System).

## Categorización del IDS

### Por despliegue
| Tipo | Descripción |
|------|-------------|
| **HIDS** (Host IDS) | instalado en cada host; visibilidad detallada de ese host; pesado de gestionar en redes grandes |
| **NIDS** (Network IDS) | monitorea toda la red; vista centralizada; detecta amenazas sin importar el host |

### Por detección
| Modo | Descripción |
|------|-------------|
| **Signature-based** | compara con firmas (patrones) conocidas en su BD; rápido pero **no detecta zero-days** |
| **Anomaly-based** | aprende un **baseline** de comportamiento normal y detecta desviaciones; **sí detecta zero-days**, pero genera muchos falsos positivos (se reduce con tuning) |
| **Hybrid** | combina ambas; usa firmas para lo conocido y anomalías para lo nuevo |

## Snort
IDS open-source (1998), **signature-based y anomaly-based**. Las firmas están en archivos de reglas; trae muchas reglas pre-instaladas y permite crear/deshabilitar reglas propias.

### Modos de Snort
| Modo | Descripción | Caso de uso |
|------|-------------|-------------|
| **Packet Sniffer** | lee y muestra paquetes sin análisis | monitoreo/troubleshooting de red |
| **Packet Logging** | registra el tráfico en un archivo **PCAP** | forense: análisis posterior de causa raíz |
| **NIDS** | monitorea en tiempo real y aplica reglas → alertas | detección proactiva de amenazas (modo principal) |

## Directorio y archivos
Snort guarda sus archivos en `/etc/snort` (en Snort 3 la ruta se elige al instalar; una build desde código suele usar `/usr/local/etc/snort`). Se carga una config pasando su ruta con `-c`.
```bash
ls /etc/snort
# snort.lua (config principal), rules/ (reglas), *.map, *.config ...
```
Archivo de config principal: **snort.lua** (define reglas activas, rango de red `$HOME_NET`, etc.). Reglas propias: `rules/local.rules`.

## Formato de una regla
```
alert icmp any any -> $HOME_NET any (msg:"Ping Detected"; sid:10001; rev:1;)
```
| Componente | Ejemplo | Significado |
|------------|---------|-------------|
| **Action** | `alert` | qué hacer al coincidir |
| **Protocol** | `icmp` | protocolo (ICMP = ping) |
| **Source IP** | `any` | IP origen |
| **Source port** | `any` | puerto origen |
| `->` | | dirección del tráfico |
| **Destination IP** | `$HOME_NET` | variable del rango de red (en la config) |
| **Destination port** | `any` | puerto destino |
| **msg** | `"Ping Detected"` | mensaje de la alerta |
| **sid** | `10001` | Signature ID (identificador único de la regla) |
| **rev** | `1` | número de revisión (sube al modificar la regla) |

## Crear y probar una regla (práctica)
```bash
# 1. Editar el archivo de reglas custom
sudo nano /etc/snort/rules/local.rules
# añadir (sin borrar las existentes):
alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)

# 2. Ejecutar Snort en modo detección
sudo snort -q -l /var/log/snort -i lo -A alert_fast -c /etc/snort/snort.lua
#   -q  modo silencioso        -l  carpeta de logs
#   -i  interfaz (lo=loopback) -A  formato de alerta (alert_fast)
#   -c  archivo de configuración

# 3. Disparar la regla
ping 127.0.0.1
```
Salida esperada:
```
[**] [1:1000001:1] "Loopback Ping Detected" [**] [Priority: 0] {ICMP} 127.0.0.1 -> 127.0.0.1
```
> Si tu interfaz loopback no se llama `lo`, usa el nombre correcto.

## Snort sobre archivos PCAP (forense)
```bash
sudo snort -q -l /var/log/snort -r Task.pcap -A alert_fast -c /etc/snort/snort.lua
#   -r  lee tráfico histórico desde un PCAP en vez de una interfaz en vivo
```
Útil cuando ya tienes tráfico capturado y buscas señales de intrusión.
