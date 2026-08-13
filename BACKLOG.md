# Backlog de la landing

Repo `dondebailar-legal`. **No es el repo de la app**: acá no se toca el binario
ni el ciclo de release.

Nace el **2026-08-13**, con el rediseño de `app-v2.html`. Antes no existía: las
decisiones vivían en comentarios del HTML y en los PR. Los comentarios se quedan
donde están —son la advertencia para quien edita esa línea—; acá va lo que
**todavía no se hizo** y por qué.

Misma regla que el backlog de la app: **cada ítem con el número medido que lo
justifica.** Sin número medido, no entra.

---

## 1 · Las cuatro páginas legales no tienen `canonical` ni `og:url`

**Origen.** Discovery de SEO, 2026-08-13.

**Lo medido**, sobre las seis páginas del repo:

| página | canonical | og:url |
| --- | --- | --- |
| `app.html` | `dondebailar.net/app.html` | sí |
| `app-v2.html` | `dondebailar.net/app.html` (a propósito) | sí |
| `index.html` | **ninguno** | **ninguno** |
| `privacidad.html` | **ninguno** | **ninguno** |
| `soporte.html` | **ninguno** | **ninguno** |
| `eliminar-cuenta.html` | **ninguno** | **ninguno** |

**Lo que lo hace urgente y no cosmético:** `index.html` y `privacidad.html` son
**idénticos byte a byte** (verificado con `cmp`). Google ve dos URLs con
contenido idéntico, ninguna declara cuál es la buena, y elige él. Un
`canonical` en cualquiera de las dos lo resuelve.

**Qué hacer.** PR chico y aparte: `canonical` + `og:url` en las cuatro.
Prioridad al duplicado exacto.

**Cuidado con el orden:** cuál de `index.html` y `privacidad.html` sobrevive se
decide en el **paso 5** (mudar la landing a la raíz). Poner el canonical NO
prejuzga eso, pero conviene apuntar los dos a `privacidad.html`, que es a donde
ya apunta el footer del rediseño.

---

## 2 · `robots.txt` y `sitemap.xml` — BLOQUEADOS hasta después del paso 5

**Origen.** Mismo discovery. **Lo medido:** no existe ninguno de los dos, y no
hay una sola referencia a ellos en las seis páginas.

**POR QUÉ NO SE HACEN TODAVÍA, y esta es la parte importante:** el **paso 5**
mueve `app.html` a la raíz del sitio. Cuando eso pase, **las URLs cambian**: la
landing deja de ser `/app.html` y pasa a ser `/`, y `index.html` deja de ser la
política. Un `sitemap.xml` escrito hoy listaría URLs que van a dejar de existir,
y habría que rehacerlo entero.

Hacerlo antes es trabajo que se tira. **Va después del paso 5, no antes.**

Sin `robots.txt`, GitHub Pages devuelve 404 y los rastreadores asumen "todo
permitido", que es lo que se quiere hoy. No hay urgencia.

---

## 3 · Las fuentes pesan 88 KB, el 26 % de la página

**Origen.** Mismo discovery, midiendo el peso real de `app-v2.html`.

**Lo medido**, descargando los archivos de verdad:

| qué | bytes | % del total |
| --- | --- | --- |
| `app-v2.html` | 34 603 | 10 % |
| las tres fotos WebP | 214 344 | 63 % |
| CSS de Google Fonts | 14 189 | 4 % |
| **2 woff2, subset latin** | **75 740** | **22 %** |
| **TOTAL** | **338 876** | |

Y el desglose de las fuentes:

| familia | bytes | dónde se usa |
| --- | --- | --- |
| Manrope | 24 836 | todo el cuerpo |
| **Unbounded** | **50 904** | **solo los títulos** |

**Unbounded sola pesa 50.9 KB —más que la foto del hero, que son 52.4— y se usa
únicamente en el wordmark, el h1, los h2 y los h3.**

**Es la mayor oportunidad de peso que queda.** Y aun así: **es identidad de
marca y no se decide por bytes.** Se anota para que el número esté a la vista el
día que alguien discuta el rendimiento, no como una tarea pendiente.

Si alguna vez se quisiera atacar, las vías son subsetear Unbounded a los
caracteres que realmente aparecen, o self-hostear las dos y sacarse de encima
los dos orígenes externos.

**Tres cosas que ya se descartaron al medir**, para que nadie las repita:

- **No son 291 KB.** Ese es el total de los 11 woff2 que declara el CSS. El
  navegador solo baja los subsets que necesita.
- **No son 201 KB.** Manrope y Unbounded son **fuentes variables**: un solo
  archivo por familia y subset cubre todos los pesos. Sumar las 7 caras "latin"
  cuenta el mismo archivo cuatro veces.
- **`latin-ext` no se descarga.** Se extrajeron los 55 caracteres visibles
  únicos de la página y los 55 caen dentro del rango `latin`. Los acentos y la
  ñ viven en U+00xx, no en latin-ext.

---

## 4 · El i18n no llega al `<head>`, y para Google la página es monolingüe

**Origen.** Mismo discovery.

**Lo medido:** cero atributos `data-es` dentro del `<head>`. El `<title>` y la
`meta description` quedan **siempre en español**, para cualquier idioma.

Eso es **deliberado** y está escrito en el archivo: los crawlers leen el HTML
crudo, así que cambiarlos por JS no serviría de nada. No se toca.

Lo que sí queda sin resolver, y es distinto: **no hay `hreflang` ni URLs
separadas por idioma**, así que para Google la versión en inglés no existe. Un
usuario con el navegador en inglés ve la página traducida pero el snippet del
buscador le sale en español.

Resolverlo de verdad pide dos URLs (`/es/` y `/en/`) con contenido servido, no
un toggle de cliente. Es un cambio de arquitectura del sitio, no un ajuste.
**No hay número que lo justifique todavía**: sin analítica no se sabe cuánto
tráfico en inglés hay. Se anota y se espera.
