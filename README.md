# Página de mantenimiento de GustApp

Página estática de una sola pantalla, completamente independiente de la
app (Next.js). No depende de `npm install`, de ninguna build ni de la
base de datos — es HTML + CSS puro, sin JavaScript, sin scripts de
terceros, sin analítica y sin enlaces salientes. Usa el logo oficial
`logo_gustapp_web.png` (copia exacta del que ya vive en la raíz del
repo) y el fondo/texto del propio sistema de color de GustApp
(`--bg`/`--text`/`--muted` de `app/globals.css`).

Todo lo de este directorio está pensado para poder publicarse de forma
totalmente aislada del resto del repositorio.

## 1. Probarla en local

No hace falta ningún servidor: basta con abrir el archivo directamente
en el navegador.

```bash
open maintenance-page/index.html
```

Si prefieres verla servida por HTTP (más fiel a cómo la vería GitHub
Pages), cualquier servidor estático vale, por ejemplo:

```bash
cd maintenance-page
python3 -m http.server 8000
# abre http://localhost:8000
```

## 2. Publicarla en GitHub Pages

Esta página vive dentro de `maintenance-page/`, una subcarpeta del
repo principal. La configuración estándar de GitHub Pages (Settings →
Pages) solo permite publicar la raíz de una rama o una carpeta
`/docs` — no una subcarpeta arbitraria — así que el camino más simple,
sin tocar el histórico de `main` ni añadir un workflow de GitHub
Actions, es extraer esta carpeta a su propia rama `gh-pages` con
`git subtree split` y publicar esa rama:

```bash
# Desde la raíz del repo, con main ya actualizado:
git subtree split --prefix=maintenance-page -b gh-pages
git push origin gh-pages
```

Después, en GitHub: **Settings → Pages → Source** → rama `gh-pages`,
carpeta `/ (root)`, y guardar. GitHub Pages publicará exactamente el
contenido de esta carpeta (incluido el archivo `CNAME` de abajo) en
`https://mindora-data.github.io/gustapp/` hasta que el dominio propio
esté configurado.

Si más adelante cambia el contenido de `maintenance-page/` en `main`,
repite el `git subtree split` (puedes forzar la rama con
`git subtree split --prefix=maintenance-page -b gh-pages --rejoin` o
simplemente borrar y recrear `gh-pages` localmente) y vuelve a hacer
`push`.

## 3. DNS para `gustapp.mindoralabs.net`

Esta carpeta ya incluye un archivo `CNAME` con el contenido
`gustapp.mindoralabs.net` — es el mecanismo propio de GitHub Pages
para servir un dominio personalizado y ya está preparado; no hay que
crearlo a mano.

Lo que sí hay que hacer, en el proveedor DNS del dominio
`mindoralabs.net` (fuera de este repo, no lo gestiona Claude), es
añadir un registro:

| Tipo  | Nombre/Host | Valor                   |
|-------|-------------|-------------------------|
| CNAME | `gustapp`   | `mindora-data.github.io` |

(`mindora-data` porque el repo vive en la organización de GitHub
`Mindora-data` — `github.com/Mindora-data/gustapp` — el hostname de
Pages para una organización siempre es `<organización-en-minúsculas>.github.io`.)

Usa un TTL bajo (por ejemplo 300s) mientras dure esta fase, para poder
revertir rápido en el paso siguiente sin esperar una propagación larga.

**Este cambio de DNS no lo hace Claude** — se aplica manualmente en el
panel del proveedor cuando decidáis activar la página de mantenimiento
de verdad.

## 4. Revertir el DNS cuando el VPS limpio esté listo

Cuando la reconstrucción del servidor termine y GustApp vuelva a
funcionar de verdad:

1. En el proveedor DNS, sustituye el registro `CNAME gustapp →
   mindora-data.github.io` por el registro que apuntaba al servidor
   antes del incidente (normalmente un registro `A` con la IP pública
   del nuevo VPS; si antes había IPv6, también el `AAAA`
   correspondiente). El valor exacto depende de cómo esté configurado
   el nuevo servidor — no lo asumas de este documento.
2. Espera a que se propague (el TTL bajo del paso 3 ayuda a que sea
   rápido) y confirma con `dig gustapp.mindoralabs.net` que ya resuelve
   a la IP del VPS.
3. Opcional: desactiva GitHub Pages para este repo (Settings → Pages →
   "Unpublish site") y borra la rama `gh-pages` si ya no hace falta.

No se ha tocado ningún DNS ni desplegado nada como parte de la
creación de esta página — todo lo de arriba queda documentado para
ejecutarlo manualmente cuando corresponda.
