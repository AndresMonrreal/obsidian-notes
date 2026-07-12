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
