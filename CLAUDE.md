# CLAUDE.md — granve-docs-api

## Qué es

Documentación pública del API `v1.0` de Granve (docs.granve.com), publicada en GitHub Pages desde el
remote `granveltd/docs`. Sitio estático sin build ni servidor: `index.html` (documentación completa
con consola «try it», autocontenido, cero dependencias ni CDN), `openapi.yaml` (contrato
machine-readable) y `CNAME`.

## Cómo se trabaja

- Un cambio de contrato se hace primero en `openapi.yaml` y después en `index.html`; los dos tienen
  que coincidir antes de cerrar.
- Nada de dependencias externas ni CDN en `index.html`: la consola corre en el navegador del
  integrador y tiene que seguir funcionando sin red hacia terceros.
- Las credenciales de ejemplo son de sandbox; ninguna llave real, ni en ejemplos, ni en la consola.
- La terminología visible sigue la de Granve: «operación» y «cobro» como términos paraguas; nada de
  nombres ni logos de bancos ni cifras de volumen.
- `pendiente-after_id-wallet.patch` en la raíz es un parche del dueño sin aplicar: no se toca ni se
  borra sin su OK.

## Reglas de trabajo

- Documentación de librerías y frameworks: consulta Context7 (resolve-library-id, luego query-docs) antes de asumir una API o una versión. Sin Context7, documentación oficial del proyecto.
- Subagentes solo para trabajo paralelo o con contexto aislado; lo demás se hace directo en el hilo principal.
- Lo mecánico y reversible, sin preguntar. Antes de algo irreversible o sobre producción (borrar, --force, migraciones de datos, secretos, infraestructura compartida): confirma conmigo y propone backup o dry-run. Los permisos de `.claude/settings.json` piden confirmación justo para eso; no los rodees con `sh -c`, rutas absolutas ni scripts.
- Si una instrucción tiene un error o resulta inviable, aplica el cambio mínimo que mantenga la decisión original y anótalo en el informe. Detén el proceso solo si una decisión es imposible de implementar.
- Si encuentras un bug o una mejora fuera del alcance de la tarea, no lo toques: repórtalo como seguimiento en el informe.
- Commits en rama feat/<nombre>, nunca en master, y solo después de mostrarme el diff y recibir mi OK en el chat; push igual.
- Al cerrar una tarea, limpia lo que ya no hace falta: la rama feat/* ya mergeada (local y remota), su worktree (`git worktree remove` y `git worktree prune`) y los archivos temporales, scratch o copias que creaste. Lo que no está mergeado no se borra: se reporta.

## Informe de cierre

Al terminar, responde solo con esto, en 15 líneas o menos:
- Tareas: qué se pidió, estado (hecha / parcial / bloqueada) y cómo quedó.
- Observaciones: desvíos respecto a lo pedido, supuestos tomados, seguimientos.
Una tarea con un ítem abierto no se declara terminada.
