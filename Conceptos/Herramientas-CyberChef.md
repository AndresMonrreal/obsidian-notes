# CyberChef — Navaja suiza de datos

Relacionado: [[Hashing]] · [[Criptografia-Clave-Publica]] · [[Herramientas-CAPA-Malware-Analysis]] · [[Herramientas-REMnux]]

## Qué es
Aplicación web para tareas "cyber": desde encodings simples (Base64) hasta cifrado/descifrado. Trabaja con **recetas (recipes)**: una serie de operaciones ejecutadas en orden.

- **Online**: gchq.github.io/CyberChef
- **Offline**: descargar el release y correrlo local (Windows/Linux) — recomendable la versión estable.

## Las 4 áreas de la interfaz
| Área | Qué hace |
|------|----------|
| **Operations** | catálogo completo de operaciones, categorizadas y buscables; hover sobre una para ver ejemplo/descripción |
| **Recipe** | el corazón de la herramienta: arrastras operaciones, defines argumentos. Botones: **Save/Load/Clear recipe** |
| **Input** | pegar/escribir/arrastrar texto o archivos. Botones: nueva pestaña, abrir carpeta, abrir archivo, limpiar |
| **Output** | resultado procesado. Botones: guardar a `.dat`, copiar al portapapeles, **reemplazar input con output**, maximizar |

Botón **BAKE!** procesa la receta. Checkbox **Auto Bake** la corre automáticamente sin tener que darle clic cada vez.

## Flujo de pensamiento (4 pasos)
1. **Objetivo claro** — "¿qué quiero lograr?" (ej. "encontré un string ilegible, quiero saber qué mensaje esconde").
2. **Meter los datos** al área de Input.
3. **Elegir operaciones** — investigar/probar (ej. si sospechas cifrado, prueba ROT13, Base64, Base85, ROT47).
4. **Revisar el output** — ¿es el resultado esperado? Si no, repetir desde el paso 1.

## Operaciones más usadas

### Extractors
| Operación | Qué extrae |
|-----------|-----------|
| **Extract IP addresses** | IPv4/IPv6 válidas |
| **Extract URLs** | requiere el protocolo (`http://`, etc.) para evitar falsos positivos |
| **Extract email addresses** | formato `algo@dominio.com` |

### Date / Time
```
From UNIX Timestamp   → convierte timestamp a fecha legible
To UNIX Timestamp     → convierte fecha (UTC) a timestamp
```
Ej.: `Fri Sep 6 20:30:22 +04 2024` → `1725654622`

### Data Format
| Operación | Ejemplo |
|-----------|---------|
| **From Base64** | `V2VsY29tZSB0byB0cnloYWNrbWUh` → `Welcome to tryhackme!` |
| **URL Decode** | `https%3A%2F%2F...` → `https://...` |
| **From Base85** | notación más eficiente que Base64 |
| **From Base58** | quita caracteres confusos (l, I, 0, O) para legibilidad humana |
| **To Base62** | usa un alfabeto amplio → strings más cortos que decimal/hex |

### Otras (de la tabla general)
| Operación | Ejemplo |
|-----------|---------|
| **From Morse Code** | `.... .-. . .- - ...` → `THREATS` |
| **URL Encode** | codifica caracteres especiales a `%XX` |
| **To Base64** | `This is fun!` → `VGhpcyBpcyBmdW4h` |
| **To Hex** | separa bytes en hexadecimal |
| **To Decimal** | convierte a array de enteros ordinales |
| **ROT13** | rota el alfabeto (Caesar cipher, default 13) |

## Base64 manual — cómo funciona por dentro
Codifica "THM" a mano:

**Paso 1 — a binario (8 bits c/u) y concatenar:**
`T=01010100 H=01001000 M=01001101` → `010101000100100001001101` (24 bits)

**Paso 2 — dividir en grupos de 6 bits y convertir a decimal:**
`010101 000100 100001 001101` → `21, 4, 33, 13`

**Paso 3 — mapear con la tabla índice de Base64 (0-25=A-Z, 26-51=a-z, 52-61=0-9, 62=+, 63=/):**
`21→V, 4→E, 33→h, 13→N` → resultado: **`VEhN`**

## URL Decode — caracteres comunes (UTF-8)
| Carácter | Encoded |
|----------|---------|
| `:` | `%3A` |
| `/` | `%2F` |
| `.` | `%2E` |
| `=` | `%3D` |
| `#` | `%23` |
