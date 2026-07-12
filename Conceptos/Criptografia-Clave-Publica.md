# Criptografía de clave pública (asimétrica)

Relacionado: [[Criptografia]] · [[Hashing]] · [[Seguridad-de-Red]] · [[Linux-Herramientas-y-Admin]]

## Idea central
El cifrado **asimétrico** usa un par de claves: **pública** (cifra, se comparte) y **privada** (descifra, secreta). Es lento comparado con el simétrico, así que se usa una vez para **negociar** una clave simétrica y luego se comunica con cifrado simétrico (rápido).

Analogía de la caja con candado:
| Analogía | Sistema criptográfico |
|----------|----------------------|
| Código secreto | Cifrado y clave simétrica |
| Candado | Clave pública |
| Llave del candado | Clave privada |

## RSA
Algoritmo de clave pública para transmisión segura por canales inseguros. Su seguridad se basa en que **factorizar un número grande es difícil**, aunque multiplicar dos primos grandes es fácil.

Variables clave (aparecen mucho en CTFs):
- `p`, `q` → primos grandes
- `n = p × q`
- `ϕ(n) = (p-1)(q-1)` → también `n − p − q + 1`
- Clave pública = `(n, e)`
- Clave privada = `(n, d)`
- `m` → mensaje (plaintext), `c` → cifrado (ciphertext)

Cifrar / descifrar:
```
y = x^e mod n     (cifrar)
x = y^d mod n     (descifrar)
```

Herramientas para CTFs RSA: **RsaCtfTool**, **rsatool**.

## Diffie-Hellman (intercambio de claves)
Permite establecer una **clave compartida** sobre un canal inseguro sin transmitirla. Se usa junto a RSA: DH para acuerdo de clave, RSA para firmas/autenticación.

Proceso:
1. Público: primo `p` y generador `g`.
2. Cada parte elige un privado (`a`, `b`).
3. Claves públicas: `A = g^a mod p`, `B = g^b mod p`.
4. Intercambian A y B.
5. Secreto compartido: `B^a mod p = A^b mod p = g^(ab) mod p`.

## Firmas digitales y certificados
- **Firma digital**: se firma con la clave privada, se verifica con la pública. Prueba autenticidad e integridad. En la práctica se firma el **hash** del documento.
- **Certificados**: prueban identidad (ej. HTTPS). Cadena de confianza que empieza en una **Root CA**. TLS gratis con **Let's Encrypt**.

## SSH — claves

### Generar par de claves
```bash
ssh-keygen -t ed25519
# También: -t rsa | ecdsa | dsa
```
- Clave pública: `id_ed25519.pub`
- Clave privada: `id_ed25519` (¡NUNCA compartir!)

Algoritmos: DSA, ECDSA, ECDSA-SK, **Ed25519** (recomendado), Ed25519-SK. Por defecto SSH usa RSA.

### Copiar clave pública al servidor
```bash
ssh-copy-id user@host
```
La clave pública autorizada vive en `~/.ssh/authorized_keys`.

### Usar una clave privada para conectar
```bash
chmod 600 privateKey        # permisos correctos (solo el dueño lee/escribe)
ssh -i privateKey user@host
```

### Verificar servidor (fingerprint)
Al conectar por primera vez, SSH pide confirmar el fingerprint de la clave pública del servidor. Previene ataques **man-in-the-middle**. Se guarda en `~/.ssh/known_hosts`.

### Crackear passphrase de clave SSH
Ver [[Herramientas-John-the-Ripper]]:
```bash
ssh2john id_rsa > hash.txt
john --wordlist=rockyou.txt hash.txt
```

## GPG / PGP
PGP (Pretty Good Privacy) cifra archivos, correos y firma digitalmente. **GnuPG (GPG)** es la implementación open-source del estándar OpenPGP.

### Generar par de claves
```bash
gpg --full-gen-key
# Elegir algoritmo (ECC/RSA), curva, expiración, nombre, email
```

### Importar clave y descifrar
```bash
gpg --import backup.key
gpg --decrypt confidential_message.gpg
```

### Crackear passphrase de clave GPG
```bash
gpg2john  # convierte para John, luego crackear con rockyou.txt
```

> Nunca compartas claves privadas. La passphrase solo descifra tu clave privada localmente; nunca se transmite ni te identifica ante el servidor.
