---
tags:
  - pre-security
  - virtualizacion
  - hypervisor
  - VM
  - containers
  - docker
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🖥️ Virtualización, Hypervisors y Containers

---

## ¿Por qué existe la virtualización?

Antes de la virtualización: **1 servidor = 1 aplicación**. Esto causaba:

| Problema | Descripción |
|----------|-------------|
| **Alto costo** | Hardware, electricidad, refrigeración y espacio para cada servicio |
| **Baja utilización** | La mayoría de servidores operaba al 5–20% de su capacidad |
| **Despliegue lento** | Configurar un servidor físico tardaba días o semanas |
| **Difícil escalar** | Más demanda = comprar otro servidor físico |

La virtualización resuelve esto permitiendo que **múltiples sistemas compartan el mismo hardware de forma segura**.

> [!example] Analogía del edificio
> - **Sin virtualización**: una sola persona en un edificio de 10 pisos — paga todo, usa uno
> - **Con virtualización**: el edificio se divide en apartamentos independientes — cada uno con su puerta, cocina y privacidad, compartiendo la estructura, agua y electricidad
>
> | Edificio | Virtualización |
> |----------|----------------|
> | El edificio | El servidor físico |
> | Los apartamentos | Las Máquinas Virtuales (VMs) |
> | Los inquilinos | Aplicaciones o sistemas operativos |
> | El administrador | El **Hypervisor** |

---

## Hypervisor — El Administrador

El hypervisor es el software que **crea y gestiona las máquinas virtuales**. Se encarga de:

- Dividir el hardware físico en múltiples sistemas virtuales
- Asignar CPU, RAM y almacenamiento a cada VM
- Mantener el **aislamiento** entre VMs (si una falla, las demás siguen funcionando)
- Gestionar el ciclo de vida de las VMs: iniciar, pausar, clonar, eliminar

### Tipo 1 vs Tipo 2

| | **Type 1** (Bare-Metal) | **Type 2** (Hosted) |
|-|------------------------|---------------------|
| **Corre sobre** | Directamente en el hardware físico | Sobre un OS existente (Windows/Linux/macOS) |
| **Rendimiento** | ⚡ Alto — sin OS intermedio | 🐢 Menor — comparte recursos con el OS host |
| **Uso típico** | Servidores, data centers, producción | Aprendizaje, laboratorios, testing en PC personal |
| **Ejemplos** | VMware ESXi, Microsoft Hyper-V, Proxmox | VirtualBox, VMware Workstation, Parallels |

### ¿Cuándo usar cada uno?

| Caso de uso | Type 1 | Type 2 |
|-------------|--------|--------|
| Servidor de producción | ✅ | |
| Servidor de base de datos | ✅ | |
| Data center | ✅ | |
| Analizar archivos maliciosos | | ✅ |
| Testing de software | | ✅ |
| Correr Kali Linux en tu PC | | ✅ |

> [!warning] Analizar malware en VMs
> Al testear malware en una VM, asegúrate de **aislar la VM de la red** para que no infecte el host ni se propague. Idealmente, usa un OS diferente en el guest vs el host.

---

## VM — Máquina Virtual (El Apartamento)

Una VM es un **computador virtual completo** creado por el hypervisor. Aunque es virtual, se comporta exactamente como una máquina física:

- Tiene su propio CPU virtual, RAM, almacenamiento y red
- Puede correr cualquier OS (Windows, Linux, macOS)
- Está **completamente aislada** de otras VMs
- Si una VM falla → las demás continúan funcionando sin problema

**Herramientas populares para Type 2 (tu PC):**
- **Oracle VirtualBox** — gratuito, open source
- **VMware Workstation** — más funciones, versión de pago

**Casos de uso prácticos:**
- Correr Kali Linux sin comprar otro equipo
- Crear un entorno aislado para analizar malware
- Simular una red con múltiples máquinas para practicar

---

## Containers — Las Habitaciones del Apartamento

Un container es un **entorno ligero y aislado** que corre una sola aplicación con todas sus dependencias.

### VM vs Container

```
┌─────────────────────────────────────────────────────┐
│                  SERVIDOR FÍSICO                    │
│  ┌──────────────────────────────────────────────┐  │
│  │               HYPERVISOR                     │  │
│  │  ┌────────────────┐  ┌────────────────────┐  │  │
│  │  │      VM 1      │  │        VM 2        │  │  │
│  │  │  ┌──────────┐  │  │  ┌──────┐ ┌──────┐│  │  │
│  │  │  │  Guest   │  │  │  │Cont 1│ │Cont 2││  │  │
│  │  │  │    OS    │  │  │  │(App) │ │(App) ││  │  │
│  │  │  │   App    │  │  │  └──────┘ └──────┘│  │  │
│  │  │  └──────────┘  │  │   (comparten el   │  │  │
│  │  │                │  │    kernel del OS) │  │  │
│  │  └────────────────┘  └────────────────────┘  │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

| | **VM** | **Container** |
|-|--------|--------------|
| **Incluye** | OS completo + App | Solo la App + dependencias |
| **Peso** | GB (OS completo) | MB (solo lo necesario) |
| **Arranque** | Minutos | Segundos |
| **Aislamiento** | Total (kernel propio) | Parcial (comparte kernel del host) |
| **Portabilidad** | Buena | ⭐ Excelente |
| **Compatibilidad** | Corre cualquier OS | Debe coincidir con el kernel del host |

> [!info] Restricción de containers
> Los containers **comparten el kernel del host**. Esto significa que un container Windows no puede correr en un host Linux y viceversa. Las VMs no tienen esta limitación.

### ¿Cuándo usar qué?

- **VM** → cuando necesitas aislamiento total, un OS diferente, o entornos de seguridad
- **Container** → cuando despliegas aplicaciones que necesitan ser rápidas, portátiles y escalables

---

## Docker — La Herramienta de Containers

**Docker** es la plataforma open-source más popular para crear, desplegar y gestionar containers.

```
Imagen Docker (recipe/template)
       ↓
   docker run
       ↓
Container en ejecución (instancia de la imagen)
```

- **Image** = plantilla/receta con todo lo que necesita la app
- **Container** = instancia en ejecución de una imagen
- **Docker Hub** = repositorio público de imágenes pre-construidas

> [!tip] En ciberseguridad
> Muchas herramientas de seguridad se distribuyen como imágenes Docker (Metasploit, Burp Suite, herramientas de análisis). Entender Docker es cada vez más esencial en el trabajo diario de un analista o pentester.

---

## Beneficios de la Virtualización

| Beneficio | Descripción |
|-----------|-------------|
| **Ahorro de costos** | Un servidor físico reemplaza a muchos |
| **Mejor uso de recursos** | CPU y RAM al 70-90% en vez de 5-20% |
| **Testing seguro** | Malware, exploits y configuraciones en entornos aislados |
| **Despliegue rápido** | Una VM nueva en minutos, no semanas |
| **Flexibilidad** | Cualquier OS, cualquier configuración |
| **Portabilidad** | Mueve VMs o containers entre servidores fácilmente |
| **Escalabilidad** | Añade más VMs/containers según la demanda |
| **Gestión centralizada** | Administra todo desde un único hypervisor |

---

## Notas relacionadas

- [[Sistemas-Hardware]] — El hardware físico sobre el que corre el hypervisor
- [[Cloud-Computing]] — La nube usa virtualización y containers a gran escala
- [[Arquitectura-Web]] — Los web servers corren dentro de VMs o containers
- [[Seguridad-de-Red]] — Las VMs son entornos aislados clave para análisis de seguridad
