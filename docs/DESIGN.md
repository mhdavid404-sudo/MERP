# MERP — Sistema de diseño
> Sala de control interna: oscura, plana, compacta; los datos mandan.

**Tema:** oscuro en pantalla (la aplicación no tiene modo claro) · tema claro **solo** para PDF (§11) · **Idioma de la interfaz:** español · **Aplicación:** web interna (roles Admin y Dealer)

> **Nota de alcance (provisional).** Este sistema es **provisional en lo relativo a marca**. Los lineamientos de marca de Magma (colores, logotipo, tipografía) están pendientes (ANOTACIONES_RECONSTRUCCION §4). **Esto no bloquea F0 ni ninguna fase posterior:** se construye ya con estos valores y, cuando lleguen los lineamientos, se ajustarán el acento, el logotipo de la barra superior y las familias tipográficas cambiando **solo variables** (`:root` y `.theme-print`, §5 y §11), sin tocar componentes. La base es una adaptación de la referencia de diseño de `docs/referencias/` (no se modifica aquí); se quitó todo lo propio de un sitio comercial.

> **Regla de tokens (obligatoria).** **Todo color, familia tipográfica y radio sale de los tokens.** Ningún componente lleva escrito un valor directo: ni hex, ni `px` de radio, ni nombre de fuente. Las medidas de layout (alturas, paddings, anchos) también citan su token cuando existe (§5). Así, pasar a la marca de Magma es cambiar variables y nada más. Los hex que aparecen entre paréntesis en este documento son solo referencia visual.

**Decisiones del dueño del proyecto (26-oct-2026), ya incorporadas:**
1. Color de error propio `--color-danger`, distinto del rojo de datos (§3.6).
2. Patrones en los tres tramos rojos de la gráfica de unidades paradas (§7.6).
3. Tema claro solo para PDF (§11).
4. Documento provisional sin bloquear F0 + regla de tokens (esta nota y §5).

Todos los datos de ejemplo de este documento (VIN, órdenes, dealers, personas) son **ficticios**.

---

## 1. Principios

1. **Los datos son el protagonista.** Chrome mínimo; el color se reserva para significar algo.
2. **Superficies por tono, no por elevación.** Sin sombras, sin degradados. Se separa con `--surface-canvas` (`#0a0a0a`) → `--surface-section` (`#141414`) → `--surface-card` (`#1e1e1e`) → `--surface-raised` (`#313131`) y bordes de 1 px.
3. **Un solo acento cromático (`--accent`, `#6798ff`)** para navegación activa, foco, selección, enlaces y la serie principal de las gráficas. **Dos excepciones**, ambas acotadas: los **colores de datos** (§3) y el **color de error** `--color-danger` (§3.6, decisión del dueño 26-oct-2026).
4. **El color nunca es la única señal.** Siempre hay texto, ícono, forma, patrón o posición además del color.
5. **Densidad compacta.** Filas de `--row-height`, controles de `--control-height`, texto base `--text-body-sm`.
6. **Calcula el servidor, presenta el frontend** (MERP-103 P-3): ningún componente decide un color de semáforo; recibe el dato ya calculado.
7. **Todo sale de tokens** (regla de tokens, arriba): cambiar de marca = cambiar variables.

---

## 2. Tokens de color

### 2.1 Neutros y acento

| Token | Valor | Uso |
|---|---|---|
| `--surface-canvas` | `#0a0a0a` | Fondo de página, barra superior |
| `--surface-section` | `#141414` | Bandas, paneles anidados, fondo de encabezado de tabla |
| `--surface-card` | `#1e1e1e` | Tarjetas, tablas, campos, zona de arrastre |
| `--surface-raised` | `#313131` | Separadores, hover de fila/botón fantasma, marcos, tooltip |
| `--border-subtle` | `#313131` | Divisores 1 px |
| `--border-strong` | `#454545` | Borde de botón secundario (el botón tiene texto, no necesita 3:1) |
| `--border-control` | `#7c7c7c` | Borde de campos y controles sin texto propio (cumple 3:1, §3.4) |
| `--text-primary` | `#ffffff` | Texto principal, encabezados |
| `--text-secondary` | `#a7a7a7` | Texto secundario, etiquetas, placeholder |
| `--text-disabled` | `#7c7c7c` | **Solo** texto deshabilitado (exento de contraste) |
| `--accent` | `#6798ff` | Enlace activo, foco, selección, serie principal de gráficas |
| `--accent-bg` | `#14243f` | Fondo tenue del acento (selección, hover de zona de arrastre) |
| `--color-danger` | `#ff70e0` | **Solo** errores (§3.6) |
| `--color-danger-bg` | `#3a1233` | Fondo tenue de la alerta de error (§3.6) |

Reglas: el acento **nunca** es fondo de botón ni de bloque de contenido. `#000000` no se usa. `--text-disabled`/`--border-control` (`#7c7c7c`) **no** pasa 4.5:1 sobre `--surface-card` (3.99:1): queda restringido a texto deshabilitado y a bordes/marcas (§3.4).

---

## 3. Colores de datos y color de error (las dos únicas excepciones a la regla del acento único)

Verde, amarillo y rojo se usan **solo** para (a) el semáforo de órdenes y (b) los tramos de unidades paradas. Todas las demás gráficas (tendencias, cumplimiento de promesa, facturación, desempeño por asesor) usan `--accent` y escalas de gris. Verde/amarillo/rojo **no** se usan en alertas, botones, validación de formularios ni estados de carga. Para errores existe, por decisión del dueño, `--color-danger` (§3.6), que es un tono distinto del rojo de datos.

### 3.1 Semáforo (RN-002, días de calendario, DA-01)

| Estado | Días | Token | Hex | Texto de la insignia |
|---|---|---|---|---|
| Verde | 0 a 4 | `--data-green` | `#3ecf6e` | "Verde · 3 d" |
| Amarillo | 5 a 7 | `--data-yellow` | `#f2c94c` | "Amarillo · 6 d" |
| Rojo | más de 7 | `--data-red` | `#ff5c5c` | "Rojo · 12 d" |

### 3.2 Tramos de unidades paradas (solo Admin)

Secuencia: los tres primeros reutilizan el semáforo; los dos últimos son rojos cada vez más saturados e intensos (el tono gira hacia el carmesí) y **siguen siendo claros** contra el fondo oscuro. Nunca se oscurece hacia casi negro. Los tres rojos llevan además un **patrón** (§7.6) que crece en densidad.

| Tramo | Token | Hex | Equivale a | Patrón |
|---|---|---|---|---|
| 0–4 días | `--bucket-1` | `#3ecf6e` | Verde del semáforo | sólido |
| 5–7 días | `--bucket-2` | `#f2c94c` | Amarillo del semáforo | sólido |
| 8–15 días | `--bucket-3` | `#ff5c5c` | Rojo del semáforo | `pat-bucket-3`: rayado diagonal fino |
| 16–30 días | `--bucket-4` | `#ff3355` | Rojo saturado | `pat-bucket-4`: punteado |
| más de 30 días | `--bucket-5` | `#e8003d` | Rojo intenso (carmesí) | `pat-bucket-5`: rayado cruzado |

Límite honesto: los tres rojos difieren poco entre sí en luminancia (ratios entre vecinos de 1.18 y 1.30), por lo que **el color no basta para distinguirlos**. Por eso, además del patrón, la gráfica separa los segmentos con 1 px de `--surface-card`, la leyenda y el eje llevan el texto del tramo ("16–30 d"), cada segmento muestra su valor al pasar el cursor o enfocar, y existe una tabla equivalente (§7.6).

### 3.3 Fondos tenues para insignias (hex sólidos, sin transparencias)

| Token | Hex | Texto de la insignia sobre él |
|---|---|---|
| `--data-green-bg` | `#10301c` | `--text-primary` (14.37:1) |
| `--data-yellow-bg` | `#33290a` | `--text-primary` (14.36:1) |
| `--data-red-bg` | `#3a1212` | `--text-primary` (16.47:1) |
| `--accent-bg` | `#14243f` | `--text-primary` (15.51:1) |

La insignia = fondo tenue + borde 1 px del color pleno + punto del color pleno (≥3:1 sobre el tenue) + **texto `--text-primary`**. El texto no depende del color.

### 3.4 Verificación de contraste (calculada, fórmula WCAG 2.x)

Valores obtenidos con un script de luminancia relativa (sRGB → lineal; `(L1+0.05)/(L2+0.05)`), ejecutado con `python -I`. Umbrales: texto 4.5:1, marcas gráficas/componentes 3:1. Columnas: los tres fondos oscuros.

| Color | Hex | vs `#0a0a0a` | vs `#141414` | vs `#1e1e1e` | Texto (4.5) | Marca (3.0) |
|---|---|---|---|---|---|---|
| Blanco | `#ffffff` | 19.80 | 18.42 | 16.67 | Pasa | Pasa |
| Gris secundario | `#a7a7a7` | 8.23 | 7.66 | 6.93 | Pasa | Pasa |
| Gris deshabilitado | `#7c7c7c` | 4.74 | 4.41 | **3.99** | Pasa solo en canvas; falla en sección y tarjeta | Pasa |
| Acento | `#6798ff` | 7.08 | 6.59 | 5.96 | Pasa | Pasa |
| Verde | `#3ecf6e` | 9.76 | 9.08 | 8.22 | Pasa | Pasa |
| Amarillo | `#f2c94c` | 12.48 | 11.61 | 10.51 | Pasa | Pasa |
| Rojo | `#ff5c5c` | 6.54 | 6.09 | 5.51 | Pasa | Pasa |
| Rojo saturado | `#ff3355` | 5.52 | 5.14 | 4.65 | Pasa | Pasa |
| Rojo intenso | `#e8003d` | 4.24 | 3.95 | 3.57 | **Falla (no usar como texto)** | Pasa |
| **Danger (magenta)** | `#ff70e0` | **8.15** | **7.58** | **6.86** | **Pasa** | **Pasa** |
| Borde decorativo | `#313131` | 1.52 | 1.42 | 1.28 | No aplica | No (solo decorativo) |
| Borde de botón | `#454545` | 2.06 | 1.92 | 1.74 | No aplica | No (el botón tiene texto) |

Conclusiones y reglas que se derivan:

- `--text-disabled`/`--border-control` (`#7c7c7c`): sobre `#141414` da 4.41 y sobre `#1e1e1e` 3.99, así que **no se usa para texto legible**; solo texto deshabilitado (exento) y bordes de control (3.99 / 4.74 ≥ 3). El placeholder usa `--text-secondary`.
- `--bucket-5` (`#e8003d`, >30 días) **no se usa como texto**; solo como relleno de segmento y del punto de la leyenda. El texto del tramo siempre va en `--text-primary` o `--text-secondary`.
- El borde de campos (`--border-control`) cumple el 3:1 de componentes (3.99 sobre tarjeta). Los bordes `--border-subtle`/`--border-strong` son solo separadores.
- `--color-danger` pasa texto (≥4.5) sobre los tres fondos oscuros, así que puede usarse como texto de mensaje de error de 12 px y como borde.

### 3.5 Gráficas que no son semáforo/tramos

Serie principal `--chart-1` (`--accent`); series adicionales `--chart-2` (`#a7a7a7`) y `--chart-3` (`#7c7c7c`, marcas, 6.93/3.99 sobre tarjeta); ejes y cuadrícula `--chart-grid` (decorativos; el valor siempre está en texto). Hasta 3 series; si hay más, usar agrupación o tabla. **`--color-danger` nunca se usa en gráficas.**

### 3.6 Color de error `--color-danger` (decisión del dueño, 26-oct-2026)

El dueño decidió que el error tenga un color propio, **distinto del rojo de datos**, para que un aviso de error no se lea como "unidad en rojo". Es la segunda excepción a la regla del acento único.

| Token | Hex | Uso |
|---|---|---|
| `--color-danger` | `#ff70e0` | Texto del mensaje de error, borde de campo inválido, borde/barra/ícono de alerta de error |
| `--color-danger-bg` | `#3a1233` | Fondo tenue sólido de la alerta de error (hex sólido, sin transparencia) |

**Por qué este tono:** un rosa-magenta de tono 313°, girado hacia el púrpura. Queda alejado del rojo de datos y de los tres rojos de tramos, y es claro (como todos los colores de este sistema) para pasar contraste sobre fondos oscuros.

**Diferencia de tono (hue, HSV, calculada):**

| Comparado con | Hue del otro | Hue de `--color-danger` | Diferencia |
|---|---|---|---|
| `#ff5c5c` (semáforo rojo / `--bucket-3`) | 0.0° | 313.0° | **47.0°** |
| `#ff3355` (`--bucket-4`) | 350.0° | 313.0° | **37.0°** |
| `#e8003d` (`--bucket-5`) | 344.2° | 313.0° | **31.2°** |

Las tres diferencias son ≥ 30°.

**Contraste (§3.4 y script):** `--color-danger` vs `#0a0a0a` 8.15, vs `#141414` 7.58, vs `#1e1e1e` 6.86. Sobre su fondo tenue `--color-danger-bg` (`#3a1233`): `--text-primary` 15.98:1, `--text-secondary` 6.64:1 y el propio `--color-danger` 6.58:1; el fondo tenue contra `#1e1e1e` es solo 1.04:1 (es un tinte, la alerta se delimita con borde y barra).

**Reglas de uso:**
1. Solo en: (a) mensajes de error, (b) bordes de campos inválidos, (c) alertas de error (borde, barra, ícono y fondo tenue).
2. **Siempre acompañado de ícono y texto** (octágono con "×" y prefijo "Error:"). Nunca el color solo.
3. **Nunca en gráficas**, ni en el semáforo, tramos, insignias, botones o deltas.
4. El botón de **peligro** no usa `--color-danger` (decisión de diseño, §7.1): es una acción, no un error, y conserva borde blanco + ícono + texto explícito + confirmación. Así el magenta significa una sola cosa: "algo salió mal".

---

## 4. Tipografía

- **Inter** (`--font-sans`; 400 cuerpo, 500 navegación y botones, 600 encabezados) y **JetBrains Mono** (`--font-mono`; 400) para VIN, números de orden, identificadores y códigos de archivo.
- **Tamaño máximo dentro de la aplicación: 40 px** (`--text-heading`, solo cifra de tarjeta de estadística). No existen tamaños de 56 ni 64.
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

Cuerpo mínimo `--text-body-sm`; solo etiquetas auxiliares y microtexto llegan a `--text-caption`.

---

## 5. Tokens como variables CSS

> **Regla de tokens:** los componentes solo usan `var(--…)`. Ningún hex, `px` de radio ni nombre de fuente fuera de este bloque y del bloque `.theme-print` (§11). Excepción técnica única: los puntos de corte de `@media` (560 px y 900 px) no admiten `var()`; se escriben como constantes en la hoja global de layout, nunca en componentes.

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

  /* Botón primario */
  --button-primary-bg: #ffffff;
  --button-primary-fg: #0a0a0a;
  --button-primary-hover: #e6e6e6;

  /* Acento */
  --accent: #6798ff;
  --accent-bg: #14243f;

  /* Error (segunda excepción al acento único, decisión del dueño 26-oct-2026):
     solo mensajes de error, bordes de campo inválido y alertas de error;
     siempre con ícono y texto; nunca en gráficas. */
  --color-danger: #ff70e0;
  --color-danger-bg: #3a1233;

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

  /* Patrones de los tramos rojos (§7.6): ids de <pattern> y geometría */
  --pattern-ink: #0a0a0a;          /* color de las líneas/puntos: oscuro sobre el rojo */
  --pattern-angle: 45deg;
  --pattern-3-cell: 8px;           /* id: pat-bucket-3, rayado diagonal fino */
  --pattern-3-stroke: 0.75px;
  --pattern-4-cell: 6px;           /* id: pat-bucket-4, punteado */
  --pattern-4-dot-radius: 1.2px;
  --pattern-5-cell: 8px;           /* id: pat-bucket-5, rayado cruzado */
  --pattern-5-stroke: 0.75px;

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
  --form-max-width: 480px;
  --modal-width: 440px;
  --panel-width: 480px;
  --table-viewport-offset: 160px;
  --chart-label-max-width: 200px;
  --empty-state-max-width: 360px;

  /* Radios (sin píldoras) */
  --radius-tag: 4px;
  --radius-control: 8px;
  --radius-card: 8px;
  --radius-button: 8px;

  /* Bordes y trazos */
  --border-width: 1px;
  --border-width-strong: 2px;
  --alert-bar-width: 3px;
  --alert-bar-width-error: 4px;
  --icon-stroke: 1.5px;

  /* Alturas y tamaños */
  --topbar-height: 48px;
  --row-height: 40px;
  --table-header-height: 40px;
  --control-height: 36px;
  --control-height-lg: 40px;
  --badge-height: 22px;
  --badge-dot-size: 8px;
  --chip-height: 24px;
  --icon-size-sm: 12px;
  --icon-size: 16px;
  --icon-size-lg: 20px;
  --semaforo-col-width: 160px;
  --bar-height: 24px;
  --bar-gap: 12px;
  --legend-swatch-size: 16px;
  --dropzone-min-height: 160px;
  --progress-height: 4px;
  --pagination-height: 40px;
  --pagination-button-size: 32px;
  --switch-width: 32px;
  --switch-height: 18px;

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

- Radio `--radius-control`/`--radius-card`/`--radius-button` (8 px) en botones, campos, tarjetas, tablas y modales; `--radius-tag` (4 px) en etiquetas, insignias y chips. Prohibido el radio de píldora.
- **Sin sombras y sin degradados** en ninguna superficie, botón o fondo. No hay motivo de cuadrícula decorativo de fondo.
- Los modales se separan con un velo sólido `--surface-canvas` sobre el contenido (sin desenfoque ni sombra) y una tarjeta `--surface-card` con borde `--border-subtle`.
- Íconos: trazo `--icon-stroke`, tamaño `--icon-size` (`--icon-size-lg` en estados vacíos), color heredado del texto; `--accent` solo en elementos activos.
- Foco: anillo `--focus-ring-width` en `--focus-ring-color` (`--accent`) con separación `--focus-ring-offset`, en **todo** elemento interactivo (7.08:1 sobre canvas, 5.96:1 sobre tarjeta). Nunca se quita sin sustituto. Orden de tabulación = orden visual.

---

## 7. Componentes

**Cada especificación cita el token como referencia principal**; el valor entre paréntesis es solo referencia visual (regla de tokens). Si un valor no tiene token, se añade a §5 antes de usarlo.

Estados comunes: **hover** (fondo `--surface-raised` o sube un tono), **foco** (anillo, §6), **deshabilitado** (texto `--text-disabled`, sin hover, `cursor: not-allowed`, `aria-disabled`), **error** (§7.8).

### 7.1 Botones

Alto `--control-height` (36 px); padding 0 `--space-16`; `--font-sans`, peso `--font-weight-medium`, `--text-body-sm`; radio `--radius-button`; ícono opcional de `--icon-size` a la izquierda con `--space-8` de separación. Sin flechitas decorativas.

| Variante | Fondo | Texto | Borde | Hover | Deshabilitado |
|---|---|---|---|---|---|
| **Primario** | `--button-primary-bg` (`#ffffff`) | `--button-primary-fg` (`#0a0a0a`) | ninguno | fondo `--button-primary-hover` (`#e6e6e6`, hex sólido) | fondo `--surface-raised`, texto `--text-disabled` |
| **Secundario** | transparente | `--text-primary` | `--border-width` `--border-strong` | fondo `--surface-card`, borde `--border-control` | borde `--border-subtle`, texto `--text-disabled` |
| **Fantasma** | transparente | `--text-primary` | ninguno | fondo `--surface-raised` | texto `--text-disabled` |
| **Peligro** | transparente | `--text-primary` | `--border-width` `--text-primary` | fondo `--surface-raised` | borde `--border-subtle`, texto `--text-disabled` |

Reglas:
- Un solo botón primario por vista o modal. Acciones de baja frecuencia: secundario o fantasma.
- **Peligro: decisión de diseño — se queda sin `--color-danger`.** `--color-danger` está reservado por el dueño a mensajes, campos y alertas de error (§3.6); un botón de acción destructiva no es un error. Se distingue por borde blanco, ícono de advertencia (triángulo) y texto explícito ("Dar de baja usuario", no "Aceptar"), y siempre pide **confirmación** en un modal que nombra el objeto afectado y cuyo botón de confirmar repite la acción. Si luego se quisiera teñirlo, hay que ampliar la regla del §3.6 con aprobación del dueño. El botón primario del modal de confirmación sigue siendo blanco.
- Estado de carga: el texto cambia ("Subiendo…"), se muestra un indicador de `--icon-size` de trazo, el botón queda deshabilitado y conserva su ancho.
- `--button-primary-hover` (`#e6e6e6`) es el único gris añadido a la paleta neutra, usado solo como estado hover del primario (texto `--button-primary-fg` sobre él: 15.86:1).

### 7.2 Barra superior con navegación por rol

- Alto `--topbar-height` (48 px), fondo `--surface-canvas`, borde inferior `--border-width` `--border-subtle`, fija arriba (`position: sticky; top: 0`), contenido alineado a `--page-max-width-wide`.
- Izquierda: marca provisional "MERP" (`--font-sans`, `--font-weight-semibold`, `--text-body`) con `--space-24` de separación; después los enlaces de navegación (`--font-weight-medium`, `--text-body-sm`, `--text-secondary`, separados `--space-8`, cada uno con padding 0 `--space-12` y alto `--topbar-height`).
- Hover: texto `--text-primary`. **Enlace activo:** texto `--text-primary` y línea inferior de `--border-width-strong` en `--accent`, más `aria-current="page"`. Foco: anillo estándar.
- Derecha: nombre del usuario y rol en `--text-secondary` ("Dealer · Dealer Demo Norte"), menú de usuario (botón fantasma) con "Cambiar contraseña" y "Cerrar sesión".
- **Navegación por rol:**
  - **Admin:** Dashboard · Panel · Unidades paradas · Dealers · Usuarios · Cargas · Perfiles de importación · Reportes. "Panel" abre con un selector de dealer, que incluye los dados de baja marcados «(baja)».
  - **Dealer:** Dashboard · Panel · Cargas · Reportes. "Cargas" = zona de arrastre + resumen + historial (MERP-112 §8).
  - Esta lista es una **propuesta a confirmar con las pantallas de cada fase**.
  - Un enlace que el rol no tiene **no se muestra** (no se deshabilita). El servidor igualmente valida (P-3).
- En pantallas <900 px los enlaces pasan a un menú desplegable (botón "Menú" con `aria-expanded`).

### 7.3 Insignia de semáforo

- Alto `--badge-height` (22 px), padding 0 `--space-8`, radio `--radius-tag`, `--font-weight-medium`, `--text-caption`, texto `--text-primary`, fondo tenue `--data-*-bg` y borde `--border-width` del color pleno `--data-*` (§3.3), punto redondo de `--badge-dot-size` del color pleno a la izquierda con `--space-8` de separación.
- Contenido: etiqueta + días, con numerales tabulares: "Verde · 3 d", "Amarillo · 6 d", "Rojo · 12 d". Sin días (resumen): solo "Verde".
- `aria-label` completo: "Semáforo rojo, 12 días". Es solo lectura: no tiene hover ni acción.
- Una orden completada con semáforo congelado (RN-003) añade un ícono de candado de `--icon-size-sm` y `title`/texto "congelado".

### 7.4 Tablas densas

Se usan en dashboard, lista de unidades, cargas, usuarios y dealers.

- Contenedor: fondo `--surface-card`, radio `--radius-card`, borde `--border-width` `--border-subtle`, `overflow: auto`, `max-height: calc(100vh - var(--topbar-height) - var(--table-viewport-offset))` (la tabla hace scroll propio dentro del contenedor).
- **Encabezado fijo (sticky):** alto `--table-header-height`, fondo `--surface-section`, borde inferior `--border-width` `--border-subtle`, `--font-weight-medium`, `--text-caption`, `--text-secondary`, `position: sticky; top: 0`.
- **Filas** de `--row-height`, `--text-body-sm`, padding horizontal `--space-12`, divisor `--border-width` `--border-subtle`. Hover: fondo `--surface-raised`. Fila seleccionada: borde izquierdo `--border-width-strong` `--accent` y fondo `--accent-bg`. Foco de fila clicable: anillo estándar inset.
- **Columna de semáforo:** primera o segunda columna, ancho fijo `--semaforo-col-width`, contiene la insignia (§7.3).
- **Números:** alineados a la **derecha**, `tabular-nums`; el encabezado de esa columna también a la derecha. Importes con separador de miles y moneda explícita; VIN y orden en `--font-mono` `--text-mono`, a la izquierda.
- **Orden por columna:** el encabezado ordenable es un botón; muestra ícono de flecha (↑ ascendente, ↓ descendente) de `--icon-size-sm` y `aria-sort`. Clic alterna ascendente → descendente → sin orden. Columna ordenada: texto `--text-primary`. El orden por defecto en dashboard lo define el servidor (DA-07: grupos alfabéticos por dealer, urgencia dentro del grupo).
- Texto largo: una línea con elipsis y `title`. Sin franjas alternadas.
- Estados: cargando (filas esqueleto de `--row-height` en `--surface-raised` sin animación de brillo), vacío (§7.11), error (alerta en lugar del cuerpo, §7.12).
- Encabezado de grupo (Dashboard agrupado por dealer): fila de `--row-height`, fondo `--surface-section`, nombre del dealer en `--font-weight-semibold` `--text-body-sm` y conteo en `--text-secondary`.

### 7.5 Tarjeta de estadística

- Fondo `--surface-card`, borde `--border-width` `--border-subtle`, radio `--radius-card`, padding `--card-padding-lg`, ancho flexible en rejillas de 4 columnas (2 en <900 px, 1 en <560 px), separación `--element-gap-lg`.
- Orden: etiqueta (`--font-weight-medium`, `--text-body-sm`, `--text-secondary`) arriba; cifra (`--font-weight-semibold`, `--text-heading` (40 px), `--text-primary`, `tabular-nums`) debajo; nota o comparación (`--text-caption`, `--text-secondary`, p. ej. "+3 vs. semana anterior") al fondo. Ícono opcional de `--icon-size` en `--accent`.
- Las tarjetas del resumen de semáforo (en proceso, verde, amarillo, rojo, completadas) llevan además la insignia/punto del color y su nombre en texto; el valor sigue en `--text-primary`.
- Si es clicable (abre la lista filtrada): es un `<a>`/botón, hover borde `--border-control`, foco estándar. Sin elevación.
- Estado sin dato: cifra "—" en `--text-secondary` y nota "No disponible" con su motivo (RN-023).

### 7.6 Gráfica de barras apiladas — unidades paradas (solo Admin)

- **Datos:** una barra horizontal por dealer, con cinco tramos apilados (0–4, 5–7, 8–15, 16–30, >30 d), ordenadas de más a menos unidades totales.
- **Colores:** `--bucket-1…5` (§3.2). Segmentos separados por `--border-width` de `--surface-card`; barras rectas, radio 0.
- **Patrones (decisión del dueño, 26-oct-2026):** los tres tramos rojos (8–15, 16–30, >30 d) llevan un patrón sutil además de la leyenda y las etiquetas con el número. Verde y amarillo quedan **sólidos**.

  **Secuencia y justificación.** Crece la complejidad y la cobertura de tinta, de modo que se lee como "cada vez más intenso": rayado diagonal fino (una sola dirección) → punteado (más masa visual por celda) → rayado cruzado (dos direcciones). La cobertura de tinta es 9.4 % → 12.6 % → 17.9 %. Los tres son distinguibles también en escala de grises e impresión monocroma.

  | Tramo | Id SVG | Patrón | Celda | Línea / punto | Ángulo | Color de tinta | Cobertura |
  |---|---|---|---|---|---|---|---|
  | 8–15 d | `pat-bucket-3` | rayado diagonal fino | `--pattern-3-cell` (8 px) | trazo `--pattern-3-stroke` (0.75 px) | `--pattern-angle` (45°) | `--pattern-ink` | 9.4 % |
  | 16–30 d | `pat-bucket-4` | punteado en rejilla | `--pattern-4-cell` (6 px) | radio `--pattern-4-dot-radius` (1.2 px) | sin rotación | `--pattern-ink` | 12.6 % |
  | >30 d | `pat-bucket-5` | rayado cruzado (rombos) | `--pattern-5-cell` (8 px) | trazo `--pattern-5-stroke` (0.75 px), dos direcciones | `--pattern-angle` (45°) | `--pattern-ink` | 17.9 % |

  **Implementación SVG** (un solo `<defs>` por documento; las barras y la leyenda referencian los mismos ids). Las medidas salen de los tokens: el componente lee los tokens una vez con `getComputedStyle` y los pasa a los atributos geométricos (`width`, `height`, `x1`…), porque esos atributos no aceptan `var()`; los colores sí se aplican con `var()` en `style`. Los valores leídos se convierten con `parseFloat()` antes de escribirlos en los atributos (`rotate(45)`, `width="8"`, `cx = celda / 2`); los que van en `style` (`stroke-width`, `fill`) se usan con `var()` sin convertir.

  ```svg
  <defs>
    <!-- 8–15 d: rayado diagonal fino. Celda = --pattern-3-cell; la línea va centrada (x = celda/2) -->
    <pattern id="pat-bucket-3" patternUnits="userSpaceOnUse"
             width="[--pattern-3-cell]" height="[--pattern-3-cell]"
             patternTransform="rotate([parseFloat(--pattern-angle)])">
      <rect width="[--pattern-3-cell]" height="[--pattern-3-cell]" style="fill: var(--bucket-3)"/>
      <line x1="[celda/2]" y1="0" x2="[celda/2]" y2="[--pattern-3-cell]"
            style="stroke: var(--pattern-ink); stroke-width: var(--pattern-3-stroke)"/>
    </pattern>

    <!-- 16–30 d: punteado. Punto centrado en la celda -->
    <pattern id="pat-bucket-4" patternUnits="userSpaceOnUse"
             width="[--pattern-4-cell]" height="[--pattern-4-cell]">
      <rect width="[--pattern-4-cell]" height="[--pattern-4-cell]" style="fill: var(--bucket-4)"/>
      <circle cx="[celda/2]" cy="[celda/2]" r="[--pattern-4-dot-radius]"
              style="fill: var(--pattern-ink)"/>
    </pattern>

    <!-- >30 d: rayado cruzado. Dos líneas perpendiculares centradas; girado 45° forma rombos -->
    <pattern id="pat-bucket-5" patternUnits="userSpaceOnUse"
             width="[--pattern-5-cell]" height="[--pattern-5-cell]"
             patternTransform="rotate([parseFloat(--pattern-angle)])">
      <rect width="[--pattern-5-cell]" height="[--pattern-5-cell]" style="fill: var(--bucket-5)"/>
      <line x1="[celda/2]" y1="0" x2="[celda/2]" y2="[--pattern-5-cell]"
            style="stroke: var(--pattern-ink); stroke-width: var(--pattern-5-stroke)"/>
      <line x1="0" y1="[celda/2]" x2="[--pattern-5-cell]" y2="[celda/2]"
            style="stroke: var(--pattern-ink); stroke-width: var(--pattern-5-stroke)"/>
    </pattern>
  </defs>
  ```

  Las líneas se centran en la celda (no en el borde) para que no se recorten a la mitad al teselar. Segmentos 1 y 2 usan `fill: var(--bucket-1)` / `var(--bucket-2)` sin patrón. Para que el patrón se imprima en el navegador, la gráfica en PDF usa `print-color-adjust: exact` (§11).

  **Por qué el relleno sigue cumpliendo 3:1.** La tinta `--pattern-ink` es oscura (canvas, `#0a0a0a`) y cubre como máximo 17.9 % del segmento; los píxeles restantes conservan el color pleno del tramo, que ya cumple 3:1. Además la tinta oscura **no puede aclarar** la marca: solo la oscurece hacia el fondo, y el fondo es oscuro, así que la zona tintada se funde con el fondo en lugar de competir. Aun así se verificó el peor caso: luminancia media del segmento (mezcla lineal de relleno y tinta según la cobertura) contra el fondo de tarjeta y de canvas.

  | Tramo + patrón | Cobertura | Relleno solo vs `#1e1e1e` | Medio con patrón vs `#1e1e1e` | Medio con patrón vs `#0a0a0a` | Tinta vs relleno (se distingue el patrón) |
  |---|---|---|---|---|---|
  | `--bucket-3` + `pat-bucket-3` | 9.4 % | 5.51 | **5.07** | 6.02 | 6.54 |
  | `--bucket-4` + `pat-bucket-4` | 12.6 % | 4.65 | **4.17** | 4.95 | 5.52 |
  | `--bucket-5` + `pat-bucket-5` | 17.9 % | 3.57 | **3.08** | 3.66 | 4.24 |

  Todos ≥ 3:1. El tramo >30 d es el más ajustado (3.08 sobre tarjeta): **por eso la cobertura del rayado cruzado no debe aumentarse** (el techo teórico sobre `#1e1e1e` es ≈ 20.7 %). Si se cambiaran `--pattern-5-cell` o `--pattern-5-stroke`, hay que recalcular.
- **Medidas:** barra de `--bar-height` (24 px), `--bar-gap` (12 px) entre barras, etiqueta del dealer a la izquierda (`--text-body-sm`, máx. `--chart-label-max-width` con elipsis), total al final (`--text-body-sm`, tabular, `--text-primary`). Eje de valores en `--text-secondary` `--text-caption`; cuadrícula `--chart-grid` de `--border-width`.
- **Leyenda:** arriba de la gráfica, fila horizontal; cada ítem = cuadro de `--legend-swatch-size` (16 px) que **muestra el mismo patrón** (un `<svg>` con `fill: url(#pat-bucket-N)`; los tramos 1 y 2, el color sólido) + texto del tramo ("16–30 d") en `--text-primary` `--text-caption`. Cada ítem es un botón para **resaltar** el tramo (los demás segmentos se atenúan a `--chart-3` (3.99:1)); opcional.
- **Interacción:** hover/foco sobre un segmento muestra tooltip sólido (`--surface-raised`, sin sombra): "Dealer Demo Norte · 8–15 d · 14 unidades". **Clic en un segmento abre la lista** de esas unidades (VIN, modelo, orden, dealer, días) en un panel lateral de `--panel-width` o en una vista con tabla (§7.4). Cada segmento es un elemento enfocable con `role="button"` y `aria-label` completo; teclado: Tab entra a la gráfica; flechas ←/→ entre tramos y ↑/↓ entre dealers (tabindex itinerante); Enter abre la lista.
- **Alternativa accesible:** botón "Ver como tabla" que muestra la misma información como tabla (dealer × tramos) con las cinco columnas; es la fuente textual para quien no distingue los tonos de rojo.
- Estado vacío: "No hay unidades paradas." con ícono; sin datos de un dealer: barra omitida.

### 7.7 Filtros y selectores

- **Campo de selección / filtro:** alto `--control-height`, fondo `--surface-card`, borde `--border-width` `--border-control`, radio `--radius-control`, texto `--text-body-sm`; ícono de chevron `--icon-size` a la derecha. Hover: borde `--text-secondary`. Foco: anillo estándar. Abierto: lista en tarjeta `--surface-card` con borde `--border-subtle`, opción activa con fondo `--surface-raised`, opción elegida con check `--accent` y `aria-selected`.
- **Filtro por dealer** (Admin): selector con búsqueda; opción "Todos los dealers" por defecto.
- **Chips de filtro aplicados:** alto `--chip-height`, radio `--radius-tag`, fondo `--surface-raised`, texto `--text-caption`, botón "Quitar" (×) con `aria-label`. Enlace fantasma "Limpiar filtros".
- **Filtro por semáforo / tramo:** grupo de botones conmutables (`aria-pressed`): seleccionado = borde `--accent` y texto `--text-primary`, con **la etiqueta de texto**, no solo el color.
- La barra de filtros va sobre la tabla, en fila con separación `--element-gap-sm`, fondo `--surface-section`, padding `--space-12`, radio `--radius-card`. Aplicar filtros es inmediato; el estado queda en la URL.

### 7.8 Formularios y validación

- **Etiqueta** sobre el campo (`--font-weight-medium`, `--text-body-sm`, `--text-primary`), texto de ayuda debajo (`--text-caption`, `--text-secondary`); separación entre campos `--element-gap-lg`; ancho máximo de formulario `--form-max-width`.
- **Campo de texto:** alto `--control-height`, fondo `--surface-card`, borde `--border-width` `--border-control`, radio `--radius-control`, padding 0 `--space-12`, placeholder `--text-secondary`. Hover: borde `--text-secondary`. Foco: borde `--accent` más anillo estándar. Deshabilitado: fondo `--surface-section`, borde `--border-subtle`, texto `--text-disabled`.
- **Error (usa `--color-danger`, §3.6):** borde `--border-width-strong` `--color-danger` (6.86:1 sobre tarjeta), ícono de error (octágono con ×, `--icon-size`, en `--color-danger`) a la derecha dentro del campo, y debajo un mensaje en `--color-danger` `--text-caption` **con prefijo de texto**: "Error: la contraseña debe tener al menos 10 caracteres y un carácter especial." (6.86:1 sobre tarjeta, 8.15:1 sobre canvas). El campo recibe `aria-invalid="true"` y `aria-describedby` apuntando al mensaje. El color nunca va solo: ícono, prefijo "Error:" y borde más grueso (2 px frente a 1 px). Resumen de errores arriba del formulario cuando hay varios, con enlaces a cada campo (usa la alerta de error, §7.12).
- **Cuándo validar:** al salir del campo (blur) y al enviar; nunca mientras se escribe por primera vez. El mensaje dice qué corregir, no solo qué falló.
- **Obligatorio:** se marca en la etiqueta con "(obligatorio)" en `--text-secondary`; no solo con asterisco.
- **Contraseña:** botón fantasma "Mostrar/Ocultar" dentro del campo; criterios (mínimo 10 caracteres, un carácter especial, DA-05) listados debajo con ícono check/punto y texto, cumplidos en `--text-primary`, pendientes en `--text-secondary` (los pendientes no usan `--color-danger`: aún no son un error).
- **Casillas y opciones:** cuadro de `--icon-size`, borde `--border-width` `--border-control`, radio `--radius-tag`; marcado = fondo `--accent` con check `--text-on-light`. Interruptor de `--switch-width` × `--switch-height` solo si hace falta, con texto de estado ("Activo").
- Envío: botón primario; mientras se envía, deshabilitado con "Guardando…". Éxito: alerta de éxito (§7.12).

### 7.9 Zona de arrastrar archivo (.xlsx)

- Tarjeta `--surface-card`, borde **discontinuo** `--border-width` `--border-control`, radio `--radius-card`, alto mínimo `--dropzone-min-height`, contenido centrado: ícono de subir (`--icon-size-lg`, `--accent`), texto "Arrastra tu archivo .xlsx aquí" (`--font-weight-medium`, `--text-body-sm`) y "o selecciónalo" como botón secundario; ayuda en `--text-caption` `--text-secondary`: "Solo .xlsx. Tamaño máximo según la configuración del sistema."
- **Hover / arrastre encima:** borde `--border-width` sólido `--accent` y fondo `--accent-bg`; texto "Suelta para cargar".
- **Foco:** anillo estándar (la zona es un botón que abre el selector de archivos; también se puede pegar con teclado).
- **Archivo elegido:** muestra nombre (`--font-mono` `--text-mono`), tamaño y botón fantasma "Quitar"; botón primario "Subir archivo".
- **Subiendo:** barra de progreso de `--progress-height` (`--surface-raised` pista, `--accent` avance) con texto "Procesando… (n %)"; botón deshabilitado.
- **Error de archivo** (formato no .xlsx, vacío, ilegible): borde sólido `--border-width-strong` `--color-danger`, ícono de error y mensaje en `--color-danger` con prefijo "Error:" ("Error: el archivo no es un .xlsx válido."). Deshabilitado: borde `--border-subtle`, texto `--text-disabled`.
- Solo un archivo a la vez. El Excel original no se guarda (DA-13); el texto de ayuda no promete conservarlo.

### 7.10 Resumen de carga

Se muestra tras subir (RN de carga: filas totales = nuevas + actualizadas + sin cambios + rechazadas).

- Rejilla de tarjetas de estadística pequeñas (§7.5, cifra de `--text-heading-sm` aquí para no dominar): **Nuevas · Actualizadas · Sin cambios · Rechazadas** (suman las filas totales, mostradas como "Filas totales: 30") y, separada por un divisor `--border-width` `--border-subtle`, la tarjeta **Avisos** (no suma al total). Se muestra también "Versión del perfil: vN" (RN-026). Todas en `--text-primary`/`--text-secondary`, sin color de semáforo. Rechazadas y Avisos llevan un ícono (octágono / triángulo) además del número; si su valor es mayor que 0 el borde de la tarjeta pasa a `--text-primary` (las dos tarjetas; nunca `--color-danger`, que se reserva a la alerta).
- Debajo, si hay rechazadas: tabla densa (§7.4) con **Fila · Motivo**, ordenada por fila, precedida de una alerta de error (§7.12) con el texto: "Error: 3 filas rechazadas. Corrige estas filas en tu sistema de origen y vuelve a subir el archivo."
- Si hay avisos (datos ilegibles cargados con aviso, VIN modificado, etc.): alerta de advertencia (§7.12) con la lista.
- Si cambió el perfil de importación: alerta informativa que lo indica.
- Una carga sin ninguna fila válida muestra alerta de error con "0 filas válidas" y la tabla de rechazos; la carga igual queda registrada.
- Acciones: botón secundario "Descargar PDF de la carga" y primario "Ir al dashboard".
- Ejemplo de datos ficticios: 30 filas — 18 nuevas, 4 actualizadas, 5 sin cambios, 3 rechazadas.

### 7.11 Estados vacíos

- Bloque centrado dentro de la tarjeta o tabla, padding `--space-40`: ícono de `--icon-size-lg` en `--text-secondary`, título `--font-weight-semibold` `--text-body`, descripción `--text-body-sm` `--text-secondary` (máx. `--empty-state-max-width`) y, si procede, un botón (primario si es la acción siguiente natural).
- Ejemplos: "Aún no has subido archivos" · "Sube tu primer .xlsx para ver tu dashboard." [Subir archivo]; "No hay unidades paradas" · "Ninguna unidad supera los 4 días."; "Sin resultados" · "Prueba quitando algún filtro." [Limpiar filtros].
- Sin ilustraciones decorativas, sin tono humorístico.

### 7.12 Alertas y avisos

**Decisión:** éxito, advertencia e información **no** usan verde, amarillo ni rojo (esos son solo del semáforo y los tramos). El **error** usa `--color-danger` (decisión del dueño, 26-oct-2026, §3.6), un magenta distinto del rojo de datos, de modo que un aviso de error no se lea como "unidad en rojo". Cada tipo se distingue por **ícono de forma distinta + prefijo de texto + borde**, nunca solo por color.

Estructura común: tarjeta `--surface-card`, radio `--radius-card`, padding `--space-12` `--space-16`, ícono de `--icon-size` a la izquierda, título en `--font-weight-medium` `--text-body-sm` (con prefijo), cuerpo `--text-body-sm` `--text-secondary`, botón de cerrar (fantasma, `aria-label="Cerrar aviso"`) cuando es descartable. Contenedor con `role="status"` (info/éxito) o `role="alert"` (advertencia/error).

| Tipo | Ícono (forma) | Prefijo del título | Fondo | Borde |
|---|---|---|---|---|
| Información | círculo con "i" | "Información:" | `--surface-card` | `--border-width` `--border-subtle` + barra izquierda `--alert-bar-width` `--accent`; ícono `--accent` |
| Éxito | círculo con check | "Listo:" | `--surface-card` | `--border-width` `--text-secondary` |
| Advertencia | triángulo con "!" | "Advertencia:" | `--surface-card` | `--border-width` `--text-primary` |
| Error | octágono con "×" | "Error:" | `--color-danger-bg` (`#3a1233`) | `--border-width-strong` `--color-danger` + barra izquierda `--alert-bar-width-error` `--color-danger`; ícono `--color-danger`; título y cuerpo en `--text-primary` |

Verificación del error: `--text-primary` sobre `--color-danger-bg` 15.98:1; `--text-secondary` sobre él 6.64:1; `--color-danger` (ícono, borde) sobre él 6.58:1 y sobre `--surface-card` 6.86:1.

Comportamiento: éxito se descarta solo tras 6 s (y queda accesible al lector de pantalla); advertencia y error **no** se descartan solos. Los avisos de página van arriba del contenido; los de campo, bajo el campo (§7.8). Errores de red incluyen acción "Reintentar".

### 7.13 Paginación

- Barra bajo la tabla, alto `--pagination-height`, fondo `--surface-section`, borde superior `--border-width` `--border-subtle`, radio inferior `--radius-card`.
- Izquierda: "Mostrando 21–40 de 132" (`--text-caption`, `--text-secondary`, `tabular-nums`) y selector "Filas por página" (20, 50, 100).
- Derecha: botones fantasma de `--pagination-button-size`: "Anterior", números (página actual con borde `--border-width` `--accent` y `aria-current="page"`), "Siguiente"; deshabilitados en los extremos (texto `--text-disabled`). Elipsis en rangos largos.
- Se usa en historial de cargas (DA-06, con filtro por dealer para el Admin) y listas largas. El cambio de página devuelve el foco al inicio de la tabla.

### 7.14 Modal de confirmación

Tarjeta `--surface-card` de `--modal-width` de ancho, borde `--border-width` `--border-subtle`, radio `--radius-card`, padding `--card-padding-lg`, título `--font-weight-semibold` `--text-subheading`, cuerpo `--text-body-sm`; acciones alineadas a la derecha (secundario "Cancelar" + primario o peligro). Foco atrapado, `Esc` cierra, el foco vuelve al disparador, `role="dialog"` con `aria-labelledby`.

---

## 8. Layout

- Contenedor: `--page-max-width` (1200 px) para formularios y paneles; `--page-max-width-wide` (1440 px) para tablas y gráficas. Padding lateral `--page-padding-x` (24 px); `--space-16` en <560 px.
- Estructura de página: barra superior (`--topbar-height`) → título de página (`--text-heading-sm`) con acciones a la derecha → filtros → contenido. Separación vertical entre bloques `--section-gap-sm`; entre secciones mayores `--section-gap-lg`.
- Sin barra lateral. Sin pie de página de sitio; opcionalmente una línea de `--text-caption` `--text-secondary` con "Última carga: …" (dato útil, no marketing).
- Responsive: diseño pensado para escritorio; hasta 560 px, las tablas hacen scroll horizontal dentro de su contenedor con la primera columna fija; las rejillas de tarjetas colapsan.

## 9. Accesibilidad (resumen)

- Contraste: texto ≥ 4.5:1 y marcas ≥ 3:1 contra `--surface-canvas`, `--surface-section` y `--surface-card` (§3.4). Excepciones controladas: `--text-disabled`/`--border-control` (`#7c7c7c`; solo texto deshabilitado, bordes de control y serie 3 de gráficas), `--bucket-5` (`#e8003d`; solo marca, con patrón verificado ≥3:1, §7.6). `--color-danger` pasa texto y marca sobre los tres fondos (8.15 / 7.58 / 6.86).
- Foco visible en todo; operable por teclado (menús, tabla, gráfica, zona de arrastre, modales).
- El color nunca es la única señal: semáforo con texto y días; tramos con **patrón**, etiqueta y tabla alterna; alertas con ícono y prefijo; errores de campo con ícono, prefijo "Error:" y borde más grueso; `--color-danger` siempre con ícono y texto.
- `prefers-reduced-motion` respetado; movimiento solo de `--motion-fast` en hover/foco.
- Lectores de pantalla: `aria-sort`, `aria-current`, `aria-invalid`, `role="alert"/"status"`, `aria-label` completo en insignias y segmentos.
- Idioma de la página `lang="es-MX"`; fechas `dd/mm/aaaa`, hora local de planta; moneda con separador de miles.

## 10. Qué NO hacer

- No hardcodear colores, fuentes ni radios en componentes: todo sale de tokens (regla de tokens).
- No usar verde, amarillo ni rojo fuera del semáforo y los tramos (ni en alertas, validación, botones, deltas "sube/baja").
- No usar `--color-danger` fuera de mensajes de error, bordes de campos inválidos y alertas de error; ni solo (sin ícono y texto); ni en gráficas; ni en el botón de peligro.
- No usar el acento como fondo de botón o bloque.
- No sombras, degradados, píldoras, fondos decorativos, ilustraciones de relleno ni tamaños de texto mayores a 40 px.
- No mostrar datos del cliente final en ninguna pantalla, reporte o PDF (DA-03).
- No texto en `--text-disabled` fuera de estados deshabilitados.
- No agregar un modo claro a la aplicación: el tema claro es solo para PDF (§11).
- No mayúsculas decorativas, flechitas en botones ni "eyebrows" sobre títulos.

---

## 11. Tema de impresión (PDF)

**Decisión del dueño (26-oct-2026):** los PDF (de la carga y semanal) usan un tema **claro**. **La aplicación en pantalla se queda en tema oscuro; no existe modo claro en la app.** El tema de impresión es solo para la plantilla del PDF.

**Qué cambia y qué no:**
- Fondo blanco, texto oscuro. **Se mantienen** las mismas fuentes (`--font-sans`, `--font-mono`), radios (`--radius-*`), espaciado (`--space-*`), escala tipográfica y estructura de componentes.
- Se redefinen **los mismos tokens de color** con otros valores en un bloque `.theme-print`. La plantilla del PDF pone `class="theme-print"` en su elemento raíz (`<html>`) y no incluye ninguna regla propia de color; también puede activarse por `@media print` repitiendo el mismo bloque.
- Semáforo y tramos se oscurecen para contrastar contra blanco. Los **mismos patrones** (`pat-bucket-3/4/5`, §7.6) se usan en los tres tramos rojos, con tinta oscura.
- Los tokens derivados (`--bucket-1…3`, `--chart-1`, `--focus-ring-color`) se **redeclaran como alias** (`var(--data-green)`, etc.) dentro de `.theme-print`: un `var()` declarado en `:root` ya viene resuelto con los valores oscuros y no cambiaría al sobrescribir el token original en un descendiente. `--pattern-ink` se redeclara con hex propio. El bloque usa `:root.theme-print` para ganar a `:root` sin depender del orden de carga.
- Los componentes interactivos (botones, campos) no existen en el PDF; sus tokens no se redefinen (el foco solo se redeclara como alias para no heredar un valor resuelto). `--color-danger` sí, solo para una alerta de error del PDF (por ejemplo, "Error: 3 filas rechazadas"), con las mismas reglas de §3.6.

```css
:root.theme-print,
.theme-print {
  color-scheme: light;
  -webkit-print-color-adjust: exact;
  print-color-adjust: exact;

  /* Superficies */
  --surface-canvas: #ffffff;
  --surface-section: #f0f0f0;
  --surface-card: #f5f5f5;
  --surface-raised: #e0e0e0;

  /* Bordes */
  --border-subtle: #d4d4d4;
  --border-strong: #a3a3a3;
  --border-control: #767676;

  /* Texto */
  --text-primary: #0a0a0a;
  --text-secondary: #525252;
  --text-disabled: #8a8a8a;

  /* Acento (ajustado a fondo blanco) */
  --accent: #1f5fd6;
  --accent-bg: #e3ecff;

  /* Error */
  --color-danger: #b01589;
  --color-danger-bg: #fde6f6;

  /* Colores de datos */
  --data-green: #137a3a;
  --data-yellow: #8a6500;
  --data-red: #d93a3a;
  --data-green-bg: #dff3e6;
  --data-yellow-bg: #fff1c7;
  --data-red-bg: #fde3e3;
  --bucket-1: var(--data-green);
  --bucket-2: var(--data-yellow);
  --bucket-3: var(--data-red);
  --bucket-4: #c8102e;
  --bucket-5: #b0002a;

  /* Patrones: tinta oscura sobre el rojo (geometría igual que en pantalla) */
  --pattern-ink: #0a0a0a;

  /* Series de gráficas generales */
  --chart-1: var(--accent);
  --chart-2: #525252;
  --chart-3: #767676;
  --chart-grid: #d4d4d4;

  /* Foco (alias; no hay foco en el PDF) */
  --focus-ring-color: var(--accent);

  /* Fondo y texto base de la hoja */
  background: var(--surface-canvas);
  color: var(--text-primary);
}
```

### 11.1 Valores y contraste en impresión (calculado, WCAG 2.x, `python -I`)

Fondos claros: `#ffffff` (`--surface-canvas`), `#f5f5f5` (`--surface-card`), `#f0f0f0` (`--surface-section`) y `#e0e0e0` (`--surface-raised`). Todas las marcas de color de datos dan ≥ 3:1 también sobre `#e0e0e0` (mínimo 3.45). Umbrales: texto 4.5:1, marcas 3:1.

| Token | Hex | vs `#ffffff` | vs `#f5f5f5` | vs `#f0f0f0` | vs `#e0e0e0` | Texto | Marca |
|---|---|---|---|---|---|---|---|
| `--text-primary` | `#0a0a0a` | 19.80 | 18.16 | 17.37 | 15.00 | Pasa | Pasa |
| `--text-secondary` | `#525252` | 7.81 | 7.17 | 6.86 | 5.92 | Pasa | Pasa |
| `--accent` | `#1f5fd6` | 5.73 | 5.25 | 5.03 | 4.34 | Pasa | Pasa |
| `--data-green` / `--bucket-1` | `#137a3a` | 5.42 | 4.97 | 4.76 | 4.11 | No se usa como texto | Pasa |
| `--data-yellow` / `--bucket-2` | `#8a6500` | 5.33 | 4.89 | 4.67 | 4.04 | No se usa como texto | Pasa |
| `--data-red` / `--bucket-3` | `#d93a3a` | 4.55 | 4.17 | 3.99 | 3.45 | No se usa como texto | Pasa |
| `--bucket-4` | `#c8102e` | 5.88 | 5.40 | 5.16 | 4.46 | No se usa como texto | Pasa |
| `--bucket-5` | `#b0002a` | 7.29 | 6.69 | 6.40 | 5.52 | No se usa como texto | Pasa |
| `--color-danger` | `#b01589` | 6.34 | 5.81 | 5.56 | 4.80 | Pasa | Pasa |
| `--border-control` / `--chart-3` | `#767676` | 4.54 | 4.17 | 3.99 | 3.44 | Pasa en blanco | Pasa |
| `--text-disabled` | `#8a8a8a` | 3.45 | 3.17 | 3.03 | 2.62 | Exento (solo deshabilitado) | Exento (no es marca) |

Reglas derivadas: en el PDF los colores de datos (`--data-*`, `--bucket-*`) **nunca se usan como texto**: solo relleno, borde y punto (umbral 3:1). Todo texto del PDF va en `--text-primary` o `--text-secondary`. Los grises `--border-subtle` (`#d4d4d4`, 1.48 sobre blanco) y `--border-strong` (`#a3a3a3`, 2.52) son separadores, no marcas. En el PDF, el amarillo es un ocre oscuro (`#8a6500`): es el costo de cumplir 3:1 sobre blanco; se identifica por la etiqueta de texto igual que en pantalla.

**Diferencia de tono del error en impresión:** `--color-danger` `#b01589` tiene hue 315.1°; contra `--bucket-3` (0.0°) 44.9°, contra `--bucket-4` (350.2°) 35.1° y contra `--bucket-5` (345.7°) 30.6°. Todas ≥ 30°.

### 11.2 Insignias del semáforo en el PDF

Texto **oscuro** (`--text-primary`) sobre fondo tenue **claro** sólido, borde `--border-width` del color pleno y punto del color pleno. Misma estructura que en pantalla (§7.3), solo cambian los valores de los tokens.

| Insignia | Fondo tenue | Texto `--text-primary` (`#0a0a0a`) | `--text-secondary` | Punto/borde vs fondo tenue |
|---|---|---|---|---|
| Verde | `--data-green-bg` `#dff3e6` | 17.06:1 | 6.73:1 | `#137a3a`: 4.67:1 |
| Amarillo | `--data-yellow-bg` `#fff1c7` | 17.59:1 | 6.94:1 | `#8a6500`: 4.73:1 |
| Rojo | `--data-red-bg` `#fde3e3` | 16.28:1 | 6.43:1 | `#d93a3a`: 3.74:1 |
| Acento | `--accent-bg` `#e3ecff` | 16.69:1 | 6.59:1 | `#1f5fd6`: 4.83:1 |
| Error | `--color-danger-bg` `#fde6f6` | 16.81:1 | 6.63:1 | `#b01589`: 5.38:1 |

Todos los textos ≥ 4.5:1 y todos los puntos/bordes ≥ 3:1. Los fondos tenues contra blanco son 1.13–1.22:1 (son tintes; la insignia se delimita con borde y punto).

### 11.3 Patrones en impresión

Los patrones de §7.6 se usan tal cual (misma geometría), con `--pattern-ink: #0a0a0a`. La tinta es oscura también en impresión, así que **solo oscurece** la marca respecto del fondo blanco y su contraste con él aumenta. Verificación (mezcla lineal según cobertura):

| Tramo + patrón | Cobertura | Relleno solo vs `#ffffff` | Medio con patrón vs `#ffffff` | vs `#f5f5f5` | vs `#f0f0f0` | Tinta vs relleno |
|---|---|---|---|---|---|---|
| `--bucket-3` + `pat-bucket-3` | 9.4 % | 4.55 | **4.90** | 4.50 | 4.30 | 4.35 |
| `--bucket-4` + `pat-bucket-4` | 12.6 % | 5.88 | **6.45** | 5.92 | 5.66 | 3.37 |
| `--bucket-5` + `pat-bucket-5` | 17.9 % | 7.29 | **8.22** | 7.54 | 7.21 | 2.71 |

Todos ≥ 3:1. Límite honesto: en el tramo >30 d la tinta se distingue del relleno con solo 2.71:1; el patrón sigue siendo visible pero sutil. Por eso la leyenda y las etiquetas con el número son obligatorias también en el PDF.

### 11.4 Contenido del PDF

El resumen del semáforo es una fila de cifras (en proceso, verde, amarillo, rojo, completadas) con insignias de texto (§11.2), comparación con la semana anterior, una barra horizontal por color y una línea de lectura. Las demás gráficas usan `--chart-1…3`. Sin datos del cliente final (DA-03). El PDF debe incluir la leyenda con los mismos patrones.
