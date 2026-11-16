# Sistema de Diseño — App Activa Biblioteca

Guía visual y de componentes para **Biblioteca Municipal**. Derivada de los mockups de
`docs/ux-ui/`, los [casos de uso](./casos-de-uso.md) y la [especificación técnica](./TECH_SPEC.md).

**Stack de referencia:** HTML5 + CSS3 + TypeScript (sin framework de UI). Los tokens se
expresan como **variables CSS nativas** (`:root { --token }`) para poder consumirse directamente
desde la capa *View* del patrón MVC.

> **Mobile-first y responsive.** Los mockups de `docs/ux-ui/` son la vista **desktop** (la meta
> visual), pero el CSS se escribe **de móvil hacia arriba**: los estilos base son los del móvil y
> cada `@media (min-width: …)` *añade* complejidad para pantallas mayores. Nunca al revés. La
> metodología y el comportamiento por componente están en [§4.5 Responsive](#45-responsive--mobile-first).

---

## 1. Principios

1. **Dos superficies, una identidad.** El sistema tiene dos contextos visuales que comparten
   tokens pero cambian de layout:
   - **Público (OPAC):** catálogo en línea consultable sin identificarse. Cabecera horizontal
     clara, fondo crema, tono acogedor. Actores: *Usuario no registrado*, *Socio*.
   - **Gestión (back-office):** panel con barra lateral oscura. Actores: *Bibliotecario*, *Admin*.
2. **La disponibilidad es el dato rey.** En catálogo y resultados, el estado de los ejemplares
   (disponibles / prestados / reservados) se comunica siempre con color + texto + número, nunca
   solo con color.
3. **Permisos visibles.** El rol condiciona qué se ve y qué se puede hacer. Las acciones
   restringidas se muestran deshabilitadas con el motivo, no se ocultan sin más (ver mockup 3 y 4).
4. **Accesible por defecto.** Contraste AA mínimo, foco visible, objetivos táctiles ≥ 44 px,
   nunca color como único portador de significado.
5. **Sobrio y legible.** Mucho espacio en blanco, tipografía grande, bordes suaves. Es una
   herramienta de uso diario para personal y un servicio público para la ciudadanía.
6. **Mobile-first.** Se diseña y se codifica primero para el móvil (≈360 px), la pantalla más
   restrictiva. Lo esencial —buscar, ver disponibilidad, identificarse, prestar/devolver— debe
   funcionar con una sola mano y una sola columna. El layout de los mockups (multi-columna,
   sidebar, tablas anchas) es el **estado ampliado** que se progresivamente enriquece con media
   queries `min-width`.

---

## 2. Tokens de color

> Valores extraídos de los mockups. Define todos en `:root` y consúmelos por nombre — nunca
> escribas un hex suelto en un componente.

### 2.1 Marca y superficies

```css
:root {
  /* Verde de marca (botones primarios, logo, enlaces) */
  --color-brand:            #1F5540;  /* verde bosque — acción primaria */
  --color-brand-hover:      #17412F;  /* hover/pressed */
  --color-brand-strong:     #163A2B;  /* "Entrar", cabecera gestión */
  --color-ink-button:       #1A2420;  /* casi negro verdoso — botón "Continuar" */

  /* Fondos */
  --color-bg:               #F3F1E9;  /* crema — fondo de página */
  --color-surface:          #FFFFFF;  /* tarjetas, inputs, paneles */
  --color-surface-muted:    #F7F5EE;  /* relleno sutil, filas tabla, campos readonly */
  --color-sidebar:          #14201A;  /* barra lateral y footer (verde casi negro) */
  --color-sidebar-ink:      #EDEDE6;  /* texto sobre sidebar/footer */
  --color-sidebar-muted:    #9BA69F;  /* texto secundario sobre sidebar */

  /* Texto */
  --color-text:             #1A1D1A;  /* titulares y cuerpo principal */
  --color-text-muted:       #6E6E66;  /* descripciones, metadatos, labels */
  --color-text-subtle:      #8E8E84;  /* placeholders, pies */

  /* Bordes y líneas */
  --color-border:           #E4E1D6;  /* borde de tarjetas e inputs */
  --color-border-strong:    #CFCBBD;  /* separadores, foco de input */
  --color-focus:            #1F5540;  /* anillo de foco (= marca) */
}
```

### 2.2 Estado (semánticos)

Usados en badges de disponibilidad, alertas y validación. Cada uno con pareja fondo/texto que
cumple contraste AA sobre su fondo.

```css
:root {
  /* Éxito / disponible */
  --color-success:          #1F5540;
  --color-success-bg:       #DCE9DD;

  /* Aviso / todos prestados · ficha bloqueada */
  --color-warning:          #8A5A24;
  --color-warning-bg:       #F4E7C9;

  /* Error / acceso restringido · validación */
  --color-danger:           #9B2C2C;
  --color-danger-bg:        #F8E3E2;

  /* Informativo / consulta como visitante */
  --color-info:             #1F5540;
  --color-info-bg:          #E3EFE4;
}
```

### 2.3 Roles (chips de rol)

```css
:root {
  --role-socio-bg:          #D7E8DB;  --role-socio-text:    #1F5540;
  --role-staff-bg:          #E7E3F3;  --role-staff-text:    #4A3E7A;  /* Bibliotecario / Admin */
}
```

### 2.4 Categorías / materias (portadas de libro)

Cada materia tiene un color de lomo. Se usa en el bloque de portada generado (iniciales sobre
color) y en la etiqueta de categoría. Mapea cada `Genre`/materia a uno de estos; reparte de forma
estable (p. ej. por hash del nombre) para las materias sin color asignado.

```css
:root {
  --cat-novela:             #2E5A46;  /* verde — Novela / Literatura española */
  --cat-geografia:          #3F597A;  /* azul acero — Geografía · Viajes */
  --cat-juvenil:            #8A5A24;  /* ocre — Literatura juvenil */
  --cat-infantil:           #7B3B3B;  /* granate — Literatura infantil */
  --cat-historia:           #5A4A2E;  /* marrón — Historia / Ensayo */
  --cat-cifi:               #35506B;  /* azul profundo — Ciencia ficción */
  --cat-default:            #44524A;  /* gris verdoso — sin materia */
}
```

> Texto siempre **blanco** sobre el color de categoría (todos superan 4.5:1).

---

## 3. Tipografía

Los mockups usan un **grotesco sans-serif** de amplia caja para titulares y un sans legible para
cuerpo. Stack recomendada (sin dependencia obligatoria de Google Fonts; degrada a sistema):

```css
:root {
  --font-display: "Inter", "Helvetica Neue", Arial, system-ui, sans-serif; /* titulares */
  --font-body:    "Inter", system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
  --font-mono:    "IBM Plex Mono", ui-monospace, SFMono-Regular, Menlo, monospace; /* ISBN, códigos */
}
```

### Escala tipográfica

Tamaños **fluidos** con `clamp(mín, preferido, máx)`: el primer valor es el del **móvil** y crece
hasta el valor desktop de los mockups. Así no hace falta redefinir tamaños en cada breakpoint.

| Token | Móvil → Desktop | Peso | Uso |
|-------|-----------------|------|-----|
| `--text-display` | 30 → 48 px | 700 | H1 hero público («¿Qué quieres leer hoy?») |
| `--text-h1` | 26 → 34 px | 700 | Título de página («Resultados…», «Alta de volumen») |
| `--text-h2` | 20 → 24 px | 700 | Secciones («Explorar por materia», «Datos del título») |
| `--text-h3` | 17 → 18 px | 600 | Título de tarjeta, subsección numerada |
| `--text-body` | 16 px (fijo) | 400 | Cuerpo, campos, descripciones |
| `--text-sm` | 14 px (fijo) | 400 | Metadatos, autor, ayuda de campo |
| `--text-overline` | 12 px (fijo) | 600 | Etiquetas en versalitas (`LITERATURA JUVENIL`, `CATÁLOGO EN LÍNEA`) — `letter-spacing: .06em; text-transform: uppercase` |
| `--text-mono` | 15 px (fijo) | 500 | ISBN, signaturas, códigos de barras |

> El cuerpo se mantiene en **16 px** en todos los tamaños: nunca por debajo, para evitar el zoom
> automático de iOS en inputs y garantizar legibilidad en móvil.

```css
:root {
  /* clamp(min-móvil, fluido-con-vw, max-desktop) */
  --text-display: 700 clamp(1.875rem, 1.2rem + 3.4vw, 3rem)/1.1 var(--font-display);
  --text-h1:      700 clamp(1.625rem, 1.3rem + 1.6vw, 2.125rem)/1.2 var(--font-display);
  --text-h2:      700 clamp(1.25rem, 1.1rem + 0.7vw, 1.5rem)/1.25 var(--font-display);
  --text-h3:      600 clamp(1.0625rem, 1rem + 0.3vw, 1.125rem)/1.33 var(--font-body);
  --text-body:    400 1rem/1.5 var(--font-body);
  --text-sm:      400 0.875rem/1.43 var(--font-body);
}
```

---

## 4. Espaciado, radios, sombras

Escala base de **4 px**.

```css
:root {
  --space-1: 4px;   --space-2: 8px;   --space-3: 12px;  --space-4: 16px;
  --space-5: 20px;  --space-6: 24px;  --space-8: 32px;  --space-10: 40px;
  --space-12: 48px; --space-16: 64px; --space-20: 80px;

  /* Radios */
  --radius-sm:   6px;   /* inputs, botones, chips */
  --radius-md:   10px;  /* tarjetas, paneles, alertas */
  --radius-lg:   14px;  /* tarjetas grandes, hero, sidebar logo */
  --radius-full: 999px; /* pills de disponibilidad y rol */

  /* Sombras (sutiles; la UI se apoya más en bordes que en sombra) */
  --shadow-sm: 0 1px 2px rgba(20, 32, 26, .04);
  --shadow-md: 0 2px 8px rgba(20, 32, 26, .06);
  --shadow-pop: 0 8px 24px rgba(20, 32, 26, .12); /* dropdowns, modales */

  /* Layout */
  --container-max: 1200px;
  --sidebar-width: 264px;
  --gutter: clamp(16px, 4vw, 24px); /* padding lateral del contenedor: 16px móvil → 24px desktop */
}
```

**Grid:** contenedor público centrado a `--container-max` con `padding` lateral de `--gutter`
(16 px en móvil, 24 px en desktop). En gestión, `--sidebar-width` fija a partir de `lg` + contenido
fluido. Tarjetas de catálogo en grid responsivo `repeat(auto-fill, minmax(min(100%, 240px), 1fr))`
— el `min(100%, …)` garantiza **1 columna** en móvil sin desbordes.

### Breakpoints

Mobile-first: los estilos base son de móvil; cada token marca el `min-width` a partir del cual se
añade complejidad. No hay media queries `max-width`.

```css
/* Convención de uso (CSS no soporta variables en @media, documentadas aquí como referencia) */
--bp-sm:  480px;   /* móvil grande / landscape */
--bp-md:  768px;   /* tablet — aparecen 2 columnas, nav se despliega */
--bp-lg:  1024px;  /* desktop — sidebar fija, filas de resultado a 2 zonas */
--bp-xl:  1280px;  /* desktop amplio — grids a 3–4 columnas */
```

| Rango | Ancho | Catálogo (grid) | Nav pública | Gestión |
|-------|-------|-----------------|-------------|---------|
| Base  | < 480 | 1 columna | menú hamburguesa | nav en drawer / barra inferior |
| `md`  | ≥ 768 | 2 columnas | nav horizontal visible | drawer; contenido 1 col |
| `lg`  | ≥ 1024 | 3 columnas | nav + búsqueda en cabecera | **sidebar fija** |
| `xl`  | ≥ 1280 | 4 columnas | — | aside de resumen a la derecha |

### 4.5 Responsive — mobile-first

**Cómo se escribe el CSS.** Estilos base = móvil (1 columna, apilado). Las media queries solo
*suman*:

```css
/* Base: móvil. Todo en una columna, de arriba abajo. */
.resultado { display: grid; gap: var(--space-4); }

/* md: la ficha y la disponibilidad empiezan a convivir */
@media (min-width: 768px) {
  .resultado { grid-template-columns: 1fr; }
}

/* lg: dos zonas lado a lado, como en el mockup desktop */
@media (min-width: 1024px) {
  .resultado { grid-template-columns: 1fr 320px; }
}
```

**Reglas transversales**

- **Objetivos táctiles ≥ 44×44 px** en móvil: botones, pills de acceso rápido, filas de tabla,
  stepper `− +`, ítems de nav. En móvil los botones de acción principal van a **ancho completo**.
- **Sin scroll horizontal del documento.** Lo que no cabe (tablas anchas, código) scrollea dentro
  de su propio contenedor `overflow-x: auto`, nunca la página.
- **Imágenes y media** con `max-width: 100%`. Bloques de portada mantienen `aspect-ratio`.
- **Una mano, un pulgar.** Las acciones primarias quedan al alcance; evita menús colgantes al
  borde superior en flujos de uso frecuente (préstamo, devolución).

**Comportamiento por zona (móvil → desktop)**

| Zona | Móvil (base) | `md` ≥ 768 | `lg` ≥ 1024 |
|------|--------------|-----------|-------------|
| **Cabecera pública** | logo + icono ☰; nav y acciones en menú desplegable | nav horizontal visible | + búsqueda integrada en cabecera |
| **Hero + login** | apilados: primero buscador, luego tarjeta «Identificarse» | 2 columnas | 2 columnas con más aire |
| **Grid de catálogo / novedades** | 1 columna | 2 columnas | 3–4 columnas |
| **Fila de resultado** | apilada: portada+datos arriba, disponibilidad debajo; stat tiles en fila de 3 | igual, más ancho | 2 zonas lado a lado (mockup 2) |
| **Mini-tabla por sede** | se convierte en lista etiquetada (`Sede · Ejempl · Prest · Disp`) | tabla | tabla |
| **Filtros (OPAC)** | ocultos tras botón `Filtrar` → panel/drawer deslizante | sidebar de filtros fija | sidebar fija |
| **Sidebar de gestión** | oculta; se abre como **drawer** desde icono ☰ (o barra inferior con los accesos clave) | drawer | **fija** a `--sidebar-width` |
| **Formulario de alta** | 1 columna; «Resumen del alta» colapsable al inicio o fijo al pie | 1 columna | aside de resumen a la derecha |
| **Tabla de búsqueda (gestión)** | tarjetas apiladas (título, editorial, ISBN, ejempl.) | tabla con scroll propio | tabla completa |
| **Barra de acciones** (`Descartar / Guardar / Dar de alta`) | fija al pie (`sticky bottom`), botones a ancho completo | alineada a la derecha | a la derecha |
| **Footer** | columnas apiladas en acordeón o lista | 2 columnas | 4 columnas |

---

## 5. Layouts

### 5.1 Cabecera pública
Logo cuadrado verde con inicial (`B`) + nombre en dos líneas · navegación horizontal
(`Inicio · Catálogo · Novedades · Agenda · Horario y sedes`) con pestaña activa resaltada en
`--color-success-bg` · a la derecha botones `Identificarse` (secundario) y `Hazte socio` (primario).
En páginas internas, la búsqueda sube a la cabecera (ver mockup 2).

### 5.2 Footer
Fondo `--color-sidebar`, texto claro, 4 columnas: *Horario · Sedes · Contacto · Accesibilidad*.
En listados se compacta a una sola línea.

### 5.3 Panel de gestión
Barra lateral oscura (`--color-sidebar`) fija con:
- Logo + etiqueta «Gestión».
- Selector **Sede de trabajo** (dropdown oscuro) — determina a qué sede se asignan altas/ejemplares.
- Nav principal: `Catálogo` (con contador), `Préstamos`, `Devoluciones`, `Reservas`, `Socios`, `Informes`.
- Grupo **SOLO ADMINISTRADOR**: `Usuarios y roles`, `Sedes` (solo visible con rol admin — CU 7, 12, 31, 33).
- Pie: avatar + nombre + rol + `Salir`.

Cabecera de contenido: breadcrumb (`Catálogo › Alta de volumen`), H1 y chip de acceso a la derecha
(`Acceso: bibliotecario o administrador`).

---

## 6. Componentes

### 6.1 Botones

| Variante | Aspecto | Uso |
|----------|---------|-----|
| **Primario** | fondo `--color-brand`, texto blanco, `--radius-sm` | acción principal: `Buscar`, `Hazte socio`, `Dar de alta`, `Añadir N ejemplares` |
| **Primario oscuro** | fondo `--color-ink-button` | acción de confirmación neutra: `Continuar` (login público) |
| **Secundario** | fondo `--color-surface`, borde `--color-border-strong`, texto `--color-text` | `Identificarse`, `Descartar cambios`, `Guardar y añadir otro`, `Comprobar` |
| **Outline marca** | borde `--color-brand`, texto `--color-brand`, fondo transparente | acción bloqueada por permiso con candado: `🔒 Identifícate para reservar` |
| **Enlace / texto** | sin fondo, texto `--color-brand`, subrayado | `Ver ficha completa →`, `Ver todo el catálogo →`, `Limpiar`, `Solicitar adquisición` |

```css
.btn { font: 600 1rem/1 var(--font-body); padding: 12px 20px; border-radius: var(--radius-sm);
       border: 1px solid transparent; cursor: pointer; }
.btn--primary   { background: var(--color-brand); color: #fff; }
.btn--primary:hover { background: var(--color-brand-hover); }
.btn--secondary { background: var(--color-surface); border-color: var(--color-border-strong); color: var(--color-text); }
.btn--outline   { background: transparent; border-color: var(--color-brand); color: var(--color-brand); }
.btn:focus-visible { outline: 3px solid color-mix(in srgb, var(--color-focus) 40%, transparent); outline-offset: 2px; }
.btn:disabled   { opacity: .5; cursor: not-allowed; }
```

### 6.2 Campos de formulario
Label en `--text-sm` peso 600 encima del campo. Campo con fondo blanco, borde `--color-border`,
`--radius-sm`, `padding: 12px 14px`. Obligatorios marcados con `*` rojo (`--color-danger`).
Placeholder en `--color-text-subtle`. Campos de solo lectura (p. ej. `Sede` en alta) con fondo
`--color-surface-muted`.

```css
.field label { display:block; font: 600 .875rem/1.4 var(--font-body); margin-bottom: 6px; }
.field .req { color: var(--color-danger); }
.input, .select, .textarea {
  width: 100%; background: var(--color-surface); border: 1px solid var(--color-border);
  border-radius: var(--radius-sm); padding: 12px 14px; font: var(--text-body); color: var(--color-text);
}
.input:focus { border-color: var(--color-focus); outline: 3px solid color-mix(in srgb, var(--color-focus) 25%, transparent); }
.input--readonly { background: var(--color-surface-muted); color: var(--color-text-muted); }
.input.is-error { border-color: var(--color-danger); }
```

Incluye: `input` texto, `select`, `textarea` (sinopsis), **stepper numérico** (`− 2 +` para cantidad
de ejemplares), **uploader** de portada (recuadro punteado «Sin portada» + botón `Subir imagen`),
y **search field** con botón integrado a la derecha.

### 6.3 Chips / pills
- **Accesos rápidos** (hero): pill `--color-surface-muted`, texto `--color-text`, clicable.
- **Disponibilidad** (badge): ver §6.4.
- **Rol:** pill con tokens de §2.3 (`Socio` verde, `Bibliotecario / Admin` lavanda).
- **Chip de acceso de página:** lavanda, p. ej. `Acceso: bibliotecario o administrador`.

### 6.4 Badge de disponibilidad
Comunica el estado agregado de ejemplares de un título. Siempre texto + color.

| Estado | Token fondo / texto | Texto ejemplo |
|--------|--------------------|---------------|
| Disponible | `--color-success-bg` / `--color-success` | `2 disponibles` |
| Todos prestados | `--color-warning-bg` / `--color-warning` | `Todos prestados` / `0 disponibles` |

```css
.badge { display:inline-block; font: 600 .8125rem/1 var(--font-body); padding: 6px 10px; border-radius: var(--radius-full); }
.badge--ok   { background: var(--color-success-bg); color: var(--color-success); }
.badge--warn { background: var(--color-warning-bg); color: var(--color-warning); }
```

### 6.5 Tarjeta de libro (catálogo)
Estructura: **bloque de portada** (color de categoría §2.4 + iniciales del título en blanco +
etiqueta de materia en versalitas) → **título** (`--text-h3`) → **autor/es** (`--text-sm` muted) →
**badge de disponibilidad**. Borde `--color-border`, `--radius-md`, fondo blanco.

### 6.6 Fila de resultado (OPAC)
Tarjeta ancha de dos zonas:
- **Izquierda:** portada + materia (overline) + título + autor + metadatos (`Editorial · Año · Págs · Idioma`) + `ISBN` y `Signatura` en mono + enlace `Ver ficha completa →`.
- **Derecha:** panel de disponibilidad → tres **stat tiles** (`Ejemplares / Prestados / Disponibles`) + mini-tabla por sede + nota (`Recógelo en…`, `Próxima devolución…`, reservas en espera) + CTA contextual (`🔒 Identifícate para reservar` si no hay disponibles; nada si hay stock → «no se reservan»).

### 6.7 Stat tile
Caja compacta con número grande (`--text-h2`) y etiqueta debajo (`--text-sm` muted). Fondo
`--color-surface-muted`, borde sutil. La columna «Disponibles» tiñe su fondo según stock
(verde si >0, ámbar si 0).

### 6.8 Tabla
Cabecera en `--text-sm` peso 600 muted, filas separadas por `--color-border`. Fila seleccionada
con fondo `--color-success-bg` y punto verde (ver «Título ya catalogado», mockup 4b). Números
alineados a la derecha. Usada en: resultados de búsqueda de gestión, disponibilidad por sede,
listado de socios (CU 12), auditoría (CU 33).

### 6.9 Alertas / banners
Caja `--radius-md`, icono + título + texto. Cuatro tonos semánticos (§2.2):

| Tono | Uso (casos) |
|------|-------------|
| **Info** (verde claro) | «Consultas como visitante. Para reservar necesitas identificarte» + enlace `Identificarse` |
| **Éxito** (verde claro) | resultado de comprobación ISBN sin duplicado: «No hay ningún título con este ISBN…» |
| **Aviso** (ámbar) | ficha bloqueada: «Este título ya existe… solo se añaden ejemplares» (CU 6/14) |
| **Error** (rojo) | «Acceso restringido»: socio intenta abrir una pantalla de gestión (CU 1, 4, 7, 9…) |

### 6.10 Segmented control (tabs)
Dos opciones grandes en tarjeta, con icono + título + descripción; la activa con borde
`--color-brand`. Ejemplo: `Título nuevo` / `Título ya catalogado` en Alta de volumen.

### 6.11 Panel resumen (aside)
Caja lateral de solo lectura que refleja el estado del formulario en vivo (`Resumen del alta`:
Modo, ISBN, Tipo, Ejemplares, **Pendiente** en rojo con campos que faltan). Patrón reutilizable
para confirmaciones de préstamo, sanción, etc.

### 6.12 Breadcrumb, avatar, contador
- **Breadcrumb:** `Inicio › Resultados`, separador `›`, último segmento sin enlace.
- **Avatar:** círculo con iniciales sobre color de marca (socios) o dorado (personal); tamaño 40 px.
- **Contador de nav:** número a la derecha del ítem (`Catálogo  1.284`).

---

## 7. Mapa de estados del dominio → color

Alinea la UI con los estados que manejan los casos de uso y el esquema (`Item.estado`,
`Loan`, `Sanction.estado`, `Reservation.estado`, `User.estado`).

| Dominio | Estado | Representación |
|---------|--------|----------------|
| Ejemplar (`Item`) | Disponible | badge éxito |
| | Prestado | texto neutro / stat «Prestados» |
| | Reservado | nota ámbar «N reservas en espera» |
| | Dañado (CU 24) | badge aviso + icono |
| | Perdido (CU 15) | badge error |
| Préstamo (`Loan`) | Activo | neutro |
| | Devuelto | muted |
| | Con retraso / devolución hoy | aviso ámbar («devolución prevista hoy») |
| Socio (`User`) | Activo | chip rol `Socio` |
| | Suspendido (CU 4) / Sancionado | badge error en ficha; bloquea préstamo/reserva/renovación |
| | De baja (CU 8) | muted |
| Sanción (`Sanction`) | Activa | badge error |
| | Anulada / vencida (CU 16) | muted |
| Solicitud (`Request`) | Pendiente / Resuelta (CU 22) | aviso / éxito |

**Regla de permisos (CU transversal «El usuario no tiene permisos»):** toda acción no autorizada
para el rol se renderiza como botón **outline con candado** + tooltip de motivo, o como banner de
**Acceso restringido** si es una pantalla completa. Nunca un error silencioso.

---

## 8. Iconografía y voz

- **Iconos:** set lineal de trazo ~1.75 px, esquinas suaves (coherente con los radios). Candado
  para permisos, lupa para búsqueda, `+`/`≡` en tabs, flecha `→` en enlaces de avance, `−`/`+`
  en steppers.
- **Tono de voz:** cercano, en segunda persona, claro y sin jerga («¿Qué quieres leer hoy?»,
  «No hace falta identificarse»). Mensajes de estado explican **qué pasa y qué puede hacer** el
  usuario, no solo el error.
- **Idioma:** español; fechas `DD/MM/AAAA`; ISBN y códigos de barras siempre en `--font-mono`.

---

## 9. Accesibilidad (checklist)

- [ ] Contraste texto/fondo ≥ 4.5:1 (3:1 para texto ≥ 24 px). Todas las parejas de §2 cumplen.
- [ ] Color nunca es el único indicador: disponibilidad, estados y validación llevan texto/icono.
- [ ] Foco visible en todo elemento interactivo (`:focus-visible` con anillo de §6.1).
- [ ] Labels asociados (`for`/`id`); `*` obligatorio también anunciado a lectores (`aria-required`).
- [ ] Objetivos táctiles ≥ 44×44 px; navegación operable por teclado (tabs, stepper, tabla).
- [ ] Jerarquía de encabezados correcta (un solo H1 por página).
- [ ] Imágenes de portada con `alt`; bloques de iniciales son decorativos (`aria-hidden`), el
      título va en texto real.

---

## 10. Snippet base (`tokens.css`)

Punto de entrada único para la capa *View*. Importar antes que cualquier hoja de componente.

```css
/* tokens.css — importar en todas las vistas */
:root {
  /* color */
  --color-brand:#1F5540; --color-brand-hover:#17412F; --color-brand-strong:#163A2B;
  --color-ink-button:#1A2420;
  --color-bg:#F3F1E9; --color-surface:#FFFFFF; --color-surface-muted:#F7F5EE;
  --color-sidebar:#14201A; --color-sidebar-ink:#EDEDE6; --color-sidebar-muted:#9BA69F;
  --color-text:#1A1D1A; --color-text-muted:#6E6E66; --color-text-subtle:#8E8E84;
  --color-border:#E4E1D6; --color-border-strong:#CFCBBD; --color-focus:#1F5540;
  --color-success:#1F5540; --color-success-bg:#DCE9DD;
  --color-warning:#8A5A24; --color-warning-bg:#F4E7C9;
  --color-danger:#9B2C2C;  --color-danger-bg:#F8E3E2;
  --color-info:#1F5540;    --color-info-bg:#E3EFE4;
  --role-socio-bg:#D7E8DB; --role-socio-text:#1F5540;
  --role-staff-bg:#E7E3F3; --role-staff-text:#4A3E7A;
  --cat-novela:#2E5A46; --cat-geografia:#3F597A; --cat-juvenil:#8A5A24;
  --cat-infantil:#7B3B3B; --cat-historia:#5A4A2E; --cat-cifi:#35506B; --cat-default:#44524A;
  /* type */
  --font-display:"Inter","Helvetica Neue",Arial,system-ui,sans-serif;
  --font-body:"Inter",system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;
  --font-mono:"IBM Plex Mono",ui-monospace,Menlo,monospace;
  /* space & radius */
  --space-1:4px;--space-2:8px;--space-3:12px;--space-4:16px;--space-5:20px;--space-6:24px;
  --space-8:32px;--space-10:40px;--space-12:48px;--space-16:64px;--space-20:80px;
  --radius-sm:6px;--radius-md:10px;--radius-lg:14px;--radius-full:999px;
  --shadow-sm:0 1px 2px rgba(20,32,26,.04); --shadow-md:0 2px 8px rgba(20,32,26,.06);
  --shadow-pop:0 8px 24px rgba(20,32,26,.12);
  --container-max:1200px; --sidebar-width:264px;
  --gutter:clamp(16px,4vw,24px);
  /* breakpoints de referencia (usar literales en @media): 480 / 768 / 1024 / 1280 */
}
*, *::before, *::after { box-sizing: border-box; }
html { -webkit-text-size-adjust: 100%; }         /* evita reescalado de texto en móvil */
body { margin:0; background:var(--color-bg); color:var(--color-text); font:var(--text-body);
       font-family:var(--font-body); -webkit-font-smoothing:antialiased; }
img, picture, svg { max-width:100%; display:block; } /* nada desborda en móvil */
```

> **Imprescindible en el `<head>` de toda vista** (sin esto el móvil renderiza a 980 px):
> ```html
> <meta name="viewport" content="width=device-width, initial-scale=1" />
> ```

---

*Última actualización: 16/11/2026. Fuente: mockups `docs/ux-ui/` · casos de uso 1–33 · TECH_SPEC.*
