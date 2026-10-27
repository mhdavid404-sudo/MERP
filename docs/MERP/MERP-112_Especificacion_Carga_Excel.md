# MERP-112 — Especificación de la carga de Excel
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.
**Supera a:** MERP-108 (histórico).

> Fuente única para la carga de órdenes. La Parte A especifica lo que la versión nueva debe hacer, con criterios de aceptación; la Parte B describe el comportamiento exacto del prototipo, incluidos los casos que acepta sin avisar. Pruebas citadas: rutas relativas a `backend/tests/`.

---

# Parte A — Especificación

> **Alcance de este documento tras RF-018.** La versión nueva no lee un formato fijo: aplica el **perfil de importación** del dealer que sube (MERP-102, RN-021 a RN-029; MERP-104). Lo que sigue describe el archivo del **primer distribuidor**, y sus secciones 2 a 4 son, en la práctica, el contenido de su perfil: columnas, mapeo, formatos de fecha y equivalencias. Las secciones 5 a 8 (validación de filas, creación y actualización, resumen, pantalla) valen para cualquier perfil.

## 1. Qué es la carga

El distribuidor exporta de su sistema de taller un reporte de órdenes en Excel y lo sube a MERP. Cada fila es una orden de taller. MERP crea las órdenes que no conocía y actualiza las que ya tenía: cada archivo es una **foto** del estado de las órdenes en ese momento.

| Aspecto | Definición |
|---|---|
| Quién sube | Solo el rol Dealer (RN-015) |
| A qué dealer van las órdenes | Al del usuario que sube; el archivo no lo indica (RN-013) |
| Frecuencia | Miércoles y viernes por acuerdo operativo. El sistema acepta cargas cualquier día y cuantas veces se quiera [DA-17] |
| Procesamiento | Inmediato, en la misma petición (MERP-103, ADR-005) |
| Atomicidad | O se guarda la carga completa con sus órdenes válidas, o nada (RN-017) |

## 2. Formato del archivo

| Aspecto | Requisito |
|---|---|
| Tipo | `.xlsx` |
| Tamaño máximo | 5 MB (el archivo de ejemplo, de 62 órdenes, pesa unos 25 KB) |
| Hoja | Se lee la primera hoja |
| Encabezado | Una fila con los nombres de columna. Puede haber filas de título encima |
| Filas | Una orden por fila. Las filas totalmente vacías se ignoran |
| Columnas obligatorias | `ORDEN`, `SERIE`, `ALTA` |

El archivo real tiene **40 columnas**, en este orden:

`TIPO`, `ORDEN`, `P&D`, `CLIENTE`, `ALTA`, `CIERRE`, `FECHA FAC`, `FACTURA`, `CVE`, `ASESOR`, `MOD`, `SERIE`, `KIL`, `UNIDAD`, `HRAS`, `CTO REF`, `REEQ`, `CTO TF`, `M.OBRA`, `REF`, `HP`, `TTF`, `OTR`, `IVA`, `TOTAL`, `PLACA`, `COLOR`, `TORRE`, `PK`, `TDS1`, `THYP`, `DIAA`, `DIAC`, `DIAF`, `PROMESA`, `HRA.PROM`, `TELS`, `CONTACTO`, `EMAIL`, `TIPO SERV`.

Comprobado en la copia anonimizada del archivo real (`ordenes_ejemplo.xlsx`, hoja `Hoja1`, 62 filas).

## 3. Mapeo de columnas

### 3.1 Columnas con campo propio (18)

| Columna | Dato de la orden | Obligatoria | Largo máximo | Notas |
|---|---|---|---|---|
| `ORDEN` | Número de orden | Sí | 50 | Identifica la orden dentro del dealer (RN-016) |
| `SERIE` | VIN | Sí | 50 | [DA-10] |
| `ALTA` | Fecha de entrada | Sí | — | Inicio del semáforo |
| `CIERRE` | Fecha de salida | No | — | Vacía = en proceso. Es el fin del trabajo en taller, no la fecha de factura |
| `PROMESA` + `HRA.PROM` | Fecha de promesa | No | — | Sin hora = 23:59:59 (RN-019) |
| `UNIDAD` | Modelo | No | 100 | Ejemplos: MINIVAN, CAP 7.5, CAPTAIN |
| `MOD` | Año | No | — | Entero |
| `PLACA` | Placas | No | 20 | |
| `ASESOR` | Asesor | No | 150 | Texto libre |
| `TIPO SERV` | Tipo de servicio | No | 50 | Texto libre. En el ejemplo, siempre `paquete` |
| `TOTAL` | Total | No | — | Importe |
| `M.OBRA` | Mano de obra | No | — | Importe |
| `REF` | Refacciones | No | — | Importe |
| `CLIENTE`, `CONTACTO`, `TELS`, `EMAIL` | Datos del cliente final | No | 200, 150, 100, 150 | **Si se guardan está abierto [DA-03]** |

### 3.2 Columna de clasificación

| Columna | Uso |
|---|---|
| `TDS1` | Determina el tipo de ingreso (RN-018). `mant` → mantenimiento. Cualquier otro valor, o vacío → sin clasificar |
| `THYP` | No se usa para clasificar. Vale `0` en las 62 filas del ejemplo |

Los valores originales de ambas se conservan con cada orden. En el ejemplo: `mant` 54 filas, `diag` 7, `repme` 1. **Pendiente con Magma [DA-02]:** qué significan `diag`, `repme` y `THYP`.

### 3.3 Resto de columnas (20)

`TIPO`, `P&D`, `FECHA FAC`, `FACTURA`, `CVE`, `KIL`, `HRAS`, `CTO REF`, `REEQ`, `CTO TF`, `HP`, `TTF`, `OTR`, `IVA`, `COLOR`, `TORRE`, `PK`, `DIAA`, `DIAC`, `DIAF`. Ningún cálculo las usa. **Si se guardan como respaldo está abierto [DA-03].**

## 4. Lectura tolerante

El archivo sale de otro sistema y no se controla su formato. El lector debe aceptar las variantes conocidas.

| # | Requisito | Evidencia en el prototipo |
|---|---|---|
| L-1 | Los encabezados se reconocen sin distinguir mayúsculas y con espacios sobrantes | `test_encuentra_encabezado_debajo_de_un_titulo_y_sin_importar_mayusculas` |
| L-2 | El encabezado puede no estar en la primera fila | Ídem |
| L-3 | Las filas vacías se ignoran y no alteran el número de fila que se informa | `test_ignora_filas_vacias_y_conserva_numero_de_fila_real` |
| L-4 | Una fecha puede venir como fecha de Excel o como texto `dd/mm/aaaa` con o sin hora | `test_fechas_como_texto_dd_mm_aaaa` |
| L-5 | La hora de promesa puede venir como hora, como texto o como fracción de día | `test_promesa_con_hora_como_texto`, `test_hora_de_promesa_como_fraccion_de_dia` |
| L-6 | Los textos se guardan sin espacios al inicio ni al final | `test_valores_numericos_y_de_texto_raros` |
| L-7 | Un número entero que Excel entrega con decimales (2025.0) se lee como entero | Ídem |
| L-8 | El número de fila informado es el que el usuario ve en Excel | `test_ignora_filas_vacias_y_conserva_numero_de_fila_real` |

Todas en `unit/infrastructure/test_xlsx_order_reader.py`, junto con `test_lee_el_excel_de_ejemplo_completo` y `test_mapea_columnas_a_campos`.

## 5. Validación

### 5.1 Del archivo completo

El archivo se rechaza entero, con un mensaje legible, y **no se registra ninguna carga**, cuando:

| Caso | Evidencia |
|---|---|
| No es `.xlsx` | `test_archivo_invalido_no_registra_carga` |
| Está vacío | — |
| Excede el tamaño máximo | `test_archivo_demasiado_grande` |
| No se puede abrir como Excel | `test_archivo_que_no_es_excel_es_invalido` |
| Le falta alguna columna obligatoria | `test_archivo_sin_columnas_obligatorias`, `test_archivo_sin_columnas_obligatorias_es_invalido` |
| Le falta **cualquier** columna que el perfil del dealer espera, obligatoria u opcional (RN-028) | No construido |
| El dealer no tiene perfil de importación (RN-021) | No construido |

### 5.2 De cada fila

La fila se rechaza, con su número y su motivo, y las demás se procesan (RN-017):

| Caso | Evidencia |
|---|---|
| Falta `ORDEN`, `SERIE` o `ALTA` | `test_rechaza_fila_sin_columnas_obligatorias` |
| `ALTA` o `CIERRE` no es una fecha | `test_rechaza_fila_si_alta_o_cierre_no_son_fecha` |
| `CIERRE` anterior a `ALTA` (RN-010) | `test_rechaza_cierre_anterior_a_alta_rn_010` |
| Un texto excede su largo máximo | `test_rechaza_texto_mas_largo_que_la_columna` |
| El número de orden ya apareció antes en el archivo; se conserva la primera | `test_orden_repetida_en_el_archivo_se_toma_la_primera` |

### 5.3 Datos opcionales que no se pueden interpretar · [DA-09]

La versión nueva **no debe perder un dato en silencio** (RF-017). Para cada caso hay que decidir si rechaza la fila o si la acepta con un aviso visible en el resumen:

| Caso | Prototipo |
|---|---|
| Importe que no es un número | Lo guarda vacío, sin aviso |
| Fecha u hora de promesa ilegible | La guarda vacía, sin aviso |
| Año que no es un entero | Lo guarda vacío, sin aviso |
| Valor de `TDS1` desconocido | Sin clasificar, sin aviso |
| Promesa anterior a la fecha de alta | La acepta |
| Número de orden `0` u otro valor sin sentido | Lo acepta como orden |

## 6. Creación y actualización

| # | Regla | Criterio de aceptación | Evidencia |
|---|---|---|---|
| U-1 | La orden se busca por (dealer, número de orden) | La misma orden en dos cargas del mismo dealer es una sola | `test_subir_el_mismo_archivo_dos_veces_no_duplica` |
| U-2 | Si no existe, se crea | La primera carga crea todas las órdenes válidas | `test_primera_carga_crea_todas_las_ordenes` |
| U-3 | Si existe y algún dato cambió, se actualiza con lo que trae el archivo | Una orden que el miércoles estaba en proceso y el viernes trae cierre queda completada | `test_carga_del_viernes_cierra_la_orden_que_el_miercoles_estaba_en_proceso` |
| U-4 | La carga más reciente manda, incluso si quita un dato | Una carga que quita el cierre devuelve la orden a en proceso | `test_la_ultima_carga_manda_aunque_quite_un_cierre` |
| U-5 | Si existe y llega idéntica, es "sin cambios" | El mismo archivo dos veces: todo "sin cambios" la segunda vez | `test_subir_el_mismo_archivo_dos_veces_no_duplica` |
| U-6 | Las órdenes que no vienen en la carga no se tocan [DA-08] | Una orden ausente conserva todos sus datos | `test_ordenes_que_no_vienen_en_la_carga_no_se_tocan` |
| U-7 | El mismo número en otro dealer es otra orden | Dos dealers con el mismo número no se pisan | `test_misma_orden_en_otro_dealer_es_otra_orden` |
| U-8 | Cada orden creada o actualizada apunta a esta carga | — | `test_orden_apunta_a_su_ultima_carga` |

**Abierto [DA-10]:** si una actualización puede cambiar el VIN de una orden existente y si el VIN se normaliza a mayúsculas.

## 7. Resumen de la carga

Cada carga registra y devuelve:

| Dato | Definición |
|---|---|
| Filas totales | Filas no vacías del archivo |
| Nuevas | Órdenes creadas |
| Actualizadas | Órdenes existentes con algún cambio |
| Sin cambios | Órdenes existentes que llegaron idénticas. **Se calcula**: totales − nuevas − actualizadas − rechazadas |
| Rechazadas | Filas rechazadas |
| Detalle | Por cada fila rechazada: número de fila y motivo, ordenadas por fila |
| Avisos | Solo si se adopta RF-017 [DA-09] |
| Versión del perfil | La versión con la que se leyó el archivo (RN-026). No construido |
| Cambio de perfil | Si la versión es distinta de la de la carga anterior del dealer (RN-029). No construido |

**Criterios de aceptación**
1. Filas totales = nuevas + actualizadas + sin cambios + rechazadas, siempre.
2. Una carga sin ninguna fila válida queda registrada, con todas sus filas rechazadas.
3. El detalle usa el número de fila que el usuario ve en Excel.

**Evidencia:** `test_dealer_sube_el_excel_de_ejemplo`, `test_carga_sin_filas_validas_igual_queda_registrada`, `test_rechazos_del_lector_se_suman_y_errores_quedan_ordenados_por_fila`, `test_upload_cuenta_rechazadas_a_partir_de_errores`.

## 8. Pantalla de cargas

- El Dealer arrastra el archivo o lo elige. Antes de enviarlo, la pantalla comprueba tipo y tamaño para avisar pronto; la validación que cuenta es la del servidor.
- Tras subir, muestra el resumen con los cuatro contadores y, si hay rechazos, la tabla de fila y motivo, con la indicación de corregirlos en el sistema de origen y volver a subir.
- Debajo, el historial (RF-011).
- El Admin ve solo el historial, con el nombre del dealer.

## 8 bis. Segundo distribuidor: formato conocido

Según lo indicado por el dueño del proyecto el 8 de octubre de 2026. **[VERIFICAR]** el archivo no se ha revisado para esta serie.

| Aspecto | Valor |
|---|---|
| Tipo | `.xls`, reporte de Crystal Reports |
| Fila de encabezados | 7 |
| Columnas | `Orden`, `Serie`, `FechaOrd`, `FechaFsct`, `Status`, `Mano Obra`, `Refacciones`, `TOTAL`, `Días` |
| Sin columna de | Promesa |

| Columna | Campo canónico probable | Situación |
|---|---|---|
| `Orden` | Número de orden | Por confirmar con el archivo |
| `Serie` | VIN | Por confirmar |
| `FechaOrd` | Fecha de entrada | Por confirmar |
| `FechaFsct` | **Desconocido** | **[DA-22]** Si es una fecha de facturación, no debe usarse como fecha de salida |
| `Status` | **Desconocido** | **[DA-22]** Posible equivalencia a abierta o cerrada; ver MERP-104, estado reportado |
| `Mano Obra`, `Refacciones`, `TOTAL` | Mano de obra, refacciones, total | Por confirmar |
| `Días` | Ninguno | MERP calcula los días; no se asigna |

Consecuencias ya visibles:
- Sin promesa, el cumplimiento de promesa será "no disponible" para este dealer (RN-023).
- Sin asesor ni tipo de servicio, esas secciones del Panel también.
- **Si ninguna columna resulta ser una fecha de salida fiable, sus órdenes no se completan** y casi todo el Panel queda "no disponible" (RN-022, advertencia).
- El prototipo solo lee `.xlsx`: aceptar `.xls` es una decisión [DA-19] y requiere otra forma de lectura.

## 9. Archivos de prueba

| Archivo | Contenido | Dónde |
|---|---|---|
| `ordenes_ejemplo.xlsx` | Copia **anonimizada** del archivo real, 62 órdenes | Versionado en el repositorio |
| Archivo real del distribuidor | Datos personales de clientes | **Nunca versionado** (RNF-006) |
| `ordenes_carga_normal.xlsx` | 8 órdenes variadas | Fuera del repositorio |
| `ordenes_casos_dificiles.xlsx` | 21 filas con casos límite | Fuera del repositorio |

La versión nueva debe incorporar al repositorio un archivo de casos límite sin datos reales, como prueba automatizada de los §4, §5 y §6.

---

# Parte B — Prototipo

## 10. Dónde está

| Pieza | Archivo |
|---|---|
| Ruta de subida e historial | `backend/app/api/routers/uploads.py` (`POST /api/v1/uploads`, `GET /api/v1/uploads`) |
| Lector del archivo | `backend/app/infrastructure/excel/xlsx_order_reader.py` (openpyxl) |
| Proceso de la carga | `backend/app/application/use_cases/process_upload.py` |
| Anonimizador | `backend/scripts/anonimizar_excel.py` |
| Pantalla | `frontend/src/features/cargas/` |

## 11. Detalles de implementación

- El encabezado se busca en las **10 primeras filas**.
- Formatos de fecha en texto aceptados: `dd/mm/aaaa`, `dd/mm/aaaa hh:mm`, `dd/mm/aaaa hh:mm:ss` y los mismos con `aaaa-mm-dd`. También el número de serie de Excel.
- En los importes se quitan el signo `$` y las comas antes de convertir; se guardan con dos decimales.
- Las columnas sin campo propio, **incluidas `TDS1` y `THYP`**, se guardan en el campo `datos_crudos` de la orden, con las fechas convertidas a texto (`test_datos_crudos_guarda_columnas_sin_campo_propio_y_tds1_thyp`, `test_datos_crudos_convierte_fechas_a_texto`).
- Los cuatro datos del cliente final se guardan en campos propios y ninguna pantalla los usa.
- "Actualizada" significa que **cualquier** dato de la orden difiere, también una columna de `datos_crudos`.
- La tabla de cargas guarda el número de rechazadas; "sin cambios" no es una columna.
- La fecha de la carga es la hora del servidor en el momento del proceso.
- El archivo original **no se guarda**: solo las órdenes y el registro de la carga.
- **No hay perfiles de importación.** Nombres de columna, búsqueda del encabezado, formatos de fecha y la equivalencia de `TDS1` están fijos en el código; el archivo del segundo distribuidor no se podría cargar.

## 12. Comportamiento observado con el archivo de casos límite

Simulación del 8 de octubre de 2026 con `ordenes_casos_dificiles.xlsx` como Magma Toluca, sin guardar: 21 filas, **14 nuevas, 1 actualizada, 0 sin cambios, 6 rechazadas**.

| Caso | Resultado | Valoración |
|---|---|---|
| Sin `ORDEN`, sin `SERIE`, sin `ALTA` | Rechazadas, cada una con su motivo | Correcto |
| `CIERRE` anterior a `ALTA` | Rechazada | Correcto |
| Orden repetida en el archivo | Se toma la primera; la segunda se rechaza citando la fila de la primera | Correcto |
| `CIERRE` con texto que no es fecha | Rechazada | Correcto |
| `ORDEN` con espacios alrededor | Se carga sin los espacios | Correcto |
| `ALTA` como texto `31/08/2026 09:36` | Se interpreta | Correcto |
| Sin hora de promesa | Promesa a las 23:59:59 | Correcto (RN-019) |
| Cliente con Ñ, `&` y emoji | Texto intacto | Correcto |
| `ALTA` igual a `CIERRE` | 0 días | Correcto |
| `TDS1` con un valor desconocido | Sin clasificar, sin aviso | DA-09 |
| `TOTAL` con `N/A` | Total vacío, sin aviso | DA-09 |
| `SERIE` en minúsculas | Se guarda en minúsculas | DA-10 |
| Orden existente con otro VIN | El VIN se reemplaza | DA-10 |
| `ORDEN` con valor `0` | Se crea la orden "0" | DA-09 |

Con `ordenes_carga_normal.xlsx` (subido el 2 de octubre como dealer PRUEBA: 8 nuevas, 0 rechazadas) se observó además que cinco órdenes traen una promesa anterior a su fecha de alta y se aceptan (DA-09).

Estos resultados los registró la verificación del 8 de octubre sobre el prototipo en marcha; el archivo de casos límite **no se ha cargado** de forma definitiva.
