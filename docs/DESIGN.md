# MERP — Sistema de diseño
> Sala de control interna: oscura, plana, compacta; los datos mandan.

**Tema:** oscuro · **Idioma de la interfaz:** español · **Aplicación:** web interna (roles Admin y Dealer)

> **Nota de alcance (provisional).** Este sistema es **provisional en lo relativo a marca**. Los lineamientos de marca de Magma (colores, logotipo, tipografía) están pendientes (ANOTACIONES_RECONSTRUCCION §4). Cuando lleguen se ajustarán el acento, el logotipo de la barra superior y las familias tipográficas; los tokens de este documento están pensados para cambiarse en un solo lugar (`:root`). La base es una adaptación de la referencia de diseño de `docs/referencias/` (no se modifica aquí); se quitó todo lo propio de un sitio comercial.

Todos los datos de ejemplo de este documento (VIN, órdenes, dealers, personas) son **ficticios**.

---

## 1. Principios

1. **Los datos son el protagonista.** Chrome mínimo; el color se reserva para significar algo.
2. **Superficies por tono, no por elevación.** Sin sombras, sin degradados. Se separa con `#0a0a0a` → `#141414` → `#1e1e1e` → `#313131` y bordes de 1 px.
3. **Un solo acento cromático (`#6798ff`)** para navegación activa, foco, selección, enlaces y la serie principal de las gráficas. Única excepción: los **colores de datos** (§3).
4. **El color nunca es la única señal.** Siempre hay texto, ícono, forma o posición además del color.
5. **Densidad compacta.** Filas de 40 px, controles de 36 px, texto base de 14 px.
6. **Calcula el servidor, presenta el frontend** (MERP-103 P-3): ningún componente decide un color de semáforo; recibe el dato ya calculado.

---

## 2. Tokens de color

### 2.1 Neutros y acento

| Token | Valor | Uso |
|---|---|---|
| `--surface-canvas` | `#0a0a0a` | Fondo de página, barra superior |
| `--surface-section` | `#141414` | Bandas, paneles anidados, fondo de encabezado de tabla |
| `--surface-card` | `#1e1e1e` | Tarjetas, tablas, campos, zona de arrastre |
| `--surface-raised` | `#313131` | Separadores, hover de fila/botón fantasma, marcos |
| `--border-subtle` | `#313131` | Divisores 1 px |
| `--border-strong` | `#454545` | Borde de botón secundario (el botón tiene texto, no necesita 3:1) |
| `--border-control` | `#7c7c7c` | Borde de campos y controles sin texto propio (cumple 3:1, §3.5) |
| `--text-primary` | `#ffffff` | Texto principal, encabezados |
| `--text-secondary` | `#a7a7a7` | Texto secundario, etiquetas, placeholder |
| `--text-disabled` | `#7c7c7c` | **Solo** texto deshabilitado (exento de contraste) |
| `--accent` | `#6798ff` | Enlace activo, foco, selección, serie principal de gráficas |

Reglas: el acento **nunca** es fondo de botón ni de bloque de contenido. `#000000` no se usa. `#7c7c7c` **no** pasa 4.5:1 sobre `#1e1e1e` (3.99:1): queda restringido a texto deshabilitado y a bordes/marcas (§3.5).

---

## 3. Colores de datos (única excepción a la regla del acento único)

Verde, amarillo y rojo se usan **solo** para (a) el semáforo de órdenes y (b) los tramos de unidades paradas. Todas las demás gráficas (tendencias, cumplimiento de promesa, facturación, desempeño por asesor) usan el acento `#6798ff` y escalas de gris. Verde/amarillo/rojo **no** se usan en alertas, botones, validación de formularios ni estados de carga.

### 3.1 Semáforo (RN-002, días de calendario, DA-01)

| Estado | Días | Token | Hex | Texto de la insignia |
|---|---|---|---|---|
| Verde | 0 a 4 | `--data-green` | `#3ecf6e` | "Verde · 3 d" |
| Amarillo | 5 a 7 | `--data-yellow` | `#f2c94c` | "Amarillo · 6 d" |
| Rojo | más de 7 | `--data-red` | `#ff5c5c` | "Rojo · 12 d" |

### 3.2 Tramos de unidades paradas (solo Admin)

Secuencia: los tres primeros reutilizan el semáforo; los dos últimos son rojos cada vez más saturados e intensos (el tono gira hacia el carmesí) y **siguen siendo claros** contra el fondo oscuro. Nunca se oscurece hacia casi negro.

| Tramo | Token | Hex | Equivale a |
|---|---|---|---|
| 0–4 días | `--bucket-1` | `#3ecf6e` | Verde del semáforo |
| 5–7 días | `--bucket-2` | `#f2c94c` | Amarillo del semáforo |
| 8–15 días | `--bucket-3` | `#ff5c5c` | Rojo del semáforo |
| 16–30 días | `--bucket-4` | `#ff3355` | Rojo saturado |
| más de 30 días | `--bucket-5` | `#e8003d` | Rojo intenso (carmesí) |

Límite honesto: los tres rojos difieren poco entre sí en luminancia (ratios entre vecinos de 1.18 y 1.30), por lo que **el color no basta para distinguirlos**. Por eso la gráfica separa los segmentos con 1 px de `#1e1e1e`, la leyenda y el eje llevan el texto del tramo ("16–30 d"), cada segmento muestra su valor al pasar el cursor o enfocar, y existe una tabla equivalente (§7.6).

### 3.3 Fondos tenues para insignias (hex sólidos, sin transparencias)

| Token | Hex | Texto de la insignia sobre él |
|---|---|---|
| `--data-green-bg` | `#10301c` | `#ffffff` (14.37:1) |
| `--data-yellow-bg` | `#33290a` | `#ffffff` (14.36:1) |
| `--data-red-bg` | `#3a1212` | `#ffffff` (16.47:1) |
| `--accent-bg` | `#14243f` | `#ffffff` (15.51:1) |

La insignia = fondo tenue + borde 1 px del color pleno + punto del color pleno (≥3:1 sobre el tenue) + **texto blanco**. El texto no depende del color.

### 3.4 Verificación de contraste (calculada, fórmula WCAG 2.x)

Valores obtenidos con un script de luminancia relativa (sRGB → lineal; `(L1+0.05)/(L2+0.05)`). Umbrales: texto 4.5:1, marcas gráficas/componentes 3:1.

| Color | Hex | vs `#0a0a0a` | vs `#1e1e1e` | Texto (4.5) | Marca (3.0) |
|---|---|---|---|---|---|
| Blanco | `#ffffff` | 19.80 | 16.67 | Pasa | Pasa |
| Gris secundario | `#a7a7a7` | 8.23 | 6.93 | Pasa | Pasa |
| Gris deshabilitado | `#7c7c7c` | 4.74 | **3.99** | Pasa solo en canvas; falla en sección (4.41) y tarjeta (3.99) | Pasa |
| Acento | `#6798ff` | 7.08 | 5.96 | Pasa | Pasa |
| Verde | `#3ecf6e` | 9.76 | 8.22 | Pasa | Pasa |
| Amarillo | `#f2c94c` | 12.48 | 10.51 | Pasa | Pasa |
| Rojo | `#ff5c5c` | 6.54 | 5.51 | Pasa | Pasa |
| Rojo saturado | `#ff3355` | 5.52 | 4.65 | Pasa | Pasa |
| Rojo intenso | `#e8003d` | 4.24 | 3.57 | **Falla (no usar como texto)** | Pasa |
| Borde decorativo | `#313131` | 1.52 | 1.28 | No aplica | No (solo decorativo) |
| Borde de botón | `#454545` | 2.06 | 1.74 | No aplica | No (el botón tiene texto) |

Conclusiones y reglas que se derivan:

- `#7c7c7c`: sobre `#141414` da 4.41 y sobre `#1e1e1e` 3.99, así que **no se usa para texto legible**; solo texto deshabilitado (exento) y bordes de control (3.99 / 4.74 ≥ 3). El placeholder usa `#a7a7a7`.
- `#e8003d` (>30 días) **no se usa como texto**; solo como relleno de segmento y del punto de la leyenda. El texto del tramo siempre va en `#ffffff` o `#a7a7a7`.
- El borde de campos (`#7c7c7c`) cumple el 3:1 de componentes (3.99 sobre tarjeta). Los bordes `#313131`/`#454545` son solo separadores.

### 3.5 Gráficas que no son semáforo/tramos

Serie principal `--accent`; series adicionales `#a7a7a7` y `#7c7c7c` (marcas, 6.93/3.99 sobre tarjeta); ejes y cuadrícula `#313131` (decorativos; el valor siempre está en texto). Hasta 3 series; si hay más, usar agrupación o tabla.

---

## 4. Tipografía

- **Inter** (400 cuerpo, 500 navegación y botones, 600 encabezados) y **JetBrains Mono** (400) para VIN, números de orden, identificadores y códigos de archivo.
- **Tamaño máximo dentro de la aplicación: 40 px** (solo cifra de tarjeta de estadística). No existen tamaños de 56 ni 64.
- Los números en tablas y cifras usan `font-variant-numeric: tabular-nums`.
- Sin MAYÚSCULAS decorativas en etiquetas; sentence case en español.

| Rol | Familia | Peso | Tamaño | Interlínea | Tracking | Token |
|---|---|---|---|---|---|---|
| Microtexto | Inter | 400 | 12 px | 1.4 | 0 | `--text-caption` |
| Cuerpo / tabla | Inter | 400 | 14 px | 1.5 | -0.17 px | `--text-body-sm` |
| Cuerpo amplio | Inter | 400 | 16 px | 1.5 | -0.19 px | `--text-body` |
| Subtítulo | Inter | 600 | 20 px | 1.4 | -0.42 px | `--text-subheading` |
| Título de página | Inter | 600 | 24 px | 1.33 | -0.5 px | `--text-heading-sm` |
| Cifra de estadística | Inter | 600 | 40 px | 1.2 | -0.84 px | `--text-heading` |
| Código / VIN | JetBrains Mono | 400 | 13 px | 1.4 | 0 | `--text-mono` |

Cuerpo mínimo 14 px; solo etiquetas auxiliares y microtexto llegan a 12 px.

---

## 5. Tokens como variables CSS

```css
:root {
  color-scheme: dark;

  /* Superficies */
  --surface-canvas: #0a0a0a;
  --surface-section: #141414;
  --surface-card: #1e1e1e;
  --surface-raised: #313131;

  /* Bordes */
  --border-subtle: #313131;
  --border-strong: #454545;
  --border-control: #7c7c7c;

  /* Texto */
  --text-primary: #ffffff;
  --text-secondary: #a7a7a7;
  --text-disabled: #7c7c7c;
  --text-on-light: #0a0a0a;
  --button-primary-hover: #e6e6e6;

  /* Acento */
  --accent: #6798ff;
  --accent-bg: #14243f;

  /* Colores de datos: SOLO semáforo y tramos */
  --data-green: #3ecf6e;
  --data-yellow: #f2c94c;
  --data-red: #ff5c5c;
  --data-green-bg: #10301c;
  --data-yellow-bg: #33290a;
  --data-red-bg: #3a1212;
  --bucket-1: var(--data-green);
  --bucket-2: var(--data-yellow);
  --bucket-3: var(--data-red);
  --bucket-4: #ff3355;
  --bucket-5: #e8003d;

  /* Series de gráficas generales */
  --chart-1: var(--accent);
  --chart-2: #a7a7a7;
  --chart-3: #7c7c7c;
  --chart-grid: #313131;

  /* Tipografía: familias */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* Tipografía: escala (máximo 40 px) */
  --text-caption: 12px;
  --leading-caption: 1.4;
  --tracking-caption: 0;
  --text-body-sm: 14px;
  --leading-body-sm: 1.5;
  --tracking-body-sm: -0.17px;
  --text-body: 16px;
  --leading-body: 1.5;
  --tracking-body: -0.19px;
  --text-subheading: 20px;
  --leading-subheading: 1.4;
  --tracking-subheading: -0.42px;
  --text-heading-sm: 24px;
  --leading-heading-sm: 1.33;
  --tracking-heading-sm: -0.5px;
  --text-heading: 40px;
  --leading-heading: 1.2;
  --tracking-heading: -0.84px;
  --text-mono: 13px;
  --leading-mono: 1.4;

  /* Pesos */
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;

  /* Espaciado (base 8 px, con medio paso de 4) */
  --space-4: 4px;
  --space-8: 8px;
  --space-12: 12px;
  --space-16: 16px;
  --space-24: 24px;
  --space-32: 32px;
  --space-40: 40px;
  --space-64: 64px;
  --space-96: 96px;

  /* Layout */
  --page-max-width: 1200px;
  --page-max-width-wide: 1440px;
  --page-padding-x: 24px;
  --section-gap-sm: 32px;
  --section-gap-lg: 64px;
  --card-padding-sm: 16px;
  --card-padding-lg: 24px;
  --element-gap-sm: 8px;
  --element-gap-lg: 16px;

  /* Radios (sin píldoras) */
  --radius-tag: 4px;
  --radius-control: 8px;
  --radius-card: 8px;
  --radius-button: 8px;

  /* Alturas */
  --topbar-height: 48px;
  --row-height: 40px;
  --table-header-height: 40px;
  --control-height: 36px;
  --control-height-lg: 40px;
  --badge-height: 22px;
  --icon-size: 16px;

  /* Foco */
  --focus-ring-color: var(--accent);
  --focus-ring-width: 2px;
  --focus-ring-offset: 2px;

  /* Movimiento (escaso) */
  --motion-fast: 120ms;
}

:focus-visible {
  outline: var(--focus-ring-width) solid var(--focus-ring-color);
  outline-offset: var(--focus-ring-offset);
}

body {
  margin: 0;
  background: var(--surface-canvas);
  color: var(--text-primary);
  font: var(--font-weight-regular) var(--text-body-sm) / var(--leading-body-sm) var(--font-sans);
  letter-spacing: var(--tracking-body-sm);
}

@media (prefers-reduced-motion: reduce) {
  * { transition: none !important; animation: none !important; }
}
```

---

## 6. Reglas de forma y superficie

- Radio **8 px** en botones, campos, tarjetas, tablas y modales; **4 px** en etiquetas, insignias y chips. Prohibido el radio de píldora.
- **Sin sombras y sin degradados** en ninguna superficie, botón o fondo. No hay motivo de cuadrícula decorativo de fondo.
- Los modales se separan con un velo sólido `#0a0a0a` sobre el contenido (sin desenfoque ni sombra) y una tarjeta `#1e1e1e` con borde `#313131`.
- Íconos: trazo de 1.5 px, 16 px (20 px en estados vacíos), color heredado del texto; el acento solo en elementos activos.
- Foco: anillo de 2 px `#6798ff` con separación de 2 px, en **todo** elemento interactivo (7.08:1 sobre canvas, 5.96:1 sobre tarjeta). Nunca se quita sin sustituto. Orden de tabulación = orden visual.

---

## 7. Componentes

En implementación, usar siempre el token; los hex de esta sección son solo referencia.

Estados comunes: **hover** (fondo `--surface-raised` o sube un tono), **foco** (anillo, §6), **deshabilitado** (texto `--text-disabled`, sin hover, `cursor: not-allowed`, `aria-disabled`), **error** (§7.8).

### 7.1 Botones

Altura `--control-height` (36 px); padding 0 16 px; Inter 500 14 px; radio 8 px; ícono opcional de 16 px a la izquierda con 8 px de separación. Sin flechitas decorativas.

| Variante | Fondo | Texto | Borde | Hover | Deshabilitado |
|---|---|---|---|---|---|
| **Primario** | `#ffffff` | `#0a0a0a` | ninguno | fondo `--button-primary-hover` (`#e6e6e6`, hex sólido) | fondo `#313131`, texto `#7c7c7c` |
| **Secundario** | transparente | `#ffffff` | 1 px `#454545` | fondo `#1e1e1e`, borde `#7c7c7c` | borde `#313131`, texto `#7c7c7c` |
| **Fantasma** | transparente | `#ffffff` | ninguno | fondo `#313131` | texto `#7c7c7c` |
| **Peligro** | transparente | `#ffffff` | 1 px `#ffffff` | fondo `#313131` | borde `#313131`, texto `#7c7c7c` |

Reglas:
- Un solo botón primario por vista o modal. Acciones de baja frecuencia: secundario o fantasma.
- **Peligro sin otro color:** se distingue por borde blanco de 1 px, ícono de advertencia (triángulo) y texto explícito ("Dar de baja usuario", no "Aceptar"). Siempre pide **confirmación** en un modal que nombra el objeto afectado y cuyo botón de confirmar repite la acción. El botón primario del modal de confirmación sigue siendo blanco.
- Estado de carga: el texto cambia ("Subiendo…"), se muestra un indicador de 16 px de trazo, el botón queda deshabilitado y conserva su ancho.
- `#e6e6e6` es el único gris añadido a la paleta neutra, usado solo como estado hover del primario (texto `#0a0a0a` sobre él: 15.86:1).

### 7.2 Barra superior con navegación por rol

- Alto 48 px, fondo `#0a0a0a`, borde inferior 1 px `#313131`, fija arriba (`position: sticky; top: 0`), contenido alineado a `--page-max-width-wide`.
- Izquierda: marca provisional "MERP" (Inter 600 16 px) con 24 px de separación; después los enlaces de navegación (Inter 500 14 px, `#a7a7a7`, separados 8 px, cada uno con padding 0 12 px y alto 48 px).
- Hover: texto `#ffffff`. **Enlace activo:** texto `#ffffff` y línea inferior de 2 px `#6798ff`, más `aria-current="page"`. Foco: anillo estándar.
- Derecha: nombre del usuario y rol en texto secundario ("Dealer · Dealer Demo Norte"), menú de usuario (botón fantasma) con "Cambiar contraseña" y "Cerrar sesión".
- **Navegación por rol:**
  - **Admin:** Dashboard · Panel · Unidades paradas · Dealers · Usuarios · Cargas · Perfiles de importación · Reportes. "Panel" abre con un selector de dealer, que incluye los dados de baja marcados «(baja)».
  - **Dealer:** Dashboard · Panel · Cargas · Reportes. "Cargas" = zona de arrastre + resumen + historial (MERP-112 §8).
  - Esta lista es una **propuesta a confirmar con las pantallas de cada fase**.
  - Un enlace que el rol no tiene **no se muestra** (no se deshabilita). El servidor igualmente valida (P-3).
- En pantallas <900 px los enlaces pasan a un menú desplegable (botón "Menú" con `aria-expanded`).

### 7.3 Insignia de semáforo

- Alto 22 px, padding 0 8 px, radio 4 px, Inter 500 12 px, texto `#ffffff`, fondo tenue y borde 1 px del color (§3.3), punto de 8 px redondo del color pleno a la izquierda con 6 px de separación.
- Contenido: etiqueta + días, con numerales tabulares: "Verde · 3 d", "Amarillo · 6 d", "Rojo · 12 d". Sin días (resumen): solo "Verde".
- `aria-label` completo: "Semáforo rojo, 12 días". Es solo lectura: no tiene hover ni acción.
- Una orden completada con semáforo congelado (RN-003) añade un ícono de candado de 12 px y `title`/texto "congelado".

### 7.4 Tablas densas

Se usan en dashboard, lista de unidades, cargas, usuarios y dealers.

- Contenedor: fondo `#1e1e1e`, radio 8 px, borde 1 px `#313131`, `overflow: auto`, `max-height: calc(100vh - var(--topbar-height) - 160px)` (la tabla hace scroll propio dentro del contenedor).
- **Encabezado fijo (sticky):** alto 40 px, fondo `#141414`, borde inferior 1 px `#313131`, Inter 500 12 px `#a7a7a7`, `position: sticky; top: 0`.
- **Filas de 40 px**, Inter 400 14 px, padding horizontal 12 px, divisor 1 px `#313131`. Hover: fondo `#313131`. Fila seleccionada: borde izquierdo 2 px `#6798ff` y fondo `#14243f`. Foco de fila clicable: anillo estándar inset.
- **Columna de semáforo:** primera o segunda columna, ancho fijo 160 px, contiene la insignia (§7.3).
- **Números:** alineados a la **derecha**, `tabular-nums`; el encabezado de esa columna también a la derecha. Importes con separador de miles y moneda explícita; VIN y orden en JetBrains Mono 13 px, a la izquierda.
- **Orden por columna:** el encabezado ordenable es un botón; muestra ícono de flecha (↑ ascendente, ↓ descendente) 12 px y `aria-sort`. Clic alterna ascendente → descendente → sin orden. Columna ordenada: texto `#ffffff`. El orden por defecto en dashboard lo define el servidor (DA-07: grupos alfabéticos por dealer, urgencia dentro del grupo).
- Texto largo: una línea con elipsis y `title`. Sin franjas alternadas.
- Estados: cargando (filas esqueleto de 40 px en `#313131` sin animación de brillo), vacío (§7.11), error (alerta en lugar del cuerpo, §7.12).
- Encabezado de grupo (Dashboard agrupado por dealer): fila de 40 px, fondo `#141414`, nombre del dealer en Inter 600 14 px y conteo en texto secundario.

### 7.5 Tarjeta de estadística

- Fondo `#1e1e1e`, borde 1 px `#313131`, radio 8 px, padding 24 px, ancho flexible en rejillas de 4 columnas (2 en <900 px, 1 en <560 px), separación 16 px.
- Orden: etiqueta (Inter 500 14 px `#a7a7a7`) arriba; cifra (Inter 600 **40 px**, `#ffffff`, `tabular-nums`) debajo; nota o comparación (12 px `#a7a7a7`, p. ej. "+3 vs. semana anterior") al fondo. Ícono opcional de 16 px en `#6798ff`.
- Las tarjetas del resumen de semáforo (en proceso, verde, amarillo, rojo, completadas) llevan además la insignia/punto del color y su nombre en texto; el valor sigue en blanco.
- Si es clicable (abre la lista filtrada): es un `<a>`/botón, hover borde `#7c7c7c`, foco estándar. Sin elevación.
- Estado sin dato: cifra "—" en `#a7a7a7` y nota "No disponible" con su motivo (RN-023).

### 7.6 Gráfica de barras apiladas — unidades paradas (solo Admin)

- **Datos:** una barra horizontal por dealer, con cinco tramos apilados (0–4, 5–7, 8–15, 16–30, >30 d), ordenadas de más a menos unidades totales.
- **Colores:** `--bucket-1…5` (§3.2). Segmentos separados por 1 px de `#1e1e1e`; barras rectas, radio 0.
- **Medidas:** barra de 24 px de alto, 12 px de separación entre barras, etiqueta del dealer a la izquierda (Inter 400 14 px, máx. 200 px con elipsis), total al final (14 px, tabular, `#ffffff`). Eje de valores en `#a7a7a7` 12 px; cuadrícula `#313131` 1 px.
- **Leyenda:** arriba de la gráfica, fila horizontal; cada ítem = cuadro de 12 px del color + texto del tramo ("16–30 d") en blanco 12 px. Cada ítem es un botón para **resaltar** el tramo (los demás segmentos se atenúan a `#7c7c7c` (3.99:1)); opcional.
- **Interacción:** hover/foco sobre un segmento muestra tooltip sólido (`#313131`, sin sombra): "Dealer Demo Norte · 8–15 d · 14 unidades". **Clic en un segmento abre la lista** de esas unidades (VIN, modelo, orden, dealer, días) en un panel lateral de 480 px o en una vista con tabla (§7.4). Cada segmento es un elemento enfocable con `role="button"` y `aria-label` completo; teclado: Tab entra a la gráfica; flechas ←/→ entre tramos y ↑/↓ entre dealers (tabindex itinerante); Enter abre la lista.
- **Alternativa accesible:** botón "Ver como tabla" que muestra la misma información como tabla (dealer × tramos) con las cinco columnas; es la fuente textual para quien no distingue los tonos de rojo.
- Estado vacío: "No hay unidades paradas." con ícono; sin datos de un dealer: barra omitida.

### 7.7 Filtros y selectores

- **Campo de selección / filtro:** alto 36 px, fondo `#1e1e1e`, borde 1 px `#7c7c7c`, radio 8 px, texto Inter 400 14 px; ícono de chevron 16 px a la derecha. Hover: borde `#a7a7a7`. Foco: anillo estándar. Abierto: lista en tarjeta `#1e1e1e` con borde `#313131`, opción activa con fondo `#313131`, opción elegida con check `#6798ff` y `aria-selected`.
- **Filtro por dealer** (Admin): selector con búsqueda; opción "Todos los dealers" por defecto.
- **Chips de filtro aplicados:** alto 24 px, radio 4 px, fondo `#313131`, texto 12 px, botón "Quitar" (×) con `aria-label`. Enlace fantasma "Limpiar filtros".
- **Filtro por semáforo / tramo:** grupo de botones conmutables (`aria-pressed`): seleccionado = borde `#6798ff` y texto blanco, con **la etiqueta de texto**, no solo el color.
- La barra de filtros va sobre la tabla, en fila con separación 8 px, fondo `#141414`, padding 12 px, radio 8 px. Aplicar filtros es inmediato; el estado queda en la URL.

### 7.8 Formularios y validación

- **Etiqueta** sobre el campo (Inter 500 14 px `#ffffff`), texto de ayuda debajo (12 px `#a7a7a7`); separación entre campos 16 px; ancho máximo de formulario 480 px.
- **Campo de texto:** alto 36 px (`--control-height`), fondo `#1e1e1e`, borde 1 px `#7c7c7c`, radio 8 px, padding 0 12 px, placeholder `#a7a7a7`. Hover: borde `#a7a7a7`. Foco: borde `#6798ff` más anillo estándar. Deshabilitado: fondo `#141414`, borde `#313131`, texto `#7c7c7c`.
- **Error:** borde 2 px `#ffffff`, ícono de error (octágono con ×) 16 px a la derecha dentro del campo, y debajo un mensaje en blanco 12 px **con prefijo de texto**: "Error: la contraseña debe tener al menos 10 caracteres y un carácter especial." El campo recibe `aria-invalid="true"` y `aria-describedby` apuntando al mensaje. Resumen de errores arriba del formulario cuando hay varios, con enlaces a cada campo.
- **Cuándo validar:** al salir del campo (blur) y al enviar; nunca mientras se escribe por primera vez. El mensaje dice qué corregir, no solo qué falló.
- **Obligatorio:** se marca en la etiqueta con "(obligatorio)" en texto secundario; no solo con asterisco.
- **Contraseña:** botón fantasma "Mostrar/Ocultar" dentro del campo; criterios (mínimo 10 caracteres, un carácter especial, DA-05) listados debajo con ícono check/punto y texto, cumplidos en blanco, pendientes en `#a7a7a7`.
- **Casillas y opciones:** cuadro de 16 px, borde 1 px `#7c7c7c`, radio 4 px; marcado = fondo `#6798ff` con check `#0a0a0a`. Interruptor de 32×18 px solo si hace falta, con texto de estado ("Activo").
- Envío: botón primario; mientras se envía, deshabilitado con "Guardando…". Éxito: alerta de éxito (§7.12).

### 7.9 Zona de arrastrar archivo (.xlsx)

- Tarjeta `#1e1e1e`, borde **discontinuo** 1 px `#7c7c7c`, radio 8 px, alto mínimo 160 px, contenido centrado: ícono de subir (20 px, `#6798ff`), texto "Arrastra tu archivo .xlsx aquí" (Inter 500 14 px) y "o selecciónalo" como botón secundario; ayuda en 12 px `#a7a7a7`: "Solo .xlsx. Tamaño máximo según la configuración del sistema."
- **Hover / arrastre encima:** borde 1 px sólido `#6798ff` y fondo `#14243f`; texto "Suelta para cargar".
- **Foco:** anillo estándar (la zona es un botón que abre el selector de archivos; también se puede pegar con teclado).
- **Archivo elegido:** muestra nombre (JetBrains Mono 13 px), tamaño y botón fantasma "Quitar"; botón primario "Subir archivo".
- **Subiendo:** barra de progreso de 4 px (`#313131` pista, `#6798ff` avance) con texto "Procesando… (n %)"; botón deshabilitado.
- **Error de archivo** (formato no .xlsx, vacío, ilegible): borde sólido 2 px `#ffffff`, ícono de error y mensaje con prefijo "Error:" ("Error: el archivo no es un .xlsx válido."). Deshabilitado: borde `#313131`, texto `#7c7c7c`.
- Solo un archivo a la vez. El Excel original no se guarda (DA-13); el texto de ayuda no promete conservarlo.

### 7.10 Resumen de carga

Se muestra tras subir (RN de carga: filas totales = nuevas + actualizadas + sin cambios + rechazadas).

- Rejilla de tarjetas de estadística pequeñas (§7.5, cifra de 24 px aquí para no dominar): **Nuevas · Actualizadas · Sin cambios · Rechazadas** (suman las filas totales, mostradas como "Filas totales: 30") y, separada por un divisor 1 px `#313131`, la tarjeta **Avisos** (no suma al total). Se muestra también "Versión del perfil: vN" (RN-026). Todas en blanco/gris, sin color de semáforo. Rechazadas y Avisos llevan un ícono (octágono / triángulo) además del número, y si su valor es mayor que 0 el borde de la tarjeta pasa a `#ffffff`.
- Debajo, si hay rechazadas: tabla densa (§7.4) con **Fila · Motivo**, ordenada por fila, y el texto: "Corrige estas filas en tu sistema de origen y vuelve a subir el archivo."
- Si hay avisos (datos ilegibles cargados con aviso, VIN modificado, etc.): alerta de advertencia (§7.12) con la lista.
- Si cambió el perfil de importación: alerta informativa que lo indica.
- Una carga sin ninguna fila válida muestra alerta de error con "0 filas válidas" y la tabla de rechazos; la carga igual queda registrada.
- Acciones: botón secundario "Descargar PDF de la carga" y primario "Ir al dashboard".
- Ejemplo de datos ficticios: 30 filas — 18 nuevas, 4 actualizadas, 5 sin cambios, 3 rechazadas.

### 7.11 Estados vacíos

- Bloque centrado dentro de la tarjeta o tabla, padding 40 px: ícono de 20 px en `#a7a7a7`, título Inter 600 16 px, descripción 14 px `#a7a7a7` (máx. 360 px) y, si procede, un botón (primario si es la acción siguiente natural).
- Ejemplos: "Aún no has subido archivos" · "Sube tu primer .xlsx para ver tu dashboard." [Subir archivo]; "No hay unidades paradas" · "Ninguna unidad supera los 4 días."; "Sin resultados" · "Prueba quitando algún filtro." [Limpiar filtros].
- Sin ilustraciones decorativas, sin tono humorístico.

### 7.12 Alertas y avisos

**Decisión:** éxito, advertencia y error **no** usan verde, amarillo ni rojo: esos colores son solo del semáforo y los tramos, y reutilizarlos haría que un aviso de error se leyera como "unidad en rojo". Cada tipo se distingue por **ícono de forma distinta + prefijo de texto + borde**, nunca por color.

Estructura común: tarjeta `#1e1e1e`, radio 8 px, padding 12 px 16 px, ícono de 16 px a la izquierda, título en Inter 500 14 px (con prefijo), cuerpo 14 px `#a7a7a7`, botón de cerrar (fantasma, `aria-label="Cerrar aviso"`) cuando es descartable. Contenedor con `role="status"` (info/éxito) o `role="alert"` (advertencia/error).

| Tipo | Ícono (forma) | Prefijo del título | Borde |
|---|---|---|---|
| Información | círculo con "i" | "Información:" | 1 px `#313131` + barra izquierda 3 px `#6798ff`; ícono `#6798ff` |
| Éxito | círculo con check | "Listo:" | 1 px `#a7a7a7` |
| Advertencia | triángulo con "!" | "Advertencia:" | 1 px `#ffffff` |
| Error | octágono con "×" | "Error:" | 2 px `#ffffff` + barra izquierda 4 px `#ffffff`; el cuerpo pasa a `#ffffff` |

Comportamiento: éxito se descarta solo tras 6 s (y queda accesible al lector de pantalla); advertencia y error **no** se descartan solos. Los avisos de página van arriba del contenido; los de campo, bajo el campo (§7.8). Errores de red incluyen acción "Reintentar".

Pendiente de decisión del dueño: si se quisiera un color propio para el error, debería ser un tono claramente distinto del rojo de datos.

### 7.13 Paginación

- Barra bajo la tabla, alto 40 px, fondo `#141414`, borde superior 1 px `#313131`, radio inferior 8 px.
- Izquierda: "Mostrando 21–40 de 132" (12 px `#a7a7a7`, `tabular-nums`) y selector "Filas por página" (20, 50, 100).
- Derecha: botones fantasma de 32 px: "Anterior", números (página actual con borde 1 px `#6798ff` y `aria-current="page"`), "Siguiente"; deshabilitados en los extremos (texto `#7c7c7c`). Elipsis en rangos largos.
- Se usa en historial de cargas (DA-06, con filtro por dealer para el Admin) y listas largas. El cambio de página devuelve el foco al inicio de la tabla.

### 7.14 Modal de confirmación

Tarjeta `#1e1e1e` de 440 px de ancho, borde 1 px `#313131`, padding 24 px, título Inter 600 20 px, cuerpo 14 px; acciones alineadas a la derecha (secundario "Cancelar" + primario o peligro). Foco atrapado, `Esc` cierra, el foco vuelve al disparador, `role="dialog"` con `aria-labelledby`.

---

## 8. Layout

- Contenedor: `--page-max-width` (1200 px) para formularios y paneles; `--page-max-width-wide` (1440 px) para tablas y gráficas. Padding lateral 24 px (16 px en <560 px).
- Estructura de página: barra superior (48 px) → título de página (24 px) con acciones a la derecha → filtros → contenido. Separación vertical entre bloques `--section-gap-sm` (32 px); entre secciones mayores `--section-gap-lg` (64 px).
- Sin barra lateral. Sin pie de página de sitio; opcionalmente una línea de 12 px `#a7a7a7` con "Última carga: …" (dato útil, no marketing).
- Responsive: diseño pensado para escritorio; hasta 560 px, las tablas hacen scroll horizontal dentro de su contenedor con la primera columna fija; las rejillas de tarjetas colapsan.

## 9. Accesibilidad (resumen)

- Contraste: texto ≥ 4.5:1 y marcas ≥ 3:1 contra `#0a0a0a` y `#1e1e1e` (§3.4). Excepciones controladas: `#7c7c7c` (solo texto deshabilitado, bordes de control y serie 3 de gráficas), `#e8003d` (solo marca).
- Foco visible en todo; operable por teclado (menús, tabla, gráfica, zona de arrastre, modales).
- El color nunca es la única señal: semáforo con texto y días; tramos con etiqueta y tabla alterna; alertas con ícono y prefijo; errores de campo con ícono y texto.
- `prefers-reduced-motion` respetado; movimiento solo de 120 ms en hover/foco.
- Lectores de pantalla: `aria-sort`, `aria-current`, `aria-invalid`, `role="alert"/"status"`, `aria-label` completo en insignias y segmentos.
- Idioma de la página `lang="es-MX"`; fechas `dd/mm/aaaa`, hora local de planta; moneda con separador de miles.

## 10. Qué NO hacer

- No usar verde, amarillo ni rojo fuera del semáforo y los tramos (ni en alertas, validación, botones, deltas "sube/baja").
- No usar el acento como fondo de botón o bloque.
- No sombras, degradados, píldoras, fondos decorativos, ilustraciones de relleno ni tamaños de texto mayores a 40 px.
- No mostrar datos del cliente final en ninguna pantalla, reporte o PDF (DA-03).
- No texto en `#7c7c7c` fuera de estados deshabilitados.
- No mayúsculas decorativas, flechitas en botones ni "eyebrows" sobre títulos.

## 11. Gráficas en reportes PDF

Los PDF (de la carga y semanal) siguen estos mismos tokens y reglas de color de datos: el resumen del semáforo es una fila de cifras (en proceso, verde, amarillo, rojo, completadas) con insignias de texto, comparación con la semana anterior, una barra horizontal por color y una línea de lectura. Las demás gráficas usan acento y grises. Si el PDF se imprime en fondo claro, se conserva la lógica (texto + color) y los hex de datos se mantienen; el ajuste de fondo claro se define al recibir los lineamientos de marca.
