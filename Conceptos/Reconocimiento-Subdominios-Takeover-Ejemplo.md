# Reconocimiento — Ejemplo con Subdomain Takeover (TakeOver, TryHackMe)

Relacionado: [[Reconocimiento-Pasivo]] · [[Reconocimiento-Activo]] · [[DNS]] · [[Protocolos-Aplicacion]] · [[TLS-SSL-Fundamentos]]

## Contexto
Objetivo: `futurevera.thm`. Cadena: **vhost fuzzing (activo) → inspección del certificado TLS de un subdominio → subdominio "escondido" revelado en el SAN → ese subdominio resulta vulnerable a subdomain takeover**. Combina enumeración pasiva (certificados) con técnicas activas (fuzzing, conexión TLS directa).

## Fase 1 — Agregar el dominio a `/etc/hosts`
```bash
echo "MACHINE_IP futurevera.thm" | sudo tee -a /etc/hosts
```
`sudo tee -a` escribe con privilegios root al final del archivo (`-a` = append). Se usa `tee` en vez de `echo ... >> archivo` porque la redirección `>>` corre con tus permisos normales aunque antepongas `sudo` al `echo` — bash resuelve la redirección antes de aplicar el `sudo`, así que ese patrón falla por permisos.

Verificar que la entrada quedó bien antes de seguir (`cat /etc/hosts`) y confirmar que el nombre resuelve a la IP esperada:
```bash
ping -c 1 futurevera.thm
```
`-c 1` manda un solo paquete ICMP — aquí no importa si responde, solo que muestre la IP correcta.

## Fase 2 — Confirmar que el sitio responde
```bash
curl -sk -o /dev/null -w "%{http_code}\n" https://futurevera.thm
```
`-s` modo silencioso · `-k` ignora errores de certificado SSL autofirmado (típico en estas VMs) · `-o /dev/null` descarta el body, solo interesa el código · `-w "%{http_code}\n"` imprime el código HTTP devuelto (200 = vivo, 000 = no conectó).

## Fase 3 — Revisar el sitio principal a mano
Antes de leer todo a mano, un primer filtro rápido con grep ahorra tiempo:
```bash
curl -sk https://futurevera.thm | grep -iE "subdomain|support|blog|portal"
```
`grep -iE "a|b|c"` busca varios patrones a la vez (`-E` = extended regex), ignorando mayúsculas (`-i`).

```bash
curl -sk https://futurevera.thm | less
```
Sin filtros, para leer el HTML completo buscando comentarios, nombres de archivos o pistas de subdominios (`less` permite navegarlo con calma, `q` para salir).

## Fase 4 — Enumeración de subdominios (vhost fuzzing)
Si no sabes la ruta exacta de la wordlist, localizarla primero:
```bash
find / -iname "*subdomain*" 2>/dev/null
find / -iname "*seclists*" -maxdepth 6 2>/dev/null
```
`find / -iname "*patron*"` busca en todo el sistema ignorando mayúsculas · `2>/dev/null` descarta errores de "permiso denegado" · `-maxdepth 6` limita la profundidad para que sea más rápido.

```bash
gobuster vhost -u https://futurevera.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  -t 50 -k --append-domain
```
`vhost` → modo de gobuster para descubrir virtual hosts que responden distinto aunque compartan la misma IP · `-u` → objetivo base contra el que se prueba cada palabra · `-w` → wordlist de SecLists con nombres comunes de subdominios · `-t 50` → 50 hilos en paralelo · `-k` → ignora verificación SSL · `--append-domain` → arma cada intento como `PALABRA.futurevera.thm` en vez de mandar la palabra sola.

Resultado: `support.futurevera.thm` y `blog.futurevera.thm` (código **421** "Misdirected Request" — normal en gobuster contra HTTPS por desajuste de SNI/Host, no es error real, confirma que el vhost existe).

## Fase 5 — Agregar los subdominios encontrados a `/etc/hosts`
```bash
sudo sed -i '/futurevera.thm/d' /etc/hosts
echo "MACHINE_IP futurevera.thm blog.futurevera.thm support.futurevera.thm" | sudo tee -a /etc/hosts
```
`sed -i '/futurevera.thm/d' archivo` edita en el sitio (`-i`) y borra (`d`) cualquier línea que contenga `futurevera.thm`, para limpiar antes de reescribir todo junto en una sola línea con la IP y los subdominios ya conocidos.

## Fase 6 — Explorar cada subdominio
```bash
curl -sk https://blog.futurevera.thm | less
curl -sk https://support.futurevera.thm | less
```
Mismo patrón que la Fase 3, ahora contra cada vhost — `support` es el relevante según el contexto de la sala ("están reconstruyendo el soporte").

## Fase 7 — Revisar el certificado SSL de `support.futurevera.thm` (paso clave)
```bash
openssl s_client -connect support.futurevera.thm:443 -servername support.futurevera.thm </dev/null 2>/dev/null \
  | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
```
`openssl s_client -connect host:443` → abre una conexión TLS manual al puerto 443 para inspeccionar el certificado que entrega el servidor · `-servername` → envía el nombre correcto vía **SNI**, asegurando que el servidor devuelva el certificado de ese vhost específico (sin esto, podría devolver el certificado default) · `</dev/null` → evita que la conexión se quede colgada esperando input interactivo · `2>/dev/null` → descarta mensajes de diagnóstico de la conexión · `openssl x509 -noout -text` → parsea el certificado a texto legible (`-noout` = no repetir el bloque base64 crudo) · `grep -A1 "..."` → busca esa sección y muestra 1 línea después (`-A1`), donde aparecen los dominios adicionales cubiertos por el certificado.

Resultado: reveló `secrethelpdesk934752.support.futurevera.thm` — un subdominio único/aleatorio que **nunca habría salido por fuerza bruta**, solo se descubre inspeccionando el certificado (mismo principio que crt.sh en [[Reconocimiento-Pasivo]], pero consultado directo contra el servidor en vez del log público).

## Fase 8 — Agregar el subdominio "escondido" y revisar sus headers
```bash
sudo sed -i '/futurevera.thm/d' /etc/hosts
echo "MACHINE_IP futurevera.thm blog.futurevera.thm support.futurevera.thm secrethelpdesk934752.support.futurevera.thm" | sudo tee -a /etc/hosts
curl -skI http://secrethelpdesk934752.support.futurevera.thm
curl -skI https://secrethelpdesk934752.support.futurevera.thm
```
`-I` → pide solo los headers HTTP de respuesta, sin bajar el body completo — rápido para detectar redirecciones (`302 Found` + header `Location:`). Esto reveló el recurso final al que apunta el DNS de ese subdominio: el punto de subdomain takeover.

## Concepto clave: Subdomain Takeover
Ocurre cuando un registro DNS (típicamente **CNAME**) sigue apuntando a un servicio de terceros (S3, GitHub Pages, Heroku, etc.) que **ya fue eliminado o nunca se reclamó**. Cualquiera puede registrar ese mismo recurso en el proveedor externo y "tomar el control" de ese subdominio de la víctima — queda sirviendo contenido del atacante bajo un dominio de confianza (útil para phishing, robo de cookies de sesión si comparte dominio padre, etc.).

## Lecciones clave
- La enumeración de subdominios no termina en la wordlist — un certificado TLS puede filtrar subdominios que nadie adivinaría (SAN). Revisarlo siempre en objetivos con HTTPS.
- `openssl s_client -servername` es la versión "activa" de lo mismo que se busca pasivamente en crt.sh (ver [[Reconocimiento-Pasivo]]) — útil cuando el certificado no está en un log público indexado, o para confirmar en vivo.
- `-I` (solo headers) es suficiente para detectar el punto de un subdomain takeover — no hace falta bajar el body para ver a dónde redirige o qué servicio externo dice "not found, claim this".
- Mantener `/etc/hosts` limpio con `sed -i '/dominio/d'` antes de reescribir evita arrastrar entradas viejas/duplicadas entre sesiones de reconocimiento.
