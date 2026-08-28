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

## 2 · `robots.txt` y `sitemap.xml` — ↑ AHORA VAN CON EL PASO 5, en D2

**Origen.** Mismo discovery. **Lo medido:** no existe ninguno de los dos, y no
hay una sola referencia a ellos en las seis páginas.

**POR QUÉ NO SE HACEN TODAVÍA, y esta es la parte importante:** el **paso 5**
mueve `app.html` a la raíz del sitio. Cuando eso pase, **las URLs cambian**: la
landing deja de ser `/app.html` y pasa a ser `/`, y `index.html` deja de ser la
política. Un `sitemap.xml` escrito hoy listaría URLs que van a dejar de existir,
y habría que rehacerlo entero.

Hacerlo antes es trabajo que se tira. **Va CON el paso 5, en la misma tanda**
— ver D2, donde los dos se fundieron el 2026-08-25.

⚠ **Y LA ÚLTIMA LÍNEA DE ESTE ÍTEM ERA FALSA.** Decía: *"Sin `robots.txt`,
GitHub Pages devuelve 404 y los rastreadores asumen «todo permitido»… No hay
urgencia."* Lo primero es cierto y lo segundo no. Search Console, 2026-08-25:
`app.html` y `soporte.html` son *"URL is unknown to Google"*, discovery **sin
sitemap y sin página referente**. Que los rastreadores tengan permiso no sirve
de nada si no saben que las URLs existen. Sí había urgencia.

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

Los cuatro ítems de arriba son de higiene del sitio. Esto es otra cosa.

⚠ **ACTUALIZADO 2026-08-25 con lo medido en Search Console.** Lo que decía acá
—"el sitio no está indexado"— era cierto cuando se escribió y ya no lo es, y la
realidad resultó peor de lo que decía:

```
dondebailar.net/      INDEXADA  ← y lo que Google tiene ahí es la POLÍTICA
                                   DE PRIVACIDAD, no la landing
app.html              "URL is unknown to Google" · last crawl N/A
soporte.html          "URL is unknown to Google" · last crawl N/A
                      Discovery: sin sitemap y sin página referente
```

O sea: **el dominio SÍ está indexado, con la página equivocada, y la landing no
existe para Google.** No es que falte visibilidad: la poca que hay apunta a un
texto legal.

Y el dato que cierra la discusión sobre el sitemap: **`soporte.html` está
enlazada desde el footer de la política —una página que Google SÍ indexó— y aun
así no la descubrió.** El enlace interno no alcanzó. Sin sitemap, el sitio es
casi invisible.

> **Esta foto es el ANTES.** Es la medición que justificó D2, y se deja tal
> cual porque es la línea base contra la que se va a comparar. El paso 5 y el
> sitemap ya se hicieron y el sitemap **ya se envió** —ver D2—; lo que estas
> tres líneas describen es lo que todavía **falta confirmar que cambió**.

## BLOQUEANTES — van primero

### D1 · ✅ HECHO · Alta en Google Search Console

La propiedad existe y responde: de ahí salen las mediciones del 2026-08-25 que
encabezan esta sección. La raíz figura indexada.

Lo que el alta NO resolvió, y por eso el resto de la sección sigue viva: Google
indexó la raíz —que hoy sirve la política— y no descubrió ni `app.html` ni
`soporte.html`.

### D2 · ✅ HECHO (2026-08-25) · Paso 5 **+ sitemap** — era UN SOLO ÍTEM

**Hecho sin esperar el número de Play Console**, y el motivo queda escrito:
25 instalaciones activas, app publicada hace tres semanas, 0.7.14 del 16-ago,
actualizaciones automáticas por defecto. La población en versiones anteriores
es marginal, y la mitigación estaba puesta desde antes: la landing lleva un
link visible a la política, y —lo que más pesa— **`PRIVACY_URL` ya apuntaba a
`/privacidad.html` y no a la raíz desde v0.7.14** (commit `11360bc`, presente
en los tags v0.7.14 y v0.7.15). O sea que la app moderna no usaba la raíz
para nada.

Lo que se hizo: `index.html` pasa a servir la landing con canonical a `/`;
`app.html` **se queda** sirviendo lo mismo con canonical a la raíz (no se
renombró: `/app.html` está publicada y la enlazan `evento.html` y
`entidad.html`, que ya circulan); `sitemap.xml` con las cuatro URLs finales y
`robots.txt` apuntándolo.

✅ **EL SITEMAP YA SE ENVIÓ A MANO — Search Console, 2026-08-25.**

| campo | valor |
| --- | --- |
| URL enviada | `https://dondebailar.net/sitemap.xml` |
| estado | **Success** |
| páginas descubiertas | **4** |

Cuatro es el número correcto: son exactamente las cuatro URLs que el archivo
lista. Que diga "Success" y descubra 4 quiere decir que Google **leyó y
entendió el XML**; no quiere decir todavía que las haya indexado.

Se envió a mano y no se esperó al descubrimiento pasivo, por la evidencia que
ya estaba medida: `soporte.html` está enlazada desde el pie de una página que
Google SÍ indexó y aun así figuraba "URL is unknown to Google". El
descubrimiento pasivo ya había fallado una vez acá.

⚠ **LO QUE SIGUE PENDIENTE, y hay que ir a mirarlo — días después del envío.**
El "Success" de arriba es sobre el ARCHIVO, no sobre las páginas. Lo que hay
que confirmar en Search Console, con la herramienta de inspección de URL:

- que **`app.html` y `soporte.html` dejen de ser "URL is unknown to Google"** —
  esta es la que prueba que el sitemap sirvió para algo;
- que la **raíz** pase a mostrar la landing en el índice, y no la política.

Nadie avisa cuando ocurre. Si dentro de una o dos semanas siguen "unknown", el
problema no es el sitemap y hay que volver a mirar esto con datos nuevos.

<details><summary>El planteo original, para contexto</summary>


Estaban separados —el paso 5 acá y el sitemap más abajo, "después del paso 5"—
y la medición del 2026-08-25 mostró que son la misma jugada. Se hacen juntos.

**La mitad del paso 5.** Hoy `dondebailar.net` **es la política de privacidad**.
Quien busque la marca y llegue a la raíz cae en un texto legal, no en la app.

Y ahora se sabe que **no es cosmético**, que es como estaba planteado: la raíz
es **la única URL que Google tiene indexada**. Mudar `app.html` a la raíz no es
"ordenar el sitio" — es hacer que la landing **herede la única indexación que
existe**, en vez de nacer desconocida como está hoy `app.html`.

**La mitad del sitemap.** No existe `sitemap.xml` ni `robots.txt`, y el efecto
está medido: `app.html` y `soporte.html` son *"URL is unknown to Google"*, sin
un solo rastreo. `soporte.html` está enlazada desde el footer de la política
—página que Google sí indexó— **y aun así no la descubrió**. El enlace interno
no alcanza.

**POR QUÉ VAN JUNTOS Y NO UNO DESPUÉS DEL OTRO.** El motivo por el que el
sitemap estaba diferido era correcto: escrito antes del paso 5, listaría URLs
que van a dejar de existir. Pero hacerlos en la misma tanda elimina esa
dependencia —el sitemap se escribe con las URLs nuevas, que ya se conocen— y
evita el peor de los dos órdenes: mudar la landing a la raíz sin sitemap la deja
heredando la indexación de la raíz pero con el resto del sitio igual de invisible.

**DEPENDE de que 0.7.14 esté publicada.** Los binarios que ya están en la calle
llevan `PRIVACY_URL` clavado a la raíz: si la raíz deja de ser la política antes
de que esos binarios se retiren, esos usuarios se quedan sin poder llegar a ella.
No se adelanta. **Esa sigue siendo la única dependencia real de todo el bloque.**

</details>

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

### D3 · ↑ FUNDIDO EN D2 (2026-08-25)

`robots.txt` y `sitemap.xml` ya no son un ítem aparte: van con el paso 5, en la
misma tanda. Ver **D2**. Se deja el encabezado para que quien venía siguiendo la
numeración no lo busque abajo y concluya que se perdió.

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

# Volver a badges oficiales de tienda, cuando estén los dos en negro

Anotado el 2026-08-27, al pasar la landing a la paleta clara.

**Hoy los dos botones de tienda son botones de TEXTO**, simétricos: mismo
fondo `#14141A`, mismo alto, mismo radio, y en cada uno una línea chica arriba
("Descargalo en" / "Disponible en") con el nombre de la tienda abajo.

No es preferencia estética. Es que sobre papel no había badge usable:

- El de Apple que hay en `/badges/` es la variante **BLANCA**
  (`Download_on_the_App_Store_Badge_ES_RGB_wht_100217`), pensada para fondo
  oscuro. Sobre `#FBF7F2` es exactamente la equivocada.
- **De Google Play nunca hubo badge.** Era una pastilla de texto.

O sea que lo que había era un badge oficial al lado de una pastilla casera, y
eso ya estaba mal antes de este cambio. Dos botones iguales es peor de marca y
mejor de diseño, y sobre todo es coherente.

### Qué hace falta para revertirlo

1. Bajar de los recursos oficiales el badge de Apple en **negro**, en ES y en
   US (`toolbox.marketingtools.apple.com`).
2. Bajar el badge oficial de **Google Play**, en ES y en EN.
3. Resolver la diferencia de forma que ya estaba anotada en el código: el de
   Apple es monocromo 120×40 y el de Play es a color con proporción propia.
   Ponerlos lado a lado sin resolver eso desbalancea los dos botones.

⚠ **Los dos SVG de Apple siguen en `/badges/` y NO se borraron**, aunque hoy
no los use nadie. Cuando se consigan los negros, esto se revierte a badges de
verdad y esos archivos son el punto de partida.

⚠ **No se redibujan ni se recolorean a mano.** Son marcas con lineamientos: la
variante negra existe y se baja, no se fabrica invirtiendo la blanca.

---

# Ajustes de la paleta clara — variantes, no cambios de marca

Anotado el 2026-08-28.

Cuando la landing pasó a fondo `#FBF7F2`, dos colores de marca dejaron de
funcionar sobre papel. **Ninguno de los dos se reemplazó: se les calculó su
variante para fondo claro.** El color de la app sigue siendo el suyo.

Si algún día hay manual de marca, esta sección es de dónde salen los valores
de fondo claro.

### El ámbar del subrayado del wordmark

| | valor | contraste sobre `#FBF7F2` |
|---|---|---|
| fondo oscuro (original) | `#F5C04A` | **1,57:1** — desaparece |
| fondo claro (en uso) | `#C98A1E` | **2,76:1** |

Es decorativo y `aria-hidden`, así que **no hay umbral WCAG que cumplir**: lo
único que importaba era que se viera. Entre las variantes que siguen leyéndose
ámbar se eligió la que menos desaparece. `#E0A32C` daba 2,08:1 y seguía al
borde de lo invisible.

### El coral del wordmark — SIN cambiar, y anotado por si alguien lo mira

`#FF4D5E` sobre `#FBF7F2` da **3,04:1**. Alcanza para texto grande en negrita,
que es exactamente lo que es (21–25px, peso 800), y **no alcanzaría para texto
corrido**. Por eso no se usa en ningún otro lado de la página.

Si alguna vez alguien quiere usar el coral para un párrafo, un botón o un
enlace: no se puede con este fondo, y hay que calcularle su variante como se
hizo con el ámbar.

---

# `app.html` — decidir si se borra o se redirige

Anotado el 2026-08-28. **Es cambio propio de Roberto**, porque el archivo está
en Search Console y esa es la parte que no se ve desde el repo.

Medido el 2026-08-27, los tres datos que hacen falta para decidir:

1. **No lo enlaza NADIE.** Cero `<a href="app.html">` en todo el repo. Las 9
   menciones que existen son todas comentarios de código.
2. **No está en `sitemap.xml`.** El sitemap tiene 4 `<loc>`: la raíz,
   `soporte.html`, `privacidad.html` y `eliminar-cuenta.html`.
3. **`robots.txt` no lo bloquea.** Es `Allow: /` a secas.

O sea: es un archivo huérfano de enlaces, servido pero no referenciado, con
`canonical` apuntando a la raíz.

⚠ El comentario del `<head>` de `index.html` dice que **no se borró a
propósito**: `evento.html` y `entidad.html` publican links a `/app.html` que
ya circulan en manos de la gente, y un 404 los rompería. Eso hay que
re-verificarlo antes de decidir — si esos links ya no se publican, el motivo
para conservarlo caducó.

Las dos salidas son borrar, o dejar un redirect 301 a la raíz. La consolidación
de indexación hoy la hace el `canonical`; el redirect la haría más explícita.

---

# Descartado

Lo que se evaluó y NO se va a hacer. Está acá para que no reaparezca como
idea nueva dentro de seis meses.

- **Publicar la agenda como páginas web** (salones, orquestas y eventos
  generados desde Supabase, por provincia o por ritmo). Se evaluó y se
  descartó: **canibaliza la app** y **expone a raspado el único activo que la
  diferencia**, que son los datos curados a mano.
