# Ataques de Sniffing y Man-in-the-Middle (MITM)

Relacionado: [[Protocolos-Aplicacion]] · [[TLS-SSL-Fundamentos]] · [[Principios-de-Seguridad-CIA-DAD]] · [[Reconocimiento-Activo]] · [[Herramientas-Hydra]]

## CIA vs DAD
Lo que se protege (**CIA** — Confidentiality, Integrity, Availability, ver [[CIA-Triad]]) tiene su contraparte de lo que un atacante busca causar (**DAD** — Disclosure, Alteration, Destruction, ver [[Principios-de-Seguridad-CIA-DAD]]):

| Ataque | Rompe | Causa |
|--------|-------|-------|
| **Sniffing** (captura de paquetes) | Confidentiality | Disclosure |
| **MITM** | Integrity (y a veces Confidentiality) | Alteration |
| **Password attack** | Confidentiality/Authentication | Disclosure |

## Sniffing Attack (captura de paquetes)

### Qué es
Capturar tráfico de red con una herramienta de packet capture. Si el protocolo va en **texto plano**, cualquiera con acceso a ese tráfico puede leer credenciales y contenido de los mensajes.

### Dónde sigue siendo relevante
Redes corporativas internas sin cifrado entre sistemas · sistemas legacy con protocolos en texto plano (servidores de correo viejos, dispositivos embebidos, ICS) · servicios mal configurados donde TLS está disponible pero no forzado · dispositivos IoT · redes inalámbricas al alcance del atacante · tras un MITM exitoso que degrada/quita el cifrado.

### Herramientas
| Herramienta | Tipo |
|-------------|------|
| **tcpdump** | CLI, ligero, viene por defecto en la mayoría de Linux |
| **Wireshark** | GUI, filtrado potente, disección de protocolos |
| **tshark** | versión CLI de Wireshark, útil para scripting |
| tcpflow, ngrep, NetworkMiner | herramientas especializadas complementarias |

### Requisito para el ataque
Acceso al tráfico de red: wiretap, switch con port mirroring, **ARP spoofing** en red local, sistema comprometido en el mismo segmento, o resultado de un MITM exitoso.

### Ejemplo práctico: capturar credenciales POP3
```bash
sudo tcpdump port 110 -A
```
- `sudo` → captura de paquetes requiere privilegios root
- `port 110` → filtra solo tráfico del servidor POP3 (puerto 110 por defecto)
- `-A` → muestra el contenido de los paquetes en **ASCII** (revela credenciales en claro)

En el output se ven en paquetes separados: `USER frank` y `PASS D2xc9CgD`. En Wireshark, el filtro de display equivalente es simplemente `pop`.

### Filtros útiles de tcpdump
```bash
sudo tcpdump port 23 -A            # capturar Telnet
sudo tcpdump host 10.20.30.148 -A  # tráfico de/hacia un host específico
sudo tcpdump port 80 -A            # HTTP (puede revelar credenciales en POST)
sudo tcpdump port 21 -A            # FTP (credenciales en claro)
sudo tcpdump -w capture.pcap       # guardar la captura a un archivo
tcpdump -r capture.pcap -A         # leer y analizar un archivo .pcap
```

### Mitigación
Cualquier protocolo en texto plano es vulnerable — el único requisito del atacante es estar en el camino del tráfico. Mitigación principal: **añadir una capa de cifrado (TLS)** — ver [[TLS-SSL-Fundamentos]].

Adicionales: **network segmentation** (aísla qué sistemas ven el tráfico de cuáles) · VLANs y túneles cifrados · **802.1X** (autenticación a nivel de puerto antes de dar acceso a la red) · **Zero Trust** (cifra todo el tráfico, incluso interno — ver [[Principios-de-Seguridad-CIA-DAD]]) · monitoreo de ARP spoofing.

---

## Man-in-the-Middle (MITM)

### Qué es
El atacante (E) se posiciona entre dos partes (A y B) que creen estar comunicándose directamente. E puede leer **y alterar** los mensajes en tránsito sin que ninguna de las partes lo note, si no verifican autenticidad/integridad de cada mensaje.

### Técnicas para posicionarse en el medio
| Técnica | Cómo funciona |
|---------|---------------|
| **ARP Spoofing** | mensajes ARP falsificados asocian la MAC del atacante con la IP del gateway/objetivo → el tráfico se redirige al atacante (redes locales) |
| **DNS Spoofing** | respuestas DNS falsas redirigen a servidores del atacante (DNS comprometido, cache poisoning, o responder más rápido que el servidor legítimo) |
| **Rogue Access Points** | punto de acceso Wi-Fi falso con nombre convincente (ej. `Airport_WiFi_Free`) — todo el tráfico de quien se conecta pasa por el atacante |
| **BGP Hijacking** | a nivel de enrutamiento de internet; el atacante anuncia rutas BGP falsas — ataque sofisticado, suele apuntar a orgs/regiones específicas |

### Herramientas
| Herramienta | Uso |
|-------------|-----|
| **Bettercap** | sucesor moderno de Ettercap; ARP spoofing, DNS spoofing, proxy HTTP/HTTPS, arquitectura modular |
| **Ettercap** | clásico para MITM en LAN; ofrece varias interfaces (texto, ncurses, GTK); menos preferido hoy que Bettercap |
| **mitmproxy** | proxy HTTPS interactivo — inspeccionar y modificar tráfico |
| **Responder** | entorno Windows; explota LLMNR/NBT-NS (protocolos de fallback cuando falla la resolución DNS estándar) para capturar credenciales — técnica común en pentests internos de Active Directory |

### MITM contra tráfico cifrado
- **SSL Stripping**: degrada HTTPS a HTTP. El atacante intercepta la petición, mantiene HTTPS con el servidor real, pero sirve el contenido a la víctima por HTTP sin cifrar. La víctima puede no notar la falta del candado.
- **Fake Certificates**: el atacante presenta su propio certificado y establece dos conexiones cifradas separadas (una con cada parte). Funciona si la víctima acepta una advertencia de certificado inválido, o si la CA está comprometida.
- **Compromised/Rogue CAs**: si el atacante controla (o engaña a) una CA confiable, puede generar certificados válidos para cualquier dominio.

### Defensas modernas
| Defensa | Qué previene |
|---------|---------------|
| **HTTPS Everywhere** | la mayoría de sitios usan HTTPS por defecto |
| **HSTS** (HTTP Strict Transport Security) | el navegador se niega a conectar por HTTP tras haber visto el header una vez → previene SSL stripping. Hay listas de **preload** con sitios que nunca deben cargar sin HTTPS |
| **Certificate Transparency (CT)** | CAs deben loguear todo certificado emitido en logs públicos auditables → dificulta certificados fraudulentos sin ser detectados |
| **Certificate Pinning** | la app especifica exactamente qué certificados/claves públicas son válidos — protege incluso si una CA se compromete (común en apps móviles) |
| **DANE** | usa DNSSEC para publicar info de certificados en registros DNS, como ruta de confianza alternativa a las CAs |

A pesar de esto, MITM sigue siendo posible si el usuario ignora advertencias de certificado, la app no valida certificados correctamente, se usan protocolos en claro, se compromete una CA confiable, o hay redes internas sin cifrado.
