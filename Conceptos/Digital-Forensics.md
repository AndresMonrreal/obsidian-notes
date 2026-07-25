# Digital Forensics (DFIR)

Relacionado: [[Defensiva-Blue-Team]] · [[Incident-Response]] · [[SOC-Centro-de-Operaciones]] · [[Sistemas-Operativos]] · [[Linux-Herramientas-y-Admin]]

## Qué es
La forense digital aplica métodos y procedimientos para investigar cibercrímenes: encontrar y analizar evidencia en dispositivos digitales para acciones legales.

## Las 4 fases (NIST)
`Collection → Examination → Analysis → Reporting`

1. **Collection**: identificar y recolectar datos de todos los dispositivos (PCs, laptops, USBs, cámaras...). No alterar la data original; documentar cada ítem.
2. **Examination**: filtrar el volumen enorme de datos y extraer solo lo de interés (ej. media de una fecha concreta, o de un usuario específico).
3. **Analysis**: correlacionar múltiples evidencias para sacar conclusiones; reconstruir las actividades en orden **cronológico**.
4. **Reporting**: informe con metodología, hallazgos y recomendaciones; incluir **resumen ejecutivo** para audiencias no técnicas.

## Tipos de forense digital
| Tipo | Investiga |
|------|-----------|
| **Computer** | computadoras (el más común) |
| **Mobile** | móviles (llamadas, SMS, GPS) |
| **Network** | toda la red (logs de tráfico) |
| **Database** | intrusiones/modificación/exfiltración en BD |
| **Cloud** | datos en infraestructura cloud (poca evidencia, complejo) |
| **Email** | phishing / fraude en correos |

## Adquisición de evidencia (buenas prácticas)
- **Proper Authorization**: obtener autorización de la autoridad correspondiente ANTES de recolectar. Sin ella, la evidencia puede ser **inadmisible** en corte.
- **Chain of Custody** (cadena de custodia): documento formal con todos los detalles de la evidencia:
  - Descripción (nombre, tipo).
  - Quién la recolectó.
  - Fecha y hora de recolección.
  - Ubicación de almacenamiento.
  - Registro de accesos (quién y cuándo). → prueba integridad y confiabilidad.
- **Write Blockers** (bloqueadores de escritura): impiden que el workstation forense altere timestamps u otros datos del disco del sospechoso → mantiene el estado original. **Garantiza la integridad durante la colección.**

## Forense en Windows — imágenes forenses
Copias **bit a bit**. Dos categorías:

| Imagen | Contenido | Volatilidad |
|--------|-----------|-------------|
| **Disk image** | todo el almacenamiento (HDD/SSD): archivos, media, historial | **no volátil** (sobrevive reinicio) |
| **Memory image** | RAM: procesos, archivos abiertos, conexiones de red | **volátil** → capturar PRIMERO (se pierde al apagar) |

### Herramientas
| Herramienta | Uso |
|-------------|-----|
| **FTK Imager** | GUI; toma y analiza **disk images** |
| **Autopsy** | open-source; analiza disk images (keyword search, recuperación de archivos borrados, metadata, extension mismatch) |
| **DumpIt** | CLI; toma **memory image** de Windows |
| **Volatility** | open-source; analiza memory images con plugins (Windows, Linux, macOS, Android) |

## Metadata — comandos prácticos

### Metadata de PDF con `pdfinfo`
Muestra título, autor, creador, fechas, etc. (paquete `poppler-utils`).
```bash
# Instalar en Kali si falta:
sudo apt install poppler-utils

pdfinfo ransom-letter.pdf
# Creator:      Microsoft® Word for Office 365
# Author:       Ann Gree Shepherd
# CreationDate: Wed Oct 10 21:47:53 2018 EEST
# Pages:        20 ...
```
> Exportar a PDF conserva la mayoría de la metadata del documento original.

### EXIF de imágenes con `exiftool`
EXIF = Exchangeable Image File Format. Guarda: modelo de cámara/móvil, fecha/hora, ajustes (focal, apertura, ISO) y, si el móvil tiene GPS, **coordenadas**.
```bash
# Instalar en Kali si falta:
sudo apt install libimage-exiftool-perl

exiftool IMAGE.jpg
# GPS Position : 51 deg 31' 4.00" N, 0 deg 5' 48.30" W
```
Para geolocalizar: pasa las coordenadas a un mapa reemplazando `deg` por `°` y quitando espacios extra → `51°31'4.0"N 0°05'48.3"W`. Google/Bing Maps revela la calle donde se tomó la foto.

> Caso "Gado kidnapping": con `pdfinfo` se saca el autor de la carta de rescate y con `exiftool` la calle donde tomaron la foto → así se identifica la ubicación del secuestrador.
