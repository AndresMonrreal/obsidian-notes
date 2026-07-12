# Hashing — Conceptos y comandos

Relacionado: [[Criptografia]] · [[Herramientas-John-the-Ripper]] · [[Representacion-de-Datos]]

## Qué es una función hash
Toma datos de **cualquier tamaño** y produce un resumen (digest) de **tamaño fijo**. Propiedades:
- Es **unidireccional**: fácil calcular el hash, computacionalmente inviable revertirlo.
- **Efecto avalancha**: cambiar un solo bit de entrada cambia radicalmente la salida.
- No usa clave (a diferencia del cifrado).

Fundamento teórico: problema **P vs NP**. Calcular el hash es "P" (rápido); revertirlo sería "NP" (intratable).

## Algoritmos comunes
| Algoritmo | Tamaño salida |
|-----------|---------------|
| MD5 | 16 bytes (128 bits) |
| SHA-1 | 20 bytes (160 bits) |
| SHA-256 | 32 bytes (256 bits) |
| NTLM | variante de MD4 |

> MD5 y SHA-1 están **rotos** (se pueden fabricar colisiones). No usarlos para contraseñas ni integridad seria.

## Colisiones (hash collision)
Dos entradas distintas producen la misma salida. Inevitables por el **efecto palomar** (pigeonhole): más entradas posibles que salidas. Un buen algoritmo hace que la probabilidad sea despreciable.
- Salida de N bits → `2^N` valores posibles. Ej: 8 bits → 256 valores.

## Comandos para calcular hashes (Linux)
```bash
md5sum archivo.txt
sha1sum archivo.txt
sha256sum archivo.txt
sha512sum archivo.txt
```
Salida en **hexadecimal** (cada byte = 2 dígitos hex).

Ver diferencia de un bit entre dos archivos:
```bash
hexdump -C file1.txt   # T = 0x54 = 01010100
hexdump -C file2.txt   # U = 0x55 = 01010101
md5sum *.txt           # hashes completamente distintos
```

## Usos del hashing

### 1. Almacenamiento de contraseñas
El servidor **no guarda la contraseña**, guarda su hash. Al iniciar sesión, hashea lo que envías y compara.

Malas prácticas históricas (brechas reales):
- Texto plano (RockYou).
- Cifrado deprecado + pistas en claro (Adobe).
- Hash inseguro sin salt (LinkedIn, SHA-1).

### 2. Verificación de integridad de archivos
```bash
sha256sum archivo.iso   # comparar contra el checksum publicado
```
Si coincide, el archivo es idéntico al original.

## Salt (sal)
Valor aleatorio **único por usuario** que se concatena a la contraseña antes de hashear. Evita rainbow tables y que dos usuarios con la misma contraseña tengan el mismo hash. No necesita ser secreto.

Flujo seguro:
1. Elegir función segura: **Argon2, scrypt, bcrypt o PBKDF2**.
2. Generar salt único (`Y4UV*^(=go_!`).
3. Concatenar: `AL4RMc10k` + salt → `AL4RMc10kY4UV*^(=go_!`.
4. Hashear la combinación.
5. Guardar hash + salt.

> Bcrypt/scrypt manejan el salt automáticamente. bcrypt además resiste GPUs.

## Rainbow Tables
Tabla de búsqueda hash→texto plano. Rápida pero solo funciona con hashes **sin salt**. Servicios: CrackStation, Hashes.com.

## HMAC (Keyed-Hash Message Authentication Code)
Combina función hash + clave secreta para verificar **autenticidad** (clave) e **integridad** (hash).
```
HMAC(K,M) = H((K⊕opad) || H((K⊕ipad) || M))
```

## Prefijos de hash en /etc/shadow (Linux)
Formato: `$prefijo$opciones$salt$hash`
| Prefijo | Algoritmo |
|---------|-----------|
| `$y$` | yescrypt (recomendado, moderno) |
| `$gy$` | gost-yescrypt |
| `$7$` | scrypt |
| `$2b$ $2y$ $2a$` | bcrypt |
| `$6$` | sha512crypt |
| `$1$` | md5crypt |

Ver hashes (requiere root):
```bash
sudo cat /etc/shadow | grep usuario
man 5 crypt   # más info de prefijos
```

## Crackeo con Hashcat
Sintaxis: `hashcat -m <tipo> -a <modo> hashfile wordlist`
- `-m` → tipo de hash numérico (ej. `-m 1000` = NTLM, `-m 3200` = bcrypt)
- `-a 0` → modo diccionario (straight)

```bash
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
Modos útiles: NTLM `1000`, bcrypt `3200`, SHA2-256 `1400`, HMAC-SHA512(pass) `1750`, Cisco-ASA MD5 `2410`.

> GPUs aceleran mucho el crackeo. Las VMs normalmente no tienen acceso a la GPU del host, así que Hashcat corre mejor en el host. John the Ripper usa CPU por defecto y funciona bien en VM. Ver también [[Herramientas-John-the-Ripper]].
