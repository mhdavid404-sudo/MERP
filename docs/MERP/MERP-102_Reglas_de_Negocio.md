# MERP-102 — Reglas de Negocio
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 3.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> Reglas que la versión nueva debe cumplir. Cada una lleva un criterio de aceptación verificable y la prueba del prototipo que ya la demuestra (rutas relativas a `backend/tests/`). Los números RN-001 a RN-011 se conservan porque el código y los planes del prototipo los citan; los retirados no se reutilizan. RN-012 a RN-020 se numeraron en esta serie para comportamiento que el prototipo ya tenía. RN-021 a RN-029 son nuevas: regulan los perfiles de importación por distribuidor, que el prototipo no tiene.

---

# Parte A — Especificación

## 1. Resumen

| Regla | Tema | Estado | Decisión abierta |
|---|---|---|---|
| RN-001 | Estado de la orden | Vigente | — |
| RN-002 | Rangos del semáforo | Vigente | DA-01 |
| RN-003 | Semáforo congelado al salir | Vigente | — |
| RN-004, 005, 006, 009 | Captura manual y empleados | Retiradas | — |
| RN-007 | Solo el Admin gestiona dealers y usuarios | Vigente | DA-04 |
| RN-008 | Bajas lógicas | Vigente | DA-04 |
| RN-010 | La salida no puede ser anterior a la entrada | Vigente | — |
| RN-011 | Acceso condicionado al dealer activo | Vigente | — |
| RN-012 | Aislamiento entre dealers | Vigente | — |
| RN-013 | El dealer de una orden sale del usuario | Vigente | — |
| RN-014 | Usuario sin dealer asignado | Vigente | — |
| RN-015 | Solo el Dealer sube archivos | Vigente | — |
| RN-016 | Identidad de la orden y actualización entre cargas | Vigente | DA-08, DA-10 |
| RN-017 | Rechazo por fila, no por carga | Vigente | DA-09 |
| RN-018 | Clasificación del tipo de ingreso | Vigente | DA-02 |
| RN-019 | Promesa sin hora | Vigente | — |
| RN-020 | Alta de usuarios | Vigente | — |
| RN-021 | Un perfil de importación por dealer | Vigente, no construida | DA-19 |
| RN-022 | Contrato mínimo del perfil | Vigente, no construida | DA-18 |
| RN-023 | Campo opcional ausente: métrica "no disponible" | Vigente, no construida | DA-18 |
| RN-024 | Quién configura y quién ve el perfil | Vigente, no construida | DA-20 |
| RN-025 | Vista previa y validación de tipos antes de guardar | Vigente, no construida | — |
| RN-026 | Versionado del perfil | Vigente, no construida | — |
| RN-027 | Registro de cambios del perfil | Vigente, no construida | — |
| RN-028 | Columna esperada ausente | Vigente, no construida | — |
| RN-029 | Aviso de cambio de perfil en el resumen | Vigente, no construida | — |

## 2. Reglas vigentes

### RN-001 — Estado de la orden
Una orden está **en proceso** mientras no tiene fecha de salida y **completada** cuando la tiene. El estado se deriva de la fecha de salida y nunca puede contradecirla.

**Criterios de aceptación**
1. Una orden sin fecha de salida está en proceso; con fecha de salida, completada.
2. Si una carga posterior trae la fecha de salida, la orden pasa a completada; si una carga posterior la quita, vuelve a en proceso.
3. No es posible guardar una orden cuyo estado contradiga su fecha de salida, ni siquiera escribiendo directo en la base.

**Evidencia:** `unit/domain/test_order.py` (`test_orden_sin_cierre_esta_en_proceso`, `test_orden_con_cierre_esta_completada`, `test_estado_sigue_a_fecha_salida_cuando_una_carga_posterior_trae_cierre`); `integration/test_order_repository.py::test_estado_se_guarda_derivado_de_fecha_salida`; `unit/application/test_process_upload.py::test_la_ultima_carga_manda_aunque_quite_un_cierre`.

### RN-002 — Rangos del semáforo · [DA-01]
El color depende de los días transcurridos desde la fecha de entrada: verde de 0 a 4, amarillo de 5 a 7, rojo con más de 7.

**Criterios de aceptación**
1. 4 días → verde; 5 → amarillo; 7 → amarillo; 8 → rojo.
2. El número de días se obtiene según la definición que cierre DA-01, y hay una prueba con una orden que entró el lunes a las 18:00 consultada el martes a las 09:00 que fija el resultado esperado.

**Evidencia:** `unit/domain/test_semaforo_calculator.py::test_calcular_semaforo_respeta_los_rangos_de_rn_002`; `integration/test_dashboard_endpoint.py::test_dashboard_admin_ve_los_cuatro_colores_con_los_dias_correctos`.
**Abierto [DA-01]:** el prototipo cuenta periodos completos de 24 horas (en el ejemplo del criterio 2 da 0 días). El texto original decía "días naturales", que daría 1. Hay que elegir.

### RN-003 — Semáforo congelado al salir
En proceso, los días se cuentan contra el momento de la consulta. Con fecha de salida, contra esa fecha, y el resultado ya no cambia. El color no se almacena.

**Criterios de aceptación**
1. Una orden en proceso consultada en dos momentos distintos puede dar días distintos.
2. Una orden completada da los mismos días y el mismo color hoy y dentro de un año.
3. No existe ningún campo almacenado con el color.

**Evidencia:** `unit/domain/test_semaforo_calculator.py` (`test_en_proceso_cuenta_contra_ahora`, `test_completado_cuenta_contra_su_propia_fecha_salida_no_contra_ahora`); `unit/application/test_get_dashboard.py::test_dashboard_orden_cerrada_muestra_semaforo_congelado`.

### RN-007 — Solo el Admin gestiona dealers y usuarios
Solo el Admin puede dar de alta, editar, dar de baja o reactivar dealers, y solo el Admin puede crear usuarios.

**Criterio de aceptación:** cada una de esas operaciones, intentada por un Dealer, se rechaza por permisos; intentada sin sesión, se rechaza por autenticación.
**Evidencia:** `test_rol_dealer_no_puede_administrar_dealers`, `test_rol_dealer_no_puede_crear_admins`, `test_rol_dealer_devuelve_403`, `test_crear_admin_sin_token_devuelve_401`.
**Abierto [DA-04]:** si un Dealer podrá gestionar a los usuarios de su propio dealer.

### RN-008 — Bajas lógicas
Dar de baja un dealer o un usuario es una desactivación, nunca un borrado. Se conserva el historial de órdenes y cargas.

**Criterios de aceptación**
1. Tras la baja de un dealer, el registro existe como inactivo y sus órdenes siguen ahí.
2. Tras la baja de un usuario, el registro existe como inactivo, las cargas que subió conservan su autor y no puede entrar.
3. El sistema no ofrece ninguna operación de borrado físico de dealers, usuarios, órdenes o cargas, salvo lo que defina la política de retención [DA-03].

**Evidencia:** criterio 1: `test_baja_logica_no_borra_el_dealer_ni_sus_ordenes_historicas`, `test_deactivate_dealer_marca_activo_false_sin_borrar`. Criterio 2: **no construido** en el prototipo, que no tiene baja de usuarios (RF-015).

### RN-010 — La salida no puede ser anterior a la entrada
**Criterios de aceptación**
1. Una fila con cierre anterior al alta se rechaza con un motivo legible.
2. Cierre igual al alta se acepta y da 0 días.
3. La base de datos rechaza ese caso aunque se escriba directo en ella.

**Evidencia:** `unit/domain/test_order.py::test_rechaza_cierre_anterior_a_alta_rn_010`; `unit/application/test_process_upload.py::test_rechaza_cierre_anterior_a_alta_rn_010`; `integration/test_order_repository.py::test_check_de_bd_rechaza_cierre_anterior_a_alta`. El criterio 2 se observó al ejecutar el prototipo; sin prueba dedicada.

### RN-011 — Acceso condicionado al dealer activo
Un usuario solo puede iniciar sesión y usar el sistema si está activo y, cuando pertenece a un dealer, ese dealer también está activo. La baja de un dealer no modifica a sus usuarios: les retira el acceso mientras dure.

**Criterios de aceptación**
1. Usuario activo de dealer inactivo: inicio de sesión rechazado como credenciales inválidas.
2. Una sesión abierta antes de la baja se rechaza en la siguiente petición a cualquier operación.
3. Al reactivar el dealer, recuperan el acceso solo los usuarios que siguen activos.
4. La baja y la reactivación no escriben en los registros de los usuarios.
5. Un usuario cuyo dealer no existe no tiene acceso.
6. Un Admin no depende de ningún dealer.

**Evidencia:** `unit/application/test_access_policy.py` (5 funciones, 8 casos); `unit/application/test_authenticate_user.py` (`test_login_de_usuario_con_dealer_dado_de_baja_lanza_authentication_error`, `test_login_con_dealer_inexistente_lanza_authentication_error`, `test_login_de_admin_no_consulta_dealers`); `integration/test_dealers_endpoint.py` (`test_baja_de_dealer_corta_el_acceso_de_sus_usuarios`, `test_reactivar_dealer_devuelve_el_acceso_solo_a_los_usuarios_activos`); `integration/test_panel_endpoint.py::test_dealer_dado_de_baja_durante_la_prueba_401_rn011`.

### RN-012 — Aislamiento entre dealers
Un Dealer solo ve y opera lo de su dealer. Cuando pide algo ajeno, la respuesta no revela si existe.

**Criterios de aceptación**
1. Consultar una orden de otro dealer responde exactamente igual que consultar una inexistente.
2. Pedir el Panel de otro dealer se rechaza con el mismo mensaje exista o no ese dealer, y sin consultar sus datos.
3. Los listados de órdenes, dashboard e historial de un Dealer contienen solo lo suyo, aunque pida otro dealer.

**Evidencia:** `test_detalle_de_orden_de_otro_dealer_devuelve_404`, `test_dealer_con_dealer_id_ajeno_403_igual_exista_o_no`, `test_dealer_con_param_ajeno_403_sin_consultar_nada`, `test_dealer_con_param_ajeno_inexistente_da_el_mismo_error`, `test_listado_aislado_por_dealer`, `test_dashboard_aislado_por_dealer`, `test_historial_por_rol`.

### RN-013 — El dealer de una orden sale del usuario, no del archivo
Toda orden cargada queda en el dealer del usuario que sube el archivo, tomado de su sesión y revalidado en el servidor.

**Criterio de aceptación:** dos dealers suben el mismo archivo; cada uno obtiene sus propias órdenes y ninguno ve las del otro.
**Evidencia:** `integration/test_uploads_endpoint.py::test_cada_dealer_carga_en_su_propio_dealer`; `unit/application/test_process_upload.py::test_misma_orden_en_otro_dealer_es_otra_orden`.

### RN-014 — Usuario sin dealer asignado
Un usuario que no es Admin y no tiene dealer recibe un rechazo por permisos en toda operación que dependa del dealer, antes de consultar nada.

**Criterio de aceptación:** ese usuario recibe el rechazo al listar órdenes, consultar una orden, ver el dashboard, ver el historial, ver el Panel y subir un archivo.
**Evidencia:** `unit/application/test_dealer_id_requerido.py` (6 casos); `integration/test_users_dealers_endpoint.py` (`test_dealer_sin_dealer_id_recibe_403_en_lecturas`, `test_dealer_sin_dealer_id_recibe_403_al_subir`); `test_dealer_con_dealer_id_none_en_el_usuario_authorization_error`.

### RN-015 — Solo el Dealer sube archivos
**Criterio de aceptación:** un Admin que intenta subir un archivo recibe un rechazo por permisos y no se registra ninguna carga.
**Evidencia:** `integration/test_uploads_endpoint.py::test_solo_el_dealer_puede_subir`; `integration/test_auth_authorization.py::test_require_role_solo_dealer_rechaza_al_admin`.

### RN-016 — Identidad de la orden y actualización entre cargas · [DA-08] [DA-10]
Una orden se identifica por la pareja (dealer, número de orden), única. Una carga que trae una orden existente la actualiza; si no existe, la crea. La carga más reciente es la foto vigente.

**Criterios de aceptación**
1. La misma orden en dos cargas del mismo dealer es una sola orden.
2. El mismo número de orden en dos dealers son dos órdenes distintas.
3. Una orden idéntica a la ya guardada cuenta como "sin cambios", no como "actualizada".
4. La base de datos impide dos órdenes con el mismo número en un mismo dealer.
5. Cada orden guarda cuál fue la última carga que la creó o la modificó.

**Evidencia:** `unit/application/test_process_upload.py` (`test_primera_carga_crea_todas_las_ordenes`, `test_carga_del_viernes_cierra_la_orden_que_el_miercoles_estaba_en_proceso`, `test_la_ultima_carga_manda_aunque_quite_un_cierre`, `test_misma_orden_en_otro_dealer_es_otra_orden`, `test_ordenes_que_no_vienen_en_la_carga_no_se_tocan`); `integration/test_order_repository.py` (`test_misma_orden_dos_veces_en_un_dealer_viola_unicidad`, `test_orden_apunta_a_su_ultima_carga`); `integration/test_uploads_endpoint.py::test_subir_el_mismo_archivo_dos_veces_no_duplica`.
**Abierto:** qué pasa con una orden en proceso que deja de venir en el archivo [DA-08]; el prototipo no la toca y queda en proceso para siempre. Si una carga puede cambiar el VIN de una orden existente [DA-10]; el prototipo lo permite.

### RN-017 — Rechazo por fila, no por carga · [DA-09]
Una fila inválida se rechaza sola, con su número de fila y su motivo; el resto se procesa. La carga es atómica: o se guardan el registro de la carga y todas sus órdenes válidas, o nada.

Motivos de rechazo de una fila:
- falta el número de orden, la serie o la fecha de alta;
- la fecha de alta o la de cierre no es una fecha;
- la fecha de cierre es anterior a la de alta;
- un texto excede el largo máximo de su campo;
- el número de orden ya apareció antes en el mismo archivo (se conserva la primera).

Motivos de rechazo del archivo completo, sin registrar carga: no es `.xlsx`, está vacío, excede el tamaño máximo, no se puede abrir, o le faltan las columnas obligatorias.

**Criterios de aceptación**
1. Un archivo con una fila de cada motivo carga las filas válidas y lista las rechazadas, ordenadas por fila, con el número de fila que el usuario ve en Excel.
2. Un archivo sin ninguna fila válida deja registrada la carga con todas las filas rechazadas.
3. Un archivo inválido no deja registro de carga.
4. Si el guardado falla a mitad, no queda ninguna orden de esa carga.

**Evidencia:** `unit/application/test_process_upload.py` (`test_rechaza_fila_sin_columnas_obligatorias`, `test_orden_repetida_en_el_archivo_se_toma_la_primera`, `test_rechazos_del_lector_se_suman_y_errores_quedan_ordenados_por_fila`, `test_carga_sin_filas_validas_igual_queda_registrada`); `unit/infrastructure/test_xlsx_order_reader.py` (`test_rechaza_fila_si_alta_o_cierre_no_son_fecha`, `test_rechaza_texto_mas_largo_que_la_columna`, `test_ignora_filas_vacias_y_conserva_numero_de_fila_real`, `test_archivo_sin_columnas_obligatorias_es_invalido`, `test_archivo_que_no_es_excel_es_invalido`); `integration/test_uploads_endpoint.py` (`test_filas_invalidas_se_reportan_sin_tirar_la_carga`, `test_archivo_invalido_no_registra_carga`, `test_archivo_demasiado_grande`). El criterio 4 no tiene prueba dedicada.
**Abierto [DA-09]:** qué hacer con los datos opcionales ilegibles, que el prototipo guarda vacíos sin avisar (RF-017).

### RN-018 — Clasificación del tipo de ingreso · [DA-02]
El tipo de ingreso de una orden es garantía, mantenimiento o **sin clasificar**. Se obtiene del código de servicio del archivo (columna `TDS1`). Los valores originales de `TDS1` y `THYP` se conservan con la orden para poder reclasificar.

**Criterios de aceptación**
1. `mant` (sin distinguir mayúsculas) → mantenimiento.
2. Un valor sin correspondencia confirmada, o vacío, → sin clasificar; nunca se adivina.
3. Tras confirmar una correspondencia nueva, las órdenes ya cargadas pueden reclasificarse sin volver a pedir los archivos.

**Evidencia:** `unit/infrastructure/test_xlsx_order_reader.py` (`test_tipo_ingreso_sale_de_tds1`, `test_datos_crudos_guarda_columnas_sin_campo_propio_y_tds1_thyp`). El criterio 3 es posible con los datos guardados; **no hay operación de reclasificación construida**.
**Abierto [DA-02]:** el significado de `diag` y `repme` y de la columna `THYP`. Hoy ningún valor produce "garantía".
**Con perfiles de importación:** `TDS1` es la columna del primer distribuidor. La correspondencia entre los valores de un archivo y el tipo de ingreso pasa a ser una **equivalencia de valores del perfil** de cada dealer (RN-021); la regla general se mantiene: lo que no tiene equivalencia confirmada queda sin clasificar.

### RN-019 — Promesa sin hora
Si el archivo trae fecha de promesa sin hora, se toma el final de ese día (23:59:59).

**Criterio de aceptación:** una orden prometida para un día sin hora y cerrada ese mismo día a las 18:00 cuenta como entregada a tiempo.
**Evidencia:** `unit/infrastructure/test_xlsx_order_reader.py::test_promesa_sin_hora_cuenta_hasta_el_final_del_dia`; `unit/domain/test_panel_distribuidor.py::test_salida_igual_a_promesa_cuenta_a_tiempo`.

### RN-020 — Alta de usuarios
El correo de un usuario es único en todo el sistema, sin distinguir mayúsculas. No se crean usuarios para un dealer inexistente ni para uno dado de baja. El rol de un usuario Dealer lo fija el sistema.

**Criterios de aceptación**
1. `Ana@x.com` y `ana@x.com` no pueden coexistir.
2. Dos altas simultáneas con el mismo correo: una se crea y la otra recibe "duplicado"; ninguna produce un error interno.
3. La existencia y el estado del dealer se comprueban antes que el correo.

**Evidencia:** `test_email_duplicado_devuelve_409_sin_distinguir_mayusculas`, `test_orden_de_validacion_dealer_antes_que_email`, `test_dealer_dado_de_baja_devuelve_422`. El criterio 2 **no se cumple** en el prototipo: responde con un error interno (MERP-114, D-06).

## 2 bis. Perfiles de importación por distribuidor

**Origen.** Cada distribuidor puede exportar su archivo con otro formato. Ejemplo real aportado el 8 de octubre de 2026: un reporte `.xls` de Crystal Reports con los encabezados en la fila 7, columnas `Orden`, `Serie`, `FechaOrd`, `FechaFsct`, `Status`, `Mano Obra`, `Refacciones`, `TOTAL` y `Días`, y sin columna de promesa. **[VERIFICAR]** ese archivo no se ha revisado para esta serie; se describe según lo indicado por el dueño del proyecto. No se escribirá código ni un caso de uso por distribuidor: las diferencias se expresan en un perfil, que es un dato.

**Riesgo que justifica los controles.** Un mapeo incorrecto **no falla**: calcula mal. Si se asigna una fecha de facturación como fecha de salida, la carga se procesa sin errores y el semáforo, el cumplimiento y los tiempos salen equivocados sin que nada lo señale. Por eso la vista previa (RN-025), el versionado (RN-026), el registro de cambios (RN-027), el aviso en el resumen (RN-029) y la visibilidad para el Admin (RN-024) son obligatorios, no opcionales.

### RN-021 — Un perfil de importación por dealer · [DA-19]
Cada dealer tiene un único perfil de importación vigente. Toda carga se interpreta con el perfil del dealer del usuario que sube. El perfil se guarda como dato y contiene: hoja y fila de encabezados; mapeo de cada columna del archivo a un campo de MERP; formato de las fechas; y equivalencias de valores.

**Criterios de aceptación**
1. Dos dealers con archivos de formato distinto cargan correctamente, cada uno con su perfil, **sin ningún cambio de código** entre uno y otro.
2. Un dealer sin perfil no puede cargar: se le indica que debe configurarlo.
3. Un archivo con los encabezados en una fila distinta de la primera se lee bien si el perfil lo indica.
4. Una columna del archivo que el perfil no asigna a ningún campo se ignora sin error.
5. El perfil de un dealer nunca se aplica a la carga de otro.

**Abierto [DA-19]:** qué formatos de archivo se aceptan (`.xls`, reportes con encabezados en otra fila) o si solo `.xlsx` limpio.

### RN-022 — Contrato mínimo del perfil · [DA-18]
Un perfil debe asignar, como mínimo, los campos obligatorios: **número de orden, VIN y fecha de entrada**. Todos los demás son opcionales: fecha de salida, fecha de promesa, asesor, importes, tipo de servicio, tipo de ingreso, modelo, año y placas.

**Criterios de aceptación**
1. Un perfil sin alguno de los campos obligatorios no se puede guardar, y el sistema dice cuál falta.
2. Un perfil con solo los campos obligatorios se puede guardar y sus cargas se procesan.
3. Un mismo campo no puede asignarse a dos columnas, ni una columna a dos campos.

**Abierto [DA-18]:** la lista definitiva de campos obligatorios la define Magma. **Advertencia:** si la fecha de salida queda como opcional y un distribuidor no la asigna, ninguna de sus órdenes se completa nunca: el semáforo corre para siempre y todo el Panel queda "no disponible" (RN-023).

### RN-023 — Campo opcional ausente: métrica "no disponible" · [DA-18]
Si el perfil de un dealer no asigna un campo opcional, las métricas que dependen de él se muestran como **"no disponible"** para ese dealer. "No disponible" (el distribuidor no aporta el dato) es distinto de "sin datos en el periodo" (lo aporta, pero no hubo órdenes).

| Campo no asignado | Lo que queda "no disponible" |
|---|---|
| Fecha de salida | Órdenes completadas, semáforo congelado, y todo lo que se calcula sobre órdenes completadas: cumplimiento de promesa, ingresos y tiempo de cierre por asesor |
| Fecha de promesa | Cumplimiento de promesa |
| Asesor | Desempeño por asesor |
| Total, mano de obra o refacciones | Ingresos facturados o su desglose, según cuál falte |
| Tipo de servicio | Mezcla por tipo de servicio |
| Tipo de ingreso | Mezcla por tipo de ingreso; estadística por tipo de ingreso del dashboard |

**Criterios de aceptación**
1. El Panel de un dealer cuyo perfil no asigna la promesa muestra el cumplimiento como "no disponible", no como 0 % ni como "sin datos".
2. El mismo Panel muestra con normalidad las secciones cuyos campos sí están asignados.
3. La disponibilidad se decide por la versión del perfil vigente, no fila por fila: un dato vacío en una fila de un campo que sí está asignado sigue las reglas actuales (por ejemplo, completadas sin promesa se cuentan aparte).

### RN-024 — Quién configura y quién ve el perfil · [DA-20]
El perfil lo configura y lo edita el propio distribuidor, solo para su dealer. El Admin de Magma ve el perfil de cualquier dealer y su historial, en solo lectura.

**Criterios de aceptación**
1. Un Dealer crea y edita el perfil de su dealer.
2. Un Dealer no puede ver ni modificar el perfil de otro dealer, y la respuesta no revela si existe (RN-012).
3. Un Admin ve el perfil vigente, las versiones y los cambios de cualquier dealer, y no tiene ninguna operación que lo modifique.

**Abierto [DA-20]:** si el Admin puede revertir un perfil a una versión anterior. Si se permite, la reversión crea una versión nueva y queda registrada como cualquier otro cambio.

### RN-025 — Vista previa y validación de tipos antes de guardar
Ningún perfil se guarda, ni al crearlo ni al editarlo, sin haber mostrado una vista previa con filas de un archivo real ya interpretadas, y sin que cada columna asignada pueda leerse con el tipo de su campo.

**Criterios de aceptación**
1. Guardar exige un archivo de muestra; la vista previa enseña, por cada fila mostrada, el valor original y el valor interpretado de cada campo asignado.
2. Una columna asignada a un campo de fecha debe poder leerse como fecha con el formato indicado; a un importe, como número; al año, como entero. Si no, no se guarda y se indica la columna y un valor de ejemplo que falla.
3. El formato de fecha indicado se comprueba contra la muestra: una muestra con fechas que solo son válidas en otro orden de día y mes impide guardar.
4. La vista previa no crea órdenes ni registra ninguna carga.

**Límite.** La validación de tipos detecta una columna que no es una fecha; **no** detecta una fecha equivocada. Que la columna elegida sea la fecha correcta solo lo puede confirmar una persona mirando la vista previa.

### RN-026 — Versionado del perfil
Cada cambio de un perfil crea una versión nueva, numerada. Las versiones no se modifican ni se borran. Cada carga queda ligada a la versión con la que se leyó.

**Criterios de aceptación**
1. Editar un perfil dos veces deja tres versiones consultables.
2. Cada carga del historial indica la versión del perfil con la que se interpretó, y esa versión se puede consultar entera.
3. Cambiar el perfil **no reinterpreta** ninguna carga ni modifica ninguna orden ya guardada: solo afecta a las cargas posteriores.
4. Ninguna operación permite modificar o borrar una versión existente.

**Consecuencia a tener presente.** Tras corregir un mapeo equivocado, las órdenes cargadas con la versión errónea conservan los datos erróneos hasta que una carga posterior las traiga de nuevo (RN-016). Las que ya no vengan en el archivo no se corrigen solas.

### RN-027 — Registro de cambios del perfil
De cada versión se registra quién la creó, cuándo y qué cambió respecto a la anterior.

**Criterios de aceptación**
1. El historial de un perfil lista cada versión con su autor, su fecha y las diferencias: columnas reasignadas, formato de fecha, fila de encabezados y equivalencias añadidas, cambiadas o quitadas.
2. El registro no se puede editar ni borrar.
3. Lo ven el Dealer dueño del perfil y el Admin.

### RN-028 — Columna esperada ausente
Si en una carga falta una columna que el perfil vigente espera, el archivo se rechaza completo, sin registrar la carga, indicando qué columnas faltan.

**Criterios de aceptación**
1. Falta una columna asignada a un campo obligatorio → rechazo, nombrándola.
2. Falta una columna asignada a un campo opcional → **también** rechazo, nombrándola: un archivo que deja de traer una columna indica que el formato cambió y el perfil debe revisarse.
3. Sobran columnas que el perfil no conoce → se carga con normalidad.
4. El mensaje nombra todas las columnas que faltan, no solo la primera.

### RN-029 — Aviso de cambio de perfil en el resumen
El resumen de una carga indica si la versión del perfil usada es distinta de la que usó la carga anterior del mismo dealer.

**Criterios de aceptación**
1. La primera carga tras editar el perfil lo indica en su resumen y en el historial; la siguiente, ya no.
2. El aviso lo ven el Dealer y el Admin.

## 3. Reglas retiradas

No describen el sistema. Se conservan para que sus números no se reutilicen.

| Regla | Texto original | Motivo |
|---|---|---|
| RN-004 | El tipo de ingreso se define en la entrada y puede corregirse en el módulo de trabajo. | Ya no hay captura manual. Sustituida por RN-018. |
| RN-005 | Un Dealer solo administra empleados de su dealer. | Se eliminó el rol Empleado. El aislamiento queda en RN-012. |
| RN-006 | Un Empleado solo opera camiones de su dealer. | Se eliminó el rol Empleado. Sustituida por RN-012 y RN-015. |
| RN-009 | No hay trabajo sin entrada, ni salida sin trabajo. | Una orden llega completa en una fila del archivo. |

## 4. Convenciones que no son reglas

- **Cargas miércoles y viernes.** Frecuencia acordada con el negocio. El sistema acepta cargas cualquier día. Si debe medirla o avisar, se decide en DA-17.

---

# Parte B — Prototipo

| Regla | Dónde vive en el prototipo |
|---|---|
| RN-001 | Propiedad derivada en `domain/entities/order.py`; restricción `ck_orders_estado_fecha_salida` |
| RN-002, RN-003 | `domain/services/semaforo_calculator.py`. Cuenta `(referencia − entrada).days`, es decir, periodos completos de 24 horas |
| RN-008 | Columna `activo` en dealers y usuarios. Solo los dealers tienen operación de baja |
| RN-010 | Validación en la entidad y restricción `ck_orders_fecha_salida_after_entrada` |
| RN-011 | `application/access_policy.py`, usada en el inicio de sesión y en cada petición autenticada |
| RN-012 | Orden ajena → 404; Panel ajeno → 403; historial ajeno → devuelve el propio |
| RN-014 | `AuthenticatedUser.dealer_id_requerido()` |
| RN-016 | Restricción `uq_orders_dealer_orden_externa`; `application/use_cases/process_upload.py` |
| RN-017 | Lector `infrastructure/excel/xlsx_order_reader.py` y `process_upload.py`. Tamaño máximo 5 MB |
| RN-018 | Tabla `_TIPO_INGRESO_POR_TDS1` en el lector; valores originales en el campo `datos_crudos` |
| RN-020 | Búsqueda de correo en minúsculas; la unicidad de la base sí distingue mayúsculas |
| RN-021 a RN-029 | **No existen.** El lector tiene fijo en el código el formato del primer distribuidor: nombres de columna, búsqueda del encabezado en las 10 primeras filas, formatos de fecha y la equivalencia `mant` → mantenimiento. Solo acepta `.xlsx` |
