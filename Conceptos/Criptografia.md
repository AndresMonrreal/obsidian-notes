---
tags:
  - pre-security
  - criptografia
  - cifrado
  - simetrico
  - asimetrico
  - TLS
  - HTTPS
  - certificados
  - CIA-triad
fecha: 2026-06-27
ruta: SEC0
fuente: TryHackMe
---

# 🔑 Criptografía — Cifrado Simétrico y Asimétrico

---

## Conceptos Fundamentales

| Término | Definición |
|---------|-----------|
| **Plaintext** | El mensaje original, legible. Ej: `HELLO` o `Nombre: Alice Smith` |
| **Ciphertext** | El mensaje cifrado, ilegible sin la clave. Ej: `KHOOR` |
| **Key (clave)** | El secreto que controla cómo se cifra y descifra. Como una contraseña que usa el algoritmo |
| **Algorithm (algoritmo)** | La receta pública de pasos para cifrar/descifrar. La seguridad NO viene del algoritmo (es público), sino de mantener la clave secreta |

### El proceso

```
Cifrar:   Plaintext + Algoritmo + Clave  →  Ciphertext
Descifrar: Ciphertext + Algoritmo + Clave →  Plaintext
```

> [!example] Analogía de la caja fuerte
> - El **algoritmo** = cómo funciona la cerradura (público, cualquiera puede verlo)
> - La **clave** = tu llave específica (secreta, solo tú la tienes)
> - El **plaintext** = la carta dentro de la caja
> - El **ciphertext** = la caja cerrada viajando por correo
>
> Nadie trata de ocultar cómo funcionan los candados para hacerlos seguros — la seguridad está en mantener tu llave privada.

---

## Cifrado Simétrico

**Una sola clave** para cifrar y descifrar. El mismo secreto lo tienen ambas partes.

```
Alice ──[clave K]──→ cifra → Ciphertext → descifra ←[clave K]── Bob
```

### El Cifrado César — El ejemplo didáctico más simple

Inventado por Julio César hace 2000+ años. Desplaza cada letra del mensaje N posiciones en el alfabeto. N es la clave.

**Con clave = 3:**
```
Cifrar HELLO:
H→K  E→H  L→O  L→O  O→R  →  KHOOR

Descifrar KHOOR (desplazar -3):
K→H  H→E  O→L  O→L  R→O  →  HELLO ✅
```

**¿Por qué no es seguro?** Solo hay 25 claves posibles (1 a 25). Un humano puede probarlas todas en minutos. Una computadora en milisegundos. Esta técnica se llama **brute force**.

> [!warning] El César solo es pedagógico
> Nunca se usa en sistemas reales. Se estudia para entender el concepto de clave + algoritmo. En la práctica se usa **AES (Advanced Encryption Standard)**.

### Ventajas y desventajas del cifrado simétrico

| ✅ Ventajas | ❌ Desventajas |
|------------|---------------|
| Muy rápido — procesa grandes volúmenes de datos | **Problema de distribución de claves**: ¿cómo compartes la clave de forma segura? |
| Eficiente para cifrar archivos, discos, tráfico de red | Si interceptan la clave, pueden descifrar todo |
| Algoritmos modernos (AES) son extremadamente seguros | Ambas partes deben tener la misma clave antes de comunicarse |

### El problema de distribución de claves

Si Alice le envía la clave a Bob por internet sin cifrar, un atacante la intercepta y puede leer todos los mensajes. ¿Pero si cifra la clave? Necesitaría otra clave para cifrarla, y así al infinito. Este es el **key distribution problem** — el talón de Aquiles del cifrado simétrico cuando se usa solo.

---

## Cifrado Asimétrico

La solución al problema de distribución de claves: **dos claves matemáticamente enlazadas**.

| Clave | Quién la tiene | Uso |
|-------|---------------|-----|
| **Public Key** (clave pública) | Cualquiera puede conocerla y usarla | Cifrar mensajes para el dueño |
| **Private Key** (clave privada) | Solo el dueño, nunca se comparte | Descifrar mensajes cifrados con su pública |

### La regla fundamental

```
Cifrado con Public Key  → Solo la Private Key puede descifrar
Cifrado con Private Key → Cualquiera con la Public Key puede descifrar
                          (se usa para firmas digitales)
```

Las dos claves están enlazadas matemáticamente, pero es computacionalmente imposible derivar la privada de la pública (tardaría cientos o miles de años con computadoras convencionales).

> [!example] Analogía del buzón de correo
> - **La ranura del buzón** = Public Key → cualquiera puede meter cartas
> - **La llave de la puerta** = Private Key → solo el dueño puede sacar las cartas
>
> Bob publica su ranura (public key) en internet. Alice mete su mensaje cifrado. Solo Bob puede abrirlo con su llave privada. Aunque un atacante intercepte el sobre, no puede abrirlo sin la llave.

### Flujo de comunicación asimétrica

```
1. Bob genera su par de claves (pública + privada)
2. Bob publica su clave pública (en su web, en un key server)
3. Alice descarga la clave pública de Bob
4. Alice cifra su mensaje con la clave pública de Bob
5. Alice envía el mensaje cifrado (ciphertext) por internet
6. Bob descifra con su clave privada → lee el mensaje ✅

En ningún momento viajó ningún secreto por la red.
```

---

## HTTPS — Cifrado Simétrico + Asimétrico juntos

HTTPS usa los dos tipos de cifrado en un enfoque híbrido:

```
1. Tu navegador pide la public key del servidor
2. El servidor la envía en un certificado digital
3. [Asimétrico] Se negocia una clave de sesión temporal sin exponerla
4. [Simétrico] A partir de ahí, toda la comunicación usa esa clave de sesión
                → rápido y eficiente para el tráfico masivo
```

**¿Por qué híbrido?**
- El cifrado asimétrico es **lento** — solo se usa para intercambiar la clave de sesión
- El cifrado simétrico es **rápido** — toma el relevo para los datos reales

---

## Certificados Digitales y Certificate Authority (CA)

**Problema:** ¿Cómo sabe Alice que la clave pública que recibió realmente pertenece a Bob y no a un atacante?

**Solución:** Los **certificados digitales**.

### ¿Qué es un certificado?

Un documento digital que contiene:
- La **clave pública** del sitio/entidad
- **A quién pertenece** (ej: `tryhackme.com`)
- **Quién lo firma** — una **Certificate Authority (CA)** de confianza
- **Fechas de validez** (válido desde / hasta)

### Certificate Authority (CA)

Entidades de confianza que firman certificados. Tu navegador y OS vienen preinstalados con una lista de CAs de confianza (ej: DigiCert, Let's Encrypt, Comodo).

```
Cuando visitas https://tryhackme.com:
1. El servidor envía su certificado
2. Tu navegador verifica: ¿firmó este cert una CA de confianza? ✅
3. Tu navegador verifica: ¿está vigente (no expirado)? ✅
4. Tu navegador muestra el 🔒 y confía en la public key
5. Comienza el handshake TLS
```

Si algo falla → el navegador muestra advertencia y puede rechazar la conexión.

### Cómo ver un certificado en el navegador

1. Ve a cualquier sitio HTTPS (ej: `https://tryhackme.com`)
2. Haz clic en el 🔒 de la barra de direcciones
3. Busca "Certificado" o "Connection is secure" → "View certificate"
4. Verás: emisor, a quién se emitió, fechas de validez

---

## Simétrico vs Asimétrico — Comparación

| | **Simétrico** | **Asimétrico** |
|-|--------------|----------------|
| **Número de claves** | 1 (la misma para cifrar y descifrar) | 2 (pública + privada) |
| **Compartir claves** | Ambos deben tener la misma clave secreta | La pública se puede compartir libremente |
| **Velocidad** | ⚡ Muy rápido | 🐢 Más lento |
| **Uso principal** | Cifrar grandes volúmenes de datos | Intercambio de claves, certificados |
| **Problema** | Distribución de claves | — |
| **Algoritmos** | AES, ChaCha20, DES (obsoleto) | RSA, ECC, Diffie-Hellman |
| **Analogía** | Una llave abre y cierra la misma caja | Buzón: cualquiera mete, solo el dueño saca |

### En la práctica: los sistemas usan ambos

```
VPNs     → Asimétrico para autenticar y negociar clave → Simétrico para el tráfico
HTTPS    → Asimétrico para el TLS handshake           → Simétrico para los datos
WhatsApp → Asimétrico para compartir claves           → Simétrico para los mensajes
```

---

## Conexión con CIA Triad

| Principio CIA | Cómo lo protege la criptografía |
|---------------|---------------------------------|
| **Confidentiality** | Cifrado — nadie sin la clave puede leer los datos |
| **Integrity** | Firmas digitales y hashing — cualquier modificación es detectable |
| **Availability** | TLS garantiza que la conexión sea con el servidor legítimo (evita ataques que interrumpirían el servicio) |

---

## Para Certificaciones

| Certificación | Temas relevantes de esta nota |
|---------------|-------------------------------|
| **Security+** | Symmetric vs Asymmetric, PKI, Certificates, CA, TLS/HTTPS |
| **Network+** | HTTPS, TLS, cómo funcionan los certificados |
| **CEH** | Criptografía como vector de ataque/defensa, PKI |
| **CISSP** | Dominio de Criptografía completo |

---

## TLS — Historia y Evolución

En los primeros días de Internet, cualquier atacante en la misma red podía poner su tarjeta de red en **modo promiscuo** y capturar todos los packets — incluyendo contraseñas y chats en texto plano. No había nada que el usuario pudiera hacer.

| Año | Hito |
|-----|------|
| **1995** | Netscape Communications lanza **SSL 2.0** — primera versión pública de SSL (Secure Sockets Layer) |
| **1996** | SSL 3.0 — versión revisada con mejoras de seguridad |
| **1999** | IETF publica **TLS 1.0** (Transport Layer Security) — sucesor estándar de SSL 3.0 con mejoras en cifrado y autenticación |
| **2018** | **TLS 1.3** — revisión mayor: elimina algoritmos inseguros, handshake más rápido, mejor seguridad |

> [!info] SSL vs TLS
> SSL y TLS hacen lo mismo: proveer cifrado y autenticación en la capa de transporte del modelo OSI. TLS es el sucesor estándar de SSL. En la práctica, cuando alguien dice "SSL" hoy en día casi siempre se refiere a TLS. Los certificados a veces todavía se llaman "SSL certificates" aunque el protocolo sea TLS.

TLS opera en la **capa de Transporte (Layer 4) del modelo OSI** y garantiza **confidencialidad e integridad** de las comunicaciones entre cliente y servidor sobre una red insegura.

---

## TLS — Cómo Añade Seguridad a Cualquier Protocolo

Una de las características más poderosas de TLS es que puede añadirse encima de casi cualquier protocolo de aplicación sin modificar las capas inferiores (TCP/IP no se toca) ni superiores (HTTP sigue siendo HTTP):

| Protocolo inseguro | Protocolo seguro (con TLS) | Puerto seguro |
|--------------------|---------------------------|---------------|
| HTTP | **HTTPS** | 443 |
| SMTP | **SMTPS** | 465 / 587 |
| POP3 | **POP3S** | 995 |
| IMAP | **IMAPS** | 993 |
| DNS | **DoT** (DNS over TLS) | 853 |
| FTP | **FTPS** | 990 / 21 |
| SIP (VoIP) | **SIPS** | 5061 |
| MQTT (IoT) | **MQTTS** | 8883 |

La "S" appended siempre significa Secure gracias a TLS.

> [!warning] Sin TLS = texto plano capturado
> Si capturas tráfico en una red donde se usa SMTP (port 25) o POP3 (port 110) sin TLS, puedes extraer **credenciales de login en claro**. Con SMTPS o IMAPS el tráfico aparece como datos cifrados ilegibles.

---

## Proceso de Obtención de un Certificado TLS

Para que un servidor pueda usar TLS, necesita un certificado firmado. El proceso estándar:

```
1. El administrador del servidor crea una CSR
   (Certificate Signing Request — contiene la public key y datos del dominio)
            │
            ▼
2. Se envía la CSR a una CA (Certificate Authority)
   La CA verifica que el solicitante realmente controla el dominio
            │
            ▼
3. La CA firma el certificado con su private key
   → Certificado digital emitido: public key + identidad + firma CA
            │
            ▼
4. El servidor instala el certificado firmado
   → Ahora puede identificarse ante cualquier cliente
            │
            ▼
5. El cliente (navegador) conecta al servidor
   → Recibe el certificado → verifica la firma de la CA
   → Si la CA está en su lista de confianza → 🔒 conexión segura
```

**CAs de confianza conocidas:** DigiCert, Let's Encrypt (gratuita), Comodo, GlobalSign, Sectigo

> [!tip] Let's Encrypt
> [Let's Encrypt](https://letsencrypt.org/) permite obtener certificados TLS **gratuitos** y se renueva automáticamente cada 90 días. Es el estándar para sitios personales y proyectos open source.

### Certificados Auto-firmados (Self-signed)

Un certificado auto-firmado es uno que **no está firmado por ninguna CA** — el dueño lo firma con su propia clave privada.

- ❌ **No prueba la autenticidad del servidor** — cualquiera puede crear uno para cualquier dominio
- ❌ El navegador muestra advertencia de seguridad (NET::ERR_CERT_AUTHORITY_INVALID)
- ✅ Útil en entornos internos/labs donde no se necesita autenticidad pública
- ⚠️ Nunca usar en producción para sitios públicos

> [!warning] Certificados self-signed en pentesting
> En pentesting o CTFs, es común encontrar servidores con certificados self-signed. No es un error de seguridad crítico por sí solo, pero indica que el dominio no fue verificado por una CA de confianza. Siempre revisa el certificado con `openssl s_client -connect dominio:443`.

---

## HTTPS — Tres Pasos vs Dos Pasos

La diferencia concreta entre HTTP y HTTPS es la adición del paso TLS:

**HTTP (inseguro):**
```
1. TCP Three-Way Handshake    (SYN → SYN/ACK → ACK)
2. HTTP application data      (GET / HTTP/1.1 → en texto plano)
```

**HTTPS (con TLS):**
```
1. TCP Three-Way Handshake    (SYN → SYN/ACK → ACK)
2. TLS Handshake              (negociación de cifrado, intercambio de certificado, acuerdo de clave de sesión)
3. HTTP application data      (GET / HTTP/1.1 → cifrado, ilegible sin la clave)
```

En Wireshark, el paso 3 aparece como "Application Data" — el analizador sabe que es HTTPS pero no puede ver el contenido sin la clave de descifrado.

> [!info] La clave de sesión
> Después del TLS handshake, ambos lados tienen una clave de sesión simétrica compartida. Todo el tráfico HTTP real se cifra con esa clave. Sin acceso a esa clave, capturar los packets solo te da bytes aleatorios.

---

## Notas relacionadas

- [[CIA-Triad]] — La criptografía es la herramienta principal para Confidentiality e Integrity
- [[HTTP-Web]] — HTTPS = HTTP + TLS (cifrado asimétrico + simétrico)
- [[Seguridad-de-Red]] — VPN usa IPSec y otros protocolos criptográficos; SSH como protocolo seguro
- [[Representacion-de-Datos]] — Los algoritmos cifran sobre bits y bytes
- [[Protocolos-Aplicacion]] — SMTPS/POP3S/IMAPS: email sobre TLS; SSH como sustituto seguro de TELNET
