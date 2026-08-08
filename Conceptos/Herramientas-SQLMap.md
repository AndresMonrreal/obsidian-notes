# SQLmap — Explotación automatizada de SQL Injection

Relacionado: [[Bases-de-Datos-SQL]] · [[OWASP-Top-10-2025]] · [[Herramientas-Burp-Suite]]

## Qué es
Herramienta que automatiza la detección y explotación de **SQL Injection**: dado un parámetro sospechoso, prueba distintas técnicas de inyección (booleana, basada en tiempo, UNION, error-based, stacked queries) y, si confirma la vulnerabilidad, permite enumerar bases de datos, tablas, columnas y extraer datos sin escribir el payload a mano.

## Instalación
```bash
pipx install sqlmap
```
Ver [[Linux-Herramientas-y-Admin]] para instalar herramientas Python vía `pipx` sin romper el sistema.

## Uso básico — confirmar y enumerar, sin extraer nada todavía
```bash
sqlmap -u "http://SITIO/ruta?id=1042" --risk=1 --level=1 --dbs
```
- `-u` → URL con el parámetro a probar (aquí `id=1042`).
- `--risk=1` → nivel de riesgo de los payloads (1 = los más seguros/estándar, hasta 3 = payloads más agresivos que pueden dañar datos — usar con cuidado y solo con autorización clara).
- `--level=1` → qué tan a fondo prueba (1 = solo el parámetro GET/POST obvio, hasta 5 = también cookies, headers, etc. — más lento).
- `--dbs` → solo **lista** las bases de datos disponibles, no extrae ni una fila todavía. Primer paso de la enumeración, antes de decidir si vale la pena seguir.

## Flujo típico de enumeración (de menos a más invasivo)
```bash
sqlmap -u "http://SITIO/ruta?id=1042" --risk=1 --level=1 --dbs           # 1. listar bases de datos
sqlmap -u "http://SITIO/ruta?id=1042" -D nombre_bd --tables               # 2. listar tablas de una BD
sqlmap -u "http://SITIO/ruta?id=1042" -D nombre_bd -T tabla --columns     # 3. listar columnas de una tabla
sqlmap -u "http://SITIO/ruta?id=1042" -D nombre_bd -T tabla --dump        # 4. extraer los datos (el paso más invasivo)
```

## Otras banderas útiles
- `--batch` → responde automáticamente "sí" a las preguntas interactivas (útil en scripts, pero revisa antes qué va a asumir).
- `--cookie="PHPSESSID=..."` → prueba el parámetro estando autenticado.
- `--data="campo=valor"` → prueba un parámetro enviado por POST en vez de en la URL.
- `-p parametro` → si la URL tiene varios parámetros, apunta solo a uno específico.

> [!warning] Usar solo dentro del alcance autorizado
> `sqlmap` genera mucho tráfico y puede llegar a modificar datos con `--risk` alto — nunca correrlo contra un objetivo sin autorización explícita y alcance claro.
