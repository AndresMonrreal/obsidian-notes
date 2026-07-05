---
tags:
  - linux
  - pre-security
  - permisos
  - chmod
  - usuarios
  - filesystem
  - directorios
  - su
fecha: 2026-06-29
ruta: SEC0
fuente: TryHackMe — Linux Fundamentals
---

# 🐧 Linux — Permisos, Usuarios y Sistema de Archivos

---

## Permisos de Archivos en Linux

### Leer la salida de `ls -l`

```
-rw-r--r-- 1 cmnatic cmnatic 0 Feb 19 10:37 file1
│└─┬──┘└─┬──┘└─┬──┘
│  │Owner │Group │Others
│  │rw-   │r--   │r--
│
└── Tipo: - = archivo, d = directorio, l = enlace simbólico
```

La primera columna tiene **10 caracteres**:
- Carácter 1: tipo de archivo
- Caracteres 2-4: permisos del **propietario (owner)**
- Caracteres 5-7: permisos del **grupo (group)**
- Caracteres 8-10: permisos de **otros (others)**

---

### Los 3 tipos de permiso

| Símbolo | Nombre | Significado |
|---------|--------|-------------|
| `r` | Read | Leer el contenido del archivo / listar directorio |
| `w` | Write | Escribir o modificar el archivo / crear archivos en directorio |
| `x` | Execute | Ejecutar el archivo como programa / entrar al directorio |
| `-` | Sin permiso | Permiso no concedido |

---

### Permisos en Formato Numérico (chmod)

Cada permiso tiene un valor numérico:

| Permiso | Valor |
|---------|-------|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |
| Sin permiso (-) | 0 |

El valor de cada grupo = suma de sus permisos activos.

**Ejemplo: `rwxrwxrwx`**

| Grupo | Permisos | Cálculo | Valor |
|-------|----------|---------|-------|
| Owner | rwx | 4+2+1 | **7** |
| Group | rwx | 4+2+1 | **7** |
| Others | rwx | 4+2+1 | **7** |

→ `rwxrwxrwx = 777`

### Tabla de referencias rápidas

| Simbólico | Numérico | Significado |
|-----------|----------|-------------|
| `rwxrwxrwx` | `777` | Todos tienen acceso completo |
| `rwxr-xr-x` | `755` | Owner: todo · Grupo/Otros: leer y ejecutar |
| `rw-r--r--` | `644` | Owner: leer/escribir · Otros: solo leer |
| `rwx------` | `700` | Solo el owner tiene acceso |
| `rwxr-x---` | `750` | Owner: todo · Grupo: leer/ejecutar · Otros: nada |
| `rw-------` | `600` | Solo el owner puede leer/escribir |

### Usar `chmod`

```bash
# Cambiar permisos con notación numérica
chmod 755 script.sh

# Cambiar permisos con notación simbólica
chmod u+x script.sh       # añadir execute al owner
chmod o-r archivo.txt     # quitar read a others
chmod g=rw datos.txt      # asignar exactamente rw al grupo
```

> [!tip] Para archivos sensibles usa 600 o 640
> Archivos con contraseñas, claves SSH, configs privadas → `chmod 600 archivo` (solo el owner puede leer/escribir, nadie más puede ver nada).

> [!warning] El permiso 777 es un riesgo de seguridad
> `chmod 777` en archivos del sistema o archivos de configuración permite que cualquier usuario del sistema los modifique. Es una vulnerabilidad común de escalada de privilegios.

---

## Usuarios y Grupos en Linux

### El modelo de propietario

Cada archivo pertenece a:
1. Un **usuario (owner)** — el que lo creó o al que se le asignó
2. Un **grupo** — un conjunto de usuarios con permisos compartidos

```
-rw-r--r-- 1 cmnatic cmnatic 0 Feb 19 file1
             │       │
             │       └── Grupo: "cmnatic"
             └────────── Owner: "cmnatic"
```

### Caso de uso real: servidor web

```
El sistema del webserver (www-data) necesita:
  → leer y escribir archivos del sitio web

Los clientes de hosting quieren:
  → subir sus propios archivos

Solución con permisos de grupo:
  → www-data es el owner
  → Los clientes están en un grupo con permisos específicos
  → Sin que ningún cliente pueda tocar los archivos de otro
```

---

## El comando `su` — Cambiar de Usuario

### Uso básico

```bash
su user2            # Cambiar a user2 (pide contraseña)
su -l user2         # Cambiar a user2 con su entorno completo
su - user2          # Equivalente a -l
sudo su             # Cambiar a root (requiere sudo)
```

### `su` vs `su -l`

| | `su user2` | `su -l user2` (o `su - user2`) |
|-|-----------|-------------------------------|
| **Directorio actual** | Mantiene el directorio anterior | Va al home del usuario (`~`) |
| **Variables de entorno** | Hereda las del usuario original | Hereda las del nuevo usuario |
| **Simula** | Solo cambio de identidad | Login completo como ese usuario |

**Ejemplo:**

```bash
# Sin -l: quedamos en el directorio anterior
tryhackme@linux2:~$ su user2
user2@linux2:/home/tryhackme$    # ← directorio del usuario anterior

# Con -l: vamos al home del nuevo usuario
tryhackme@linux2:~$ su -l user2
user2@linux2:~$ pwd
/home/user2                       # ← home del usuario correcto
```

> [!info] ¿Cuándo usar `su -l`?
> Usa `-l` cuando necesites trabajar como ese usuario de verdad (configurar su entorno, ejecutar comandos con sus variables de entorno). Sin `-l` puede causar problemas si el programa busca archivos en el home del usuario.

---

## Directorios Raíz Importantes del Sistema Linux

```
/
├── etc/     → Configuración del sistema
├── var/     → Datos variables (logs, bases de datos)
├── root/    → Home del superusuario root
├── tmp/     → Archivos temporales (se borran al reiniciar)
├── home/    → Homes de usuarios normales
├── bin/     → Binarios del sistema
└── proc/    → Procesos en ejecución (virtual)
```

---

### `/etc` — Configuración del Sistema

Almacena archivos de configuración que usa el sistema operativo.

```bash
tryhackme@linux2:/etc$ ls
shadow  passwd  sudoers  sudoers.d
```

| Archivo | Contenido |
|---------|-----------|
| `sudoers` | Lista de usuarios/grupos con permisos para usar `sudo` |
| `sudoers.d/` | Directorio con configs de sudo adicionales |
| `passwd` | Información de cuentas de usuario (username, UID, home, shell) |
| `shadow` | Contraseñas de los usuarios en formato hash **sha512** (solo root puede leer) |

> [!warning] `/etc/shadow` es crítico para la seguridad
> Si un atacante obtiene `/etc/shadow`, puede intentar crackear los hashes de contraseñas offline con herramientas como John the Ripper o hashcat. Por eso solo root puede leerlo (`-rw-r----- root shadow`).

---

### `/var` — Datos Variables

"Variable data" — datos frecuentemente modificados por servicios y aplicaciones en ejecución.

```bash
tryhackme@linux2:/var$ ls
backups  log  opt  tmp
```

| Subdirectorio | Contenido |
|--------------|-----------|
| `/var/log` | Logs de servicios (Apache, SSH, syslog, etc.) |
| `/var/backups` | Copias de seguridad del sistema |
| `/var/tmp` | Temporal que persiste entre reinicios (a diferencia de `/tmp`) |

---

### `/root` — Home del Superusuario

El directorio home del usuario **root** (≠ `/home/root`).

```bash
root@linux2:~# ls
myfile  myfolder  passwords.xlsx
```

> [!info] No confundir `/root` con `/`
> - `/` → el directorio raíz del sistema de archivos completo
> - `/root` → el home exclusivo del usuario root

---

### `/tmp` — Archivos Temporales

Datos temporales que solo se necesitan brevemente. **Se borran al reiniciar el sistema.**

```bash
root@linux2:/tmp# ls
todelete  trash.txt  rubbish.bin
```

**Características de seguridad clave:**
- Cualquier usuario puede escribir en `/tmp` por defecto
- No se mantiene entre reinicios

> [!tip] `/tmp` en pentesting
> Al obtener acceso a una máquina, `/tmp` es el lugar ideal para guardar scripts de enumeración, herramientas y resultados temporales. Todo usuario puede escribir aquí y el contenido no requiere privilegios especiales.

> [!warning] Riesgo de `/tmp` escribible por todos
> Actores maliciosos en un sistema comprometido también pueden dejar herramientas o backdoors en `/tmp`. Siempre revísalo durante un análisis forense o respuesta a incidentes.

---

## Comandos de Permiso — Resumen Rápido

```bash
# Ver permisos
ls -l archivo
ls -la          # incluye archivos ocultos (.)

# Cambiar permisos
chmod 755 archivo
chmod u+x archivo       # owner + execute
chmod g-w archivo       # group - write
chmod o=r archivo       # others = solo read

# Cambiar propietario
chown usuario archivo
chown usuario:grupo archivo

# Cambiar grupo
chgrp grupo archivo

# Ver a qué grupos perteneces
id
groups

# Ver quién está conectado
who
whoami

# Cambiar usuario
su usuario
su -l usuario
sudo su          # cambiar a root
```

---

## Notas relacionadas

- [[Linux-Herramientas-y-Admin]] — nano, wget, scp, cron, apt, logs
- [[Sistemas-Operativos]] — Kernel space vs user space, privilegios del sistema
- [[Defensiva-Blue-Team]] — Permisos mal configurados = vector de escalada de privilegios
- [[Ofensiva-Pentesting]] — Enumeración de permisos en post-explotación
