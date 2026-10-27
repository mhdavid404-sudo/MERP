# MERP-104 — Modelo de Datos (Dominio)
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> Reemplaza a la versión 1.0, que describía las entidades Truck y WorkRecord de la captura manual. La Parte A define el modelo que la versión nueva debe tener; la Parte B, las entidades tal como existen en el código.

---

# Parte A — Especificación

## 1. Entidades

```
Dealer 1 ──── N Usuario        (un Admin no pertenece a ningún dealer)
Dealer 1 ──── N Orden
Dealer 1 ──── N Carga
Usuario 1 ─── N Carga          (quién la subió)
Carga  1 ──── N Fila rechazada
Carga  1 ──── N Orden          (última carga que creó o modificó la orden)
Dealer 1 ──── 1 Perfil de importación
Perfil 1 ──── N Versión de perfil
Versión de perfil 1 ── N Carga (versión con la que se leyó)
Usuario 1 ─── N Versión de perfil (quién la creó)
```

### Dealer
| Dato | Obligatorio | Notas |
|---|---|---|
| Identificador | Sí | Generado por el sistema |
| Nombre | Sí | No vacío. Unicidad abierta [DA-11] |
| Activo | Sí | Baja lógica (RN-008) |

### Usuario
| Dato | Obligatorio | Notas |
|---|---|---|
| Identificador | Sí | |
| Nombre | Sí | No vacío |
| Correo | Sí | Único sin distinguir mayúsculas (RN-020) |
| Contraseña | Sí | Solo su hash. El dominio no conoce el algoritmo |
| Rol | Sí | Admin o Dealer |
| Dealer | Según rol | Admin: nunca. Dealer: siempre |
| Activo | Sí | Baja lógica (RN-008) |

### Orden
| Dato | Obligatorio | Origen en el archivo | Notas |
|---|---|---|---|
| Identificador | Sí | — | Generado por el sistema |
| Dealer | Sí | — | Del usuario que sube (RN-013) |
| Número de orden | Sí | `ORDEN` | Único dentro del dealer (RN-016) |
| VIN | Sí | `SERIE` | No es único. Normalización y cambio entre cargas abiertos [DA-10] |
| Fecha de entrada | Sí | `ALTA` | |
| Fecha de salida | No | `CIERRE` | Vacía = en proceso. No anterior a la entrada (RN-010) |
| Fecha de promesa | No | `PROMESA` + `HRA.PROM` | Sin hora = fin del día (RN-019) |
| Modelo | No | `UNIDAD` | |
| Año | No | `MOD` | |
| Placas | No | `PLACA` | |
| Tipo de ingreso | No | `TDS1` | Garantía, mantenimiento o sin clasificar (RN-018) [DA-02] |
| Tipo de servicio | No | `TIPO SERV` | Texto libre |
| Asesor | No | `ASESOR` | Texto libre |
| Total, mano de obra, refacciones | No | `TOTAL`, `M.OBRA`, `REF` | Decimal exacto, dos decimales |
| Datos del cliente final | No | `CLIENTE`, `CONTACTO`, `TELS`, `EMAIL` | **Si se guardan o no está abierto [DA-03]** |
| Valores originales de clasificación | Sí | `TDS1`, `THYP` | Para poder reclasificar (RN-018) |
| Resto de columnas | — | Las demás | **Si se guardan o no está abierto [DA-03]** |
| Última carga | No | — | La carga que la creó o modificó por última vez |
| Estado | Derivado | — | En proceso o completada (RN-001). No se asigna |

### Carga
| Dato | Obligatorio | Notas |
|---|---|---|
| Identificador | Sí | |
| Dealer | Sí | |
| Usuario que la subió | Sí | |
| Nombre del archivo | Sí | |
| Fecha y hora | Sí | Momento del proceso |
| Filas totales | Sí | Filas no vacías del archivo |
| Nuevas, actualizadas | Sí | |
| Filas rechazadas | Sí | Lista de (número de fila, motivo) |
| Rechazadas | Derivado | Cantidad de filas rechazadas |
| Sin cambios | Derivado | Totales − nuevas − actualizadas − rechazadas |
| Avisos | Abierto | Solo si se adopta RF-017 [DA-09] |
| Versión del perfil | Sí | La versión del perfil de importación con la que se leyó el archivo (RN-026) |
| Cambio de perfil | Derivado | Verdadero si la versión es distinta de la de la carga anterior del mismo dealer (RN-029) |

### Perfil de importación
Describe cómo leer el archivo de un dealer. **Es un dato, no código** (RN-021). No se construye en el prototipo.

| Dato | Obligatorio | Notas |
|---|---|---|
| Identificador | Sí | |
| Dealer | Sí | Uno por dealer |
| Versión vigente | Sí | La última versión creada |

### Versión de perfil
Inmutable: una vez creada no se modifica ni se borra (RN-026).

| Dato | Obligatorio | Notas |
|---|---|---|
| Identificador | Sí | |
| Perfil | Sí | |
| Número de versión | Sí | Consecutivo dentro del perfil, desde 1 |
| Hoja | Sí | Nombre o posición de la hoja |
| Fila de encabezados | Sí | Número de fila tal como se ve en el archivo |
| Mapeo | Sí | Lista de parejas (columna del archivo → campo canónico). Debe cubrir los campos obligatorios (RN-022) |
| Formato de fechas | Sí | Cómo interpretar las fechas que llegan como texto: orden de día, mes y año |
| Equivalencias de valores | No | Por campo: lista de (valor en el archivo → valor canónico). Ejemplos: código de servicio → tipo de ingreso; estado del archivo → abierta o cerrada |
| Creada por | Sí | Usuario |
| Fecha de creación | Sí | |
| Qué cambió | Sí, desde la versión 2 | Diferencias respecto a la versión anterior (RN-027) |
| Origen | Sí | Configurada por el dealer, plantilla de Magma, o reversión [DA-20] |

### Campos canónicos
Son los campos de la orden a los que un perfil puede asignar una columna. El sistema trabaja siempre con ellos, venga el archivo como venga.

| Campo canónico | Tipo | Contrato | Lo que depende de él |
|---|---|---|---|
| Número de orden | Texto | **Obligatorio** | Identidad de la orden |
| VIN | Texto | **Obligatorio** | — |
| Fecha de entrada | Fecha y hora | **Obligatorio** | Semáforo |
| Fecha de salida | Fecha y hora | Opcional | Estado, semáforo congelado, todo lo calculado sobre completadas |
| Fecha de promesa | Fecha | Opcional | Cumplimiento de promesa |
| Hora de promesa | Hora | Opcional | Precisión de la promesa |
| Asesor | Texto | Opcional | Desempeño por asesor |
| Total | Importe | Opcional | Ingresos |
| Mano de obra | Importe | Opcional | Desglose de ingresos |
| Refacciones | Importe | Opcional | Desglose de ingresos |
| Tipo de servicio | Texto | Opcional | Mezcla por tipo de servicio |
| Tipo de ingreso | Valor con equivalencias | Opcional | Mezcla por tipo de ingreso |
| Modelo, año, placas | Texto, entero, texto | Opcional | Solo informativos |
| Datos del cliente final | Texto | Opcional | Ninguna métrica. **Si existen como campos está abierto [DA-03]** |

La columna "Contrato" refleja el contrato mínimo propuesto; **la lista definitiva la define Magma [DA-18]**.

**Estado reportado por el distribuidor.** Algunos archivos traen una columna de estado (abierta o cerrada). En MERP el estado **se deriva de la fecha de salida** (RN-001), así que una columna de estado no puede asignarse sin más: una orden "cerrada" sin fecha de salida no se podría representar. Cómo se usa esa columna depende de qué significan los campos del archivo del segundo distribuidor **[DA-22]**.

## 2. Valores derivados

Nunca se almacenan como dato de origen; se calculan:

| Valor | Regla |
|---|---|
| Estado de la orden | RN-001 |
| Días transcurridos y color del semáforo | RN-002, RN-003 |
| Rechazadas y sin cambios de una carga | Definiciones de arriba |
| Métricas del Panel | MERP-109 |

## 3. Invariantes

Cada invariante debe cumplirse en el dominio **y** en la base de datos.

| # | Invariante | Criterio de aceptación |
|---|---|---|
| I-1 | Un Admin no tiene dealer; un Dealer siempre tiene uno. | No se puede crear ni guardar un usuario que lo incumpla |
| I-2 | Número de orden y VIN no vacíos. | Una orden sin ellos no se puede construir |
| I-3 | Fecha de salida ≥ fecha de entrada. | RN-010 |
| I-4 | Estado coherente con la fecha de salida. | RN-001, criterio 3 |
| I-5 | (Dealer, número de orden) único. | RN-016, criterio 4 |
| I-6 | Correo único sin distinguir mayúsculas. | RN-020, criterio 1 |
| I-7 | Nombre de dealer y de usuario no vacíos. | No se puede crear con nombre en blanco |
| I-8 | Filas totales = nuevas + actualizadas + sin cambios + rechazadas. | RF-010, criterio 6 |
| I-9 | Un dealer tiene como máximo un perfil de importación. | No se puede crear un segundo perfil para el mismo dealer |
| I-10 | Toda versión de perfil asigna los campos obligatorios, y no repite campo ni columna. | RN-022 |
| I-11 | Las versiones de un perfil son inmutables y su numeración es consecutiva y única. | RN-026, criterios 1 y 4 |
| I-12 | Toda carga está ligada a una versión de perfil del mismo dealer. | No se puede guardar una carga sin versión, ni con la versión de otro dealer |

**Evidencia en el prototipo:** `unit/domain/test_order.py` (I-2, I-3, I-4, I-8: `test_rechaza_orden_sin_campos_obligatorios`, `test_rechaza_cierre_anterior_a_alta_rn_010`, `test_campos_opcionales_pueden_venir_vacios`, `test_upload_cuenta_rechazadas_a_partir_de_errores`); `integration/test_order_repository.py` (I-3, I-4, I-5); `integration/test_users_dealers_endpoint.py::test_crear_admin_con_nombre_en_blanco_devuelve_422` (I-7). I-6 se cumple en la aplicación pero **no en la base** del prototipo.

## 4. Requisitos del modelo que el prototipo no cumple

| Requisito | Origen |
|---|---|
| Fechas de creación y de última modificación en cada entidad | RNF-019 |
| Unicidad del correo sin distinguir mayúsculas, impuesta por la base | RN-020 |
| Posibilidad de dar de baja a un usuario | RF-015 |
| Perfil de importación, versiones y su vínculo con cada carga | RF-018, RN-021 a RN-029 |

---

# Parte B — Prototipo

Nombres tal como están en `backend/app/domain/`.

| Entidad | Campos |
|---|---|
| `Dealer` | `id`, `nombre`, `activo` |
| `User` | `id`, `nombre`, `email`, `password_hash`, `rol`, `dealer_id`, `activo` |
| `Order` | `id`, `dealer_id`, `orden_externa`, `vin`, `fecha_entrada`, `fecha_salida`, `fecha_promesa`, `modelo`, `anio`, `placas`, `tipo_ingreso`, `tipo_servicio`, `asesor`, `total`, `mano_obra`, `refacciones`, `cliente`, `contacto`, `telefono`, `email`, `datos_crudos`, `ultima_carga_id`; `estado` es una propiedad |
| `Upload` | `id`, `dealer_id`, `user_id`, `nombre_archivo`, `fecha_carga`, `filas_total`, `nuevas`, `actualizadas`, `errores`; `rechazadas` y `sin_cambios` son propiedades |
| `FilaRechazada` | `fila`, `motivo` |

| Valor | Opciones |
|---|---|
| `UserRole` | `admin`, `dealer` |
| `TipoIngreso` | `garantia`, `mantenimiento` (sin clasificar = vacío) |
| `OrderEstado` | `en_proceso`, `completado` |
| `Semaforo` | `verde`, `amarillo`, `rojo` |
| `PeriodoPanel` | `semana` (7 días), `mes` (30 días) |

Notas del prototipo:
- `datos_crudos` guarda **todas** las columnas del archivo que no tienen campo propio, incluidas `TDS1` y `THYP`, como texto estructurado.
- Los cuatro datos del cliente final se guardan y ninguna pantalla ni respuesta de la API los usa.
- Las invariantes I-1, I-2, I-3 e I-7 se validan al construir la entidad.
- No existen las entidades `Truck` y `WorkRecord` de la versión 1; se eliminaron junto con sus tablas.
- **No existe el perfil de importación.** Lo que el perfil contendría para el primer distribuidor está repartido en el código del lector: los nombres de columna y su campo, los formatos de fecha aceptados y la equivalencia `mant` → mantenimiento.
- Los usuarios que tenían el rol Empleado quedaron como `dealer` con `activo=false`.
