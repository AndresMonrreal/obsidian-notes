---
tags:
  - pre-security
  - hardware
  - sistemas-operativos
  - boot
  - UEFI
  - tipos-computadoras
  - IoT
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 💻 Sistemas, Hardware y Proceso de Arranque

---

## Tipos de Computadoras

No todas las computadoras tienen pantalla ni están hechas para que una persona se siente frente a ellas. Existen 4 tipos principales en entornos profesionales:

| Tipo | Pantalla/Teclado | Propósito principal |
|------|-----------------|---------------------|
| **Laptop** | ✅ Sí | Computación portátil para uso diario |
| **Desktop** | ✅ Sí | Rendimiento sostenido en un lugar fijo |
| **Workstation** | ✅ Sí | Precisión y fiabilidad para tareas profesionales (CAD, simulaciones, 3D) |
| **Server** | ❌ No | Proveer servicios a múltiples usuarios simultáneamente a través de la red |

### Diferencias clave

**Laptop** — portable, batería, refrigeración limitada. Sacrifica rendimiento por movilidad.

**Desktop** — usa corriente de pared, mejor refrigeración, más consistente que un laptop en tareas largas.

**Workstation** — parece un desktop pero usa componentes especializados que priorizan la **precisión y fiabilidad** (reducción de errores en cómputos complejos). Usada por ingenieros, animadores, científicos.

**Server** — nunca se apaga, atiende requests de múltiples usuarios al mismo tiempo. No tiene interfaz visual directa. Es la máquina detrás de los servicios que usas cada día.

> [!info] Relevancia en ciberseguridad
> Los servidores son los objetivos más valiosos para los atacantes (datos, servicios críticos). Entender que funcionan 24/7 sin pantalla es clave para entender su gestión y los vectores de ataque remotos (SSH, RDP, exploits de servicios).

---

## Otros tipos de computadoras

| Tipo | Qué es | Ejemplos |
|------|--------|---------|
| **Smartphone** | Computadora de bolsillo optimizada para batería y conectividad | iPhone, Android |
| **Tablet** | Computadora touch con pantalla más grande | iPad, tablets de dibujo |
| **IoT Device** | Dispositivo conectado a red con un propósito único | Termostato inteligente, timbre con cámara, fitness tracker |
| **Embedded Computer** | Computadora integrada dentro de otro dispositivo | Controlador de cafetera, sensor de puerta automática, dimmer de luz |

### IoT vs Embedded — la diferencia clave

| | IoT | Embedded |
|-|-----|---------|
| **Conectividad** | ✅ Se conecta a una red | ❌ Puede no conectarse a nada |
| **Función** | Reportar datos o recibir comandos remotamente | Ejecutar su tarea dentro del aparato |
| **Visibilidad** | Gestionable desde una app o panel | Invisible, corre en segundo plano durante años |
| **Ejemplo** | Cámara de seguridad WiFi | Chip controlador de la puerta automática |

> [!warning] IoT en ciberseguridad
> Los dispositivos IoT son uno de los vectores de ataque más ignorados. Muchos tienen contraseñas por defecto, firmware desactualizado y no tienen pantalla para que el usuario note algo raro. Son usados constantemente para crear **botnets** (ej. Mirai Botnet).

---

## Proceso de Arranque del Sistema (Boot Process)

Cuando presionas el botón de encendido, el sistema pasa por 5 pasos antes de mostrarte el sistema operativo.

```
[1] Power Button
       ↓
[2] UEFI/BIOS arranca
       ↓
[3] POST — Power-On Self Test
       ↓
[4] Selección del Boot Device
       ↓
[5] Bootloader carga el OS
       ↓
[Sistema Operativo activo ✅]
```

---

### Paso 1 — Power Button

Al presionar el botón, se envía una señal a la **PSU (Power Supply Unit)** para que distribuya energía a todos los componentes del sistema.

---

### Paso 2 — Firmware (UEFI / BIOS)

Una vez que los componentes físicos reciben energía, el **firmware** toma el control. Su trabajo es inicializar y activar todos los componentes de hardware.

| | BIOS | UEFI |
|-|------|------|
| **Nombre** | Basic Input/Output System | Unified Extensible Firmware Interface |
| **Estado** | Legacy (antiguo) | Reemplaza al BIOS en sistemas modernos |
| **Interfaz** | Solo texto | Gráfica (mouse + teclado) |
| **Soporte de disco** | MBR (hasta 2TB) | GPT (discos grandes, +2TB) |
| **Seguridad** | Básica | **Secure Boot** — verifica que el OS no esté comprometido |

> [!info] BIOS vs UEFI
> En la práctica, cuando alguien dice "entrar al BIOS" se refiere a entrar a la configuración del firmware, aunque sea UEFI. Los términos se usan indistintamente.

---

### Paso 3 — POST (Power-On Self Test)

El UEFI ejecuta una rutina de diagnóstico que verifica que **cada componente crítico esté presente, configurado correctamente y funcionando**.

**Qué prueba el POST:**
- RAM instalada y accesible
- CPU funcionando
- Dispositivos de almacenamiento detectados
- Tarjeta de video presente
- Periféricos básicos (teclado, etc.)

**Si algo falla:**
- **Beep codes** — una serie de pitidos que identifican el componente con problema (varía por fabricante)
- **Mensaje de error** en pantalla (en sistemas UEFI modernos)

> [!tip] En ciberseguridad forense
> Los logs del POST y el UEFI pueden revelar si alguien intentó arrancar el sistema desde un dispositivo externo (USB malicioso) o si el Secure Boot fue desactivado.

---

### Paso 4 — Selección del Boot Device

El UEFI tiene una **lista ordenada de dispositivos** donde buscar el sistema operativo (Boot Order):

```
Prioridad 1: USB Drive
Prioridad 2: SSD/HDD interno
Prioridad 3: DVD/CD
Prioridad 4: Red (PXE Boot)
```

El UEFI revisa cada dispositivo en orden hasta encontrar uno que tenga un bootloader válido.

> [!warning] Boot Order en seguridad
> Si un atacante tiene acceso físico al equipo y el Boot Order tiene USB en prioridad 1, puede arrancar un Live OS desde un USB (ej. Kali Linux) y acceder al sistema sin credenciales. **Siempre asegurar el UEFI con contraseña y poner el disco interno en primer lugar**.

---

### Paso 5 — Bootloader

Una vez identificado el dispositivo de arranque, el UEFI inicia el **bootloader** — un pequeño programa que:

1. Carga el **Sistema Operativo** desde el disco hacia la **RAM**
2. Transfiere el control de todos los componentes al OS

**Bootloaders comunes:**

| OS | Bootloader |
|----|-----------|
| Windows | Windows Boot Manager |
| Linux | GRUB (GRand Unified Bootloader) |
| macOS | iBoot |

> [!info] Dual Boot
> GRUB permite elegir entre múltiples sistemas operativos al arrancar. Esto es posible porque el bootloader se carga antes que el OS y puede ofrecer un menú de selección.

---

## Resumen visual del Boot

```
PSU activa
    ↓
UEFI inicializa hardware
    ↓
POST verifica que todo esté OK
    ↓  ← Si falla → beep codes / error
UEFI busca Boot Device (en orden de prioridad)
    ↓
Bootloader carga el OS en RAM
    ↓
OS toma control del hardware
    ↓
Pantalla de login ✅
```

---

## Notas relacionadas

- [[OSI-Model]] — Los servidores operan comunicándose a través de las capas de red
- [[Infraestructura-de-Red]] — Servidores en el contexto de la arquitectura de red
- [[Arquitectura-Web]] — El tipo "Server" es el que corre Apache/Nginx/IIS
- [[Seguridad-de-Red]] — VPN y Firewall protegen los servidores en la red
