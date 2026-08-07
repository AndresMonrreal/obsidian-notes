# Nmap — Escaneo de Puertos (TCP Connect / SYN / UDP)

Relacionado: [[Reconocimiento-Activo]] · [[TCP-UDP-Puertos]] · [[Redes-Fundamentos]]

Segunda sala de la serie de Nmap (después de host discovery, ver [[Reconocimiento-Activo]]). Ya con hosts vivos confirmados, el siguiente paso es ver **qué puertos/servicios** tienen abiertos.

## Puertos y servicios
Un puerto TCP/UDP identifica un servicio corriendo en un host, de la misma forma que una IP identifica al host entre otros. Un servidor HTTP se enlaza por default al puerto TCP 80 (443 si soporta TLS), pero el admin puede elegir otro puerto. **No puede haber más de un servicio escuchando en el mismo puerto/IP a la vez.**

## Los 6 estados de puerto según Nmap
Simplificando, un puerto está "abierto" o "cerrado" — pero en la práctica hay que contar con firewalls de por medio, así que Nmap maneja 6 estados:

| Estado | Significado |
|---|---|
| **open** | hay un servicio real escuchando y respondiendo |
| **closed** | el puerto es alcanzable, pero nada escucha ahí |
| **filtered** | Nmap no puede saber si está open o closed — algo (firewall) está bloqueando el paquete de ida o la respuesta |
| **unfiltered** | el puerto es alcanzable pero aun así no se puede determinar open/closed (típico de un ACK scan, `-sA`) |
| **open\|filtered** | Nmap no puede distinguir entre open y filtered |
| **closed\|filtered** | Nmap no puede distinguir entre closed y filtered |

Los tres últimos existen porque un firewall mete incertidumbre real — la evidencia no siempre alcanza para confirmar el estado exacto. **`open` es el estado que le importa a un pentester**: es la única superficie real donde se puede interactuar con algo (mandar peticiones, probar login, enumerar versión vulnerable). Los demás dicen "aquí no hay nada que atacar directo" o "todavía no se sabe" — útil para mapear, pero no un punto de entrada por sí solo.

## Banderas TCP relevantes
El header TCP (24 bytes, RFC 793) trae estas banderas que Nmap manipula para cada tipo de scan:

| Flag | Significado |
|---|---|
| **URG** | el urgent pointer es significativo — procesar el segmento de inmediato |
| **ACK** | el número de acknowledgement es significativo — confirma recepción |
| **PSH** | pide entregar los datos a la aplicación de inmediato |
| **RST** | resetea la conexión — la manda un firewall para cortarla, o un host cuando no hay servicio escuchando en ese puerto |
| **SYN** | inicia el 3-way handshake y sincroniza números de secuencia (primer paquete de toda conexión TCP) |
| **FIN** | el emisor ya no tiene más datos que mandar |

## TCP Connect Scan — `-sT`
Completa el 3-way handshake real (SYN → SYN/ACK → ACK) y luego cierra la conexión con RST/ACK en cuanto confirma el estado — no interesa mantenerla abierta, solo saber si el puerto responde. **Es la única opción posible sin privilegios** (root/sudo).
```bash
nmap -sT 10.66.171.192
```

## TCP SYN Scan — `-sS` (default con privilegios)
Modo default de Nmap cuando corres como root/sudo. Nunca completa el handshake: en cuanto llega el SYN/ACK, Nmap manda un RST en vez de un ACK — la conexión nunca se establece de verdad, por lo que es **menos probable que quede logueada** en el objetivo. Requiere privilegios porque arma el paquete SYN "crudo" (raw socket).
```bash
sudo nmap -sS 10.66.171.192
```

## UDP Scan — `-sU`
UDP es connectionless, no hay handshake. Un puerto UDP **abierto** normalmente no contesta nada — mandarle un paquete no dice nada por sí solo. La señal real es al revés: un puerto **cerrado** responde con **ICMP type 3, code 3** (destination unreachable / port unreachable). Se puede combinar con un scan TCP en el mismo comando.
```bash
sudo nmap -sU --top-ports 10 10.66.171.192
```
> Nota de la sala: DNS usa **UDP/53** para consultas normales (rápidas, sin necesitar confirmación de entrega) y TCP/53 solo para transferencias de zona o respuestas que exceden el tamaño de un paquete UDP — el mismo puerto detrás de `-R`/`--dns-servers` en [[Reconocimiento-Activo]]. SSH usa **TCP/22** porque necesita una conexión confiable y ordenada (comandos, transferencia de archivos, sesiones interactivas) — UDP no serviría ahí.

## Especificar puertos
- `-p22,80,443` → lista de puertos específicos.
- `-p1-1023` / `-p20-25` → rango.
- `-p-` → los 65535 puertos.
- `-F` → fast mode, top 100 puertos más comunes (en vez de los 1000 default).
- `--top-ports 10` → los N puertos más comunes según la base de datos interna de Nmap.
- `-r` → escanea en orden consecutivo en vez de aleatorio — útil para probar si un puerto abre de forma consistente (ej. mientras un objetivo está booteando).

## Control de velocidad y sigilo
- `-T<0-5>` → plantillas de timing: `-T0` paranoid (1 sonda cada 5 min, el más lento) hasta `-T5` insane (el más rápido, pero más propenso a perder paquetes/precisión por la velocidad). Default es `-T3` normal. `-T4` se usa mucho en CTFs/práctica; `-T1` en engagements reales donde el sigilo pesa más que la velocidad.
- `--min-rate <n>` / `--max-rate <n>` → controla paquetes por segundo directo (ej. `--max-rate 10` = no más de 10 pps).
- `--min-parallelism <n>` / `--max-parallelism <n>` → cuántas sondas de descubrimiento/puertos corren en paralelo (ej. `--min-parallelism 64` fuerza al menos 64 simultáneas).

## Referencia rápida
| Tipo de scan | Comando |
|---|---|
| TCP Connect | `nmap -sT 10.66.171.192` |
| TCP SYN | `sudo nmap -sS 10.66.171.192` |
| UDP | `sudo nmap -sU 10.66.171.192` |

| Opción | Propósito |
|---|---|
| `-p-` | todos los puertos |
| `-p1-1023` | rango de puertos |
| `-F` | 100 puertos más comunes |
| `-r` | orden consecutivo, no aleatorio |
| `-T<0-5>` | `-T0` el más lento, `-T5` el más rápido |
| `--max-rate 50` | tasa ≤ 50 paquetes/seg |
| `--min-rate 15` | tasa ≥ 15 paquetes/seg |
| `--min-parallelism 100` | al menos 100 sondas en paralelo |
