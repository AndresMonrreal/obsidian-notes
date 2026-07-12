# John the Ripper — Crackeo de hashes

Relacionado: [[Hashing]] · [[Criptografia]] · [[Ofensiva-Pentesting]]

## Qué es
John the Ripper ("John") es una herramienta para **crackear hashes offline** mediante ataques de fuerza bruta y diccionario. La versión más completa y recomendada es **Jumbo John** (incluye utilidades extra como `zip2john`, `rar2john`, `ssh2john`, `unshadow`).

Verificar versión:
```bash
john
# Primera línea: John the Ripper 1.9.0-jumbo-1 ...
```

## Wordlist estándar
`rockyou.txt` (de la brecha de rockyou.com, 2009). Ubicación en Kali/AttackBox:
```bash
/usr/share/wordlists/rockyou.txt
# Si está comprimida:
tar xvzf rockyou.txt.tar.gz
```

## Sintaxis básica
```bash
john [opciones] [archivo]
```

### Modo diccionario (wordlist)
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```

### Crackeo automático (John detecta el tipo)
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```
No siempre es fiable; mejor identificar el hash primero.

### Especificar formato
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```
> Nota: para hashes estándar (md5, sha1, sha256...) suele necesitarse el prefijo `raw-`.

Listar / buscar formatos disponibles:
```bash
john --list=formats
john --list=formats | grep -iF "md5"
```

## Identificar un hash
```bash
wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py
python3 hash-id.py
# Pega el hash y te da los formatos probables
```

## Single Crack Mode (mangling desde el username)
Genera candidatos mutando el nombre de usuario (Markus → Markus1, MArkus, Markus!, etc.).
```bash
john --single --format=raw-sha256 hashes.txt
```
El archivo debe llevar el usuario delante del hash:
```
mike:1efee03cdcb96d90ad48ccc7b8666033
```

## Custom Rules (reglas de mangling)
Definidas en `john.conf` (`/opt/john/john.conf` en AttackBox, o `/etc/john/john.conf`).
```
[List.Rules:PoloPassword]
cAz"[0-9] [!£$%@]"
```
- `c` → capitaliza la primera letra
- `Az` → añade caracteres al final
- `A0` → añade caracteres al inicio
- `[0-9]`, `[A-Z]`, `[a-z]`, `[!£$%@]` → conjuntos de caracteres

Llamar la regla:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rule=PoloPassword hash.txt
```
Explotan la **predictibilidad de la complejidad** (mayúscula al inicio, número + símbolo al final, ej. `Polopassword1!`).

## Herramientas de conversión "2john"
Convierten archivos protegidos a un formato que John entiende. Luego se crackea igual.

### /etc/shadow (unshadow)
```bash
unshadow local_passwd local_shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```

### ZIP protegido
```bash
zip2john secure.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```

### RAR protegido
```bash
/opt/john/rar2john secure.rar > rar_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
```

### Clave privada SSH (id_rsa)
```bash
/opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
# o: python3 /opt/john/ssh2john.py
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```

## NTLM (contraseñas Windows)
Formato usado por Windows moderno (SAM / NTDS.dit). Se obtiene con mimikatz o volcando la SAM.
```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt
```

## Recuperar contraseña ya crackeada
```bash
john --show hash.txt
```
