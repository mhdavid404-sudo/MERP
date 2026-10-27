# MERP-108 — Rediseño: ingesta de Excel (documento de cambio)
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** histórico

> **HISTÓRICO — superado por MERP-112.** Este documento registra la decisión de sustituir la captura manual por la carga de Excel. Se conserva sin cambios de contenido como registro. **No describe el sistema actual:** asume que el tipo de ingreso sale de `THYP` (sale de `TDS1`), habla de 39 columnas (son 40), permite subir archivos al rol Empleado (ya no existe) y no define la tabla de cargas. La especificación vigente de la carga es MERP-112; el modelo de datos, MERP-104 y MERP-107; el contrato de la API, MERP-106.

---

> **Nota (M5):** `trucks`/`work_records`, el enum `truck_estado` y los endpoints POST de captura manual se eliminaron en M5 (migración `a7c3e91d5b20`); los datos previos se descartaron.

**Estado:** Reemplaza las Fases 5 y 7 (módulos de captura manual: entrada/trabajo/salida) definidas en MERP-101, MERP-105 y MERP-106. El resto del diseño original (roles, dealers, semáforo) se mantiene.

---

## 1. Motivo del cambio

Al aprobarse el proyecto, se definió que el distribuidor ya tiene su propio sistema de taller del cual exporta un reporte en Excel. En vez de que el Empleado capture manualmente entrada/trabajo/salida en el sistema, el distribuidor **sube ese Excel vía drag & drop**, y el sistema lo procesa automáticamente.

**Frecuencia:** 2 cargas por semana (no es tiempo real continuo — es una foto actualizada dos veces por semana).

**Quién puede subir el archivo:** Dealer o Empleado (ambos roles tienen permiso).

## 2. Mapeo de columnas (Excel del distribuidor → sistema)

Basado en el archivo de ejemplo (`prueba.xlsx`, hoja única, 39 columnas, 1 fila por orden de taller):

| Columna Excel | Campo en el sistema | Notas |
|---|---|---|
| `ORDEN` | `orden_externa` | Identificador único de la orden — clave para detectar si es una orden nueva o una actualización de una ya existente |
| `SERIE` | `vin` | |
| `MOD` | `anio` | |
| `UNIDAD` | `modelo` | Nombre real del modelo (ej. "MINIVAN", "CAP 7.5", "CAPTAIN") |
| `ALTA` | `fecha_entrada` | |
| `CIERRE` | `fecha_salida` | **Define cuándo el camión se considera "salido"** (decisión confirmada — no se usa FECHA FAC) |
| `THYP` | `tipo_ingreso` | Mapea a `garantia` / `mantenimiento` |
| `PLACA` | `placas` | |
| `TIPO SERV` | `tipo_servicio` | Clasificación adicional del distribuidor (ej. "paquete") — se guarda, no afecta el semáforo |
| `ASESOR` | `asesor` | Para el panel de desempeño por asesor (MERP-109) |
| `PROMESA`, `HRA.PROM` | `fecha_promesa` | Para el cálculo de cumplimiento de promesa (MERP-109) |
| `TOTAL`, `M.OBRA`, `REF` | `total`, `mano_obra`, `refacciones` | Para el resumen de ingresos (MERP-109) |
| `CLIENTE`, `CONTACTO`, `TELS`, `EMAIL` | *(se guardan tal cual, sin usarse en dashboards de Magma)* | Datos del distribuidor sobre su propio cliente final |
| Resto de columnas (`KIL`, `HRAS`, `CTO REF`, `REEQ`, `CTO TF`, `HP`, `TTF`, `OTR`, `IVA`, `COLOR`, `TORRE`, `PK`, `TDS1`, `DIAA`, `DIAC`, `DIAF`, `TIPO`, `P&D`, `CVE`, `FECHA FAC`, `FACTURA`) | *(se guardan sin procesar, como respaldo)* | No se usan en ningún cálculo del prototipo; se conservan por si se necesitan después |

## 3. Cambios al modelo de datos (reemplaza parte de MERP-104)

La entidad `Truck` + `WorkRecord` se **fusionan conceptualmente en una sola entidad `Order`** (orden de taller), ya que el Excel no distingue "trabajo" como paso separado — solo trae fecha de alta y fecha de cierre.

### Order (reemplaza Truck + WorkRecord)
| Campo | Tipo | Origen |
|---|---|---|
| id | UUID | generado |
| dealer_id | UUID | FK — determinado por quién sube el archivo, no viene en el Excel |
| orden_externa | string | `ORDEN` — único por dealer (misma orden puede repetirse entre cargas) |
| vin | string | `SERIE` |
| anio | int | `MOD` |
| modelo | string | `UNIDAD` |
| placas | string | `PLACA` |
| tipo_ingreso | enum | `THYP` → garantia/mantenimiento |
| tipo_servicio | string | `TIPO SERV` |
| fecha_entrada | datetime | `ALTA` |
| fecha_salida | datetime \| null | `CIERRE` (null = sigue en proceso) |
| fecha_promesa | datetime \| null | `PROMESA` + `HRA.PROM` |
| asesor | string | `ASESOR` |
| total, mano_obra, refacciones | decimal | `TOTAL`, `M.OBRA`, `REF` |
| cliente, contacto, telefono, email | string | datos del cliente del distribuidor |
| datos_crudos | JSONB | el resto de columnas del Excel, sin procesar, por fila |
| estado | enum | en_proceso / completado — derivado de `fecha_salida` |

**Regla de actualización (upsert):** al procesar una carga, si `(dealer_id, orden_externa)` ya existe, se **actualiza** el registro (ej. una orden que en la carga del lunes no tenía `fecha_salida` y en la del jueves ya sí) — esto es lo que permite medir "cuánto se tardaron" entre cargas. Si no existe, se crea.

## 4. Cambios a la API (reemplaza parte de MERP-106)

Se eliminan: `POST /trucks`, `POST /trucks/{id}/work`, `POST /trucks/{id}/exit`.

Se agrega:

| Método | Ruta | Rol | Descripción |
|---|---|---|---|
| POST | /uploads | dealer, empleado | Sube el archivo Excel (multipart/form-data). El sistema lo procesa de inmediato (síncrono, dado el tamaño pequeño de archivo esperado) |
| GET | /uploads | admin, dealer, empleado | Historial de cargas (fecha, quién subió, cuántas órdenes nuevas/actualizadas, errores de fila si los hubo) |

`GET /trucks`, `GET /trucks/{id}` y `GET /dashboard` se mantienen igual, pero ahora leen de `Order` en vez de `Truck`+`WorkRecord`.

## 5. Semáforo (sin cambios de regla, cambia la fuente)

Sigue siendo RN-002/RN-003 de MERP-102 (0-4 verde, 5-7 amarillo, +7 rojo, congelado al salir) — el cálculo es exactamente igual, solo que ahora `fecha_entrada`/`fecha_salida` vienen del Excel (`ALTA`/`CIERRE`) en vez de ser capturadas a mano.

## 6. Validación de archivo

- Si una fila no trae `ORDEN`, `SERIE` o `ALTA`, se rechaza esa fila específica (no toda la carga) y se reporta en el resultado de `POST /uploads`.
- Si `CIERRE` < `ALTA` en una fila, se rechaza esa fila (viola RN-010).
- El resto de columnas son opcionales — su ausencia no rechaza la fila.
