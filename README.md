# Granve API — Documentación pública

Sitio estático de documentación del API `v1.0` con consola interactiva ("try it").
Pensado para publicarse en **GitHub Pages** — no necesita build ni servidor.

## Archivos

| Archivo        | Qué es |
|----------------|--------|
| `index.html`   | La documentación completa + consola interactiva. Autocontenido (cero dependencias/CDN). |
| `openapi.yaml` | Especificación OpenAPI 3.0 (fuente machine-readable del contrato). |
| `CNAME`        | Dominio personalizado de GitHub Pages (`docs.granve.com`). |

Alcance: **Introduction, Endpoint, Authentication, guías (Errors, Rate limits,
Webhooks & return URL, Testing/sandbox), Payment (PayIn), Payment (PayOut),
Wallet y Changelog** — los 11 endpoints públicos de `/v1.0` con consola try-it
y ejemplos en 7 lenguajes.

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
2. **Headers `X-Date` / `X-Sign`** — nombres canónicos de la firma desde
   jul-2026 (los navegadores no pueden enviar el header `Date`, forbidden en
   Fetch). `ApiAuthenticate` los lee con prioridad y mantiene `Date`/`Sign`
   como alias legacy para integraciones existentes. Requiere que esa versión
   esté deployada.

## Regenerar la referencia con Redocly (opcional)

`index.html` es artesanal (no lo pisa ningún build). Si además se quiere la
vista clásica de Redoc a partir del YAML:

```bash
redocly build-docs openapi.yaml --output=reference.html
```

**No** usar `--output=index.html`: destruiría la consola interactiva.
