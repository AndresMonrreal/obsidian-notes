# Bases de datos y SQL — Fundamentos

Relacionado: [[Arquitectura-Web]] · [[HTTP-Web]] · [[Ofensiva-Pentesting]]

## Qué es una base de datos
Colección organizada de información estructurada, fácilmente accesible y manipulable. Ejemplos: credenciales de login, posts de redes sociales, historial de visualización.

## Tipos de bases de datos
- **Relacionales (SQL)**: datos **estructurados** en tablas (filas y columnas). Relaciones entre tablas. Ideal cuando el formato es consistente y la precisión importa (ej. e-commerce). DBMS: MySQL, MariaDB, Oracle, PostgreSQL.
- **No relacionales (NoSQL)**: formato **no tabular** (documentos, clave-valor). Ideal cuando los datos varían mucho de formato (ej. redes sociales). DBMS: MongoDB.

## Tablas, filas y columnas
- **Tabla**: colección de datos (ej. `Books`).
- **Columna**: campo con un tipo de dato definido (String, Integer, Float/Decimal, Date/Time).
- **Fila (row)**: un registro insertado.

## Claves
- **Primary Key**: identifica de forma **única** cada registro. Solo una por tabla (ej. `id`).
- **Foreign Key**: columna que existe en otra tabla, crea el **vínculo** entre tablas. Puede haber varias.

## DBMS y acceso
El **DBMS** es la interfaz entre el usuario y la base de datos. Se interactúa con **SQL** (Structured Query Language).

Conectar a MySQL:
```bash
mysql -u root -p
# Enter password: tryhackme
```

## Sentencias de BASE DE DATOS
```sql
CREATE DATABASE thm_bookmarket_db;   -- crear
SHOW DATABASES;                      -- listar
USE thm_bookmarket_db;               -- seleccionar activa
DROP DATABASE database_name;         -- eliminar
```

## Sentencias de TABLA
```sql
CREATE TABLE book_inventory (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    book_name VARCHAR(255) NOT NULL,
    publication_date DATE
);

SHOW TABLES;                 -- listar tablas
DESCRIBE book_inventory;     -- ver columnas y tipos (o DESC)
ALTER TABLE book_inventory ADD page_count INT;   -- modificar
DROP TABLE table_name;       -- eliminar
```

## Operaciones CRUD
| Operación | Sentencia |
|-----------|-----------|
| Create | `INSERT` |
| Read | `SELECT` |
| Update | `UPDATE` |
| Delete | `DELETE` |

```sql
-- CREATE
INSERT INTO books (id, name, published_date, description)
VALUES (1, "Android Security Internals", "2014-10-14", "An In-Depth Guide...");

-- READ
SELECT * FROM books;
SELECT name, description FROM books;

-- UPDATE
UPDATE books SET description = "..." WHERE id = 1;

-- DELETE
DELETE FROM books WHERE id = 1;
```

## Cláusulas
```sql
SELECT DISTINCT name FROM books;              -- valores únicos (sin duplicados)

SELECT name, COUNT(*) FROM books
GROUP BY name;                                -- agrupar

SELECT * FROM books ORDER BY published_date ASC;   -- ordenar ascendente
SELECT * FROM books ORDER BY published_date DESC;  -- descendente

SELECT name, COUNT(*) FROM books
GROUP BY name
HAVING name LIKE '%Hack%';                    -- filtrar tras agrupar
```
- `WHERE` filtra **antes** de agrupar; `HAVING` filtra **después**.

## Operadores

### Lógicos
```sql
WHERE description LIKE "%guide%";              -- patrón (% comodín)
WHERE category = "X" AND name = "Y";          -- ambas verdaderas
WHERE name LIKE "%Android%" OR name LIKE "%iOS%";  -- al menos una
WHERE NOT description LIKE "%guide%";          -- niega
WHERE id BETWEEN 2 AND 4;                       -- rango inclusivo
```

### Comparación
```sql
WHERE name = "Designing Secure Software";   -- igual
WHERE category != "Offensive Security";     -- distinto
WHERE published_date < "2020-01-01";        -- menor que
WHERE published_date > "2020-01-01";        -- mayor que
WHERE published_date <= "2021-11-15";       -- menor o igual
WHERE published_date >= "2021-11-02";       -- mayor o igual
```

## Funciones

### De cadena (string)
```sql
SELECT CONCAT(name, " is a type of ", category, " book.") AS book_info FROM books;
SELECT category, GROUP_CONCAT(name SEPARATOR ", ") AS books FROM books GROUP BY category;
SELECT SUBSTRING(published_date, 1, 4) AS published_year FROM books;   -- primeros 4 chars
SELECT LENGTH(name) AS name_length FROM books;                        -- nº de caracteres
```

### De agregación
```sql
SELECT COUNT(*) AS total_books FROM books;      -- contar filas
SELECT SUM(price) AS total_price FROM books;    -- sumar
SELECT MAX(published_date) AS latest_book FROM books;    -- máximo
SELECT MIN(published_date) AS earliest_book FROM books;  -- mínimo
```

## Relevancia para seguridad
Entender SQL es la base para atacar/defender **SQL Injection (SQLi)**. La validación insegura de query strings o inputs de formularios (ver [[HTTP-Web]]) permite inyectar código malicioso. Herramientas como Burp Suite ([[Herramientas-Burp-Suite]]) ayudan a probar estos endpoints.
