# 📚 Índice — Vault de Ciberseguridad

Mapa de contenido (MOC) para navegar rápido. Organizado por tema, no por orden de creación. Cada nota trae ejemplos de comandos listos para copiar y estudiar.

> Consejo: usa el **Graph View** de Obsidian para ver cómo se conectan estas notas, o `Ctrl+O` (Quick Switcher) para saltar directo a una por nombre.

---

## 🧭 Fundamentos de redes
- [[Redes-Fundamentos]] — conceptos base de networking
- [[OSI-Model]] — las 7 capas
- [[TCP-UDP-Puertos]] — TCP vs UDP, three-way handshake, puertos
- [[Subnetting]] — cálculo de subredes
- [[Infraestructura-de-Red]] — dispositivos de red
- [[Protocolos-ARP-DHCP]] — ARP, DHCP
- [[Packets-and-Frames]] — encapsulación
- [[DNS]] — resolución de nombres
- [[Protocolos-Aplicacion]] — TELNET, FTP, SMTP, POP3, IMAP, WHOIS, SSH (con comandos)
- [[Seguridad-de-Red]] — conceptos generales de seguridad en red

## 🖥️ Sistemas y fundamentos
- [[Sistemas-Operativos]] · [[Sistemas-Hardware]] · [[Virtualizacion]] · [[Cloud-Computing]]
- [[Representacion-de-Datos]] — binario, hex, encoding

## 🐧 Linux / 🪟 Windows
- [[Linux-Herramientas-y-Admin]] · [[Linux-Permisos-y-Sistema]] · [[Linux-Scripting-y-Shells]]
- [[Windows-CMD]] · [[Windows-PowerShell]]

## 🌐 Web
- [[Arquitectura-Web]] — front end/back end
- [[Cliente-Servidor-HTTP]] · [[HTTP-Web]] · [[HTTP-Peticiones-y-Respuestas]] — métodos, headers, status codes
- [[JavaScript-Web]] — sintaxis, ofuscación, bypass de validación client-side
- [[Bases-de-Datos-SQL]] — CRUD, cláusulas, operadores, funciones

## 🔐 Criptografía y Hashing
- [[Criptografia]] — fundamentos
- [[Criptografia-Clave-Publica]] — RSA, Diffie-Hellman, firmas digitales, SSH keys, GPG
- [[Hashing]] — MD5/SHA, salt, rainbow tables, HMAC

## 🛠️ Herramientas ofensivas
- [[Herramientas-Hydra]] — fuerza bruta
- [[Herramientas-John-the-Ripper]] — crackeo de hashes
- [[Herramientas-Metasploit]] — msfconsole, msfvenom, Meterpreter
- [[Herramientas-Burp-Suite]] — proxy, intercepción, XSS
- [[Herramientas-CyberChef]] — encoding/decoding, recetas
- [[Herramientas-SQLMap]] — detección y explotación automática de SQL injection
- [[Herramientas-CAPA-Malware-Analysis]] — análisis estático de malware
- [[Herramientas-REMnux]] — oledump, INetSim, Volatility
- [[Herramientas-FlareVM]] — PEStudio, FLOSS, Process Monitor/Explorer

## 🎯 Pentesting — metodología y práctica
- [[Ofensiva-Pentesting]] — **nota central**: terminología, red vs pentest, vulnerabilidad/amenaza/riesgo, mindset, ética
- [[Frameworks-Pentest]] — OSSTMM, WSTG, NIST 800-115, PTES, ISSAF, MITRE ATT&CK
- [[Cyber-Kill-Chain]] — las 7 etapas de un ataque
- [[Reconocimiento-Pasivo]] — WHOIS/RDAP, dig, DNSDumpster, crt.sh, Wayback Machine, Shodan
- [[Reconocimiento-Activo]] — DevTools, ping, traceroute, telnet, netcat, nmap (host discovery)
- [[Nmap-Escaneo-Puertos]] — estados de puerto, TCP Connect/SYN/UDP scan, especificar puertos, timing y rate
- [[Nmap-Escaneo-Avanzado-y-Evasion]] — Null/FIN/Xmas/Maimon/ACK/Window scan, spoofing IP/MAC, decoy, fragmentación, idle scan, OS fingerprinting
- [[TCP-Escaneo-Riesgos-y-Defensa]] — síntesis: banderas TCP, 3-way handshake, SYN flood, qué filtra cada scan, defensas y nota legal
- [[Git-Expuesto-y-Fuga-de-Secretos]] — detectar `.git` expuesto, git-dumper/GitTools, gitleaks, auditar código recuperado
- [[Metodologia-Pentest-Web-Chain]] — caso completo: IDOR → password reset → RCE
- [[Infra-Pentest-Metasploit-Ejemplo]] — caso completo: nmap → searchsploit → Metasploit → privesc
- [[Web-Pentest-SSRF-Ejemplo]] — caso completo: fingerprint de wkhtmltopdf → HTML injection → LFI bloqueado → SSRF contra endpoint interno
- [[Reconocimiento-Subdominios-Takeover-Ejemplo]] — caso completo: vhost fuzzing → SAN del certificado TLS → subdominio escondido → subdomain takeover
- [[Web-Pentest-RCE-Privesc-Ejemplo]] — caso completo: command injection → reverse shell → privesc → pivote a servicio interno → AJAX auth → Bearer token
- [[OWASP-Top-10-2025]] — las 9 categorías con ejemplos reales
- [[Ataques-Sniffing-MITM]] — captura de paquetes (tcpdump), ARP/DNS spoofing, rogue AP, SSL stripping
- [[TLS-SSL-Fundamentos]] — historia, handshake, implicit TLS vs STARTTLS, certificados

## 🛡️ Defensiva / Blue Team
- [[Defensiva-Blue-Team]] — nota general
- [[SOC-Centro-de-Operaciones]] — pilares del SOC, roles L1/L2/L3, triage
- [[Incident-Response]] — SANS (PICERL) vs NIST, playbooks/runbooks
- [[Digital-Forensics]] — cadena de custodia, imágenes forenses, pdfinfo/exiftool
- [[Logs-y-SIEM]] — tipos de log, Event IDs, reglas de detección
- [[Firewall-Fundamentos]] — tipos de firewall, ufw
- [[IDS-Snort]] — HIDS/NIDS, reglas Snort
- [[Vulnerability-Scanning-y-CVE]] — CVE, CVSS, OpenVAS

## 🧠 Principios y roles
- [[CIA-Triad]] — Confidencialidad, Integridad, Disponibilidad
- [[Principios-de-Seguridad-CIA-DAD]] — DAD, Parkerian Hexad, Bell-LaPadula, Zero Trust, ISO/IEC 19249
- [[Roles-Ciberseguridad]] — Security Analyst, Engineer, Pentester + valor del training

---

## 📝 Cómo sigo agregando notas
1. Nueva sala/tema → decide si **amplía** una nota existente o si merece nota propia (evita duplicar).
2. Siempre incluir comandos de ejemplo, no solo teoría.
3. Enlazar con `[[nombre-de-nota]]` a temas relacionados — así el grafo tiene sentido.
4. Actualizar este README cuando se agregue una nota nueva importante.
