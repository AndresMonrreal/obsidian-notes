# Hydra — Fuerza bruta de contraseñas

Relacionado: [[Ofensiva-Pentesting]] · [[Criptografia]] · [[Seguridad-de-Red]]

## Qué es
Hydra es una herramienta de **fuerza bruta online** para descubrir contraseñas de servicios de autenticación. Recorre una lista de contraseñas (wordlist) y prueba cada una contra el servicio hasta dar con la correcta. Sirve para acelerar lo que sería adivinar manualmente.

Refuerza la importancia de usar contraseñas fuertes: si la contraseña es común, corta (< 8 caracteres) o sin caracteres especiales, es fácil de adivinar. Cámbiale siempre las credenciales por defecto a cámaras CCTV, routers, etc. (`admin:password` es el clásico).

## Protocolos soportados (selección)
FTP, SSH, SMB, SMTP, POP3, IMAP, RDP, VNC, MySQL, MS-SQL, MongoDB, LDAP, SNMP, Telnet, HTTP-GET/POST, HTTPS-FORM-GET/POST, y muchos más.

## Opciones clave
| Opción | Descripción |
|--------|-------------|
| `-l` | usuario único para login |
| `-L` | lista de usuarios (archivo) |
| `-p` | contraseña única |
| `-P` | lista de contraseñas (wordlist) |
| `-t` | número de hilos en paralelo |
| `-s` | puerto no estándar |
| `-V` | verbose: muestra cada intento |
| `-f` | detenerse al encontrar la primera credencial válida |

## Ejemplos de comandos

### FTP
```bash
hydra -l user -P passlist.txt ftp://MACHINE_IP
```

### SSH
```bash
hydra -l root -P passwords.txt MACHINE_IP -t 4 ssh
```
- `-l root` → usuario para SSH
- `-P passwords.txt` → wordlist
- `-t 4` → cuatro hilos en paralelo

### Formulario web POST
```bash
hydra -l <usuario> -P <wordlist> MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```
Estructura del string `http-post-form`: `"<ruta>:<credenciales>:<respuesta_fallo>"`
- `/` → la página de login (aquí la raíz)
- `username=^USER^&password=^PASS^` → campos del formulario; Hydra reemplaza `^USER^` y `^PASS^`
- `F=incorrect` → texto que aparece en la respuesta cuando el login falla

### Formulario en puerto no estándar
```bash
hydra -l <usuario> -P <wordlist> MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -s <puerto> -V
```

## Cómo averiguar el tipo de request
Usa la pestaña **Network** de las DevTools del navegador (o mira el código fuente) para ver si el formulario usa **GET** o **POST** y cuáles son los nombres exactos de los campos.

## Instalación
- Kali: ya viene preinstalado.
- Ubuntu/Debian: `apt install hydra`
- Fedora: `dnf install hydra`

## Flujo de estudio recomendado
1. Enumerar el servicio y su puerto (ver [[TCP-UDP-Puertos]]).
2. Identificar campos del formulario o el protocolo.
3. Elegir wordlist (ej. `rockyou.txt`).
4. Lanzar Hydra con `-V` para ver progreso.

---

## Autenticación: los 3 factores
Autenticación = probar quién dices ser. Se logra con uno o más de:
- **Algo que sabes** — contraseña/PIN (lo que ataca Hydra).
- **Algo que tienes** — teléfono, llave de seguridad, smart card.
- **Algo que eres** — huella, reconocimiento facial.

## Tipos de ataque de contraseñas
| Tipo | Cómo funciona |
|------|-----------------|
| **Password Guessing** | requiere conocer al objetivo (mascota, año de nacimiento, equipo favorito — fácil de sacar de redes sociales) |
| **Dictionary Attack** | prueba palabras comunes de una wordlist — efectivo porque muchos usan palabras reales |
| **Brute Force** | prueba todas las combinaciones posibles — exhaustivo, efectivo contra contraseñas cortas (el espacio de búsqueda crece exponencialmente con la longitud) |
| **Credential Stuffing** | usa pares usuario/contraseña filtrados de brechas previas contra **otros** servicios — explota la reutilización de contraseñas |
| **Password Spraying** | prueba **pocas** contraseñas comunes contra **muchas** cuentas (al revés del brute force) — evade bloqueos por intentos fallidos |
| **Hybrid Attack** | combina palabras de diccionario con patrones comunes (ej. `Summer2024`, `P@ssw0rd`) |

> Contraseñas comunes que se siguen viendo en brechas recientes: `123456`, `password`/`Password1`, `qwerty`, nombre de la empresa + año (`Welcome1`, `Summer2024`).

## Wordlists
| Wordlist | Dónde |
|----------|-------|
| **rockyou.txt** | `/usr/share/wordlists/rockyou.txt` (AttackBox/Kali) — el clásico, ~14M contraseñas filtradas |
| **SecLists** | `/usr/share/seclists/` — colección de múltiples wordlists para distintos propósitos |
| **CrackStation** | wordlists optimizadas para crackeo de hashes |

> La wordlist ideal depende del objetivo: usuarios franceses → wordlist en francés; empleados de una empresa → nombre de la empresa + años/estaciones. Una wordlist custom suele superar a una genérica.

## Otras herramientas de ataque de contraseñas
| Herramienta | Para qué |
|-------------|----------|
| **Medusa** | similar a Hydra, diseño modular, más estable en algunos protocolos |
| **Ncrack** | del proyecto Nmap, testing de autenticación paralelo de alta velocidad |
| **CrackMapExec / NetExec** | especializado en Windows/Active Directory — spray de contraseñas sobre SMB, WinRM, LDAP |
| **Burp Suite Intruder** | útil contra formularios web de login que Hydra no maneja bien |
| **Hashcat / John the Ripper** | crackeo de hashes **offline** (no ataca un servicio en vivo) — ver [[Herramientas-John-the-Ripper]] |

## Mitigación de ataques de contraseñas
- **Password policy moderna** (NIST SP 800-63B): priorizar **longitud** sobre reglas de complejidad, bloquear contraseñas ya filtradas en brechas, no forzar cambios periódicos sin evidencia de compromiso.
- **Account lockout** — bloquea tras N intentos fallidos; efectivo contra brute force, pero se puede evadir con password spraying.
- **Rate limiting / throttling** — delay entre intentos; molesto para bots, tolerable para usuarios reales.
- **CAPTCHA** — idealmente con análisis de comportamiento, no solo reconocimiento de imagen.
- **MFA (Multi-Factor Authentication)** — una de las defensas más efectivas.
- **Passwordless**: **passkeys** (FIDO2/WebAuthn), magic links por email, llaves de seguridad físicas (YubiKey).
- **Breached password detection** — comparar contra bases de datos de brechas conocidas (ej. API de "Have I Been Pwned") al registrar/cambiar contraseña.
- **Behavioural analysis** — detectar logins desde ubicaciones inusuales o "viajes imposibles".

## Notas relacionadas
- [[Ataques-Sniffing-MITM]] — captura de credenciales cuando el protocolo va en texto plano
- [[Protocolos-Aplicacion]] — servicios que Hydra puede atacar (FTP, SSH, SMTP, POP3, IMAP)
- [[Herramientas-John-the-Ripper]] — crackeo de hashes offline (complementario a Hydra)
