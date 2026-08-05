# TLS/SSL — Fundamentos y funcionamiento

Relacionado: [[Ataques-Sniffing-MITM]] · [[Protocolos-Aplicacion]] · [[Criptografia-Clave-Publica]] · [[HTTP-Peticiones-y-Respuestas]] · [[OSI-Model]]

## Historia rápida
- **1994** — Netscape introduce **SSL** (Secure Sockets Layer).
- **1996** — SSL 3.0.
- **1999** — nace **TLS** (Transport Layer Security) 1.0, sucesor de SSL.
- **2008** — TLS 1.2, ampliamente usado y seguro si está bien configurado.
- **2018** — TLS 1.3, el estándar actual.
- **2021** — TLS 1.0 y 1.1 deprecados (vulnerabilidades conocidas); navegadores mayores ya no los soportan.

> SSL 2.0/3.0 están **deprecados e inseguros** — nunca deben usarse. Coloquialmente la gente sigue diciendo "certificado SSL" pero en la práctica todos los sistemas modernos usan **TLS**.

## Dónde encaja TLS
Los protocolos cubiertos en [[Protocolos-Aplicacion]] van en texto plano en la capa de aplicación. TLS añade cifrado — conceptualmente en la capa de presentación del modelo OSI ([[OSI-Model]]), aunque en la práctica TLS opera entre la capa de transporte y la de aplicación, no encaja perfectamente en una sola capa OSI.

## Tabla de protocolos: antes y después de TLS
| Protocolo | Puerto default | Versión segura | Puerto con TLS |
|-----------|-----------------|-----------------|------------------|
| HTTP | 80 | **HTTPS** | 443 |
| FTP | 21 | **FTPS** | 990 |
| SMTP | 25 | **SMTPS** | 465 |
| POP3 | 110 | **POP3S** | 995 |
| IMAP | 143 | **IMAPS** | 993 |
| DNS | 53 | **DoT** (DNS over TLS) | 853 |
| DNS | 53 | **DoH** (DNS over HTTPS) | 443 |

DoT envuelve las consultas DNS estándar dentro de una conexión TLS; DoH las envía como peticiones HTTPS. Ambos evitan que alguien vea qué sitios visita el usuario.

## Implicit TLS vs STARTTLS
| Método | Cómo funciona |
|--------|---------------|
| **Implicit TLS** | puerto dedicado; la conexión está cifrada **desde el primer byte** (ej. 443, 993) |
| **STARTTLS** | el cliente conecta en el puerto normal (texto plano) y luego emite el comando `STARTTLS` para **actualizar** la conexión a TLS en el mismo puerto (común en email; puerto 587 para SMTP submission) |

> **Implicit TLS es preferible.** STARTTLS es vulnerable a **ataques de downgrade**: un atacante MITM puede eliminar el comando `STARTTLS` de la comunicación, forzando que la conexión se quede en texto plano.

## Cómo funciona HTTPS (pasos)
HTTP normal: 1) conexión TCP → 2) peticiones HTTP (GET/POST). HTTPS añade un paso: 1) conexión TCP → 2) **handshake TLS** → 3) peticiones HTTP (ya cifradas).

## TLS Handshake (TLS 1.2, simplificado)
1. **ClientHello** — el cliente envía versiones TLS soportadas, cipher suites, y un valor aleatorio.
2. **ServerHello** — el servidor responde con los parámetros elegidos, su **certificado** (firmado por una CA), y su propio valor aleatorio.
3. **Key Exchange** — cliente y servidor intercambian info para generar la clave secreta compartida (el proceso exacto depende de la cipher suite).
4. **Finished** — ambos confirman que el handshake terminó bien y cambian a comunicación cifrada.

### Mejoras de TLS 1.3
- **Handshake más rápido**: solo 1 round-trip (1-RTT) vs 2 en TLS 1.2. Soporta 0-RTT para clientes que vuelven a conectar (con algunos trade-offs de seguridad).
- **Forward secrecy por defecto**: si la clave privada del servidor se compromete en el futuro, las sesiones grabadas en el pasado siguen indescifrables.
- **Cipher suites simplificadas**: se eliminaron algoritmos inseguros/obsoletos.
- **Handshake más cifrado**: menos información visible para un observador.

**Idea central**: cliente y servidor acuerdan una clave secreta que un tercero observando el canal no puede descubrir. Todo lo que sigue viaja cifrado con esa clave.

## Certificados y confianza
El certificado prueba que el servidor es quien dice ser. Datos visibles en un certificado: **a quién** se emitió, **quién** lo emitió (la CA), y el **periodo de validez** (un certificado expirado no debe confiarse). El navegador hace toda esta verificación automáticamente.

### Ecosistema moderno de certificados
- **Let's Encrypt** (2015) — certificados TLS gratis y automatizados; eliminó la barrera de costo que frenaba la adopción de HTTPS (de <50% de tráfico web en 2015 a >95% hoy).
- **Certificate Transparency (CT)** — las CAs deben loguear todo certificado emitido en logs públicos auditables; los navegadores rechazan certificados no logueados correctamente.
- **Certificados de vida corta** — Let's Encrypt emite certificados válidos solo **90 días**, fomentando automatización y reduciendo la ventana de exposición si se comprometen.
- **ACME** (Automated Certificate Management Environment) — protocolo que usa Let's Encrypt para automatizar emisión/renovación. Herramienta común: **Certbot**.

## Herramientas para evaluar configuraciones TLS
| Herramienta | Uso |
|-------------|-----|
| **testssl.sh** | CLI, evaluación detallada de protocolos/cipher suites/vulnerabilidades — mejor opción para sistemas internos no públicos |
| **sslyze** | herramienta Python, útil para automatización/CI-CD |
| **SSL Labs** (ssllabs.com) | servicio web para análisis rápido de servidores HTTPS públicos |
| **nmap ssl-enum-ciphers** | script de Nmap que enumera cipher suites soportadas como parte de un escaneo de puertos |

Buscar: soporte de protocolos deprecados (TLS 1.0/1.1) · cipher suites débiles · ausencia de forward secrecy · problemas de certificado (expirado, self-signed, mismatch de dominio).
