# MERP-101 — Requisitos
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 3.2 · **Fecha:** 26 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> Especificación de la versión nueva. Cada requisito lleva un criterio de aceptación verificable, su situación en el prototipo y, cuando existe, la prueba que ya lo demuestra. Las rutas de prueba son relativas a `backend/tests/` del prototipo. Los números RF-001 a RF-014 y RNF-001 a RNF-008 se conservan; los retirados no se reutilizan.

---

# Parte A — Especificación

## 1. Requisitos funcionales

| ID | Requisito | En el prototipo |
|---|---|---|
| RF-001 | Gestión de dealers | Construido |
| RF-002 | Alta de usuarios Admin | Construido |
| RF-003 a RF-006 | Empleados y captura manual | Retirados |
| RF-007 | Dashboard con semáforo | Construido; agrupado y filtro planeados [DA-07] |
| RF-008 | Autenticación y autorización por rol | Construido |
| RF-009 | Alta de usuarios Dealer | Construido |
| RF-010 | Carga de órdenes desde Excel | Construido |
| RF-011 | Historial de cargas | Construido; paginación planeada [DA-06] |
| RF-012 | Consulta de órdenes | Construido |
| RF-013 | Panel del Distribuidor | Construido |
| RF-014 | Corte de acceso por baja de dealer | Construido |
| RF-015 | Gestión de usuarios | No construido [DA-04] |
| RF-016 | Cierre de sesión y gestión de la contraseña | No construido [DA-05] |
| RF-017 | Avisos de la carga | No construido [DA-09] |
| RF-018 | Perfiles de importación por distribuidor | No construido [DA-18 a DA-22] |

### RF-001 — Gestión de dealers
El Admin puede dar de alta un dealer, cambiar su nombre, darlo de baja y reactivarlo. La baja es lógica (RN-008).

**Criterios de aceptación**
1. Un Admin crea un dealer con nombre y este queda activo.
2. Un Admin cambia el nombre de un dealer y el cambio se refleja al listarlos.
3. Tras dar de baja un dealer, sigue apareciendo en el listado como inactivo y sus órdenes siguen visibles para el Admin.
4. Un Admin reactiva un dealer dado de baja.
5. Un Dealer que intenta cualquiera de estas operaciones recibe un rechazo por permisos.
6. Operar sobre un dealer inexistente responde "no encontrado".

**Evidencia en el prototipo:** `integration/test_dealers_endpoint.py` (`test_admin_puede_crear_dealer`, `test_admin_puede_actualizar_nombre_de_dealer`, `test_admin_puede_dar_de_baja_dealer`, `test_admin_puede_reactivar_dealer_via_patch`, `test_rol_dealer_no_puede_administrar_dealers`, `test_patch_a_dealer_inexistente_devuelve_404`, `test_baja_logica_no_borra_el_dealer_ni_sus_ordenes_historicas`).
**Abierto:** si el nombre del dealer debe ser único [DA-11].

### RF-002 — Alta de usuarios Admin
El Admin puede crear usuarios Admin con nombre, correo y contraseña.

**Criterios de aceptación**
1. Un Admin crea otro Admin; el nuevo usuario no tiene dealer.
2. Un correo ya registrado se rechaza como duplicado, sin distinguir mayúsculas.
3. Un Dealer o una petición sin sesión no pueden crear Admins.
4. La contraseña cumple la política vigente [DA-05]; en el prototipo, mínimo 8 caracteres.

**Evidencia:** `integration/test_users_admin_endpoint.py` (los 4 casos); `unit/application/test_create_admin.py`.

### RF-003 a RF-006 — Retirados
- RF-003, gestión de empleados por el Dealer: retirado con el rol Empleado.
- RF-004, RF-005 y RF-006, captura manual de entrada, trabajo y salida: sustituidos por RF-010.

### RF-007 — Dashboard con semáforo
El sistema muestra las órdenes con su color de semáforo (RN-002, RN-003). El Admin ve las de todos los dealers; el Dealer solo las suyas.

**Criterios de aceptación**
1. Cada orden muestra dealer, número de orden, modelo, año, tipo de ingreso, estado, días en planta y color.
2. Los días y el color coinciden con RN-002 y RN-003 para órdenes en los límites de cada rango.
3. Una orden completada conserva sus días y su color aunque pase el tiempo.
4. Un Dealer no recibe ninguna orden de otro dealer.
5. El orden de la tabla pone primero lo urgente: en proceso antes que completadas; dentro de cada grupo, rojo, amarillo, verde; después, más días primero.
6. Se muestran estadísticas: órdenes por semáforo, promedio de días en planta, órdenes por tipo de ingreso y, para el Admin, órdenes por dealer.
7. Se muestra la tendencia de órdenes entradas y de promedio de días, por semana o por mes.
8. Una orden sin modelo o sin tipo de ingreso se muestra sin error.

**Evidencia:** `integration/test_dashboard_endpoint.py` (`test_dashboard_admin_ve_los_cuatro_colores_con_los_dias_correctos`, `test_dashboard_aislado_por_dealer`, `test_dashboard_orden_sin_tipo_ni_modelo`); `unit/application/test_get_dashboard.py`. Los criterios 5, 6 y 7 viven en el frontend del prototipo y **no tienen prueba automatizada**.
**Planeado, no construido [DA-07]:** agrupar la tabla del Admin por distribuidor y filtrar por dealer.

### RF-008 — Autenticación y autorización por rol
Todo usuario inicia sesión con correo y contraseña. El sistema reconoce su rol y limita lo que puede ver y hacer.

**Criterios de aceptación**
1. Con credenciales correctas se obtiene una sesión que identifica el rol y el dealer del usuario.
2. Correo inexistente, contraseña incorrecta, usuario inactivo y dealer dado de baja producen la misma respuesta, sin indicar cuál falló.
3. El correo se compara sin distinguir mayúsculas.
4. Toda operación distinta del inicio de sesión y de la verificación de salud rechaza una petición sin sesión o con sesión inválida.
5. Una operación reservada a un rol rechaza a un usuario de otro rol.
6. La contraseña se guarda solo como hash; nunca aparece en una respuesta.

**Evidencia:** `integration/test_auth_login.py` (5 casos); `integration/test_auth_authorization.py` (`test_sin_token_devuelve_401`, `test_token_invalido_devuelve_401`, `test_rol_insuficiente_devuelve_403`); `unit/application/test_authenticate_user.py`; `unit/infrastructure/test_argon2_password_hasher.py`; `unit/infrastructure/test_jwt_token_service.py`.

### RF-009 — Alta de usuarios Dealer
El Admin puede crear usuarios con rol Dealer para un dealer existente y activo.

**Criterios de aceptación**
1. El usuario creado puede iniciar sesión y queda asociado al dealer indicado.
2. El rol lo fija el sistema: un rol enviado en la petición se ignora.
3. Dealer inexistente → "no encontrado". Dealer dado de baja → rechazo por regla de negocio. Correo duplicado → "duplicado".
4. Un Dealer o una petición sin sesión no pueden crear usuarios.

**Evidencia:** `integration/test_users_dealers_endpoint.py` (`test_admin_crea_usuario_dealer`, `test_el_usuario_creado_puede_hacer_login`, `test_el_rol_del_payload_se_ignora`, `test_dealer_inexistente_devuelve_404`, `test_dealer_dado_de_baja_devuelve_422`, `test_email_duplicado_devuelve_409_sin_distinguir_mayusculas`, `test_rol_dealer_devuelve_403`); `unit/application/test_create_dealer_user.py`.

### RF-010 — Carga de órdenes desde Excel
El Dealer sube el archivo de su sistema de taller. El sistema crea o actualiza una orden por fila. Especificación completa en MERP-112.

**Criterios de aceptación**
1. Subir el Excel de ejemplo (62 filas) crea 62 órdenes en el dealer del usuario.
2. Subir el mismo archivo dos veces no duplica órdenes: la segunda carga reporta todas como "sin cambios".
3. Una orden que en una carga venía sin cierre y en la siguiente trae cierre pasa a completada.
4. Una fila inválida se rechaza con su número de fila y su motivo; las demás se cargan.
5. Un archivo inválido se rechaza completo y no deja registro de carga.
6. El resumen cumple: filas totales = nuevas + actualizadas + sin cambios + rechazadas.
7. Un Admin no puede subir archivos.
8. Las órdenes de un dealer nunca quedan en otro dealer, aunque el archivo sea el mismo.

**Evidencia:** `integration/test_uploads_endpoint.py` (`test_dealer_sube_el_excel_de_ejemplo`, `test_subir_el_mismo_archivo_dos_veces_no_duplica`, `test_carga_miercoles_en_proceso_y_viernes_con_cierre`, `test_filas_invalidas_se_reportan_sin_tirar_la_carga`, `test_archivo_invalido_no_registra_carga`, `test_archivo_sin_columnas_obligatorias`, `test_archivo_demasiado_grande`, `test_solo_el_dealer_puede_subir`, `test_cada_dealer_carga_en_su_propio_dealer`); `unit/application/test_process_upload.py`; `unit/infrastructure/test_xlsx_order_reader.py`.

### RF-011 — Historial de cargas
El sistema muestra las cargas realizadas, de la más reciente a la más antigua, con fecha, archivo, quién la subió, los cuatro contadores y el detalle de filas rechazadas.

**Criterios de aceptación**
1. Un Dealer ve solo las cargas de su dealer, aunque pida las de otro.
2. Un Admin ve las de todos los dealers, con el nombre del dealer, y puede limitar la consulta a uno.
3. Cada carga muestra el motivo de cada fila rechazada.

**Evidencia:** `integration/test_uploads_endpoint.py::test_historial_por_rol`; `unit/application/test_process_upload.py::test_historial_por_rol`; `integration/test_order_repository.py::test_upload_guarda_y_lista_errores_por_dealer`.
**Planeado, no construido [DA-06]:** paginación; filtro por dealer en pantalla (la API del prototipo lo permite, la pantalla no).

### RF-012 — Consulta de órdenes
El sistema permite listar las órdenes y consultar una, con el mismo alcance por rol que el dashboard.

**Criterios de aceptación**
1. El Admin obtiene órdenes de todos los dealers; el Dealer, solo las suyas.
2. Consultar una orden de otro dealer responde exactamente igual que consultar una inexistente.
3. La respuesta no incluye los datos del cliente final.

**Evidencia:** `integration/test_trucks_read_endpoint.py` (6 casos); `unit/application/test_list_get_trucks.py`.

### RF-013 — Panel del Distribuidor
El sistema muestra, para un dealer y un periodo de 7 o 30 días, cumplimiento de promesa, desempeño por asesor, ingresos facturados y mezcla de servicio. Especificación y criterios en MERP-109.

**Evidencia:** `unit/domain/test_panel_distribuidor.py` (29 casos); `integration/test_panel_endpoint.py` (16 casos); `unit/application/test_get_panel_distribuidor.py`.

### RF-014 — Corte de acceso por baja de dealer
Al dar de baja un dealer, sus usuarios dejan de poder iniciar sesión y de usar una sesión ya abierta. Al reactivarlo, recuperan el acceso (RN-011).

**Criterios de aceptación**
1. Tras la baja, el inicio de sesión de un usuario de ese dealer se rechaza como credenciales inválidas.
2. Una sesión abierta antes de la baja deja de servir en la siguiente petición.
3. Tras la reactivación, vuelven a funcionar el inicio de sesión y la sesión previa, solo para usuarios activos.
4. Los registros de los usuarios no se modifican con la baja ni con la reactivación.

**Evidencia:** `integration/test_dealers_endpoint.py` (`test_baja_de_dealer_corta_el_acceso_de_sus_usuarios`, `test_reactivar_dealer_devuelve_el_acceso_solo_a_los_usuarios_activos`); `integration/test_auth_authorization.py::test_token_emitido_antes_de_la_baja_del_dealer_deja_de_servir`; `unit/application/test_access_policy.py`.

### RF-015 — Gestión de usuarios · No construido [DA-04]
El sistema debe permitir listar los usuarios y retirar el acceso a una persona concreta sin dar de baja a todo su dealer. El alcance exacto (editar, reactivar, quién puede hacerlo) depende de DA-04.

**Criterio de aceptación mínimo:** tras dar de baja a un usuario, su inicio de sesión y su sesión abierta se rechazan, y los demás usuarios de su dealer siguen entrando. La baja es lógica (RN-008).
**En el prototipo:** solo existe el alta. La parte de sesión abierta sí está cubierta para un usuario inactivo (`integration/test_auth_authorization.py::test_usuario_dado_de_baja_pierde_el_acceso_aunque_el_token_siga_vigente`), pero no hay operación que lo desactive.

### RF-016 — Cierre de sesión y gestión de la contraseña · No construido [DA-05]
El usuario debe poder cerrar su sesión y cambiar su contraseña. La duración de la sesión, la política de contraseñas y la recuperación de una contraseña olvidada dependen de DA-05.

**Criterio de aceptación mínimo:** tras cambiar la contraseña, la anterior deja de servir para iniciar sesión.
**En el prototipo:** cerrar sesión solo borra los datos del navegador; no hay cambio ni recuperación de contraseña.

### RF-017 — Avisos de la carga · No construido [DA-09]
Cuando una fila trae un dato opcional que el sistema no puede interpretar, la carga debe informarlo en vez de descartarlo en silencio. Qué casos rechazan la fila y cuáles solo avisan depende de DA-09.

**Criterio de aceptación mínimo:** una fila con un importe no numérico no pasa inadvertida: aparece en el resumen de la carga, como rechazo o como aviso.
**En el prototipo:** el dato se guarda vacío sin aviso (MERP-114, C-02 y C-08).

### RF-018 — Perfiles de importación por distribuidor · No construido [DA-18 a DA-22]
Cada distribuidor puede exportar su archivo con un formato distinto. El sistema debe poder leer cualquiera de ellos **sin código ni caso de uso nuevo por distribuidor**: hay un único caso de uso de carga (CU-014) que aplica el perfil de importación del dealer que sube. El perfil es un dato que el propio distribuidor configura para su dealer.

**Criterios de aceptación**
1. Incorporar un distribuidor con un formato nuevo no requiere ningún cambio de código ni un despliegue.
2. El perfil define hoja y fila de encabezados, mapeo de columnas a campos, formato de fechas y equivalencias de valores (RN-021).
3. No se puede guardar un perfil sin los campos obligatorios (RN-022) ni sin vista previa y validación de tipos (RN-025).
4. Un campo opcional no asignado deja sus métricas como "no disponible" para ese dealer (RN-023).
5. Cada cambio crea una versión, queda registrado y solo afecta a las cargas posteriores; cada carga indica con qué versión se leyó (RN-026, RN-027).
6. El Admin ve el perfil y su historial en solo lectura (RN-024).
7. Si falta una columna que el perfil espera, el archivo se rechaza diciendo cuál (RN-028).
8. El resumen de la carga avisa si el perfil cambió desde la carga anterior (RN-029).

**Estrategia de adopción.** Plantilla única de Magma como camino principal; perfil configurable como respaldo. Para el piloto basta un perfil configurado a mano. El asistente de configuración (CU-020) puede construirse al llegar el segundo distribuidor [DA-21], pero el modelo de datos y el caso de uso de carga nacen ya con perfiles.
**En el prototipo:** el formato del primer distribuidor está fijo en el código del lector.

## 2. Requisitos no funcionales

| ID | Requisito | En el prototipo |
|---|---|---|
| RNF-001 | Apto para datos reales | No cumplido |
| RNF-002 | Arquitectura por capas | Construido |
| RNF-003 | Usabilidad y mensajes | Construido |
| RNF-004 | Datos de demostración | Construido |
| RNF-005 | Zona horaria | Construido en local [DA-12] |
| RNF-006 | El Excel real no se versiona | Construido |
| RNF-007 | Aislamiento entre dealers | Construido |
| RNF-008 | Entorno local sin pasos manuales | Construido |
| RNF-009 | HTTPS en producción | No construido |
| RNF-010 | CORS por configuración | No construido |
| RNF-011 | Arranque de producción | No construido |
| RNF-012 | Base de datos privada y cifrada | No construido |
| RNF-013 | Secretos fuera del repositorio | Parcial |
| RNF-014 | Repositorio remoto e integración continua | No construido |
| RNF-015 | Límite de intentos de inicio de sesión | No construido |
| RNF-016 | Inicio de sesión sin fuga de existencia | Parcial |
| RNF-017 | Formato único de errores | Parcial |
| RNF-018 | Respaldos con restauración probada | No construido |
| RNF-019 | Trazabilidad mínima de cambios | No construido [DA-16] |
| RNF-020 | Pruebas automatizadas | Parcial |
| RNF-021 | Construcción reproducible | Parcial |
| RNF-022 | Nomenclatura del dominio | No cumplido |
| RNF-023 | Sin datos reales en desarrollo | [VERIFICAR] |
| RNF-024 | Configuración por entorno | Parcial |
| RNF-025 | Capacidad: 25 distribuidores y crecimiento | No comprobado |
| RNF-026 | Tiempos de respuesta | Parcial |
| RNF-027 | Uso de todos los núcleos en producción | No cumplido |
| RNF-028 | Inicio de sesión rápido y seguro | No cumplido |
| RNF-029 | Prueba de carga antes del piloto | Parcial |

### RNF-001 — Apto para datos reales
El sistema no recibe datos reales de un distribuidor en un servidor público mientras no cumpla RNF-009 a RNF-016 y RNF-018.
**Criterio:** existe una lista de verificación previa al piloto con esos requisitos, todos comprobados y fechados.

### RNF-002 — Arquitectura por capas
Dominio, aplicación, infraestructura y presentación, con las dependencias apuntando hacia el dominio (MERP-103).
**Criterio:** el dominio no importa nada de las otras capas; una comprobación automática lo verifica en la integración continua.
**Evidencia:** en el prototipo se cumple por convención, sin comprobación automática.

### RNF-003 — Usabilidad y mensajes
Lo urgente se ve primero. Los mensajes de error que ve el usuario están en lenguaje llano, sin identificadores internos ni códigos de regla. Los estados sin datos se explican con texto, no con ceros.
**Criterio:** ningún mensaje de error de la API contiene un UUID ni un código de regla; cada pantalla tiene un estado vacío con texto.

### RNF-004 — Datos de demostración
El entorno de desarrollo puede poblarse con datos de ejemplo sin integraciones externas (MERP-113).
**Criterio:** sobre una base vacía, el sembrado deja al menos dos dealers, un Admin, un usuario por dealer y dos cargas de un Excel anonimizado; repetirlo no duplica nada.

### RNF-005 — Zona horaria · [DA-12]
El servidor y la base de datos trabajan en la hora local de la planta (`America/Mexico_City`).
**Criterio:** una carga hecha a las 10:00 hora de planta queda registrada a las 10:00, y una orden que entró hace 4 días y 23 horas no aparece con 5 días.
**Evidencia:** resuelto en el entorno local del prototipo; sin prueba automatizada.

### RNF-006 — El Excel real no se versiona
**Criterio:** el repositorio no contiene ningún archivo con datos de clientes reales; las pruebas usan una copia anonimizada.
**Evidencia:** `.gitignore` excluye el archivo real; solo `ordenes_ejemplo.xlsx` está versionado.

### RNF-007 — Aislamiento entre dealers
**Criterio:** para cada operación de lectura o escritura, existe una prueba en la que un Dealer intenta alcanzar datos de otro dealer y no los obtiene, sin que la respuesta revele si existen (RN-012).
**Evidencia:** `test_dashboard_aislado_por_dealer`, `test_listado_aislado_por_dealer`, `test_detalle_de_orden_de_otro_dealer_devuelve_404`, `test_dealer_con_dealer_id_ajeno_403_igual_exista_o_no`, `test_cada_dealer_carga_en_su_propio_dealer`, `test_historial_por_rol`.

### RNF-008 — Entorno local sin pasos manuales
**Criterio:** en un equipo sin estado previo, un solo comando levanta el sistema con migraciones aplicadas y datos de demostración.
**Evidencia:** verificado manualmente durante el prototipo; sin prueba automatizada.

### RNF-009 — HTTPS en producción
**Criterio:** toda petición por HTTP se redirige a HTTPS; el certificado es válido y se renueva solo.

### RNF-010 — CORS por configuración
Los orígenes permitidos se leen de la configuración del entorno. En producción solo se permite el dominio del frontend.
**Criterio:** una petición desde un origen no listado es rechazada por el navegador; cambiar el origen permitido no requiere cambiar código.
**En el prototipo:** fijo en `*` dentro del código.

### RNF-011 — Arranque de producción
En producción el servidor arranca sin recarga automática, no siembra datos de demostración y falla de forma visible si una migración falla.
**Criterio:** en un despliegue de producción recién creado no existe ningún usuario de demostración; el primer Admin se crea por un procedimiento documentado, con contraseña propia.
**En el prototipo:** un único arranque que recarga, siembra y oculta el fallo del sembrado.

### RNF-012 — Base de datos privada y cifrada
**Criterio:** la base de datos no es alcanzable desde internet y su almacenamiento está cifrado.

### RNF-013 — Secretos fuera del repositorio
**Criterio:** ningún secreto está en el repositorio ni en la imagen; el secreto de firma de sesiones de producción es distinto del de desarrollo.
**En el prototipo:** los archivos de entorno están excluidos del repositorio; no hay entorno de producción.

### RNF-014 — Repositorio remoto e integración continua
**Criterio:** el repositorio tiene un remoto; cada cambio ejecuta las pruebas y el análisis estático, y un cambio que los rompe no se despliega.
**En el prototipo:** sin remoto y sin flujo automatizado.

### RNF-015 — Límite de intentos de inicio de sesión
**Criterio:** tras un número configurable de intentos fallidos desde un mismo origen o sobre un mismo correo, los siguientes se rechazan durante un tiempo. Los valores se cierran en DA-05.

### RNF-016 — Inicio de sesión sin fuga de existencia
El mensaje y el tiempo de respuesta de un inicio de sesión fallido no permiten distinguir si el correo existe.
**Criterio:** el tiempo medio de respuesta para un correo inexistente y para un correo existente con contraseña incorrecta no difiere de forma medible.
**En el prototipo:** el mensaje es idéntico; el tiempo no (MERP-114, D-04).

### RNF-017 — Formato único de errores
Todos los errores de la API, incluidos los de validación de datos de entrada, usan el mismo formato.
**Criterio:** una petición con un campo mal formado y una petición rechazada por una regla de negocio devuelven la misma estructura de error.
**En el prototipo:** la validación automática usa otro formato (MERP-106).

### RNF-018 — Respaldos con restauración probada
**Criterio:** existe un respaldo automático diario y se ha restaurado al menos una vez en un entorno aparte, con fecha y resultado registrados.

### RNF-019 — Trazabilidad mínima de cambios · [DA-16]
Cada registro guarda cuándo se creó y cuándo se modificó por última vez.
**Criterio:** todas las tablas de negocio tienen ambas fechas y se actualizan solas. Si además se requiere una bitácora de quién hizo qué, se decide en DA-16.

### RNF-020 — Pruebas automatizadas
Backend y frontend tienen pruebas automatizadas que corren en la integración continua, contra una base de datos exclusiva de pruebas.
**Criterio:** las reglas de negocio de MERP-102 tienen cada una al menos una prueba; la lógica de orden y de estadísticas del dashboard tiene pruebas.
**En el prototipo:** 245 pruebas de backend; ninguna de frontend; las de integración usan la base de desarrollo.

### RNF-021 — Construcción reproducible
**Criterio:** dos construcciones del mismo commit instalan exactamente las mismas versiones de dependencias y de herramientas.
**En el prototipo:** hay archivo de bloqueo, pero una construcción cambió versiones y una herramienta se instala sin versión fija (MERP-114, F-03).

### RNF-022 — Nomenclatura del dominio
Rutas, campos, mensajes y pantallas hablan de "órdenes". No queda el término "camión" para referirse a una orden.
**Criterio:** una búsqueda de "truck" y "camión" en el código y en los textos de la interfaz no devuelve usos que se refieran a una orden.

### RNF-023 — Sin datos reales en desarrollo
**Criterio:** ninguna base de desarrollo o de pruebas contiene datos personales de clientes reales.
**En el prototipo:** **[VERIFICAR]** (MERP-114, G-01).

### RNF-024 — Configuración por entorno
Todo lo que cambia entre desarrollo y producción se controla con variables de entorno documentadas, y el sistema se niega a arrancar si falta una obligatoria.
**Criterio:** arrancar sin una variable obligatoria falla de inmediato con un mensaje que la nombra.
**En el prototipo:** tres variables se leen al usarse, no al arrancar; dos de la plantilla no se leen (MERP-113).

### RNF-025 — Capacidad: 25 distribuidores y crecimiento
El sistema debe atender a **25 distribuidores** desde el inicio, con unas **50 personas** usándolo, y estar preparado para crecer. Se debe asumir que los distribuidores coinciden: si todos suben miércoles y viernes, pueden subir en la misma hora.
**Criterio:** la prueba de carga (RNF-029) cumple RNF-026 con 25 distribuidores subiendo a la vez y 50 usuarios consultando, y se repite con el doble (50 distribuidores, 100 usuarios) para conocer el margen.
**Origen:** dato aportado por el dueño del proyecto el 26 de octubre de 2026.

### RNF-026 — Tiempos de respuesta
| Operación | Objetivo |
|---|---|
| Carga de un archivo del tamaño real (unas 62 órdenes) | Menos de 2 segundos |
| Carga de un archivo de 2,000 órdenes | Menos de 10 segundos |
| Consultas de dashboard, Panel e historial | Menos de 2 segundos en el 95 % de las peticiones |
| Inicio de sesión | Menos de 0.5 segundos (RNF-028) |
| CPU del servidor con la carga esperada | Menos de 70 % sostenido |

**Criterio:** la prueba de carga los mide y todos se cumplen con la carga de RNF-025.
**En el prototipo (26-oct-2026, una carga sola, backend limitado a 2 CPU):** 62 filas en 0.21 s si son nuevas y 0.32 s si se actualizan; 500 filas en 1.09 y 2.03 s; **2,000 filas en 5.13 y 8.31 s**. Si en la versión nueva un archivo grande no cumple el objetivo, se revisa ADR-005 (procesar la carga en segundo plano).

### RNF-027 — Uso de todos los núcleos en producción
En producción, el backend corre con **varios procesos de trabajo**, al menos uno por núcleo del servidor, para que un servidor de 2 CPU use las 2.
**Criterio:** con carga, el monitoreo muestra los procesos repartidos entre todos los núcleos; ninguno se queda en 100 % mientras otro está libre.
**En el prototipo:** un solo proceso. La prueba de carga mostró que el backend llega a un núcleo completo (~104 % en el monitor de Docker) y no usa el segundo, aunque el contenedor tenga 2 CPU.

### RNF-028 — Inicio de sesión rápido y seguro
El hash de contraseñas sigue siendo Argon2, pero con parámetros ajustados para que verificar una contraseña tarde alrededor de **0.3 segundos** en el servidor de producción. La verificación no debe bloquear al resto de las peticiones.
**Criterio:** el inicio de sesión responde en menos de 0.5 segundos con un usuario y con 50 usuarios entrando a la vez, y los parámetros de Argon2 están documentados con la medición que los justifica.
**En el prototipo:** **1.1 segundos** por inicio de sesión, con solo 3 usuarios. Con 50 personas entrando al mismo tiempo sería un cuello de botella. Se complementa con el límite de intentos (RNF-015).

### RNF-029 — Prueba de carga antes del piloto
Antes de que el distribuidor cargue datos reales, se ejecuta una prueba de carga sobre un servidor con las mismas especificaciones que producción y con el backend configurado como en producción (RNF-027).
**Criterio:** existe un informe de la prueba, con fecha y commit, que cubre carga de archivos, consultas, mezcla de ambas y archivos grandes, y cumple RNF-025 y RNF-026. Se repite al sumar distribuidores.
**Reglas para que la medición sea válida**
1. Se mide la CPU real de cada contenedor, no solo el porcentaje sobre su límite: un proceso que usa un núcleo completo de dos aparece como 50 % y oculta que está saturado.
2. El equipo donde corre la prueba tiene más memoria que la suma de los límites de los contenedores; si no, la medición de memoria no es confiable.
3. Los archivos de prueba alternan versiones con datos distintos, para que cada carga reescriba las filas y no se cuenten como "sin cambios".
4. Nunca se usan archivos con datos reales.
5. La prueba corre en un entorno aislado, con su propia base, y no toca la de desarrollo.

**En el prototipo:** prueba ejecutada el 26 de octubre de 2026 con hasta 20 distribuidores y 50 usuarios, en un equipo con 4.8 GB de memoria para Docker frente a 6 GB de límites. Resultados completos pendientes (MERP-114, grupo L).

---

# Parte B — Prototipo

| Tema | Cómo está hoy |
|---|---|
| Rutas de órdenes | Se llaman `/trucks`; el dashboard devuelve la lista bajo la clave `trucks` |
| Dashboard | El servidor calcula días y color; el orden por urgencia, las estadísticas y la tendencia se calculan en el navegador. La pantalla se refresca cada 15 segundos |
| Contraseña | Mínimo 8 caracteres, sin más política |
| Sesión | Token firmado con duración configurable; se guarda en el almacenamiento local del navegador |
| Carga | `.xlsx`, máximo 5 MB, procesamiento inmediato en una sola transacción |
| Frecuencia de carga | Miércoles y viernes por convención; el sistema no la impone ni la mide |

El detalle de cada tema está en la Parte B de MERP-103, 106, 107, 112 y 113.
