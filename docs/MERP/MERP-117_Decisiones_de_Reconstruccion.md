# MERP-117 — Decisiones de reconstrucción
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> Si la junta de Magma aprueba el prototipo, MERP se reconstruye desde cero. Este documento fija qué se conserva del prototipo, qué cambia, qué se descarta y por qué, y reúne las **decisiones abiertas que deben cerrarse antes de construir**. Es el registro único de decisiones de la versión nueva.

---

## 1. Criterio general

El prototipo demostró que el producto funciona: 245 pruebas en verde y un flujo completo de carga, semáforo y Panel. Lo que no demostró es que pueda operar con datos reales en internet. Por eso:

- **Se conserva lo que es conocimiento del negocio**: reglas, modelo de datos, cálculos y las pruebas que los describen.
- **Se cambia lo que era aceptable solo en un prototipo**: seguridad, despliegue, nombres heredados y lógica repartida.
- **Se descarta lo que es historia**: restos de la versión de captura manual y datos de prueba.

## 2. Qué se conserva

| Elemento | Por qué | Dónde está especificado |
|---|---|---|
| **Arquitectura por capas** con dependencias hacia el dominio | Permitió cambiar la captura manual por la carga de Excel sin rehacer el dominio ni el semáforo | MERP-103 |
| **Modelo de datos**: dealer, usuario, orden, carga | Refleja el archivo real del distribuidor y ya resolvió la identidad de la orden | MERP-104, MERP-107 |
| **Reglas de negocio** RN-001 a RN-020 | Son el contenido del producto; casi todas tienen pruebas | MERP-102 |
| **Dos roles**: Admin y Dealer | El tercer rol se eliminó porque no aportaba al flujo de carga | MERP-100 §4 |
| **Carga como foto**: la más reciente manda; rechazo por fila | Es como trabaja el distribuidor: exporta y sube | MERP-112 |
| **El dealer de la orden sale del usuario**, nunca del archivo | Es la base del aislamiento entre distribuidores | RN-013 |
| **Baja lógica** y **corte de acceso por regla**, sin modificar usuarios | Reversible y sin escrituras en cascada | RN-008, RN-011 |
| **Cálculo en el servidor** como funciones puras, con el reloj inyectable | Hizo el Panel comprobable con 29 pruebas unitarias | MERP-103, P-2, P-3, P-6 |
| **Panel del Distribuidor** completo | Especificado, construido y probado | MERP-109 |
| **Importes en decimal exacto** | Evita errores de redondeo en las sumas | MERP-103, P-10 |
| **Las 245 pruebas**, como descripción del comportamiento esperado | Son los criterios de aceptación ya escritos | MERP-101, 102, 109, 112 |
| **Lector de Excel tolerante** | Cada tolerancia responde a un caso visto en el archivo real | MERP-112 §4 |
| **Excel real fuera del repositorio** y copia anonimizada para pruebas | Protege datos personales | RNF-006 |
| **Método de trabajo**: plan por fase, revisión de arquitectura antes de construir, revisión de seguridad al final | Encontró a tiempo el hueco de la baja de dealer | MERP-115 |

## 3. Qué cambia

| Elemento | En el prototipo | En la versión nueva | Por qué |
|---|---|---|---|
| Seguridad | CORS abierto, sin HTTPS, sin límite de intentos | Requisitos desde el primer día | El prototipo era local; la versión nueva recibe datos reales (RNF-009 a RNF-016) |
| Despliegue | No existe; un único arranque de desarrollo | Remoto, integración y despliegue continuos, arranque de producción | Casi todos los incidentes del prototipo vinieron de no probar el sistema completo hasta tarde (MERP-110, MERP-115) |
| Configuración | Tres variables leídas al usarse; dos sin uso; CORS en el código | Todo por variables, validadas al arrancar | Poder cambiar de entorno sin tocar código (RNF-024) |
| Nombres | `trucks`, "camiones" | "Órdenes" en rutas, respuestas, mensajes y pantallas | El dominio cambió y los nombres no (RNF-022) |
| Lógica del dashboard | Orden, promedio, estadísticas y tendencia en el navegador, sin pruebas | En el servidor, con pruebas | Una regla en dos lugares acaba divergiendo (MERP-103, P-3) |
| Formato de errores | Dos formatos | Uno | El frontend no debe distinguir tipos de error por su forma (RNF-017) |
| Errores internos conocidos | Dos casos responden error 500 | Ninguno | MERP-106 §2 |
| Unicidad del correo | Solo en la aplicación | También en la base | Una invariante debe estar en los dos sitios (MERP-104 §3) |
| Trazabilidad | Ninguna tabla guarda fechas de cambio | Fecha de creación y de modificación | Saber cuándo cambió un dato (RNF-019) |
| Pruebas | Solo backend, contra la base de desarrollo | Backend y frontend, con base exclusiva | RNF-020 |
| Datos opcionales ilegibles | Se guardan vacíos en silencio | Nunca en silencio | Un dato perdido sin aviso produce métricas engañosas (RF-017) |
| Gestión de usuarios | Solo alta | Poder retirar el acceso a una persona | RN-008 no estaba implementada para usuarios (RF-015) |
| Lectura del archivo | Un único formato, fijo en el código del lector | **Perfil de importación por distribuidor**, guardado como dato, con un solo caso de uso de carga | Cada distribuidor exporta con otro formato; no se escribirá código por distribuidor (§4 bis, RF-018) |
| Migraciones | Cuatro, con el historial de la versión 1 | Una inicial limpia | No arrastrar tablas y roles que ya no existen |
| Construcción | Versiones mínimas, una herramienta sin fijar, Node distinto en imagen y equipo | Versiones fijadas e idénticas | RNF-021 |
| Documentación | Dos juegos; README obsoleto | Una ubicación; se actualiza en cada fase | MERP-115, RC-18 y RC-19 |
| Registro de decisiones | Repartido entre README, planes y código | Este documento | MERP-115, RC-23 |

## 4. Qué se descarta

| Elemento | Por qué |
|---|---|
| Captura manual de entrada, trabajo y salida | Sustituida por la carga de Excel; ya retirada del prototipo |
| Rol Empleado | Ya retirado; no se reintroduce |
| Entidades y tablas `Truck` y `WorkRecord` | Ya eliminadas |
| Documentos MERP-100 a 107 de la raíz del repositorio | Versión 1, obsoleta |
| MERP-108 y MERP-111 como fuente vigente | Quedan como registro histórico |
| Las 27 capturas de pantalla actuales | Muestran pantallas que ya no existen |
| Datos de la base de desarrollo | Son de prueba; la versión nueva parte de una base vacía (DA-14) |
| La columna `THYP` como origen del tipo de ingreso | Vale 0 en todo el archivo real |
| El supuesto de "$0 el primer año" en AWS | Ya no aplica a cuentas nuevas (MERP-110 §6) |
| ADR-003, autenticación de prototipo | Sustituida por los requisitos de seguridad |

## 4 bis. Perfiles de importación por distribuidor

**Origen.** Requisito añadido el 8 de octubre de 2026 por el dueño del proyecto, al conocerse el archivo de un segundo distribuidor: un reporte `.xls` de Crystal Reports con los encabezados en la fila 7, columnas `Orden`, `Serie`, `FechaOrd`, `FechaFsct`, `Status`, `Mano Obra`, `Refacciones`, `TOTAL` y `Días`, y sin columna de promesa. El prototipo no podría cargarlo: tiene el formato del primer distribuidor fijo en el código.

**Decisión.** No se escribe código ni un caso de uso por distribuidor. Hay un único caso de uso, "Cargar archivo de órdenes" (CU-014), que aplica el perfil de importación del dealer que sube. El perfil es un dato.

| Aspecto | Decidido |
|---|---|
| Qué contiene el perfil | Hoja y fila de encabezados; mapeo de columnas a campos canónicos; formato de fechas; equivalencias de valores |
| Contrato mínimo | Número de orden, VIN y fecha de entrada. El resto, opcional. La lista definitiva la fija Magma [DA-18] |
| Campo opcional ausente | Su métrica aparece como "no disponible" para ese dealer |
| Quién lo configura | El propio distribuidor, solo para su dealer |
| Qué ve el Admin | El perfil y su historial, en solo lectura |
| Controles obligatorios | Vista previa antes de guardar; no guardar sin los obligatorios; validación de tipos; registro de cambios; versionado con cada carga ligada a su versión; rechazo del archivo si falta una columna esperada; aviso en el resumen si el perfil cambió; los cambios no reescriben cargas anteriores |
| Estrategia | Plantilla única de Magma como camino principal; perfil configurable como respaldo |
| Piloto | Un perfil configurado a mano. El asistente puede esperar al segundo distribuidor [DA-21], pero el diseño lo deja previsto desde el inicio |

**Por qué como dato y no como código.** Con código por distribuidor, cada alta exige desarrollo, pruebas y despliegue, y el número de variantes crece con el de distribuidores. Con un perfil, el alta es configuración y queda registrada.

**Por qué tantos controles.** Un mapeo incorrecto no produce un error: produce números equivocados. Asignar una fecha de facturación como fecha de salida se carga sin fallos y desvía el semáforo, el cumplimiento y los tiempos de cierre. Como el sistema no puede saber si una fecha es la correcta, solo si es una fecha, la defensa es que una persona la vea antes de guardar (vista previa), que todo cambio quede a la vista (registro, versión, aviso en el resumen) y que Magma pueda revisarlo (solo lectura).

**Qué queda previsto desde el primer día, aunque el asistente llegue después.**
1. El caso de uso de carga recibe el perfil; no conoce nombres de columna.
2. Las tablas de perfil y de versiones existen, y cada carga guarda su versión.
3. El perfil del primer distribuidor se carga como dato, no se escribe en el código.
4. Las métricas distinguen "no disponible" de "sin datos".
5. La vista de solo lectura del perfil para el Admin.

**Especificado en:** MERP-101 (RF-018), MERP-102 (RN-021 a RN-029), MERP-104 (perfil, versión y campos canónicos), MERP-105 (CU-014, CU-020 a CU-022), MERP-107 (tablas y B-9 a B-12), MERP-112 (§8 bis).
**Pendiente de propagar** cuando se cierren las decisiones: el "no disponible" en MERP-109 y en el contrato de MERP-106, y las operaciones de perfil en MERP-106.

## 5. Decisiones abiertas

**Ninguna fase de la reconstrucción puede empezar si depende de una decisión sin cerrar** (MERP-115, RC-26). Para cada una se da la pregunta, las opciones, una recomendación con su porqué y lo que bloquea.

| ID | Tema | Quién decide | Bloquea |
|---|---|---|---|
| DA-01 | Cómo cuenta los días el semáforo | Dueño del proyecto | Dashboard |
| DA-02 | Significado de `diag`, `repme` y `THYP` | Magma | Clasificación; mezcla del Panel |
| DA-03 | Datos del cliente final y retención | Magma | Modelo de datos; carga |
| DA-04 | Gestión de usuarios | Dueño del proyecto | Usuarios; pantallas de administración |
| DA-05 | Sesión y contraseñas | Dueño del proyecto | Autenticación |
| DA-06 | Paginación e historial | Dueño del proyecto | Historial de cargas |
| DA-07 | Filtro y agrupado del dashboard | Dueño del proyecto | Dashboard |
| DA-08 | Órdenes que dejan de venir en el archivo | Dueño del proyecto, con el distribuidor | Carga; dashboard; Panel |
| DA-09 | Validación del Excel: rechazar, avisar o aceptar | Dueño del proyecto | Carga |
| DA-10 | VIN: normalización y cambio entre cargas | Dueño del proyecto | Carga |
| DA-11 | Nombre de dealer único | Dueño del proyecto | Dealers |
| DA-12 | Fechas y zona horaria | Dueño del proyecto | Modelo de datos; despliegue |
| DA-13 | Guardar o no el archivo original | Magma | Carga; despliegue |
| DA-14 | Punto de partida: stack, repositorio, datos y cierre del prototipo | Dueño del proyecto | Todo |
| DA-15 | Dominio y HTTPS | Magma | Despliegue |
| DA-16 | Bitácora de acciones | Dueño del proyecto | Modelo de datos |
| DA-17 | Alcance diferido | Magma | Nada: confirma lo que queda fuera |
| DA-18 | Campos obligatorios definitivos del perfil | Magma | Perfiles; modelo de datos; Panel |
| DA-19 | Formatos de archivo aceptados | Dueño del proyecto | Lector; perfiles |
| DA-20 | Si el Admin puede revertir un perfil | Dueño del proyecto | Perfiles; permisos |
| DA-21 | Asistente de mapeo: en el piloto o después | Dueño del proyecto | Planificación; pantallas |
| DA-22 | Significado de `FechaFsct`, `Status` y `Tipo` en el archivo del segundo distribuidor | El segundo distribuidor | Su perfil |

### DA-01 — Cómo cuenta los días el semáforo

**Pregunta.** ¿Los días son de calendario o periodos de 24 horas?

| Opción | Efecto | Ejemplo: entra lunes 18:00, se consulta martes 09:00 |
|---|---|---|
| A. Periodos de 24 horas (prototipo) | Una orden cambia de color a la hora exacta en que entró | 0 días |
| B. Días de calendario | Una orden cambia de color al cambiar la fecha | 1 día |

**Recomendación: B.** Es como se redactó la regla original ("días naturales") y como lo lee una persona: "entró ayer" es un día. Con cargas dos veces por semana, la precisión de horas no aporta. Efecto: las órdenes pasan a amarillo y a rojo hasta casi un día antes que hoy.
**También hay que confirmar:** los umbrales 4 y 7, y si deben ser configurables.

### DA-02 — Significado de `diag`, `repme` y `THYP`

**Preguntas para Magma.**
1. ¿Qué tipo de ingreso son `diag` y `repme`: garantía, mantenimiento u otra categoría?
2. ¿Existe en el archivo alguna forma de distinguir una garantía? Hoy ningún valor la produce.
3. ¿Qué significa `THYP` y por qué vale siempre 0?
4. ¿Pueden aparecer otros valores en `TDS1`?

**Mientras tanto.** Se mantiene "sin clasificar" y se conservan los valores originales. **Recomendación:** construir la clasificación como una tabla de correspondencias editable sin cambiar código, y una operación para reclasificar lo ya cargado (RN-018, criterio 3).

### DA-03 — Datos del cliente final y retención

**Preguntas.**
1. ¿Se guardan nombre, contacto, teléfono y correo del cliente final? Hoy se guardan y nadie los usa.
2. ¿Se guarda el respaldo de las 20 columnas sin uso?
3. ¿Qué se hace si el distribuidor escribe datos personales en los campos de asesor o de tipo de servicio?
4. ¿Cuánto tiempo se conservan las órdenes y las cargas, y qué pasa después?
5. ¿Quién es el responsable de los datos y qué aviso de privacidad aplica? (Área legal de Magma.)

**Recomendación.** No guardar los cuatro datos del cliente final ni el respaldo completo: conservar solo `TDS1` y `THYP`. Es más fácil añadir un dato cuando haya un uso que justificar uno guardado sin uso. La retención la fija Magma; el sistema debe poder aplicarla. Propuesta completa en MERP-116 §5.

### DA-04 — Gestión de usuarios

**Preguntas.**
1. ¿Quién puede ver la lista de usuarios y dar de baja a uno: solo el Admin, o también cada Dealer sobre los usuarios de su dealer?
2. ¿Se puede reactivar a un usuario dado de baja?
3. ¿Se pueden editar el nombre y el correo?
4. ¿Qué pasa con los correos de usuarios dados de baja: quedan ocupados?

**Recomendación.** Solo el Admin: listar, dar de baja y reactivar. Sin edición de correo en la primera versión. Mantiene los dos roles tal como son y cierra el hueco real, que es no poder retirar el acceso a una persona. Los correos de usuarios dados de baja quedan ocupados, porque la baja es lógica.

### DA-05 — Sesión y contraseñas

**Preguntas.**
1. ¿Cuánto dura una sesión y se renueva sola mientras se usa?
2. ¿El cierre de sesión debe invalidar la sesión en el servidor?
3. ¿Dónde guarda el navegador la sesión?
4. ¿Qué política de contraseña se exige?
5. ¿Cómo cambia un usuario su contraseña y cómo la recupera si la olvida?
6. ¿Cuántos intentos fallidos se permiten y por cuánto tiempo se bloquea?
7. ¿Cómo se crea el primer Admin en producción?

**Recomendación.**
- Sesión corta, renovable mientras hay actividad, guardada en una cookie que el código de la página no puede leer. Reduce el daño de un fallo en el frontend; exige añadir protección contra peticiones falsificadas entre sitios.
- Cambio de contraseña por el propio usuario; restablecimiento por el Admin. La recuperación por correo exige un servicio de envío, que hoy está fuera de alcance (DA-17).
- Los valores concretos (minutos, longitud mínima, número de intentos) los fija el dueño del proyecto; el prototipo usa 8 caracteres y la plantilla propone 60 minutos. **[VERIFICAR]** la duración en uso.
- Primer Admin: comando de un solo uso, documentado (MERP-110, D-6).

### DA-06 — Paginación e historial

**Preguntas.**
1. ¿Se pagina el historial de cargas? ¿De cuántas en cuántas?
2. ¿El Admin puede filtrar por dealer en la pantalla?
3. ¿Se paginan también las órdenes y el dashboard cuando crezcan?

**Recomendación.** Paginación en el servidor para el historial, con filtro por dealer para el Admin. Para el dashboard, no paginar: su valor es ver todo lo urgente de un vistazo; limitar en su lugar las órdenes completadas que se muestran.

### DA-07 — Filtro y agrupado del dashboard

**Preguntas.**
1. ¿La vista del Admin se agrupa por distribuidor?
2. ¿En qué orden van los grupos: alfabético, o primero el que tiene más órdenes en rojo?
3. ¿Hay filtro por dealer?

**Recomendación.** Agrupar, con orden alfabético entre grupos y urgencia dentro de cada grupo. El orden por urgencia entre grupos hace saltar bloques enteros cada vez que una orden cambia de color mientras se lee la tabla. Añadir un filtro "todos / un dealer": con muchos distribuidores, el encabezado del grupo se pierde al desplazarse. El orden y los agregados los entrega el servidor, con el identificador del dealer, no el nombre.

### DA-08 — Órdenes que dejan de venir en el archivo

**Situación.** Una orden en proceso que no aparece en cargas posteriores sigue en proceso para siempre: en rojo en el dashboard y como activa en el Panel.

| Opción | Efecto |
|---|---|
| A. No tocarla (prototipo) | Fiel al dato, pero acumula órdenes fantasma |
| B. Marcarla como "no reportada" cuando falta en la última carga de su dealer | Visible y sin inventar un cierre; se puede excluir de las métricas |
| C. Cerrarla automáticamente | Inventa una fecha de salida que el archivo no trae |

**Recomendación: B.** Conserva la regla de no inventar datos y hace visible el problema.
**[VERIFICAR] con el distribuidor:** cómo exporta su sistema; si el reporte incluye siempre todas las órdenes abiertas, la ausencia significa algo; si exporta por rango de fechas, no.

### DA-09 — Validación del Excel: rechazar, avisar o aceptar

Para cada caso que el prototipo acepta en silencio:

| Caso | Recomendación |
|---|---|
| Importe que no es un número | Aceptar la fila con aviso |
| Promesa ilegible | Aceptar con aviso |
| Año que no es entero | Aceptar con aviso |
| Valor de `TDS1` sin correspondencia | Aceptar con aviso, distinguiéndolo de los pendientes de confirmar |
| Promesa anterior a la fecha de alta | Aceptar con aviso |
| Número de orden `0` | Decidir si se rechaza y qué otros valores se consideran inválidos |

**Recomendación general.** Añadir "avisos" al resumen de la carga, junto a los rechazos. Rechazar la fila solo cuando falta lo que identifica a la orden o lo que mueve el semáforo; lo demás se carga y se avisa, para que el distribuidor lo corrija en su sistema.

### DA-10 — VIN: normalización y cambio entre cargas

**Preguntas.** ¿El VIN se guarda en mayúsculas y sin espacios? ¿Puede una carga cambiar el VIN de una orden que ya existe?
**Recomendación.** Normalizar a mayúsculas. Permitir el cambio, con aviso en el resumen: el sistema del distribuidor es la fuente y una corrección legítima debe poder entrar, pero no en silencio.

### DA-11 — Nombre de dealer único

**Recomendación.** Único sin distinguir mayúsculas. Dos dealers con el mismo nombre no se pueden distinguir en ninguna pantalla.

### DA-12 — Fechas y zona horaria

| Opción | Efecto |
|---|---|
| A. Sin zona, en hora local de planta (prototipo) | Simple, pero depende de configurar bien la zona en el servidor, en la base y en la base administrada; un descuido adelanta el semáforo seis horas |
| B. Con zona | Las fechas del archivo se interpretan como hora de planta al cargarlas y se guardan sin ambigüedad |

**Recomendación: B.** El prototipo ya sufrió el error de la opción A, y el despliegue en AWS lo vuelve a exponer. Además prepara el caso de distribuidores en otra zona horaria.

### DA-13 — Guardar o no el archivo original

**Recomendación.** No guardarlo en el piloto. Las órdenes y el registro de la carga bastan para operar, y el archivo es el elemento con más datos personales. Si Magma lo quiere como respaldo: almacenamiento privado, cifrado y con borrado automático (MERP-116).

### DA-14 — Punto de partida

**Preguntas.**
1. ¿Se mantiene el mismo stack (FastAPI, PostgreSQL, React, Docker)?
2. ¿Repositorio nuevo o el actual?
3. ¿Se migra algún dato del prototipo?
4. ¿Qué se hace con el prototipo: se ejecuta la fase M8 o se congela en `5d77e51`?
5. ¿Qué se hace con la base de desarrollo, el README y los documentos de la raíz?

**Recomendación.**
- Mismo stack: el equipo lo conoce y las pruebas existentes se reutilizan casi sin cambios.
- Repositorio nuevo, con remoto desde el primer commit; el actual queda archivado como referencia.
- No migrar datos: todos son de prueba.
- Congelar el prototipo y no ejecutar M8: su alcance (agrupado, README, capturas) se rehace en la versión nueva. Si el prototipo se va a mostrar a la junta, hacer solo lo necesario para la demostración.
- **Acción inmediata, independiente de todo lo demás:** confirmar si la base de desarrollo contiene datos reales (MERP-114, G-01) y, si es así, borrarlos.

### DA-15 — Dominio y HTTPS

**Preguntas.** ¿Con qué nombre de dominio se publica? ¿Quién lo registra y lo paga?
**Recomendación.** Certificado gratuito en el propio servidor durante el piloto (MERP-110 §7).

### DA-16 — Bitácora de acciones

**Pregunta.** Además de las fechas de creación y modificación (RNF-019), ¿se registra quién hizo cada acción administrativa?
**Recomendación.** Una bitácora mínima de altas y bajas de dealers y usuarios. Las cargas ya registran quién las subió.

### DA-17 — Alcance diferido

**Confirmar que quedan fuera de la primera versión:**
- Notificaciones automáticas.
- Integración con el Excel / Power BI interno de Magma.
- Fusión con ATLAS.
- Medir o avisar del cumplimiento del calendario de cargas (miércoles y viernes).

### DA-18 — Campos obligatorios definitivos del perfil

**Pregunta para Magma.** ¿Qué datos debe aportar todo distribuidor para participar?

| Opción | Efecto |
|---|---|
| A. Número de orden, VIN y fecha de entrada (propuesta actual) | Entra cualquier distribuidor, pero uno sin fecha de salida solo tendría semáforo en marcha: sus órdenes nunca se completan y casi todo su Panel queda "no disponible" |
| B. Los tres anteriores más la fecha de salida | Garantiza semáforo completo y las métricas de órdenes cerradas; deja fuera a quien no pueda exportar el cierre |

**Recomendación: B**, o A con la advertencia aceptada de forma expresa. Sin fecha de salida el sistema no puede decir cuánto tardó una orden, que es su propósito. La promesa, el asesor y los importes sí pueden ser opcionales sin vaciar el producto.

### DA-19 — Formatos de archivo aceptados

| Opción | Efecto |
|---|---|
| A. Solo `.xlsx`, encabezados en la primera fila | Lo más simple; obliga al distribuidor a limpiar su reporte |
| B. `.xlsx` con encabezados en cualquier fila | Cubre los reportes con títulos encima. El prototipo ya busca el encabezado en las primeras filas |
| C. También `.xls` | Cubre el reporte del segundo distribuidor tal cual; exige una segunda forma de lectura, con sus pruebas |

**Recomendación: B, y C solo si el segundo distribuidor no puede exportar `.xlsx`.** La fila de encabezados es un dato del perfil y no cuesta casi nada. El `.xls` es un formato antiguo que añade una dependencia y casos propios; conviene preguntar antes si su sistema puede exportar el formato nuevo. Es coherente con la estrategia de plantilla única como camino principal.

### DA-20 — Si el Admin puede revertir un perfil

**Pregunta.** El perfil es del distribuidor. Si lo edita mal y las métricas de Magma se desvían, ¿puede el Admin devolverlo a una versión anterior?
**Recomendación: sí revertir, no editar.** El Admin elige una versión anterior y el sistema crea una versión nueva idéntica, registrada a su nombre y con origen "reversión". El distribuidor conserva la autoría del mapeo y Magma tiene una salida si detecta un error y el distribuidor no responde. Sin esto, la única defensa de Magma sería dar de baja al dealer.

### DA-21 — Asistente de mapeo: en el piloto o después

**Recomendación: después, al llegar el segundo distribuidor.** En el piloto hay un solo distribuidor y su formato ya se conoce; su perfil se carga a mano. Lo que **no** se pospone, porque rehacerlo después es caro: tablas de perfil y versiones, vínculo de cada carga con su versión, el caso de uso de carga leyendo el perfil, el "no disponible" en las métricas y la vista de solo lectura (§4 bis). Con eso, el asistente es solo una pantalla nueva sobre un modelo que ya existe.
**A tener en cuenta:** un perfil cargado a mano se salta la vista previa. En el piloto esa comprobación la hace quien lo configura, cargando el archivo real de muestra y revisando el resultado antes de darlo por bueno.

### DA-22 — Significado de `FechaFsct`, `Status` y `Tipo`

**Preguntas para el segundo distribuidor.**
1. ¿`FechaFsct` es la fecha en que terminó el trabajo en taller, la fecha de factura u otra? ¿Puede estar vacía en una orden abierta?
2. ¿Qué valores toma `Status` y qué significa cada uno? ¿Una orden puede estar cerrada sin `FechaFsct`?
3. ¿Qué es `Tipo` y qué valores toma? ¿Sirve para distinguir garantía de mantenimiento?
4. ¿El reporte incluye siempre todas las órdenes abiertas, o solo las de un rango de fechas? (Afecta también a DA-08.)
5. ¿Su sistema puede exportar en `.xlsx`? (DA-19.)
6. ¿Existe en su sistema la fecha prometida de entrega, aunque no salga en este reporte?

**Mientras tanto.** No se configura su perfil. En particular, **no se asigna `FechaFsct` como fecha de salida** hasta confirmar qué es: es justo el error que el §4 bis describe. **[VERIFICAR]** el archivo no se ha revisado para esta serie; `Tipo` se menciona en las decisiones pero no figura en la lista de columnas recibida.

## 6. Decisiones ya tomadas

| # | Decisión | Fecha u origen |
|---|---|---|
| 1 | La unidad de trabajo es la orden de taller. | Cambio de alcance tras la aprobación de los gerentes |
| 2 | Las órdenes entran por carga de Excel; no hay captura manual. | Ídem |
| 3 | Dos roles: Admin y Dealer. Solo el Dealer sube archivos. | Ídem |
| 4 | "Salida" es la fecha de cierre del trabajo, no la de factura. | Diseño de la carga |
| 5 | El tipo de ingreso sale de `TDS1`; lo no confirmado queda sin clasificar. | Al contrastar con el archivo real |
| 6 | La baja de un dealer corta el acceso por regla, sin modificar a sus usuarios. | M6.1 |
| 7 | El Panel usa una ventana móvil de 7 o 30 días para sus cuatro secciones. | M7 |
| 8 | El desglose de ingresos es sobre mano de obra más refacciones; el total va aparte. | M7 |
| 9 | Despliegue en AWS, cuenta nueva, piloto con un distribuidor, simple y barato. | Diseño del despliegue |
| 10 | La documentación vigente vive en `MERP-documentacion-completa/`. | 8-oct-2026 |
| 11 | Los números de requisito, regla y caso de uso se conservan; los retirados no se reutilizan. | 8-oct-2026 |
| 12 | Si la junta aprueba, MERP se reconstruye desde cero y esta serie es su especificación. | 8-oct-2026 |
| 13 | Seguridad y despliegue son requisitos desde el inicio. | 8-oct-2026 |
| 14 | Un único caso de uso de carga que aplica el perfil de importación del dealer. Sin código ni caso de uso por distribuidor. | 8-oct-2026 |
| 15 | El perfil lo configura el propio distribuidor para su dealer; el Admin lo ve en solo lectura. | 8-oct-2026 |
| 16 | Los cambios de un perfil se versionan, se registran y solo afectan a cargas posteriores; no reescriben las anteriores. | 8-oct-2026 |
| 17 | Si falta una columna que el perfil espera, se rechaza el archivo completo. | 8-oct-2026 |
| 18 | Plantilla única de Magma como camino principal; perfil configurable como respaldo. En el piloto, un perfil configurado a mano. | 8-oct-2026 |

## 7. Condiciones para empezar a construir

1. Aprobación de la junta de Magma.
2. DA-14 cerrada: punto de partida.
3. Cerradas las decisiones que bloquean la primera fase. Como mínimo DA-03, DA-05, DA-12 y DA-18, porque definen el modelo de datos y la autenticación, que son lo primero que se construye.
4. Enviadas a Magma las preguntas de DA-02, DA-03, DA-13, DA-15, DA-17 y DA-18, y al segundo distribuidor las de DA-22.
5. Resuelta la acción inmediata sobre los datos de la base de desarrollo.
