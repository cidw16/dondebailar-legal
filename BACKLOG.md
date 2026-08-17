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

---

# Que la landing se pueda ENCONTRAR

Los cuatro ítems de arriba son de higiene del sitio. Esto es otra cosa: hoy
**`dondebailar.net` no está indexado**. Una búsqueda del dominio no lo
devuelve. Todo lo que sigue está en el orden en que hay que hacerlo, y las
dependencias están escritas porque hacerlo al revés es tirar trabajo.

## BLOQUEANTES — van primero

### D1 · Alta en Google Search Console y solicitud de indexación

**El sitio no está indexado. Sin esto, nada de lo demás sirve**: se puede
tener el mejor `sitemap.xml`, el JSON-LD perfecto y las páginas más rápidas,
y Google no las va a mirar porque no sabe que el dominio existe.

Es **gratis** y es de las pocas cosas de esta lista que no dependen de nada.
No tiene excusa para no estar hecho.

Dependencias: **ninguna**. Se puede hacer hoy.

### D2 · Paso 5: la raíz pasa a servir la landing

Hoy `dondebailar.net` **es la política de privacidad**. Quien busque la marca
y llegue a la raíz cae en un texto legal, no en la app.

Es el problema más caro de los que se pueden arreglar sin escribir código
nuevo: no importa cuánto se optimice `app.html` si la puerta de entrada del
dominio muestra otra cosa.

**DEPENDE de que 0.7.14 esté publicada.** Los binarios que ya están en la
calle llevan `PRIVACY_URL` clavado a la raíz: si la raíz deja de ser la
política antes de que esos binarios se retiren, esos usuarios se quedan sin
poder llegar a ella. No se adelanta.

### D9 · ✅ HECHO (2026-08-17) · Los links a la política, todos relativos

Los tres links internos a la política pasan a `privacidad.html` **relativo**.
Ninguno a la raíz.

**Por qué no era cosmético.** Hoy `/` y `/privacidad.html` sirven el MISMO
documento —idénticos al normalizar comentarios y espacios—, así que los tres
funcionaban. Pero **D2 (paso 5) convierte la raíz en la landing**: el día que
eso pase, un link a `/` lleva a marketing **sin dar error**. Es el peor modo de
falla que hay: nada se rompe, ningún 404, nadie se entera. Es exactamente el
mismo razonamiento que ya está escrito para `PRIVACY_URL` en D2.

**⚠ LOS NÚMEROS DE LÍNEA QUE CIRCULABAN ESTABAN MAL, LOS TRES.** Se anotan los
medidos para que nadie vuelva a buscar donde no está:

| se decía | qué hay ahí de verdad | dónde estaba el link |
| --- | --- | --- |
| `app.html:322` | CSS: `.que-es::before{…}` | **`app.html:711`**, ya relativo |
| `soporte.html:107` | el párrafo del correo de contacto | **`soporte.html:114`** |
| `eliminar-cuenta.html:95` | el texto de "Qué se conserva" | **`eliminar-cuenta.html:102`** |

**Y dos de los tres ni siquiera eran links a `privacidad.html`:** `soporte.html`
y `eliminar-cuenta.html` apuntaban a `https://dondebailar.net/` —la raíz
absoluta—, no al archivo. `grep -n 'privacidad\.html' *.html` sobre todo el
repo devolvía **un solo `href` real**, el de `app.html`. El de `app.html` no se
tocó: ya estaba bien.

**Lección de método, que es lo que sobrevive a este ítem:** el rediseño de
`app.html` movió cada línea del archivo, así que cualquier número anotado antes
del rediseño quedó muerto. Los números de línea en un backlog envejecen mal;
anotar el `grep` que los encuentra envejece bien.

**Lo que NO entra acá:** `soporte.html:118` apunta a
`https://dondebailar.net/eliminar-cuenta.html`, también absoluto. No se tocó
porque el paso 5 no cambia esa URL —`eliminar-cuenta.html` se queda donde
está—, así que no comparte el modo de falla. Queda anotado por si alguna vez se
unifica el criterio de relativo vs absoluto en todo el sitio.

## DESPUÉS DEL PASO 5 — porque las URLs cambian

Los dos que siguen describen URLs. Si se escriben antes del paso 5, listan
direcciones que van a dejar de existir y hay que rehacerlos enteros.

### D3 · `robots.txt` y `sitemap.xml`

**No existe ninguno de los dos.** Es el ítem 2 de arriba, con el detalle
medido. Va acá en el orden porque su dependencia es el paso 5.

### D4 · `llms.txt`

Un archivo que declara de qué trata el sitio para que lo lean los motores de
IA. **Es un estándar propuesto y ningún motor confirmó que lo siga.**

Diez minutos de trabajo, sin ninguna garantía de que sirva. **Va último a
propósito**: si se hace primero, se siente productivo y no mueve nada. Que
sea barato no lo hace prioritario.

## INDEPENDIENTES — no esperan a nadie

### D5 · `canonical` y `og:url` en las cuatro páginas legales

Es el **ítem 1** de arriba. Prioridad al duplicado exacto: `index.html` y
`privacidad.html` son idénticos byte a byte y ninguno declara cuál es el
bueno.

### D6 · El `author` del JSON-LD tiene que coincidir con App Store Connect

Hoy `app-v2.html` declara `"author": {"@type":"Organization","name":"Roberto
Del Cid"}`. El mail del pie es `support@novaaisolutionscr.com`, o sea de Nova
AI Solutions CR.

**Hay que verificar cuál de los dos figura como desarrollador en la ficha del
App Store** y usar ese, textual. Un dato estructurado que no coincide con la
tienda es peor que no tenerlo: le dice a Google que la página y la ficha son
de dueños distintos.

No se puede resolver desde el repo: hay que mirar App Store Connect.

### D7 · El swap `app-v2.html` → `app.html`

Pendiente. Va **después de verificar el rediseño sobre la página publicada**,
y es un commit propio: así lo que pide marcha atrás es un rename y no una
reconstrucción. No confundir con el paso 5, que es mover la landing a la
raíz; esto es solo reemplazar el archivo viejo por el nuevo.

## PROYECTO APARTE — NO es de 0.7.14

### D8 · Directorio público en la web

Una página por entidad —salones, academias y orquestas— generada desde
Supabase, más páginas índice por categoría.

**Qué lleva cada página:** nombre, categoría, dirección, coordenadas,
contacto público, descripción, logo, y para salones y orquestas el **horario
recurrente**.

**Qué NO lleva: la tabla `eventos`.** Los eventos con fecha se quedan en la
app. No es un recorte de alcance para llegar a tiempo: es la línea que
separa este proyecto del que se descartó, y está al final de este archivo.

**Por qué este sí y la agenda no.** El directorio es **información estable
que las propias entidades quieren difundir**: un salón quiere que se sepa su
dirección y su noche fija. Publicarla no le quita nada a nadie. La agenda al
día, los recordatorios y la proximidad **son la app**, y ahí queda el valor.

**El beneficio principal no es el SEO.** Es que **cada entidad tiene una URL
propia que puede compartir**: en su bio de Instagram, en su WhatsApp, en un
flyer. Es la vía más realista de conseguir enlaces entrantes — que es
justamente lo que la sección de abajo dice que pesa más que todo lo técnico
junto. Y los pondrían las entidades, no nosotros.

**DEPENDE del paso 5 (D2). Antes no.** Mientras la raíz siga siendo la
política de privacidad, no tiene sentido colgar un árbol de URLs públicas de
un dominio cuya puerta de entrada muestra un texto legal.

**NO ENTRA EN 0.7.14.** Es un proyecto con su propio alcance: generación de
las páginas, dónde se hospedan, cada cuánto se regeneran y qué pasa cuando
una entidad se da de baja. Se anota para que no se pierda entre los arreglos
chicos, no para meterlo en el ciclo actual.

Nota de datos, ya verificada contra la base: los campos salen de `entidades`,
que hoy tiene 26 columnas e incluye `nombre`, `category`, `direccion`,
`latitud`, `longitud`, `descripcion`, `logo_url`, `horario` y los de contacto
(`telefono`, `whatsapp`, `email`, `website`, `instagram`, `facebook`,
`tiktok`). No hace falta modelo nuevo.
## Lo que pesa más que todo lo anterior junto, y no es trabajo de repo

**Las menciones de terceros.** Que una nota de prensa local, un foro de baile
o una cuenta con audiencia enlacen a `dondebailar.net` mueve más la aguja que
todos los ítems técnicos de esta lista sumados.

**No es trabajo de repositorio y no hay commit que lo resuelva.** Se anota
igual porque el riesgo real es el contrario: pasar semanas puliendo
`sitemap.xml` y JSON-LD sintiendo que se avanza, mientras lo que de verdad
mueve el ranking nunca se empieza porque no se parece a programar.

---

# Descartado

Lo que se evaluó y NO se va a hacer. Está acá para que no reaparezca como
idea nueva dentro de seis meses.

- **Publicar la agenda como páginas web** (salones, orquestas y eventos
  generados desde Supabase, por provincia o por ritmo). Se evaluó y se
  descartó: **canibaliza la app** y **expone a raspado el único activo que la
  diferencia**, que son los datos curados a mano.
