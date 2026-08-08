# Repositorio `.git` Expuesto y Fuga de Secretos

Relacionado: [[Linux-Herramientas-y-Admin]] · [[Herramientas-SQLMap]] · [[Reconocimiento-Pasivo]] · [[Nmap-Escaneo-Puertos]] · [[Linux-Permisos-y-Sistema]] · [[Herramientas-Burp-Suite]]

> Nota de confidencialidad: en esta nota `SITIO` reemplaza siempre el dominio/IP real de cualquier objetivo — nunca anotar el dominio real de un cliente en el vault, ni siquiera junto a datos ya públicos.

## Qué es
Si un servidor sirve el directorio `.git/` de un proyecto por HTTP (típico de un despliegue que copia el repo completo al servidor de producción sin excluir `.git/`), cualquiera puede **reconstruir el historial completo del código fuente** — commits viejos, ramas, y todo lo que alguna vez se subió, incluido lo que ya no está en la versión actual (credenciales hardcodeadas y luego "borradas", por ejemplo, siguen vivas en el historial).

## 1. Confirmar la exposición
```bash
curl -isk https://SITIO/.git/HEAD
```
Si responde `200` con algo como `ref: refs/heads/main`, el repo está expuesto.

## 2. Recuperar el repositorio

### Opción A — `git-dumper` (rápida, repos completos)
```bash
pipx install git-dumper
git-dumper http://SITIO/.git/ ./elrepo
# o sin instalar (una sola vez):
pipx run git-dumper http://SITIO/.git/ ./elrepo
```
Ver [[Linux-Herramientas-y-Admin]] para instalar con `pipx` sin romper el sistema, o la alternativa manual con `venv`.

### Opción B — GitTools (mejor para repos parcialmente dumpeados / corruptos)
```bash
git clone https://github.com/internetwache/GitTools.git
cd GitTools/Dumper
chmod +x gitdumper.sh
./gitdumper.sh http://SITIO/.git/ ./elrepo2

cd ../Extractor
chmod +x extractor.sh
./extractor.sh /ruta/absoluta/a/elrepo2 ~/extraido
```
`extractor.sh` reconstruye cada commit recuperable en su propia carpeta, **aunque el repo completo esté corrupto** — útil cuando `git-dumper` deja el repo a medias porque el servidor bloqueó algunas rutas del `.git/objects/`.

## 3. Diagnóstico del repo recuperado
```bash
git status
git log --all --oneline
git fsck --full                    # qué objetos faltan/están corruptos
ls -la .git/objects/pack/          # si el repo usaba pack files
```
`git fsck --full` es el chequeo de integridad — confirma qué tan completo quedó el dump antes de invertir tiempo buscando en un historial roto.

## 4. Buscar secretos en el historial recuperado
```bash
pipx install gitleaks
gitleaks detect --source . --no-git -v
```
Escanea **todo el contenido del working tree recuperado** (no solo el HEAD actual) buscando patrones de credenciales/API keys/tokens con reglas predefinidas. `--no-git` le dice que trate la carpeta como archivos planos en vez de intentar leer el historial de commits vía git (útil cuando el repo recuperado con GitTools quedó fragmentado en varias carpetas por commit, en vez de un `.git` normal navegable).

## 5. Buscar secretos hardcodeados en JS servido directamente
```bash
curl -s http://SITIO/Public/js/app.js | grep -iE "api_key|secret|password|token|authorization"

# o guardándolo para revisar con calma / reusar el archivo:
curl -s http://SITIO/Public/js/app.js -o app.js
grep -in "palabra_clave" app.js
```
El JS del cliente a veces trae claves de API o endpoints internos hardcodeados directamente — vale la pena revisarlo aunque no haya `.git` expuesto.

## 6. Auditar el código fuente recuperado
```bash
grep -rln "NOMBRE_FUNCION" .
grep -rln "NOMBRE_PARAMETRO" . --include='*.php'
find . -iname "*route*" -o -iname "*Controller*"
grep -rln "Route::" .
```
`grep -rln` (recursivo, listar solo nombres de archivo, sin distinguir mayúsculas) rastrea **quién usa/llama** a una función o parámetro específico en todo el proyecto — clave para entender el flujo real de un endpoint sospechoso sin adivinar (mismo principio que la nota en [[Linux-Permisos-y-Sistema]] sobre leer código en vez de adivinar). `find ... -iname "*route*" -o -iname "*Controller*"` localiza rápido los archivos de definición de rutas en frameworks tipo Laravel/Symfony.

## 7. Fechar la exposición pública
```bash
curl "http://web.archive.org/cdx/search/cdx?url=SITIO/.git/HEAD&output=json"
```
Confirma desde cuándo el recurso estuvo indexado públicamente — dato relevante para el reporte (ver [[Reconocimiento-Pasivo]] → *Wayback Machine*, incluye también el dato secundario de `crt.sh`).

## Recon complementario típico en este tipo de investigación
Estos pasos no son específicos de la exposición de `.git`, pero suelen acompañarla — documentados a fondo en sus notas dedicadas:
- `nmap -sV -Pn --top-ports 1000 SITIO` (rápido) y `nmap -sV -Pn -p- SITIO` (completo) → ver [[Nmap-Escaneo-Puertos]]. `-Pn` es clave aquí porque un dominio web detrás de WAF/CDN suele bloquear ICMP.
- `dig +short SITIO` · `host SITIO` · `whois IP` → ver [[Reconocimiento-Pasivo]].
- `curl -isk https://SITIO:PUERTO/ | head -30` → revisar rápido un servicio/puerto específico ya identificado.
- Si algún parámetro (`?id=...`) luce sospechoso tras auditar el código → `sqlmap`, ver [[Herramientas-SQLMap]].

## Lecciones clave
- `.git/HEAD` accesible por HTTP (`curl -isk https://SITIO/.git/HEAD` → `200`) es la señal de un chequeo de 5 segundos que vale la pena hacer siempre en cualquier objetivo web.
- El historial de git puede contener secretos que el código **actual** ya no tiene — nunca basta con mirar solo la versión más reciente del repo.
- Cuando el dump queda corrupto/parcial, `GitTools` + `extractor.sh` rescata lo que `git-dumper` solo no puede.
- Auditar código fuente recuperado (`grep -rln`) es mucho más confiable que probar endpoints a ciegas — el código dice exactamente qué parámetros espera y qué hace con ellos.
