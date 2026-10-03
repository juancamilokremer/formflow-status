# Terminar la configuración de Upptime

`.upptimerc.yml` y los 8 workflows de GitHub Actions ya están en el repo (vinieron del
template `upptime/upptime` @v1.44.1 — el `gh repo create --template` sí funcionó,
quedaron en la rama `master`). El permiso de escritura para Actions (necesario para que
los workflows puedan commitear sus resultados) ya quedó activado vía API, y los 8
workflows ya están registrados y activos en GitHub.

**Lo que ya funciona sin tocar nada más:** `Uptime CI` corre cada 5 minutos, chequea
`FormFlow API`/`FormFlow App` y commitea el resultado real a `history/`/`api/`
(verificado con una corrida manual exitosa). Hoy reportan "down" porque los dominios de
producción todavía no existen — es esperable, ver Paso 2 más abajo.

Quedan 2 pasos manuales.

## Paso 1 — GH_PAT (para que el repo se mantenga sincronizado con Upptime)

Los workflows `Setup CI` y `Update template CI` intentan sobrescribir sus propios
archivos `.github/workflows/*.yml` cada semana (para traer actualizaciones de Upptime
automáticamente). GitHub **nunca** permite que el token por defecto (`GITHUB_TOKEN`)
toque archivos de workflow, sin importar los permisos del repo — confirmado en la
práctica, falla con `refusing to allow a GitHub App to create or update workflow ...
without 'workflows' permission` en cualquiera de los 8 archivos. Esto requiere sí o sí
un Personal Access Token (classic) con scopes `repo` + `workflow`.

Esto **no bloquea el monitoreo** (`Uptime CI` ya funciona sin esto) — es solo para que
los workflows se actualicen solos cuando Upptime saque una versión nueva. Si no se
configura, el repo sigue funcionando igual, simplemente no se auto-actualiza.

1. GitHub → Settings (de tu cuenta, no del repo) → Developer settings → Personal access
   tokens → Tokens (classic) → Generate new token → marcar `repo` y `workflow`.
2. En este repo → Settings → Secrets and variables → Actions → New repository secret →
   nombre `GH_PAT`, valor el token generado.

## Paso 2 — DNS (cuando el deploy real esté listo)

`.upptimerc.yml` ya apunta a `api.formflow.app`/`app.formflow.app` (los dominios de
producción planeados) y a `status.formflow.app` como CNAME de la status page. Hasta que
el deploy real esté arriba, Upptime va a reportar esos sitios como "down" en cada
corrida — es esperable, no manda ninguna alerta a nadie porque no hay notificaciones
(Slack/email/etc) configuradas en `.upptimerc.yml` todavía.

Cuando `formflow.app` esté resolviendo:

1. En el proveedor de DNS del dominio, agregar:
   ```
   CNAME  status.formflow.app  →  juancamilokremer.github.io
   ```
2. En este repo → **Settings → Pages → Custom domain** → escribir `status.formflow.app`
   → Save (GitHub emite el certificado HTTPS automáticamente, puede tardar unos minutos)

## Cómo correrlo manualmente para probar

Sin esperar al cron de 5 minutos: **Actions → Uptime CI → Run workflow**. La primera
corrida exitosa genera `api/`, `history/` y `graphs/` con datos reales de FormFlow (hoy
el repo no los tiene — se generan solos, no hace falta crearlos a mano).
