# Terminar la configuración de Upptime

`.upptimerc.yml` y los 8 workflows de GitHub Actions ya están en el repo (copiados
directo desde https://github.com/upptime/upptime, versión @v1.44.1 — `gh repo create
--template` no los trajo porque ese repo no está marcado como "template repository" en
GitHub, así que se clonaron a mano). El permiso de escritura para Actions (necesario
para que los workflows puedan commitear sus resultados) ya quedó activado vía API.
Queda 1 paso manual, no requiere tocar código.

## Paso 1 — DNS (cuando el deploy real esté listo)

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
