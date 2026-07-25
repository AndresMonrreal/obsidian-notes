---
tags:
  - pre-security
  - roles
  - blue-team
  - SOC
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🛡️ Roles en Ciberseguridad

## Security Analyst

El **Security Analyst** (Analista de Seguridad) es el defensor digital de una organización. Forma parte del **[[Blue Team]]** — el equipo encargado de la defensa.

> [!summary] Función principal
> Monitorear, investigar y responder a alertas de actividad inusual en dispositivos y la red.

### Tareas del día a día

- Monitorear **tráfico de red** y **logs** del sistema
- Investigar accesos sospechosos (ej. empleado en Londres que aparece logueado desde otro país)
- Colaborar con otros equipos para reforzar la postura de seguridad
- Documentar incidentes y crear reportes

### Herramientas comunes

| Herramienta | Uso |
|-------------|-----|
| SIEM (Splunk, Elastic) | Centralizar y analizar logs |
| IDS/IPS | Detectar intrusiones |
| Wireshark | Análisis de tráfico de red |
| Threat Intel Platforms | Investigar amenazas conocidas |

---

## Blue Team vs Red Team

| Equipo | Rol | Enfoque |
|--------|-----|---------|
| 🔵 **Blue Team** | Defensivo | Detectar, responder, proteger |
| 🔴 **Red Team** | Ofensivo | Atacar, explotar, reportar |
| 🟣 **Purple Team** | Híbrido | Mejorar defensa usando técnicas ofensivas |

---

## Especializaciones futuras

Desde el rol de Analista puedes especializarte en:

- **Incident Response (IR)** — Respuesta a incidentes activos
- **Threat Hunting** — Búsqueda proactiva de amenazas ocultas en la red
- **Malware Analysis** — Análisis de código malicioso (reversing, sandboxing)
- **SOC Analyst (L1/L2/L3)** — Centro de Operaciones de Seguridad

> [!tip] Consejo de carrera
> El camino típico es: **SOC Analyst L1 → L2 → Threat Hunter / IR Specialist**. Cada nivel requiere más autonomía e investigación profunda.

---

## Security Engineer

Es el **arquitecto** de la ciberseguridad: construye y mantiene los sistemas y procesos que protegen la red y los dispositivos de la organización (ej. mantener un IDS, que actúa como "cámara de seguridad" digital — ver [[IDS-Snort]]).

> [!summary] Función principal
> Diseñar, construir y mantener las defensas técnicas de la organización.

### Tareas del día a día
- Diseñar y mantener sistemas de seguridad
- Mantenerse al día con las últimas técnicas de ataque
- Documentar procesos y procedimientos
- Evaluar riesgo y asegurar que sistemas/apps estén protegidos contra vulnerabilidades

### Progresión
Ya es un rol defensivo especializado, pero puede profundizar en **Application Security** o **Cloud Security**. Las habilidades también transfieren bien a roles ofensivos.

---

## Penetration Tester ("Pentester" / Ethical Hacker)

Comprueba qué tan seguros son los sistemas/software de una empresa intentando "romperlos" de forma segura y controlada, bajo un acuerdo formal con la empresa (**engagement**). Ver [[Ofensiva-Pentesting]].

> [!summary] Función principal
> Buscar y explotar vulnerabilidades como lo haría un atacante real — pero de forma autorizada y controlada — para que la empresa las corrija antes que un cibercriminal las use.

### Tareas del día a día
- Testear la seguridad de sistemas, redes y sitios web
- Realizar evaluaciones de seguridad, auditorías y análisis de políticas
- Analizar resultados y crear reportes con recomendaciones

### Progresión
Punto de partida fuerte para una carrera ofensiva. Puedes especializarte en **network testing** o **web application testing**, y avanzar a **Red Teaming** (simular ataques a gran escala y largo plazo en toda la organización, a veces incluyendo intrusión física). El Red Team es una progresión avanzada del pentesting.

---

## El valor del training (por qué invertir en capacitación)

- Aumenta la **capacidad** del equipo sin contratar más gente.
- Permite contratar juniors y acelerarlos rápido; reduce el esfuerzo repetido de los seniors enseñando lo mismo.
- Crea una **línea base común** para evaluar skills (en vez de etiquetas vagas como "junior"/"senior").
- Fomenta trabajo en equipo (ej. CTFs).

### Cálculo simplificado de ROI
```
Equipo: 10 empleados · $80,000/año c/u · training aumenta productividad 4%

Ganancia = 10 × 4% × $80,000 = $32,000
Costo del training = 10 × $500 = $5,000
ROI = $32,000 / $5,000 = 640%
```

### Preguntas para elegir un vendor de training
¿Para quién es la capacitación? ¿Cuál es su experiencia/rol/temas relevantes? ¿El vendor tiene experiencia con organizaciones similares? ¿Qué tan amplio/profundo es el contenido en los temas que importan? ¿Se puede aprender, entrenar y practicar en una sola plataforma? El costo suele quedar chico frente al beneficio de productividad del equipo.

---

## Notas relacionadas

- [[Redes-Fundamentos]] — La base técnica que necesita un analista
- [[Protocolos-ARP-DHCP]] — Protocolos que aparecen constantemente en los logs
- [[Subnetting]] — Esencial para entender la segmentación de redes
