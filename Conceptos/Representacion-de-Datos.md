---
tags:
  - pre-security
  - datos
  - binario
  - hexadecimal
  - ASCII
  - unicode
  - bits
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔢 Representación de Datos — Bits, Hex, ASCII y Unicode

---

## Bits y Bytes — La Base de Todo

Una computadora solo entiende dos estados: **0 y 1** (bajo/alto voltaje, norte/sur magnético, presencia/ausencia de luz).

| Concepto | Definición |
|----------|-----------|
| **Bit** | La unidad mínima de información — un dígito que puede ser 0 o 1 |
| **Byte** (u Octet) | Grupo de **8 bits** — puede representar 256 estados (2⁸ = 256) |
| **Nibble** | Grupo de **4 bits** — representa 16 estados, equivale a un dígito hexadecimal |

### ¿Cuántos valores puede representar N bits?

```
1 bit  → 2¹  = 2 estados    (0 o 1)
4 bits → 2⁴  = 16 estados   (0 a 15)
8 bits → 2⁸  = 256 estados  (0 a 255)
N bits → 2^N estados
```

---

## Colores RGB — El ejemplo más visual

Los colores en pantalla combinan tres luces: **Rojo (R), Verde (G) y Azul (B)**.

### Con 1 bit por color (solo ON/OFF) → 8 colores

| Bits (R G B) | Color |
|-------------|-------|
| 000 | Negro (todo apagado) |
| 001 | Azul |
| 010 | Verde |
| 100 | Rojo |
| 011 | Cian (verde + azul) |
| 101 | Magenta (rojo + azul) |
| 110 | Amarillo (rojo + verde) |
| 111 | Blanco (todo encendido) |

### Con 8 bits por canal (256 niveles) → 16 millones de colores

```
256 × 256 × 256 = 16,777,216 colores
```

Cada color usa **3 bytes (24 bits)** — uno por canal RGB.

---

## Sistema Binario (Base 2)

Los humanos usamos base 10 (dedos). Las computadoras usan base 2.

**Regla:** cada posición vale una potencia de 2 (de derecha a izquierda: 2⁰, 2¹, 2², 2³...)

### Cómo convertir binario → decimal

```
1001 en binario:
= 1×2³ + 0×2² + 0×2¹ + 1×2⁰
= 1×8  + 0×4  + 0×2  + 1×1
= 8 + 0 + 0 + 1
= 9 ✅
```

### Tabla de referencia (4 bits)

| Binario | Decimal |
|---------|---------|
| 0000 | 0 |
| 0001 | 1 |
| 0010 | 2 |
| 0011 | 3 |
| 0100 | 4 |
| 0101 | 5 |
| 0110 | 6 |
| 0111 | 7 |
| 1000 | 8 |
| 1001 | 9 |
| 1010 | 10 |
| 1011 | 11 |
| 1100 | 12 |
| 1101 | 13 |
| 1110 | 14 |
| 1111 | 15 |

---

## Sistema Hexadecimal (Base 16)

El hexadecimal agrupa **4 bits en un solo carácter**, haciendo el binario mucho más legible.

Como necesitamos 16 símbolos (0-15) y solo tenemos 10 dígitos, se usan letras para 10-15:

| Decimal | Hex | Binario |
|---------|-----|---------|
| 0-9 | 0-9 | 0000 a 1001 |
| 10 | **A** | 1010 |
| 11 | **B** | 1011 |
| 12 | **C** | 1100 |
| 13 | **D** | 1101 |
| 14 | **E** | 1110 |
| 15 | **F** | 1111 |

### Ejemplo — Color en hex

El color verde `10100011 11101010 00101010` en binario es `A3EA2A` en hex.

Mucho más cómodo de leer y escribir — por eso los colores en diseño se escriben como `#A3EA2A`.

### Convertir hex → decimal

```
9BDF (hex) = ?
= 9×16³ + 11×16² + 13×16¹ + 15×16⁰
= 9×4096 + 11×256 + 13×16 + 15×1
= 36864 + 2816 + 208 + 15
= 39,903 ✅
```

### Sistema Octal (Base 8) — Opcional

Agrupa 3 bits en un dígito (0-7). Menos común, pero aparece en permisos de Linux (`chmod 755`).

```
357 (octal) = 3×8² + 5×8¹ + 7×8⁰ = 192 + 40 + 7 = 239
```

---

## ASCII — American Standard Code for Information Interchange (1963)

Estándar que asigna un número del 0-127 a cada carácter del inglés (letras, dígitos, puntuación y caracteres de control). Usa **7 bits**.

### Puntos clave de ASCII

- Las letras están **en orden**: si `a` = 97 (0x61), entonces `b` = 98 (0x62), `c` = 99 (0x63)...
- Mayúsculas van antes que minúsculas: `A` = 65 (0x41), `a` = 97 (0x61)
- Los dígitos 0-9 van del 48 al 57

### Muestra de la tabla ASCII

| Decimal | Hex | Símbolo |
|---------|-----|---------|
| 48 | 30 | `0` |
| 57 | 39 | `9` |
| 65 | 41 | `A` |
| 90 | 5A | `Z` |
| 97 | 61 | `a` |
| 122 | 7A | `z` |
| 127 | 7F | DEL |

### "TryHackMe" en ASCII

```
T     r     y     H     a     c     k     M     e
0x54  0x72  0x79  0x48  0x61  0x63  0x6B  0x4D  0x65
```

### Limitación de ASCII

Solo cubre inglés. Para otros idiomas europeos se crearon estándares regionales:
- **ISO-8859-1** → Español, Francés, Alemán, idiomas del oeste de Europa
- **ISO-8859-2** → Polaco, Checo, Húngaro, idiomas del este de Europa

**Problema:** si el archivo fue guardado en ISO-8859-1 y se abre con ISO-8859-2, los caracteres especiales se muestran mal. La `Ø` se convierte en `Ř`.

---

## Unicode — El Estándar Universal

Unicode asigna un **code point único** a cada carácter de todos los idiomas del mundo, sistemas de escritura históricos y emojis.

- **Unicode 17.0** define ~157,000 caracteres, incluyendo ~4,000 secuencias de emoji
- Ya no hay que preocuparse por qué encoding usó el autor del archivo

### Ejemplos de code points

| Code point | Carácter | Descripción |
|-----------|---------|-------------|
| U+0041 | A | Letra latina A |
| U+03A9 | Ω | Letra griega Omega |
| U+3042 | あ | Hiragana japonés |
| U+9F8D | 龍 | Dragón en chino |
| U+1F60A | 😊 | Emoji sonrisa |
| U+265E | ♞ | Caballo de ajedrez negro |

---

## UTF-8, UTF-16 y UTF-32 — Cómo se almacena Unicode

Unicode define los code points, pero **UTF** define cómo se almacenan en bytes:

| Encoding | Bytes por carácter | Compatibilidad | Uso |
|----------|--------------------|----------------|-----|
| **UTF-8** | 1 a 4 (dinámico) | ✅ Compatible con ASCII | 🌐 Dominante en la web |
| **UTF-16** | 2 o 4 | Compatible con la mayoría de sistemas | Windows internamente, Java |
| **UTF-32** | Siempre 4 | Universal pero ineficiente | Procesamiento interno |

### UTF-8 en detalle (el más importante)

```
ASCII (U+0000 a U+007F)  → 1 byte   — igual que ASCII original
Ω (U+03A9)               → 2 bytes
🔥 (U+1F525)             → 4 bytes
```

UTF-8 es eficiente: texto en inglés ocupa lo mismo que en ASCII. Para otros idiomas usa más bytes solo cuando es necesario.

> [!tip] Para ciberseguridad
> Entender encodings es crucial en ataques como **encoding bypass** en WAFs, **SQL Injection con caracteres Unicode**, y análisis de malware donde los strings pueden estar en diferentes encodings. También aparece constantemente en análisis forense de archivos.

---

## Resumen — Los sistemas en una línea

```
Decimal (base 10): 0-9         → uso humano cotidiano
Binario (base 2):  0-1         → el lenguaje nativo del hardware
Hexadecimal (base 16): 0-9,A-F → representación compacta de binario (4 bits = 1 dígito)
Octal (base 8): 0-7            → agrupa 3 bits (permisos de Linux)
ASCII:  7 bits, 128 chars      → inglés únicamente
Unicode: code points universales → todos los idiomas + emoji
UTF-8:  1-4 bytes dinámicos    → el estándar actual de la web
```

---

## Notas relacionadas

- [[Sistemas-Operativos]] — El OS gestiona cómo los archivos se guardan con su encoding
- [[Criptografia]] — Los algoritmos de cifrado operan sobre bits y bytes
- [[HTTP-Web]] — Los headers HTTP incluyen `Content-Type: charset=UTF-8`
