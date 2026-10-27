# MERP-105 — Casos de Uso
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 3.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> La Parte A describe lo que cada actor debe poder hacer en la versión nueva, sin nombres del prototipo. Cada caso remite a los requisitos y reglas que lo verifican (MERP-101 y MERP-102), donde están los criterios de aceptación. La Parte B relaciona cada caso con el código actual. Los números CU-001 a CU-010 se conservan; los retirados no se reutilizan.

---

# Parte A — Especificación

## 1. Resumen

| Caso | Nombre | Actor | En el prototipo | Se verifica con |
|---|---|---|---|---|
| CU-001 | Iniciar sesión | Cualquier usuario | Construido | RF-008, RN-011, RNF-015, RNF-016 |
| CU-002 | Dar de alta un dealer | Admin | Construido | RF-001 |
| CU-003 | Dar de baja un dealer | Admin | Construido | RF-001, RF-014, RN-008, RN-011 |
| CU-004 | Crear un usuario Admin | Admin | Construido | RF-002, RN-020 |
| CU-005 a CU-009 | Empleados y captura manual | — | Retirados | — |
| CU-010 | Ver el dashboard | Admin, Dealer | Construido; agrupado y filtro planeados | RF-007, RN-002, RN-003, RN-012 |
| CU-011 | Editar o reactivar un dealer | Admin | Construido | RF-001, RN-011 |
| CU-012 | Listar dealers | Admin | Construido | RF-001 |
| CU-013 | Crear un usuario Dealer | Admin | Construido | RF-009, RN-020 |
| CU-014 | Cargar archivo de órdenes | Dealer | Parcial: el prototipo lee un único formato fijo en el código | RF-010, RF-018, RN-013, RN-015, RN-016, RN-017, RN-021, RN-028, RN-029 |
| CU-015 | Consultar el historial de cargas | Admin, Dealer | Construido; paginación planeada | RF-011, RN-012 |
| CU-016 | Consultar órdenes | Admin, Dealer | Construido | RF-012, RN-012 |
| CU-017 | Ver el Panel del Distribuidor | Admin, Dealer | Construido | RF-013, MERP-109 |
| CU-018 | Gestionar usuarios | Por decidir | No construido [DA-04] | RF-015 |
| CU-019 | Cerrar sesión y cambiar la contraseña | Cualquier usuario | No construido [DA-05] | RF-016 |
| CU-020 | Configurar el perfil de importación | Dealer | No construido [DA-21] | RF-018, RN-022, RN-024, RN-025 |
| CU-021 | Editar el perfil de importación | Dealer | No construido | RF-018, RN-025, RN-026, RN-027 |
| CU-022 | Consultar el perfil de importación y su historial | Dealer, Admin | No construido | RF-018, RN-024, RN-027 |

**Condición común.** Todos los casos salvo CU-001 requieren una sesión válida. En cada petición el sistema vuelve a comprobar que el usuario sigue activo y que su dealer sigue activo (RN-011).

## 2. Casos vigentes

### CU-001 — Iniciar sesión
**Flujo**
1. El usuario escribe su correo y su contraseña.
2. El sistema busca el correo sin distinguir mayúsculas.
3. Comprueba que el usuario esté activo y, si pertenece a un dealer, que el dealer esté activo.
4. Verifica la contraseña.
5. Abre una sesión que identifica el rol y el dealer del usuario.

**Alternativas:** cualquier fallo de los pasos 2 a 4 produce la misma respuesta, sin indicar cuál fue, y sin que el tiempo de respuesta lo delate (RNF-016). Superado el límite de intentos, se rechaza temporalmente (RNF-015).

### CU-002 — Dar de alta un dealer
**Flujo:** el Admin escribe el nombre → el sistema crea el dealer activo.
**Alternativas:** nombre vacío → rechazo. Nombre repetido → según DA-11.

### CU-003 — Dar de baja un dealer
**Flujo:** el Admin elige un dealer → el sistema lo marca inactivo.
**Efectos**
- Sus órdenes y cargas se conservan; el Admin las sigue viendo.
- Sus usuarios pierden el acceso mientras dure la baja, incluidas las sesiones abiertas. Sus registros no cambian.
- No se pueden crear usuarios nuevos para ese dealer.

**Alternativas:** dealer inexistente → "no encontrado".

### CU-004 — Crear un usuario Admin
**Flujo:** el Admin escribe nombre, correo y contraseña → el sistema crea el usuario con rol Admin, sin dealer.
**Alternativas:** correo ya registrado → "duplicado". Contraseña que no cumple la política → rechazo.

### CU-010 — Ver el dashboard
**Flujo**
1. El sistema reúne las órdenes del alcance del usuario: todas para el Admin, las de su dealer para el Dealer.
2. Calcula para cada una los días transcurridos y el color.
3. Las presenta ordenadas por urgencia, con estadísticas y tendencia.

**Alternativas:** sin órdenes → la pantalla lo explica y, para el Dealer, enlaza a la carga.
**Planeado [DA-07]:** el Admin agrupa por distribuidor y filtra por dealer.

### CU-011 — Editar o reactivar un dealer
**Flujo:** el Admin cambia el nombre de un dealer, su estado activo, o ambos.
**Efectos:** desactivarlo equivale a CU-003; reactivarlo devuelve el acceso a sus usuarios activos.

### CU-012 — Listar dealers
**Flujo:** el sistema devuelve todos los dealers, activos e inactivos, ordenados por nombre.

### CU-013 — Crear un usuario Dealer
**Flujo**
1. El Admin elige el dealer y escribe nombre, correo y contraseña.
2. El sistema comprueba que el dealer exista y esté activo y, después, que el correo no esté registrado.
3. Crea el usuario con rol Dealer; el rol lo fija el sistema.

**Alternativas:** dealer inexistente → "no encontrado"; dealer dado de baja → rechazo por regla de negocio; correo ya registrado → "duplicado".

### CU-014 — Cargar archivo de órdenes
Es el **único** caso de uso de carga. No hay un caso de uso ni código distinto por distribuidor: lo que cambia de un distribuidor a otro es su **perfil de importación**, que es un dato (MERP-104).

**Precondición:** el usuario es Dealer, tiene dealer asignado y su dealer tiene un perfil de importación vigente.
**Flujo**
1. El Dealer arrastra o selecciona el archivo.
2. El sistema valida el archivo: tipo y tamaño.
3. Toma la versión vigente del perfil del dealer del usuario.
4. Localiza la hoja y la fila de encabezados que indica el perfil y comprueba que estén todas las columnas que el perfil espera.
5. Interpreta cada fila según el perfil: mapeo de columnas a campos, formato de fechas y equivalencias de valores. Aparta las filas inválidas con su motivo.
6. Para cada fila válida, crea la orden o actualiza la existente, siempre en el dealer del usuario.
7. Registra la carga, **ligada a la versión del perfil con la que se leyó**, y devuelve el resumen: totales, nuevas, actualizadas, sin cambios y rechazadas, con el detalle de los rechazos y la indicación de si el perfil cambió respecto a la carga anterior.

**Alternativas**
- El dealer no tiene perfil → no se carga; se dirige al Dealer a CU-020.
- Falta en el archivo una columna que el perfil espera → se rechaza el archivo completo, diciendo qué columnas faltan, sin registrar la carga (RN-028).
- Archivo no válido → se rechaza completo, con el motivo, sin registrar la carga.
- Usuario Admin → rechazo por permisos.

**Detalle:** MERP-112.

### CU-015 — Consultar el historial de cargas
**Flujo:** el sistema devuelve las cargas de la más reciente a la más antigua, con el dealer, quién la subió, los contadores y las filas rechazadas. El Dealer ve las suyas; el Admin, todas, y puede limitarlas a un dealer.
**Planeado [DA-06]:** paginación y filtro por dealer en pantalla.

### CU-016 — Consultar órdenes
**Flujo:** el sistema devuelve la lista de órdenes del alcance del usuario, o una orden por su identificador, sin los datos del cliente final.
**Alternativas:** orden inexistente o de otro dealer → la misma respuesta "no encontrado".

### CU-017 — Ver el Panel del Distribuidor
**Flujo**
1. El usuario elige el periodo: últimos 7 o últimos 30 días. El Admin elige además el dealer.
2. El sistema calcula las cuatro secciones (MERP-109).

**Alternativas:** Admin sin dealer elegido → rechazo por dato faltante; Dealer que pide otro dealer → rechazo por permisos; Admin que pide un dealer inexistente → "no encontrado". El Admin puede consultar un dealer dado de baja.

## 3. Casos no construidos

### CU-018 — Gestionar usuarios · [DA-04]
El sistema debe permitir ver los usuarios existentes y retirar el acceso a una persona sin dar de baja a su dealer. Quién puede hacerlo, y si además se podrá editar y reactivar, se define en DA-04. La baja es lógica (RN-008).

### CU-019 — Cerrar sesión y cambiar la contraseña · [DA-05]
El usuario debe poder terminar su sesión y cambiar su contraseña. La duración de la sesión, la política de contraseñas y la recuperación se definen en DA-05.

### CU-020 — Configurar el perfil de importación · [DA-21]
**Actor:** Dealer, solo para su dealer.
**Cuándo:** en la primera carga, o cuando el dealer aún no tiene perfil.
**Flujo (asistente)**
1. El Dealer sube un archivo de muestra.
2. El sistema muestra las hojas y deja indicar la hoja y la fila de encabezados; propone la que detecta.
3. Muestra las columnas encontradas y deja asignar cada una a un campo de MERP, o dejarla sin asignar.
4. El Dealer indica el formato de las fechas y, donde aplique, las equivalencias de valores (por ejemplo, qué valores de una columna de estado significan abierta y cuáles cerrada).
5. El sistema muestra una **vista previa** con filas del archivo ya interpretadas según lo configurado.
6. El Dealer confirma y el perfil se guarda como versión 1.

**Alternativas**
- Falta asignar un campo obligatorio → no se puede guardar (RN-022).
- Una columna asignada como fecha, número o importe no se puede leer como tal en la muestra → no se puede guardar y se indica cuál (RN-025).
- El Dealer abandona el asistente → no se guarda nada.

**Estrategia.** El camino principal es una plantilla única de Magma, cuyo perfil viene dado; el asistente es el respaldo para quien no pueda usarla. En el piloto, con un solo distribuidor, el perfil se configura a mano. Cuándo se construye el asistente se decide en DA-21; el modelo de datos lo deja previsto desde el inicio.

### CU-021 — Editar el perfil de importación
**Actor:** Dealer, solo el de su dealer.
**Flujo:** el mismo del asistente, partiendo de la versión vigente, con un archivo de muestra y vista previa obligatoria. Al confirmar se crea una **versión nueva**; la anterior no se modifica.
**Efectos**
- Queda registrado quién lo cambió, cuándo y qué cambió (RN-027).
- La versión nueva se aplica solo a las cargas posteriores. Las cargas y las órdenes anteriores no se reinterpretan (RN-026).
- La siguiente carga avisa en su resumen de que el perfil cambió (RN-029).

**Abierto [DA-20]:** si el Admin puede revertir un perfil a una versión anterior.

### CU-022 — Consultar el perfil de importación y su historial
**Actores:** el Dealer, el de su dealer; el Admin, el de cualquier dealer, **solo lectura**.
**Flujo:** el sistema muestra la versión vigente (hoja, fila de encabezados, mapeo, formato de fechas, equivalencias), la lista de versiones con su autor y fecha, y qué cambió en cada una. Desde una carga del historial se puede ver la versión con la que se leyó.
**Alternativas:** un Dealer que pide el perfil de otro dealer recibe la misma respuesta que si no existiera (RN-012).

## 4. Casos retirados

| Caso | Descripción original | Motivo |
|---|---|---|
| CU-005 | El Dealer da de alta un empleado. | Se eliminó el rol Empleado. Ver CU-013. |
| CU-006 | El Dealer da de baja un empleado. | Se eliminó el rol Empleado. Ver CU-018. |
| CU-007 | El Empleado registra la entrada de un camión. | Sustituido por CU-014. |
| CU-008 | El Empleado registra el trabajo realizado. | Sustituido por CU-014. |
| CU-009 | El Empleado registra la salida de un camión. | Sustituido por CU-014. |

---

# Parte B — Prototipo

Rutas con prefijo `/api/v1`. Casos de uso en `backend/app/application/use_cases/`.

| Caso | Archivo | Ruta | Pantalla |
|---|---|---|---|
| CU-001 | `authenticate_user.py` | `POST /auth/login` | `/login` |
| CU-002 | `create_dealer.py` | `POST /dealers` | `/dealers` |
| CU-003 | `deactivate_dealer.py` | `DELETE /dealers/{id}` | `/dealers` |
| CU-004 | `create_admin.py` | `POST /users/admins` | `/dealers` |
| CU-010 | `get_dashboard.py` | `GET /dashboard` | `/` |
| CU-011 | `update_dealer.py` | `PATCH /dealers/{id}` | `/dealers` |
| CU-012 | `list_dealers.py` | `GET /dealers` | `/dealers`, `/panel` |
| CU-013 | `create_dealer_user.py` | `POST /users/dealers` | `/dealers` |
| CU-014 | `process_upload.py` | `POST /uploads` | `/cargas` |
| CU-015 | `list_uploads.py` | `GET /uploads` | `/cargas` |
| CU-016 | `list_trucks.py`, `get_truck.py` | `GET /trucks`, `GET /trucks/{id}` | Ninguna propia |
| CU-017 | `get_panel_distribuidor.py` | `GET /panel` | `/panel` |

Particularidades del prototipo:
- CU-001: no hay límite de intentos y el tiempo de respuesta distingue los correos existentes (MERP-114, D-03 y D-04).
- CU-002: se aceptan dos dealers con el mismo nombre.
- CU-010: el orden, las estadísticas y la tendencia se calculan en el navegador.
- CU-014: tamaño máximo 5 MB; procesamiento en la misma petición. **No hay perfiles de importación**: el formato del primer distribuidor está fijo en el código del lector (nombres de columna, formatos de fecha y la equivalencia `mant` → mantenimiento).
- CU-020, CU-021 y CU-022: no existe nada equivalente.
- CU-015: la API acepta el filtro por dealer para el Admin; la pantalla no lo ofrece.
- CU-016: las rutas y los nombres internos conservan "truck".
- CU-019: cerrar sesión solo borra los datos guardados en el navegador.
