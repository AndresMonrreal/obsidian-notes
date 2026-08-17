# Burp Suite — Decoder, Organizer, Comparer y Sequencer

Relacionado: [[Herramientas-Burp-Suite]] · [[Herramientas-Burp-Suite-Repeater]] · [[Herramientas-Burp-Suite-Intruder]] · [[Herramientas-CyberChef]] · [[Web-Pentest-Credential-Stuffing-IDOR-Ejemplo]]

## Decoder — codificar/decodificar/hashear datos
No solo decodifica datos capturados — también codifica datos propios para mandarlos al objetivo, genera hashes, y trae **Smart Decode** (como la función "Magic" de CyberChef) que intenta decodificar recursivamente hasta llegar a texto plano.

### Interfaz
- Caja de texto para pegar/escribir datos (o *Send to Decoder* desde otro módulo).
- Radio button **Text**/**Hex** para tratar el input como texto o bytes.
- Dropdowns **Decode as** / **Encode as** / **Hash** debajo de cada bloque de datos.
- **Smart Decode** al final — autodetecta el tipo de codificación y decodifica.

Cada operación genera un nuevo bloque debajo, permitiendo **encadenar transformaciones** (ej. texto → Base64 → ASCII Hex → Octal, todo en una sola cadena de pasos).

### Encode/Decode — tipos disponibles
| Tipo | Qué hace |
|---|---|
| Plain | texto crudo sin transformar |
| URL | sustituye caracteres por su código ASCII en hex precedido de `%` (ej. `/` → `%2F`) — esencial para requests web |
| HTML | reemplaza caracteres especiales con `&...;` (entidades HTML) — previene XSS al renderizar |
| Base64 | convierte cualquier dato a un formato compatible con ASCII |
| ASCII Hex | convierte cada carácter a su representación hexadecimal (`ASCII` → `4153434949`) |
| Hex / Octal / Binary | solo para inputs numéricos — convierte entre decimal/hex/octal/binario |
| Gzip | comprime/descomprime — el resultado no suele ser ASCII/Unicode válido |

Cada tipo de codificación viene coloreado distinto en la interfaz, para identificar rápido qué transformación se aplicó en cada bloque.

### Hex View
Alternar a vista hexadecimal (arriba de las opciones de decode) permite editar los datos byte por byte — necesario al trabajar con binarios u otros datos no-ASCII.

### Hash
Hashear es de un solo sentido (irreversible) — cualquier input, sin importar el tamaño, produce una firma de tamaño fijo, y cambiar un solo carácter cambia el hash por completo (efecto avalancha). Por eso sirve para verificar integridad de archivos y para guardar contraseñas sin almacenar el texto plano (se compara hash contra hash en cada login).

Flujo: pegar el texto → dropdown **Hash** (el tercero, después de *Decode as*/*Encode as*; la lista de algoritmos es larga) → elegir algoritmo (ej. SHA-256, MD5, MD4) → el resultado son bytes binarios crudos, así que Burp lo muestra automático en **Hex view** → si se necesita el string "clásico" de hash (los caracteres hex de siempre), aplicar **Encode as → ASCII Hex** sobre ese resultado.

> ⚠️ Distinguir entre "el hash en ASCII Hex" y "el hash en Base64" — ambos se aplican sobre los mismos bytes crudos del hash, pero dan resultados totalmente distintos (el string hex tradicional vs. un string más corto con caracteres `+/=`). Confirmar cuál se pide antes de contestar.

> MD4/MD5 están deprecados — útiles para practicar el flujo de Decoder, pero nunca recomendables en producción. Sistemas modernos (OpenSSL) pueden traer MD4 deshabilitado por default por lo inseguro que es.

### Caso práctico — identificar una llave SSH por su hash MD5
```bash
wget http://SITIO:9999/AlteredKeys.zip
mkdir -p AlteredKeysExtracted
unzip -o AlteredKeys.zip -d AlteredKeysExtracted
cd AlteredKeysExtracted
md5sum *
```
`mkdir -p` crea una carpeta dedicada (evita mezclar con archivos ya existentes) · `unzip -o archivo.zip -d carpeta` extrae el contenido a esa carpeta específica (`-d` destino, `-o` sobreescribe sin preguntar) · `md5sum *` calcula el hash de **cada archivo individual** dentro de la carpeta ya extraída.

> ⚠️ Error común: correr `md5sum *` **antes** de extraer el zip, o en un directorio con muchas subcarpetas — `md5sum` no puede hashear directorios (tira `Is a directory` por cada uno), y el `.zip` sin extraer da el hash del archivo comprimido completo, no el de las llaves de adentro.

Buscar en el output la línea cuyo hash coincida exactamente con el buscado — el nombre de archivo junto a ese hash es la llave correcta.

## Organizer — guardar y anotar requests para después
Guarda copias **de solo lectura** de requests HTTP interesantes para revisarlas más tarde o incluirlas en un reporte. Enviar desde cualquier módulo (Proxy, Repeater) con clic derecho → *Send to Organizer*, o `Ctrl+O`.

Cada request guardada aparece en una tabla con: número de índice, hora, estado de flujo de trabajo, módulo de origen, método HTTP, hostname, ruta, query string, cantidad de parámetros, código de estado de la respuesta, tamaño de la respuesta, y notas propias. Request y response son de solo lectura, pero ambos se pueden buscar con la barra de búsqueda.

## Comparer — diferencias entre dos piezas de datos
Compara dos datasets (texto o bytes) y resalta las diferencias — útil, por ejemplo, para comparar dos respuestas de tamaño distinto en un ataque de credential stuffing/brute force (ver [[Web-Pentest-Credential-Stuffing-IDOR-Ejemplo]]) y ver exactamente **qué cambió**, no solo que el tamaño es distinto.

### Interfaz
- Izquierda: tabla de datasets cargados (clic derecho en cualquier módulo → *Send to Comparer*, o Paste/Load manual).
- Arriba a la derecha: **Paste** / **Load** / **Remove** / **Clear**.
- Abajo a la derecha: elegir comparar por **Words** o **Bytes** — se puede cambiar después, no importa cuál se elige primero.

Al comparar, aparece una ventana con:
- Los datos en texto o hex, lado a lado.
- Leyenda de colores (modificado / eliminado / agregado) abajo a la izquierda.
- **Sync views**: si está activo, cambiar un lado a Hex sincroniza el otro automáticamente.
- El título de la ventana muestra el **número total de diferencias** encontradas.

### Uso típico
Capturar una request de login con credenciales inválidas → Send to Repeater → Send → mandar la respuesta a Comparer. Repetir con credenciales distintas (o las correctas) → mandar esa segunda respuesta también a Comparer → comparar por **Words** para ver exactamente qué cambió entre un intento fallido y uno exitoso (más preciso que solo comparar el tamaño en bytes).

## Sequencer — medir la aleatoriedad (entropía) de tokens
Evalúa qué tan aleatorios son tokens (cookies de sesión, tokens CSRF, tokens de reset de contraseña) — si un token no se genera de forma criptográficamente segura, en teoría se pueden predecir valores futuros. Especialmente relevante en tokens con implicaciones de seguridad altas (ej. reset de contraseña).

### Dos formas de analizar
- **Live Capture** (default, la más común): pasar una request que genera un token (ej. un POST de login) → Sequencer manda esa misma request miles de veces automáticamente, guardando cada token generado.
- **Manual Load**: cargar una lista ya existente de tokens pre-generados — evita generar tráfico masivo contra el objetivo, pero requiere ya tener esa lista.

### Flujo de Live Capture
1. Capturar la request que genera el token → *Send to Sequencer*.
2. En **"Token Location Within Response"**, elegir dónde vive el token: **Cookie**, **Form field**, o **Custom location** (ej. seleccionar el campo `loginToken` si es un form field).
3. **Start live capture** — recolecta tokens (~10,000 es un buen mínimo para precisión).
4. **Pause** (no *Stop*, si se quiere poder reanudar después) → **Analyze now**.
5. Opcional: **Auto analyze** repite el análisis cada 2000 requests para ver el progreso en vivo.

### Interpretar el reporte de entropía
- **Overall result**: evaluación general de qué tan segura es la generación del token.
- **Effective entropy**: mide la aleatoriedad real, en bits — más alto es más seguro contra predicción/fuerza bruta (ej. 117 bits se considera alto).
- **Reliability**: nivel de confianza estadística del resultado (ej. significancia del 1% → 99% de confianza).
- **Sample**: detalles de la muestra analizada (cantidad de tokens, características).

Un reporte de entropía es un indicador fuerte, pero no una prueba absoluta — otros factores pueden afectar la seguridad real del token más allá de su aleatoriedad estadística.

## Referencia rápida
| Módulo | Para qué sirve |
|---|---|
| Decoder | codificar/decodificar (URL, HTML, Base64, ASCII Hex, Gzip...) y generar hashes |
| Organizer | guardar/anotar requests de solo lectura para después |
| Comparer | ver diferencias exactas (palabra o byte) entre dos piezas de datos |
| Sequencer | medir la entropía/aleatoriedad de tokens (cookies, CSRF, reset tokens) |
