# MERP-109 — Panel del Distribuidor
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 3.0 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51` (el Panel se construyó en M7).

> La Parte A especifica el Panel que la versión nueva debe ofrecer, con criterios de aceptación. Todo lo que especifica está **construido** en el prototipo y cubierto por pruebas (rutas relativas a `backend/tests/`). La Parte B describe cómo está hecho.

---

# Parte A — Especificación

## 1. Propósito

El Panel es el valor que el distribuidor recibe a cambio de subir su archivo: métricas de su propia operación, calculadas con datos del archivo que el semáforo no usa (promesa, asesor, importes, tipo de servicio).

## 2. Quién lo ve

| Rol | Alcance |
|---|---|
| Dealer | Solo el de su dealer |
| Admin | El de cualquier dealer, incluidos los dados de baja, que se muestran marcados |

**Criterios de aceptación**
1. Un Dealer sin indicar dealer obtiene el suyo; indicando el suyo, también.
2. Un Dealer que indica otro dealer recibe un rechazo por permisos, con el mismo mensaje exista o no ese dealer y sin que se consulten sus datos.
3. Un Admin debe indicar el dealer; sin él, se le pide que lo elija.
4. Un Admin obtiene el Panel de un dealer dado de baja, marcado como inactivo.
5. Un Admin que indica un dealer inexistente recibe "no encontrado".

**Evidencia:** `unit/application/test_get_panel_distribuidor.py` (`test_dealer_sin_param_usa_el_suyo`, `test_dealer_con_param_propio_ok`, `test_dealer_con_param_ajeno_403_sin_consultar_nada`, `test_dealer_con_param_ajeno_inexistente_da_el_mismo_error`, `test_admin_sin_param_validation_error`, `test_admin_con_dealer_dado_de_baja_ok_y_marcado_inactivo`, `test_admin_con_dealer_inexistente_not_found`); `integration/test_panel_endpoint.py`.

## 3. Periodo

Un único selector aplica a las cuatro secciones.

| Opción | Ventana |
|---|---|
| Semana (por defecto) | Los últimos 7 días hasta el momento de la consulta |
| Mes | Los últimos 30 días hasta el momento de la consulta |

Es una ventana móvil, no de calendario. Ambos extremos se incluyen. "Ahora" es la hora local de planta.

**Criterios de aceptación**
1. La ventana de semana mide exactamente 7 días y la de mes 30, terminando en el instante de la consulta.
2. Una orden cerrada exactamente en el límite inicial o final cuenta; un segundo fuera, no.
3. Sin indicar periodo se usa semana.
4. Un periodo distinto de los dos se rechaza.

**Evidencia:** `unit/domain/test_panel_distribuidor.py` (`test_ventana_semana_y_mes_son_moviles_exactas`, `test_limites_de_la_ventana_inclusivos_y_un_segundo_fuera`); `integration/test_panel_endpoint.py` (`test_periodo_por_defecto_es_semana`, `test_periodo_mes_incluye_la_orden_de_hace_20_dias`, `test_periodo_invalido_422`); `test_el_reloj_inyectado_fija_la_ventana_sin_microsegundos`.

## 4. Qué órdenes cuenta cada sección

| Conjunto | Definición | Lo usan |
|---|---|---|
| **Completadas en la ventana** | Órdenes con fecha de salida dentro de la ventana | Cumplimiento, ingresos, y completadas y tiempo por asesor |
| **Activas en la ventana** | Órdenes que entraron a más tardar al final de la ventana y que siguen en proceso o salieron después de su inicio. Incluye a las completadas | Volumen por asesor y las dos mezclas |

**Criterios de aceptación**
1. Suma del volumen por asesor = suma de la mezcla por tipo de ingreso = suma de la mezcla por tipo de servicio = órdenes activas en la ventana.
2. Una orden con fecha de entrada posterior a "ahora" no cuenta en nada.
3. Una orden con salida posterior a "ahora" cuenta como activa y no como completada.

**Evidencia:** `test_invariante_volumen_ingreso_servicio_igual_a_activas`, `test_fecha_entrada_futura_queda_fuera_de_todo`, `test_salida_futura_esta_en_activas_pero_no_en_completadas`.

## 5. Las cuatro secciones

### 5.1 Cumplimiento de promesa

Porcentaje a tiempo = órdenes con salida en la fecha prometida o antes ÷ órdenes completadas en la ventana que tienen promesa. Un decimal.

**Criterios de aceptación**
1. Salida igual a la promesa cuenta como a tiempo.
2. Salida posterior a la promesa no cuenta.
3. Las completadas sin promesa se excluyen del porcentaje y se informan aparte.
4. Las órdenes en proceso no cuentan, aunque su promesa haya vencido.
5. Sin órdenes evaluables no hay porcentaje (no se muestra 0 %).

**Evidencia:** `test_salida_igual_a_promesa_cuenta_a_tiempo`, `test_salida_tarde_no_cuenta_y_porcentaje_con_un_decimal`, `test_promesa_nula_se_excluye_y_se_cuenta_aparte`, `test_orden_en_proceso_no_cuenta_aunque_su_promesa_haya_vencido`, `test_cero_evaluables_porcentaje_none`.

### 5.2 Desempeño por asesor

Por cada asesor: volumen (órdenes activas), completadas (en la ventana) y tiempo promedio de cierre en días con un decimal, calculado con la duración exacta (salida − entrada) en horas entre 24.

**Criterios de aceptación**
1. Asesor vacío, nulo o solo espacios forman un único grupo "Sin asesor".
2. Los espacios sobrantes del nombre no crean asesores distintos.
3. Un asesor con volumen pero sin órdenes completadas no tiene tiempo promedio.
4. El tiempo se redondea a un decimal, la mitad hacia arriba.
5. Orden: volumen descendente, después nombre; "Sin asesor" al final.

**Evidencia:** `test_asesor_nulo_vacio_y_espacios_se_agrupan_en_un_bucket`, `test_asesor_se_normaliza_con_strip`, `test_asesor_con_volumen_pero_sin_completadas_tiempo_none`, `test_tiempo_promedio_en_horas_exactas`, `test_tiempo_promedio_de_varias_ordenes`, `test_tiempo_promedio_redondea_half_up`, `test_orden_de_asesores_volumen_desc_nombre_y_none_al_final_sin_typeerror`, `test_volumen_cuenta_en_proceso_y_completadas_solo_las_cerradas_en_ventana`.

**Nota.** Este tiempo usa la duración exacta. Si el semáforo se define por días de calendario [DA-01], las dos cifras seguirán sin ser comparables y la pantalla debe decirlo.

### 5.3 Ingresos facturados

Sobre las órdenes completadas en la ventana: total facturado, monto de mano de obra, monto de refacciones y su desglose porcentual.

**Criterios de aceptación**
1. Solo suman las órdenes completadas en la ventana.
2. Una orden sin total no suma y se informa como "sin importe".
3. Los porcentajes se calculan sobre (mano de obra + refacciones), no sobre el total, y suman exactamente 100.
4. El total se muestra aparte; la pantalla aclara que incluye otros conceptos.
5. Si mano de obra + refacciones no es positivo, o alguno es negativo, no hay porcentajes pero los montos se muestran.
6. Sin órdenes, los importes son cero con dos decimales.

**Evidencia:** `test_ingresos_solo_de_completadas_en_ventana`, `test_total_nulo_cuenta_como_sin_importe_y_suma_cero`, `test_desglose_sobre_mo_mas_ref_y_total_aparte`, `test_porcentajes_suman_exactamente_cien`, `test_mo_mas_ref_cero_porcentajes_none_y_montos_visibles`, `test_mo_o_ref_negativos_porcentajes_none`, `test_sin_ordenes_importes_en_cero_con_dos_decimales`.

### 5.4 Mezcla de servicio

Dos conteos independientes sobre las órdenes activas: por tipo de ingreso y por tipo de servicio.

**Criterios de aceptación**
1. Garantía y mantenimiento aparecen siempre, aunque estén en cero.
2. "Sin clasificar" aparece solo si hay alguna, y al final.
3. El tipo de servicio se agrupa sin distinguir mayúsculas y se muestra en mayúsculas; vacío es "Sin tipo".
4. Orden del tipo de servicio: conteo descendente; "Sin tipo" al final en caso de empate.

**Evidencia:** `test_mezcla_garantia_y_mantenimiento_siempre_presentes_en_cero`, `test_mezcla_null_solo_si_mayor_a_cero_y_al_final`, `test_tipo_servicio_sin_distinguir_mayusculas_en_mayusculas_y_ordenado`, `test_tipo_servicio_orden_exacto_empate_con_none_al_final`.

## 6. Valores vacíos

El Panel nunca muestra un cero engañoso. Cuando no se puede calcular un valor, este no existe y la pantalla lo explica.

| Situación | Texto |
|---|---|
| Sin órdenes evaluables de promesa | "Sin datos de promesa para este periodo" |
| Sin órdenes facturadas | "Sin órdenes facturadas en este periodo" |
| Sin órdenes activas | "Sin órdenes en este periodo" |
| Activas pero ninguna cerrada | "Ninguna orden cerrada en este periodo" |

**Criterio de aceptación:** un dealer sin órdenes recibe un Panel válido, con valores inexistentes e importes en cero, sin error.
**Evidencia:** `integration/test_panel_endpoint.py::test_dealer_sin_ordenes_recibe_panel_vacio_con_nulls_e_importes_cero`.

## 7. Privacidad

**Criterio de aceptación:** la respuesta del Panel no contiene datos del cliente final, VIN, placas, identificadores ni números de orden.
**Evidencia:** `integration/test_panel_endpoint.py::test_respuesta_no_contiene_datos_de_cliente_ni_ids_de_orden`.

El nombre del asesor y el tipo de servicio son texto libre del archivo y se muestran tal cual. Qué hacer si contienen datos personales está abierto [DA-03].

## 8. Contenido de la respuesta

| Bloque | Campos |
|---|---|
| Dealer | Identificador, nombre, activo |
| Periodo | Tipo, inicio y fin de la ventana |
| Órdenes en el periodo | Número de órdenes activas |
| Cumplimiento de promesa | Evaluables, a tiempo, porcentaje, completadas sin promesa |
| Asesores | Por cada uno: asesor, volumen, completadas, tiempo promedio |
| Ingresos | Órdenes facturadas, órdenes sin importe, total, mano de obra, refacciones, los dos porcentajes |
| Mezcla | Lista por tipo de ingreso y lista por tipo de servicio, cada una con clave y número de órdenes |

Los importes viajan como texto con dos decimales. Contrato completo en MERP-106.

## 9. Dónde se calcula

Todas las métricas se calculan en el servidor, como funciones puras del dominio que reciben las órdenes del dealer y la ventana (MERP-103, P-2, P-3 y P-6). El frontend solo presenta. El Panel no tiene tablas propias.

## 10. Puntos que dependen de decisiones abiertas

| Punto | Efecto en el Panel | Decisión |
|---|---|---|
| Órdenes que dejan de venir en el archivo | Siguen contando como activas en todas las ventanas | DA-08 |
| Promesa anterior a la fecha de alta | Contará como entregada tarde al cerrar | DA-09 |
| Importe ilegible | La orden cuenta como "sin importe" | DA-09 |
| `diag` y `repme` sin clasificar | La mezcla por tipo de ingreso solo distingue mantenimiento | DA-02 |
| Asesor y tipo de servicio como texto libre | Posibles datos personales a la vista | DA-03 |

---

# Parte B — Prototipo

| Tema | Cómo está hecho |
|---|---|
| Ruta | `GET /api/v1/panel?periodo=semana\|mes&dealer_id=` |
| Cálculo | `backend/app/domain/services/panel_distribuidor.py` |
| Caso de uso | `backend/app/application/use_cases/get_panel_distribuidor.py`; recibe el reloj como parámetro y descarta los microsegundos |
| Datos | Lee todas las órdenes del dealer y filtra en memoria; no hay consultas agregadas en SQL |
| Códigos | 400 Admin sin dealer · 401 · 403 dealer ajeno o usuario sin dealer · 404 dealer inexistente · 422 periodo o identificador mal formado (formato de validación del marco, MERP-106 §6) |
| Pantalla | `/panel`, en `frontend/src/features/panel/` |

**Pantalla del prototipo**
- Encabezado con el nombre del dealer y "últimos 7 días" o "últimos 30 días".
- Admin: selector "Distribuidor" con todos los dealers; los inactivos llevan "(dado de baja)"; por defecto, el primer dealer activo.
- Selector Semana / Mes.
- Línea con el inicio y el fin de la ventana y el número de órdenes.
- Tarjetas: Cumplimiento de promesa · Ingresos facturados · Volumen por asesor · Días promedio de cierre por asesor · Mezcla por tipo de ingreso · Mezcla por tipo de servicio.
- Gráficas de barras en SVG propio, con desplazamiento horizontal cuando hay muchas barras.
- Los datos en memoria del navegador se separan por usuario y se borran al iniciar y al cerrar sesión.

**Observación de los datos de desarrollo.** Durante M7 se observó un total facturado cercano a 1.48 veces la suma de mano de obra y refacciones. No se afirma a qué corresponde la diferencia. **[VERIFICAR]** con Magma qué conceptos incluye `TOTAL`.

**Rendimiento.** Leer todas las órdenes del dealer en cada consulta es suficiente para el volumen del piloto. Con historial de años habrá que filtrar por fecha en la base.
