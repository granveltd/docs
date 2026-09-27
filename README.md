# Granve API — Documentación pública

Sitio estático de documentación del API `v1.0` con consola interactiva ("try it").
Pensado para publicarse en **GitHub Pages** — no necesita build ni servidor.

## Archivos

| Archivo        | Qué es |
|----------------|--------|
| `index.html`   | La documentación completa + consola interactiva. Autocontenido (cero dependencias/CDN). |
| `openapi.yaml` | Especificación OpenAPI 3.0 (fuente machine-readable del contrato). |
| `CNAME`        | Dominio personalizado de GitHub Pages (`docs.granve.com`). |

Alcance: **Introduction, What changes (1.2.0), Quickstart, Environments,
Credentials & permissions, Authentication, guías (Errors, Rate limits, Webhooks &
return URL, Testing/sandbox, Troubleshooting), Payment (PayIn), Payment (PayOut),
Wallet, Tools y Changelog** — las 14 operaciones públicas de `/v1.0` (13 con consola
try-it y ejemplos en 7 lenguajes; la renotificación manual se autentica sola y no la
tiene).

Versión publicada: **1.2.0** (credenciales con permisos). Se publica **junto con el
portal de comercios**, el mismo día y nunca antes: describe pantallas del portal
(Configuración › Integración) y errores que recién existen ese día. La fila de cada
versión del Changelog lleva la **fecha del día en que se publica** (hora de Caracas).

## Publicar en GitHub Pages

1. Subir esta carpeta a un repositorio (raíz o carpeta `/docs`).
2. Settings → Pages → Deploy from branch → seleccionar branch y carpeta.
3. Para el dominio `docs.granve.com`: crear un CNAME DNS apuntando a
   `<usuario>.github.io` y mantener el archivo `CNAME`. Si no se usa dominio
   propio, **eliminar el archivo `CNAME`**.

## Requisitos del lado del API para que la consola funcione

La consola ejecuta las llamadas desde el navegador del integrador, por lo que el
backend debe permitir el origen de las docs:

1. **CORS** — definir en el entorno del API:
   `CORS_ALLOWED_ORIGINS=https://docs.granve.com` (agregar también el dominio
   `*.github.io` que corresponda si se sirve sin dominio propio).
   `config/cors.php` ya permite los headers `Sign`/`X-Date` y expone
   `X-Next-Cursor`/`X-Total`.
2. **Headers `X-Date` / `X-Sign`** — nombres canónicos de la firma (los
   navegadores no pueden enviar el header `Date`, forbidden en Fetch).

   La consola **firma con el formato vigente**: `X-Sign` = HMAC-SHA256 de
   `{X-Date}\n{MÉTODO} {ruta}`, con la ruta sin host y **sin query string**
   (`endpointCtx()` expone `signPath` justo para eso — `path` conserva la query
   porque es la URL que se llama, `signPath` no porque es lo único que el
   servidor firma). Si alguna vez se toca `signWithPrivateKey()`, hay que
   tocar también los 7 ejemplos de lenguajes: son lo que el integrador copia.

## Regenerar la referencia con Redocly (opcional)

`index.html` es artesanal (no lo pisa ningún build). Si además se quiere la
vista clásica de Redoc a partir del YAML:

```bash
redocly build-docs openapi.yaml --output=reference.html
```

**No** usar `--output=index.html`: destruiría la consola interactiva.

## Verificar antes de publicar

No hay build ni suite: estas cuatro comprobaciones son la compuerta, y las cuatro
pasan antes de cualquier push, desde la raíz del repo.

```bash
# 1) El contrato OpenAPI es válido. Las tres reglas salteadas fallaban desde antes
#    de la 1.2.0 (no hay securitySchemes, operationId ni license) y no son un error
#    del contrato. Esperado: «Your API description is valid.» con 2 warnings
#    (operation-4xx-response del callback y de la renotificación).
REDOCLY_TELEMETRY=off redocly lint openapi.yaml \
  --skip-rule=security-defined --skip-rule=operation-operationId --skip-rule=info-license

# 2) El JavaScript de index.html compila. ENDPOINTS y WEBHOOK_VERIFY son template
#    literals: un backtick de más en un texto rompe la página entera, sin error visible.
node -e "const s=require('fs').readFileSync('index.html','utf8');new (require('vm').Script)(s.match(/<script>([\s\S]*)<\/script>/)[1]);console.log('js ok')"

# 3) Ninguna promesa retirada en la 1.2.0 volvió, y la versión más nueva del
#    Changelog es la de info.version y lleva la fecha de HOY (Caracas).
#    Las dos últimas assert (fila == version, fecha == hoy) solo valen el día en
#    que se publica una versión nueva: en un arreglo sin versión nueva, saltearlas.
python3 - <<'PY'
import re, yaml
from datetime import datetime
from zoneinfo import ZoneInfo
plano = lambda s: ' '.join(s.split())
Y = plano(open('openapi.yaml', encoding='utf-8').read())
html = open('index.html', encoding='utf-8').read()
H = plano(html)
retiradas = [
    'your credentials are the same in both environments', 'identical on both environments',
    'your same credentials work on both environments', 'no test-only key',
    'The same pair works in Sandbox and Production', 'same credentials, same code',
    'Request an immediate rotation', 'two access keys',
    'HMAC_SHA256(X-Date value, private_token)', 'raw_body, your_private_token',
]
vuelven = [f for f in retiradas if f in Y or f in H]
assert not vuelven, f'volvieron: {vuelven}'
version = yaml.safe_load(open('openapi.yaml', encoding='utf-8').read())['info']['version']
m = re.search(r'<tr><td><code>([\d.]+)</code></td><td>([\d-]+)</td>', html[html.index('id="changelog"'):])
hoy = datetime.now(ZoneInfo('America/Caracas')).strftime('%Y-%m-%d')
assert m.group(1) == version, f'la fila más nueva es {m.group(1)} y info.version es {version}'
assert m.group(2) == hoy, f'la fila {version} dice {m.group(2)}: se publica hoy, {hoy}'
print('sin promesas retiradas; versión', version, 'fechada', hoy)
PY

# 4) Balance de etiquetas y anclas internas de index.html — lo único que atrapa un
#    href="#…" roto. Tal cual la compuerta común del plan de credenciales v3.
python3 - <<'PY'
import re
s = open('index.html', encoding='utf-8').read()
for t in ['div', 'table', 'ul', 'ol', 'tr', 'h2', 'h3', 'h4', 'pre', 'code', 'strong', 'em', 'a', 'li', 'p', 'section']:
    d = len(re.findall(r'<%s[\s>]' % t, s)) - len(re.findall(r'</%s>' % t, s))
    assert d == (2 if t == 'pre' else 0), f'<{t}> desbalanceado: {d}'
ids = set(re.findall(r'\bid="([\w-]+)"', s))
rotos = sorted(set(re.findall(r'href=\\?"#([\w-]+)\\?"', s)) - ids)
assert not rotos, f'anclas rotas: {rotos}'
print('html ok')
PY
```
