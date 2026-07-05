---
tags:
  - pre-security
  - sistemas-operativos
  - kernel
  - linux
  - windows
  - CLI
  - GUI
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🖥️ Sistemas Operativos (OS)

---

## ¿Qué es un OS?

El sistema operativo es el **software central que coordina todo** lo que ocurre en una computadora. Se ubica entre el usuario, las aplicaciones y el hardware físico, actuando como el gestor invisible que mantiene todo funcionando como un sistema unificado.

```
┌─────────────────────────┐
│         USUARIO         │
├─────────────────────────┤
│      APLICACIONES       │  ← Browser, juegos, editores
├─────────────────────────┤
│   SISTEMA OPERATIVO     │  ← Windows, Linux, macOS
├─────────────────────────┤
│        HARDWARE         │  ← CPU, RAM, Disco, GPU
└─────────────────────────┘
```

> [!example] Analogía del aeropuerto
> - **Hardware** (CPU, RAM, disco) = pistas, aviones, radares, combustible
> - **Aplicaciones** (browser, juegos) = aerolíneas y pasajeros que quieren despegar/aterrizar
> - **OS** = el sistema completo de control de tráfico aéreo — asigna recursos, resuelve conflictos y garantiza seguridad

Sin OS, cada app tendría que controlar directamente el CPU, memoria y dispositivos — causando conflictos inmediatos.

---

## Capas de Privilegio del Sistema

Dentro del OS existen dos zonas con diferentes niveles de permiso:

```
┌─────────────────────────────────────────┐
│            USER SPACE                   │
│  Aplicaciones normales (browser, Word)  │
│  Acceso RESTRINGIDO — no tocan hardware │
│  Hacen "system calls" para pedir ayuda  │
├─────────────────────────────────────────┤
│           KERNEL SPACE                  │
│  El kernel — núcleo del OS              │
│  Acceso TOTAL a CPU, RAM, storage       │
│  Gestiona hardware directamente         │
└─────────────────────────────────────────┘
         Hardware físico
```

| | Kernel Space | User Space |
|-|-------------|-----------|
| **Quién opera** | El kernel del OS | Aplicaciones del usuario |
| **Acceso al hardware** | ✅ Directo y total | ❌ Solo a través del kernel |
| **Riesgo de un fallo** | Puede crashear todo el sistema | Solo falla esa app |
| **Analogía** | Torre de control (área restringida) | Pasajeros y aerolíneas en tierra |

### System Calls

Cuando una app necesita algo del hardware (guardar un archivo, reproducir audio, conectarse a WiFi), **no puede hacerlo sola** — debe hacer una **system call**: una solicitud formal al kernel para que actúe en su nombre.

> [!info] Por qué importa en seguridad
> La separación kernel/user space es una defensa fundamental. Un malware en user space no puede acceder directamente al hardware. Si logra ejecutarse en kernel space (escalada de privilegios), el sistema queda completamente comprometido — de ahí la gravedad de los exploits de kernel.

---

## Responsabilidades del OS

| Responsabilidad | Qué hace el OS | Ejemplo real |
|-----------------|---------------|-------------|
| **Process Management** | Crea, programa, prioriza y termina procesos. Decide cuánto CPU time recibe cada uno | Tener browser + música + Spotify abiertos sin que se congele todo |
| **Memory Management** | Asigna RAM a procesos, los aísla entre sí, recupera memoria al cerrar apps. Usa **virtual memory** cuando la RAM se llena | Abrir múltiples apps — el OS asigna RAM a cada una sin que se interfieran |
| **File System Management** | Organiza archivos en directorios, gestiona nombres, rutas, permisos y metadatos (nombre, tamaño, tipo, timestamps) | Crear carpetas, guardar fotos, poner un archivo en "solo lectura" |
| **User Management** | Gestiona cuentas, autenticación y permisos — quién puede acceder a qué | Login con contraseña y que tus archivos sean inaccesibles para otros usuarios |
| **Device Management** | Carga drivers y provee una interfaz universal (**hardware abstraction layer**) para que las apps usen hardware sin saber sus detalles técnicos | Conectar un mouse, impresora o disco externo y que funcione automáticamente |

---

## Seguridad del OS

Antes de cualquier antivirus o firewall, el OS ya aplica protecciones básicas:

| Protección | Descripción |
|-----------|-------------|
| **Authentication** | Verifica quién eres — contraseñas, biometría, PIN |
| **Permissions** | Controla qué puede leer, escribir o ejecutar cada usuario y app |
| **Isolation** | Cada proceso vive en su propio espacio protegido — no puede interferir con otros |
| **System Protection** | Protege archivos críticos del sistema de modificaciones no autorizadas |

> [!warning] El OS como superficie de ataque
> Si un atacante compromete el OS (escalada de privilegios al kernel), tiene control total de la máquina. Por eso mantener el OS actualizado (patches de seguridad) es crítico — los exploits de kernel son los más peligrosos.

---

## Interfaces del OS: GUI vs CLI

### GUI — Graphical User Interface

- Representación visual con ventanas, iconos y menús
- Interacción mediante clics y arrastre
- Intuitivo para el usuario promedio
- **Ejemplo**: El Explorador de archivos de Windows, Finder en macOS

### CLI — Command-Line Interface

- Interfaz de texto donde escribes comandos exactos
- Mayor precisión, control y velocidad
- Ideal para tareas avanzadas, automatización y scripting
- Requiere conocer la sintaxis correcta
- **Ejemplo**: Terminal en Linux/macOS, PowerShell/CMD en Windows

```
Misma tarea — dos formas:

GUI: Abres el explorador → navegas carpetas → haces clic en la carpeta home

CLI: ls /home/ubuntu
     (resultado instantáneo en una línea)
```

> [!tip] CLI en ciberseguridad
> La gran mayoría del trabajo en ciberseguridad (pentesting, análisis forense, administración de servidores) se hace desde la CLI. Los servidores Linux en producción casi nunca tienen GUI. Dominar la CLI es **no negociable** en esta carrera.

---

## Tipos de OS

| Tipo | Uso principal | Características clave |
|------|--------------|----------------------|
| **Desktop** | PCs personales, trabajo, gaming | GUI rica, multitarea, orientado al usuario |
| **Server** | Hosting web, BD, cloud, backend | Sin GUI (headless), máximo uptime, multiusuario, acceso remoto |
| **Mobile** | Smartphones y tablets | UI táctil, eficiencia de batería, siempre conectado, app sandboxing |
| **Embedded** | Electrodomésticos, autos, IoT, routers | Huella mínima, hardware limitado, función única |
| **Virtual/Cloud** | VMs, containers, instancias cloud | Ligero, escalable, despliegue rápido |

---

## OS Reales por Familia

### Desktop

| OS | Versiones comunes | Notas |
|----|-----------------|-------|
| **Windows** | 10 (EOL), 11 | El más usado en PCs personales y corporativo |
| **macOS** | Sonoma (14), Sequoia (15) | Ecosistema Apple, GUI pulida |
| **Linux** | Ubuntu, Debian, Fedora | Open source, altamente personalizable |

### Server

| OS | Versiones comunes | Notas |
|----|-----------------|-------|
| **Windows Server** | 2019, 2022, 2025 | Entornos corporativos, Active Directory |
| **Linux Server** | Ubuntu Server, CentOS, Red Hat | Domina la web — mayoría de servidores del mundo |
| **Unix** | IBM AIX, Oracle Solaris | Finanzas, telecomunicaciones, gobierno |

### Mobile

| OS | Versiones | Notas |
|----|-----------|-------|
| **Android** | 14, 15, 16 | El más usado globalmente, open source base |
| **iOS** | 17, 18 | Solo dispositivos Apple, muy seguro por diseño |

### Embedded e IoT

| OS | Ejemplos | Notas |
|----|---------|-------|
| **Embedded Linux** | OpenWrt, Ubuntu Core | Routers, dispositivos IoT |
| **Real-Time OS (RTOS)** | FreeRTOS, VxWorks, QNX | Sistemas que necesitan respuesta garantizada (aviónica, industria) |

### Virtual y Cloud

| OS | Ejemplos | Notas |
|----|---------|-------|
| **Cloud/VM** | Ubuntu LTS, Amazon Linux, Rocky Linux | Data centers, hosting |
| **Container-optimized** | Alpine Linux, Bottlerocket (AWS) | Mínimos, diseñados solo para correr containers |

---

## ¿Por qué tantos OS diferentes?

Cada entorno tiene necesidades distintas:

- **Laptop** → amigable, multitarea, experiencia de usuario
- **Servidor** → estabilidad 24/7, seguridad, sin interfaz gráfica
- **Móvil** → eficiencia de batería, hardware integrado
- **Embedded** → ligero, función específica, sin recursos de sobra

Ningún OS es perfecto para todos los casos — por eso existe un ecosistema.

> [!tip] Linux en ciberseguridad
> **Linux** domina en servidores, cloud, herramientas de seguridad y dispositivos embebidos. **Kali Linux** (basado en Debian) es el estándar en pentesting. **Ubuntu** es el más común para aprender. Conocer Linux es esencial para cualquier ruta en ciberseguridad.

---

## Notas relacionadas

- [[Sistemas-Hardware]] — El hardware que el OS gestiona
- [[Virtualizacion]] — VMs y containers corren OS dentro del OS
- [[Cloud-Computing]] — Los servidores cloud corren OS Server (Linux principalmente)
- [[Roles-Ciberseguridad]] — El analista SOC trabaja con logs del OS constantemente
