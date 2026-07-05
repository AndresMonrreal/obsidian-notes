---
tags:
  - pre-security
  - cloud
  - AWS
  - IaaS
  - PaaS
  - SaaS
  - infraestructura
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# ☁️ Cloud Computing

---

## ¿Qué es?

Cloud computing es el acceso a recursos de computación (servidores, almacenamiento, redes, software) **a través de Internet**, bajo demanda y pagando solo por lo que usas.

En vez de comprar y mantener hardware propio, alquilas la infraestructura de un proveedor que la gestiona por ti.

---

## Evolución: Del servidor físico a la nube

```
1. Servidor físico  → 1 app por máquina, alto costo, baja utilización
        ↓
2. Virtualización   → múltiples VMs en 1 servidor físico (ver [[Virtualizacion]])
        ↓
3. Cloud Computing  → virtualización a escala masiva, accesible por Internet
```

---

## Beneficios del Cloud

| Beneficio | Descripción |
|-----------|-------------|
| **Escalabilidad** | Aumenta o reduce recursos según la demanda en minutos |
| **On-demand** | Crea o elimina servidores y almacenamiento al instante |
| **Pago por uso** | Solo pagas lo que consumes — sin inversión inicial en hardware |
| **Seguridad** | Los proveedores protegen la infraestructura física |
| **Alta disponibilidad** | La app sigue corriendo aunque parte del sistema falle |
| **Acceso global** | Usuarios de cualquier parte del mundo pueden acceder |

---

## Tipos de Despliegue (Deployment Models)

| Tipo | Quién lo usa | Cuándo usarlo |
|------|-------------|---------------|
| **Public Cloud** | Startups, apps web, proyectos globales | Asequible, sin gestión de infraestructura, escala fácil |
| **Private Cloud** | Bancos, salud, gobierno | Control total, cumplimiento regulatorio, datos sensibles |
| **Hybrid Cloud** | E-commerce, empresas grandes | Datos sensibles en privado + escala pública según demanda |

> [!example] Ejemplo Hybrid Cloud
> Una tienda online guarda los datos de tarjetas de crédito en su nube privada, pero usa la nube pública para manejar el tráfico extra del Black Friday sin comprar servidores permanentes.

---

## Modelos de Servicio (IaaS / PaaS / SaaS)

La diferencia entre modelos es **cuánto gestiona el proveedor vs tú**:

```
                    Tú gestionas:
┌─────────────────────────────────────────────┐
│  SaaS  │ Solo usas la app (Gmail, Zoom)     │ ← Menos control, más simple
├─────────────────────────────────────────────┤
│  PaaS  │ Tu código/app                      │
│        │ Proveedor: OS + infraestructura    │
├─────────────────────────────────────────────┤
│  IaaS  │ Tu OS + app + datos                │ ← Más control, más responsabilidad
│        │ Proveedor: hardware físico         │
└─────────────────────────────────────────────┘
```

### IaaS — Infrastructure as a Service

- Alquilas servidores virtuales, almacenamiento y red
- Tú instalas y gestionas el OS, middleware y la app
- Máximo control, máxima responsabilidad
- **Ejemplo**: AWS EC2, Azure Virtual Machines, Google Compute Engine
- **Para**: DevOps, sysadmins, empresas con equipos técnicos

### PaaS — Platform as a Service

- El proveedor gestiona hardware + OS + runtime
- Tú solo despliegas tu código/aplicación
- Ideal para desarrolladores que no quieren gestionar servidores
- **Ejemplo**: Heroku, Google App Engine, AWS Elastic Beanstalk
- **Para**: Desarrolladores que quieren enfocarse en el producto

### SaaS — Software as a Service

- Software completo accesible desde el navegador o app
- El proveedor gestiona absolutamente todo
- **Ejemplo**: Gmail, Zoom, Salesforce, Microsoft 365, Slack
- **Para**: Usuarios finales — no requiere conocimiento técnico

> [!example] Analogía de vivienda
> - **IaaS** = Alquilas un terreno vacío — tú construyes, amueblas y mantienes
> - **PaaS** = Alquilas un apartamento vacío — solo traes tus muebles y vives
> - **SaaS** = Hotel — todo incluido, solo llegas y usas

---

## Principales Proveedores

| Proveedor | Fortaleza | Cuota de mercado |
|-----------|-----------|-----------------|
| **AWS (Amazon)** | Mayor oferta de servicios, infraestructura global, estándar de la industria | 🥇 Líder |
| **Microsoft Azure** | Entornos empresariales, integración con productos Microsoft, hybrid cloud | 🥈 |
| **Google Cloud (GCP)** | Big Data, IA/ML, análisis de datos, Kubernetes | 🥉 |
| **Alibaba Cloud** | Dominante en Asia, competitivo globalmente | |
| **IBM Cloud** | Hybrid cloud, soluciones empresariales, IA (Watson) | |
| **Oracle Cloud** | Bases de datos, aplicaciones empresariales | |

> [!tip] Para ciberseguridad
> **AWS** es el más importante conocer — es el que más aparece en ofertas de trabajo (AWS Security Specialty) y el que usan la mayoría de empresas. Azure es segundo, especialmente en entornos corporativos.

---

## Cómo lo usan empresas reales

| Empresa | Uso del Cloud |
|---------|--------------|
| **Netflix** | AWS — escala global, millones de streams simultáneos |
| **Spotify** | Google Cloud — millones de canciones y usuarios |
| **Instagram** | AWS — almacenamiento masivo de fotos/videos |
| **Tiendas online** | Escalan durante Black Friday sin hardware permanente |

---

## Cloud vs On-Premise

| | On-Premise (físico) | Cloud |
|-|--------------------|----|
| **Costo inicial** | 💲💲💲 Alto | 💲 Bajo o nulo |
| **Escalabilidad** | Lenta (comprar hardware) | Inmediata |
| **Control** | Total | Compartido con el proveedor |
| **Mantenimiento** | Tu equipo | El proveedor |
| **Seguridad física** | Tú la gestionas | El proveedor la garantiza |
| **Ideal para** | Datos ultra-sensibles, regulaciones estrictas | La mayoría de casos |

---

## Modelo de Responsabilidad Compartida

En el cloud, la seguridad es **responsabilidad compartida**:

```
Proveedor (AWS/Azure/GCP):          Cliente (tú):
✅ Hardware físico                   ✅ Datos
✅ Infraestructura de red           ✅ Configuración de accesos
✅ Hypervisor                       ✅ OS (en IaaS)
✅ Seguridad física del data center ✅ Aplicaciones
                                    ✅ Cifrado de datos
```

> [!warning] El error más común en seguridad cloud
> Asumir que "el cloud es seguro por defecto". Los **misconfigurations** (buckets S3 públicos, roles IAM sin restricciones) son la causa #1 de brechas en AWS. La responsabilidad del cliente es real.

---

## Notas relacionadas

- [[Virtualizacion]] — La tecnología base sobre la que funciona el cloud
- [[Sistemas-Hardware]] — La evolución del servidor físico al cloud
- [[Arquitectura-Web]] — Las apps web modernas corren en el cloud
- [[Seguridad-de-Red]] — Firewall y VPN también existen en el cloud (Security Groups, VPC)
