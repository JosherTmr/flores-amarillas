# Auditoría responsive inicial — "Tus flores amarillas"

Objeto: https://joshertmr.github.io/flores-amarillas/ · fuente `publicar/index.html`
Referencia: skill `responsive-design` (SKILL.md + references/details.md, fluid-layouts.md, breakpoint-strategies.md, container-queries.md)

## Nota metodológica

El panel del navegador escala la emulación. En los viewports con `dpr=1` (1920, 1440, 1366, 1024, 768, 844x390, 1024x600) `innerWidth === documentElement.clientWidth === rect(.night)` — medición fiable. En móvil el panel fuerza `dpr=2` e `innerWidth`/`innerHeight` mienten (en "390x844" devolvió `515x1115`). Cuatro sondas independientes (`clientWidth/Height`, `rect(.night)`, `100vmin`, `100vh`) confirmaron que el viewport CSS real sí era el solicitado. Todas las cifras de móvil están tomadas contra `documentElement.clientWidth/clientHeight`, y el desbordamiento horizontal se calcula geométricamente (caja real del velo) en vez de fiarse de `scrollWidth`.

## 1. Tabla resumen por viewport

`W×H` = viewport CSS real medido. "Velo fuera" = px de `.mensaje::before` fuera del viewport (izq/der). "Escena sangra" = unión de `.flowers *` fuera del viewport (izq/der/abajo).

| Solicitado | CSS real (innerWH reportado) | ar | scrollW vs clientW | scrollH vs clientH | Velo fuera L/R | Escena sangra L/R/B | Elem. contenido fuera | Colisiones | Veredicto |
|---|---|---|---|---|---|---|---|---|---|
| 1920×1080 | 1920×1080 (1920×1080) | 1.778 | 1920 = 1920 | 1281 vs 1080 (+201) | 72 / — | 0/0/199 | 0 | 0 | PASA |
| 1440×900 | 1440×900 (1440×900) | 1.600 | 1440 = 1440 | 1126 vs 900 (+226) | 82 / — | 0/0/165 | 0 | 0 | PASA |
| 1366×768 | 1366×768 (1366×768) | 1.779 | 1366 = 1366 | 1060 vs 768 (+292) | 83 / — | 0/0/144 | 0 | 0 | PASA (margen 28 px) |
| 1024×768 | 1024×768 (1024×768) | 1.333 | 1024 = 1024 | 1060 vs 768 (+292) | 90 / — | 0/0/144 | 0 | 0 | PASA (margen 28 px) |
| 768×1024 | 768×1024 (768×1024) | 0.750 | **1014 vs 768 (+246)** | 1168 vs 1024 (+144) | **246 / 246** | 3/5/141 | 0 | 0 | PASA con reservas |
| 430×932 | 430×932 (568×1232 ✗) | 0.461 | 568 vs 430 (+138) | n/d fiable | 138 / 138 | 1/1/79 | 0 | 0 | PASA |
| 393×852 | 393×852 (519×1126 ✗) | 0.461 | 519 vs 393 (+126) | n/d fiable | 126 / 126 | 1/1/72 | 0 | 0 | PASA |
| 390×844 | 390×844 (515×1115 ✗) | 0.462 | 515 vs 390 (+125) | n/d fiable | 125 / 125 | 0/9/73 | 0 | 0 | PASA |
| 375×667 | 375×667 (496×881 ✗) | 0.562 | 496 vs 375 (+121) | n/d fiable | 120 / 120 | 1/11/70 | 0 | 0 | PASA con reservas |
| 360×800 | 360×800 (475×1056 ✗) | 0.450 | 475 vs 360 (+115) | n/d fiable | 115 / 115 | 0/11/67 | 0 | 0 | PASA |
| **844×390** (móvil apaisado — añadido) | 844×390 | 2.164 | 844 = 844 | — | 94 / — | 0/0/72 | **6** | **1** | **FALLA** |
| **1024×600** (portátil corto — añadido) | 1024×600 | 1.707 | 1024 = 1024 | — | 90 / — | 0/0/114 | **3** | **1** | **FALLA** |
| **349×261** (nativo del panel — añadido) | 349×261 | 1.337 | **456 vs 349 (+107)** | 506 vs 261 (+245) | 107 / — | — | **3** | **4** | **FALLA** |

Los tres viewports que fallan los añadió el auditor: los diez de la lista del cliente no contenían ningún caso **apaisado y bajo**, que es donde el diseño se rompe.

## 2. Problemas numerados

### P1 · MENSAJE-ALTO-FIJO — BLOQUEANTE
- Selector: `.mensaje` dentro de `@media (min-aspect-ratio: 1/1)`.
- Culpable: `top: 50%; transform: translateY(-50%)` sin restricción de altura, con `padding: 3rem 2.4rem`, `gap: 1.15rem` y tipografías cuyo valor preferido es solo `vw`: `.mensaje__titulo { clamp(1.95rem, 8.6vw, 4.8rem) }`, `.mensaje__remate { clamp(1.55rem, 6.2vw, 2.6rem) }`, `.mensaje__linea { clamp(1.15rem, 4.4vw, 1.72rem) }`.
- Causa raíz: la altura del bloque es **712.1 px constantes** en cuanto los `clamp()` tocan su máximo (≥893 px de ancho). Nada la reduce cuando falta altura: la única guarda de pantalla baja, `@media (max-aspect-ratio: 1/1) and (max-height: 700px)`, es **exclusiva de vertical**. El apaisado no tiene equivalente.
- Afecta: todo apaisado con altura < ~712 px. 1366×768 y 1024×768 sobreviven con solo **28 px de margen**.
- Evidencia:
  - 844×390: `.mensaje` = `16.9,-152.8 → 362.5,542.8` → **153 px fuera por arriba y 153 por abajo**. `.mensaje__fecha` `t=-104.8 b=-88.8` → invisible. `.mensaje__titulo` `t=-70.4` → primera línea cortada. `.mensaje__remate` ("Te amo.") `t=424.4` → **invisible**. `.mensaje__firma` `t=478.8` → **invisible**. Colisión `nota × cancion = 155×34 px`.
  - 1024×600: `.mensaje` `t=-56 b=656`; `.fecha` `t=-8 b=8` (mitad cortada); `.firma` `t=592 b=608` (mitad cortada); colisión `remate × cancion = 113×31`.
  - 349×261: `.fecha` `b=-3.4` (fuera), `.firma` `t=264.4 > 261` (fuera); 4 colisiones.
- Severidad: BLOQUEANTE. El remate "Te amo." y la firma desaparecen al girar el teléfono.

### P2 · VELO-DESBORDA — IMPORTANTE
- Selector: `.mensaje::before`. Culpable: `inset: -45% -32%`.
- Causa raíz: los `%` de `inset` se resuelven contra la caja de `.mensaje`, así que el velo mide siempre **164 % del ancho × 190 % del alto** del mensaje. Crece con él, sin techo.
- Afecta: todos. Es **la única fuente de desbordamiento horizontal de documento de toda la página**.
- Evidencia: 768×1024 → velo `L=-246 R=1014` = 246 px fuera por lado; coincide exacto con `scrollWidth 1014` vs `clientWidth 768`. 430×932 → 138/138 · 393×852 → 126/126 · 390×844 → 125/125 · 375×667 → 120/120 · 360×800 → 115/115. Apaisado: 1920 → 72 · 1440 → 82 · 1366 → 83 · 1024 → 90. Vertical: velo `B=1126` y `scrollHeight` = 1126 exacto en 1440×900.
- Severidad: IMPORTANTE. Invisible (el degradado llega a `transparent`), pero es lo que obliga a `overflow: hidden` horizontal.

### P3 · BOTON-TACTIL — IMPORTANTE
- Selector: `.cancion`. Culpable: `padding: 0.5rem 1rem 0.5rem 0.8rem` + `font-size: 0.66rem`, sin `min-height`.
- Afecta: **los 13 viewports, sin excepción**.
- Evidencia: caja **184.5 × 34.0 px** en todos (`pass44 = false`). Altura 34 px frente al mínimo de 44. `font-size` resuelto **10.56 px** con `letter-spacing: 0.2em` en mayúsculas.
- Respaldo: SKILL.md §Best Practices 7 y §Common Issues.

### P4 · REPRODUCTOR-VISIBLE — IMPORTANTE (requisito del cliente)
- Selector: `.reproductor` + su bloque en `@media (min-aspect-ratio: 1/1)`.
- Culpables: `opacity: 0.26`, `:hover{opacity:.8}`, `width: clamp(124px,16vmin,168px)`, `aspect-ratio: 16/9`, `pointer-events: auto`.
- Evidencia: caja de **124×69.8 px** (móvil) a **168×94.5 px** (1920). Participa en colisiones: en 349×261 choca con `.cancion` (69.7×34) y con `.mensaje__nota` (124×27.5).

### P5 · TITULO-MEDIDA-ESTRECHA — IMPORTANTE
- Selector: `.mensaje { max-width: 36ch }` frente a `.mensaje__titulo` de hasta `4.8rem`.
- Causa raíz: `ch` se resuelve contra **la fuente del contenedor** (Jost 16 px), no contra la del título. Columna de **268.8 px** para una tipografía de **76.8 px**.
- Evidencia: `.mensaje__titulo` = `268.8 × 301.0 px`, `font-size: 76.8px`, `line-height: 75.26px` → **4 líneas a ~3.5 caracteres por línea**.

### P6 · TITULO-A-SANGRE-TABLET — IMPORTANTE
- Selector: `.mensaje__titulo` en vertical; `.mensaje { padding: max(2.2rem,4.5vh) 1.4rem 2.6rem }`.
- Culpable: `clamp(1.95rem, 8.6vw, 4.8rem)` — valor preferido de `vw` puro, sin término base. No alcanza el tope hasta 893 px, así que entre ~700 y 893 px el título es desproporcionado.
- Evidencia (768×1024): título = `723.2×129.4 px`, `font-size: 66.048px` → **94.2 % del ancho del viewport**, con solo 22.4 px de margen lateral.
- Es literalmente el "se ve bien en PC pero queda enorme en tablet" del encargo.

### P7 · TEXTO-PEQUENO — MENOR/IMPORTANTE
- `.mensaje__fecha { 0.72rem }`, `.mensaje__firma { 0.7rem }`, `.cancion { 0.66rem }` — **valores `rem` fijos que nunca escalan**, todos con `letter-spacing` 0.2–0.34em en mayúsculas.
- Evidencia: `.fecha` = **11.52 px** idéntico en los 13 viewports; `.firma` = **11.2 px**; `.cancion` = **10.56 px**. En 375×667 `.nota` cae a **12.16 px**.
- IMPORTANTE para `.cancion` (es un control interactivo), MENOR para el resto.

### P8 · ESCENA-SIN-CONTENEDOR — IMPORTANTE (estructural)
- Selector: `.flowers`. Culpable: `position: relative; transform: scale(0.9)` con **ancho y alto 0** (medido `0×0` en todos los viewports). Descendientes `absolute` con `left: -460%`, `left: -450%`, `left: -215%`, `.long-g--1..7 { left: -42vmin … 42vmin }`.
- Consecuencia: al ser ancla de tamaño cero, la escena **no se puede recortar, dimensionar ni desplazar como unidad**. Lo único que impide scroll de documento es `body { overflow: hidden }`.
- Evidencia (unión de `.flowers *` fuera del viewport, izq/der/abajo): 1920 → 0/0/199 · 1440 → 0/0/165 · 1366 → 0/0/144 · 1024×768 → 0/0/144 · 1024×600 → 0/0/114 · 768×1024 → 3/5/141 · 844×390 → 0/0/72 · 430×932 → 1/1/79 · 393×852 → 1/1/72 · 390×844 → 0/9/73 · 375×667 → 1/11/70 · 360×800 → 0/11/67.

### P9 · FLOWER-4-SIN-REGLA — MENOR
- **No existe ninguna regla CSS para `.flower--4`** (verificado recorriendo `document.styleSheets`: `hasF4Rule = false`). Hereda solo la base `.flower`: `left: 0px`, `animation-name: none`.
- Evidencia (1024×600): `.flower--1` = `499,168 21×378` con `moving-flower-1`; `.flower--4` = `512,249 8×297` sin animación. Es una cuarta flor **estática superpuesta a la primera**, con tallo de 55 vmin en vez de 70.

### P10 · DECLARACION-INVALIDA — MENOR
- `.flower__g-fr { left: vmin }` — `vmin` sin número no es longitud válida; el parser descarta la declaración. `rule.style.left === ""`, `getComputedStyle(...).left === "0px"`.

### P11 · BURBUJAS-PORCENTAJE-FIJO — MENOR
- `.bubble:nth-child(1..20)`: `top`/`left` en porcentajes cableados con `height`/`width` en `vmin` y `box-shadow: inset 0 0 0 Nvmin`. Dos sistemas de unidades para posición y tamaño.
- Evidencia: en 768×1024 una burbuja alcanza `right = 776` (8 px fuera); en 375×667 y 360×800 la escena llega a `right = 386` y `371` (11 px fuera).

### P12 · 100VH-MOVIL — MENOR
- `body { min-height: 100vh }`. SKILL.md §Common Issues y details.md líneas 449–465 prescriben `100dvh` / `100svh`. Con la barra de URL dinámica, la línea base de la escena (`align-items: flex-end`) se desplaza.

### P13 · BREAKPOINT-SOLO-ASPECTO — IMPORTANTE (estructural)
- `@media (max-aspect-ratio: 1/1)` y `@media (min-aspect-ratio: 1/1)`.
- Problema A: en un viewport **exactamente cuadrado** (768×768, 800×800) **ambas consultas coinciden** y gana la apaisada por orden de cascada. Accidental, no diseñado.
- Problema B (el grave): la relación de aspecto **no puede expresar "no hay altura suficiente"**, que es la restricción real. Causa estructural de P1.

## 3. Recomendación técnica por problema

| # | Qué cambiar | A qué | Técnica |
|---|---|---|---|
| P1 | Añadir el eje de **altura** a la escala tipográfica y un tope de altura al bloque | `clamp(min, base + N·vmin, max)` en vez de `vw` puro — p.ej. `.mensaje__titulo: clamp(1.6rem, 1rem + 6.4vmin, 4.2rem)`; `.mensaje__remate: clamp(1.25rem, 0.8rem + 4.6vmin, 2.4rem)`. En apaisado `max-block-size: calc(100dvh - 2rem)` y `padding`/`gap` con `clamp(0.4rem, 1.4vmin, 1.15rem)` | fluid-layouts.md §Calculating Fluid Values (`base + pendiente`, nunca `vw` puro); details.md Pattern 2 |
| P1 alt. **preferida** | Convertir el mensaje en componente con **container query** | `.mensaje-wrap { container-type: size; container-name: msg; position: fixed; inset: 0 }` y dimensionar con `cqi`/`cqh`: `font-size: clamp(1.6rem, 7cqmin, 4.2rem)`. Reacciona a ancho **y** alto sin media queries | container-queries.md; details.md Pattern 1 |
| P2 | Desacoplar el velo de la caja del mensaje | `.mensaje::before` de `inset:-45% -32%` a **`position: fixed; inset: 0`** con el foco en el propio degradado: `radial-gradient(ellipse 45% 40% at 22% 50%, rgba(6,5,14,.94), rgba(6,5,14,0) 100%)` | fluid-layouts.md §min()/max() |
| P3 | Tamaño táctil | `min-block-size: 44px; min-inline-size: 44px; padding-block: 0.75rem; padding-inline: 1.1rem; font-size: clamp(0.72rem, 0.66rem + 0.35vw, 0.82rem)` | SKILL.md §Best Practices 7 |
| P4 | Ocultar el reproductor **sin** sacarlo del árbol de render (la IFrame API se estrangula con `display:none`) | Mantener `position: fixed` y aplicar `inline-size: 1px; block-size: 1px; opacity: 0; pointer-events: none; border: 0; clip-path: inset(50%)`. **Eliminar** `aspect-ratio`, la regla `:hover/:focus-within` y el bloque `@media (min-aspect-ratio:1/1){ .reproductor{…} }`. Al ser `fixed` no deja hueco. Verificar que el audio sigue sonando | requisito de cliente |
| P5 | Medida en el **texto**, no en el contenedor | Quitar `max-width: 36ch` de `.mensaje`; usar `.mensaje { inline-size: min(46ch, 38vw) }` y `.mensaje__titulo { max-inline-size: 11ch }`. Bajar el tope del título en apaisado de `4.8rem` a `~3.6rem` | fluid-layouts.md §Content-Based Widths, §Intrinsic Sizing |
| P6 | Cortar el crecimiento del título entre 700 y 893 px | `clamp(1.95rem, 1.9rem + 4.1vw, 3.9rem)` y `padding-inline: clamp(1.4rem, 5vw, 3rem)` | fluid-layouts.md §Type Scale Generator |
| P7 | Fluidificar los fijos | `.fecha`/`.firma`: `clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem)`; `.cancion`: `clamp(0.72rem, 0.66rem + 0.35vw, 0.82rem)`. Suelo de `.nota` en rama corta de `0.76rem` a `~0.85rem` | fluid-layouts.md §Complete Type Scale |
| P8 | Dar **caja real** a la decoración | `.escena { position: fixed; inset: 0; overflow: clip; pointer-events: none; z-index: 0; container-type: size }` y dentro `.flowers { position: absolute; left: 50%; bottom: 0 }` | details.md §3 Layout Patterns |
| P9 | `.flower--4` | Darle regla propia al nivel de sus hermanas, o eliminar el marcado | — |
| P10 | `.flower__g-fr { left: vmin }` | `left: 1vmin` | — |
| P11 | Burbujas | Dentro de la capa `.escena` (que ya recorta) y misma base para posición y tamaño: `inset-block-start: N%; inset-inline-start: N%` con `inline-size: Ncqmin` | container-queries.md |
| P12 | Unidades de viewport | `body { min-height: 100vh; min-height: 100svh }` y `dvh` para el tope de `.mensaje` | details.md §Viewport Units |
| P13 | Breakpoints | Aspecto solo para la **disposición**; eje de altura independiente: `(min-aspect-ratio:1/1) and (max-height:780px)`, `(max-height:640px)`, `(max-height:480px)`. Empate en 1/1 con `(max-aspect-ratio: 0.9999/1)`. Mejor: container queries sobre `.escena`/`.mensaje-wrap` | breakpoint-strategies.md |

## 4. Veredicto sobre `overflow: hidden`

Caso **mixto**. Hay tres cosas hoy tapadas por la misma línea `body { overflow: hidden }`:

**(A) Decoración a sangre LEGÍTIMA — se puede y se debe recortar.**
La escena de flores es un fondo que se recorta a propósito, como `background-size: cover`. El sangrado es **asimétrico y casi siempre por abajo** (199 px en 1920×1080, 165 en 1440×900, 144 en 1366×768 y 1024×768, 141 en 768×1024, 67–79 px en móviles), con apenas 0–11 px por los lados. Es una maceta que se sale del encuadre por la base. **Ningún elemento de contenido está ahí.**
→ Pero el recorte debe ser **explícito y contenido**: hoy lo hace `body{overflow:hidden}` (documento entero); debe hacerlo `.escena { position: fixed; inset: 0; overflow: clip }` (solo la capa decorativa).

**(B) Desbordamiento REAL DE CONTENIDO — prohibido taparlo.**
En apaisado bajo, `.mensaje` y sus hijos salen del viewport y `overflow:hidden` los está **borrando literalmente**: en 844×390 desaparecen `.fecha`, `.remate` ("Te amo.") y `.firma`, y el título pierde su primera línea. Se corrige con P1/P13, **nunca** con recorte.

**(C) Caso intermedio: el velo del mensaje.**
`.mensaje::before` sale hasta 246 px por lado y es la única causa del desbordamiento horizontal del documento. Es decorativo en apariencia pero es **hijo del bloque de contenido** y su tamaño lo dicta el contenido. Se arregla en origen (P2).

**Separación arquitectónica — tres capas, cada una con su política:**

| Capa | Contenido | Política de overflow |
|---|---|---|
| `.escena` (z 0) | `.night`, `.flowers`, `.bubbles` | `position: fixed; inset: 0; overflow: clip; pointer-events: none` — recorte declarado y local |
| `.interfaz` (z 300) | `.mensaje`, `.marca`, `.cancion`, `.reproductor` | **`overflow` sin tocar.** Todo debe caber por construcción; cualquier recorte aquí es un bug |
| `body` / `html` | — | **Eliminar `overflow: hidden`.** Si tras P1, P2 y P8 el documento sigue haciendo scroll, queda un bug real |

Criterio de cierre: **borrar `body{overflow:hidden}` y comprobar que `documentElement.scrollWidth === clientWidth` y `scrollHeight === clientHeight` en los 13 viewports.** Hoy falla en todos.

## 5. Diagnóstico de raíz

**El layout necesita rehacerse estructuralmente. No bastan parches.**

Tres sistemas de posicionamiento incompatibles superpuestos sobre el mismo `body`:

1. **El CodePen (`.flowers`)**: `absolute` sobre un ancla de tamaño **0×0**, unidades `vmin`, desplazamientos de hasta `-460%`. Ancla al borde inferior vía `body { display:flex; align-items:flex-end }`.
2. **La interfaz**: cuatro elementos `fixed` **independientes entre sí**, cada uno con sus propios `clamp()` de offset. Ningún contrato garantiza que no se pisen — y se pisan (9 colisiones medidas en 3 viewports).
3. **Las media queries de aspecto**: reorganizan el punto 2 pero no el 1, y solo conocen el eje horizontal.

Cada colisión medida (`nota×marca`, `nota×reproductor`, `remate×cancion`, `cancion×reproductor`) es consecuencia de que nadie reserva espacio para nadie.

**Arquitectura propuesta — dos capas fijas y UNA rejilla de interfaz:**

```
body                         (sin overflow, sin flex, sin min-height:100vh)
├── .escena                  position:fixed; inset:0; overflow:clip;
│                            pointer-events:none; z-index:0;
│                            container-type:size            ← recorta la sangre
│   ├── .night  ├── .flowers  └── .bubbles
│
└── .interfaz                position:fixed; inset:0; z-index:300;
                             container-type:size;           ← contexto de CQ
                             display:grid;
                             padding: clamp(.9rem, 3vmin, 2.2rem);
                             pointer-events:none
    ├── .mensaje             grid-area; max-block-size:100%; pointer-events:none
    ├── .cancion             grid-area; pointer-events:auto; min-block-size:44px
    ├── .marca               grid-area; pointer-events:none
    └── .reproductor         1×1px, opacity:0, clip-path (fuera de la rejilla)
```

Los cuatro elementos pasan a ser **celdas de una misma rejilla**, así que las colisiones dejan de ser posibles **por construcción**. La rejilla cambia con `grid-template-areas`:

- **Vertical (`ar < 1`):** texto arriba, controles abajo.
- **Apaisado alto (`ar ≥ 1`, `height ≥ 780px`):** texto en columna izquierda.
- **Apaisado bajo (`ar ≥ 1`, `height < 780px`):** misma rejilla, mensaje escalado con `cqmin`, `.mensaje__nota` con `display:none` en el paso más agresivo, conservando siempre título, remate y firma.

Toda la tipografía pasa a tokens fluidos **bidimensionales** (`cqmin`/`vmin`, nunca `vw` puro): un viewport corto reduce el texto igual que uno estrecho. Eso elimina P1, P5, P6, P7 y P13 a la vez.

**Orden de ejecución:** P8 y la rejilla de interfaz primero (sustrato), luego P1/P13 (tipografía bidimensional + eje de altura), luego P2 y P4, y por último P3, P7, P9, P10, P11, P12.
