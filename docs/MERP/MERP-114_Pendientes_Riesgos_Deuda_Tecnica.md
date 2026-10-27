# MERP-114 — Pendientes, riesgos y deuda técnica
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.2 · **Fecha:** 26 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> Registro único de lo que el prototipo deja abierto, **clasificado para la reconstrucción**. Cada punto tiene una de tres clasificaciones:
>
> - **Requisito** — la versión nueva debe resolverlo. Se indica el requisito que lo cubre.
> - **Se acepta** — se deja como está, con el motivo.
> - **Decisión** — no se puede construir hasta cerrar la decisión abierta indicada (MERP-117 §5).
>
> Fuente de cada punto: "Código" = comprobado leyendo el repositorio; "Prueba" = observado al ejecutar el prototipo el 8 de octubre de 2026. **[VERIFICAR]** marca un dato sin comprobar del todo.

---

## 1. Resumen

| Clasificación | Puntos |
|---|---|
| Requisito de la versión nueva | 40 |
| Se acepta tal cual | 8 |
| Pendiente de decisión | 34 |
| **Total abierto** | **82** |
| Cerrados (grupo J) | 3 |

## A. Por confirmar con Magma

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| A-01 | Significado de `TDS1 = diag` y `repme`. Hoy quedan "sin clasificar" y ningún valor produce "garantía". *Código.* | Decisión | DA-02 |
| A-02 | Significado de la columna `THYP`, que vale `0` en las 62 filas del ejemplo. *Código.* | Decisión | DA-02 |
| A-03 | `ASESOR` y `TIPO SERV` son texto libre y se muestran tal cual en el Panel; si traen datos personales, quedan a la vista. *Revisión de seguridad de M7.* | Decisión | DA-03 |
| A-04 | Los datos del cliente final (cliente, contacto, teléfono, correo) se guardan y ninguna pantalla los usa. No hay política de retención. *Código.* | Decisión | DA-03 |
| A-05 | Integración con el Excel / Power BI interno de Magma, sin definir. | Decisión | DA-17 |
| A-06 | Qué conceptos incluye `TOTAL` además de mano de obra y refacciones. **[VERIFICAR]** | Se acepta | El Panel ya muestra el total aparte y lo aclara (MERP-109 §5.3). Confirmarlo mejora el texto, no cambia el cálculo |

Los valores originales de `TDS1` y `THYP` se guardan con cada orden: A-01 y A-02 pueden resolverse reclasificando lo ya cargado.

## B. Pedido para el prototipo y no construido

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| B-01 | Agrupar por distribuidor la tabla del dashboard del Admin. Hoy mezcla dealers. *Código; Prueba.* | Decisión | DA-07 |
| B-02 | Filtro por dealer en el dashboard. No existe; el único selector está en el Panel. *Código.* | Decisión | DA-07 |
| B-03 | Paginación del historial de cargas. Hoy se devuelve la lista completa. *Código.* | Decisión | DA-06 |
| B-04 | Fase M8 del prototipo (README, capturas, B-01): no iniciada; solo hay un plan a la espera de cinco decisiones. | Decisión | DA-14 |
| B-05 | Despliegue en AWS: no iniciado. | Requisito | MERP-110, D-1 a D-15 |

## C. Validación de la carga de Excel

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| C-01 | Una fila con `ORDEN = "0"` se acepta como orden. *Prueba.* | Decisión | DA-09 |
| C-02 | Un importe que no es número se guarda vacío sin aviso. *Código; Prueba.* | Requisito | RF-017. La forma (rechazo o aviso) en DA-09 |
| C-03 | Un valor desconocido en `TDS1` queda sin clasificar sin aviso; no se distingue de `diag` o `repme`. *Código; Prueba.* | Requisito | RF-017; DA-09 |
| C-04 | El VIN se guarda como viene, sin pasar a mayúsculas. *Prueba.* | Decisión | DA-10 |
| C-05 | Al actualizar una orden, la carga reemplaza también su VIN. *Código; Prueba.* | Decisión | DA-10 |
| C-06 | El VIN no es único. *Código.* | Se acepta | Es correcto: un vehículo puede tener varias órdenes |
| C-07 | Se acepta una promesa anterior a la fecha de alta; contará como entrega tarde. *Prueba.* | Decisión | DA-09 |
| C-08 | Una promesa ilegible o un año no entero se guardan vacíos sin aviso. *Código.* | Requisito | RF-017; DA-09 |
| C-09 | Las órdenes que dejan de venir en el archivo nunca se cierran: quedan en proceso y en rojo para siempre. *Código.* | Decisión | DA-08 |
| C-10 | El archivo original no se guarda; solo las órdenes y el registro de la carga. *Código.* | Decisión | DA-13 |
| C-11 | El archivo de casos límite existe fuera del repositorio y no se ha cargado de forma definitiva. | Requisito | MERP-113, T-3: incorporarlo como prueba automatizada, sin datos reales |

## D. Seguridad

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| D-01 | CORS abierto y fijo en el código; sin variable de entorno. *Código.* | Requisito | RNF-010 |
| D-02 | Sin HTTPS. | Requisito | RNF-009 |
| D-03 | Sin límite de intentos de inicio de sesión. *Código.* | Requisito | RNF-015 |
| D-04 | Fuga de tiempos en el inicio de sesión: la contraseña solo se verifica si el usuario existe y tiene acceso, porque la comprobación de RN-011 va antes; el tiempo delata qué correos existen. *Revisión de seguridad de M6.1; Código.* | Requisito | RNF-016 |
| D-05 | No se puede listar, editar ni dar de baja usuarios. RN-008 no está implementada para usuarios; tampoco se puede cambiar una contraseña. *Código.* | Requisito | RF-015 y RF-016. El alcance en DA-04 y DA-05 |
| D-06 | Dos altas simultáneas con el mismo correo responden un error interno en vez de "duplicado". *Revisión de seguridad de M6.* | Requisito | RN-020, criterio 2 |
| D-07 | Una sesión bien firmada con identificador de usuario mal formado responde un error interno. Riesgo bajo. *Revisión de seguridad de M6.1; Código.* | Requisito | MERP-106 §2, criterio 3 |
| D-08 | La validación de datos de entrada responde con otro formato de error. *Código.* | Requisito | RNF-017 |
| D-09 | La sesión se guarda en el almacenamiento local del navegador. *Código.* | Decisión | DA-05 |
| D-10 | Ninguna tabla registra cuándo se creó o modificó un dato, ni quién lo hizo. *Código.* | Requisito | RNF-019 para las fechas. La bitácora en DA-16 |
| D-11 | Duración real de la sesión. La plantilla propone 60 minutos. **[VERIFICAR]** el valor en uso. | Decisión | DA-05 |

## E. Modelo de datos y nomenclatura

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| E-01 | Rutas, clave de respuesta, casos de uso y un mensaje conservan "truck" o "camión" aunque el dominio es la orden. *Código.* | Requisito | RNF-022 |
| E-02 | La interfaz sigue diciendo "camiones". *Código.* | Requisito | RNF-022 |
| E-03 | El nombre del dealer no es único; la gráfica por dealer agrupa por nombre. *Código.* | Decisión | DA-11 |
| E-04 | El semáforo cuenta periodos de 24 horas, no días de calendario. *Código.* | Decisión | DA-01 |
| E-05 | El correo es único distinguiendo mayúsculas en la base, pero se busca sin distinguirlas. *Código.* | Requisito | RN-020, criterio 1; MERP-107 §3 |
| E-06 | Las fechas se guardan sin zona horaria. *Código.* | Decisión | DA-12 |

## F. Entorno, construcción y pruebas

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| F-01 | El arranque ignora el fallo del sembrado (`\|\| true`). *Código.* | Requisito | RNF-011; MERP-113 §4 |
| F-02 | El backend arranca con recarga automática. *Código.* | Requisito | RNF-011 |
| F-03 | Una construcción cambió versiones: `sqlalchemy` 2.0.53 → 2.0.52, `uvicorn` 0.53 → 0.52, `greenlet` 3.5.6 → 3.5.5. No rompió nada. **[VERIFICAR]** la causa; se observa que las dependencias se declaran con versión mínima y que una herramienta se instala con la etiqueta `latest`. | Requisito | RNF-021 |
| F-04 | Node 22 en la imagen del frontend y Node 24 en el equipo. *Código.* | Requisito | RNF-021 |
| F-05 | El frontend no tiene pruebas automatizadas. *Código.* | Requisito | RNF-020 |
| F-06 | Las pruebas de integración usan la base de desarrollo. *Código.* | Requisito | RNF-020 |
| F-07 | Verificaciones en navegador pendientes: formulario de alta de dealers y sesión guardada con el antiguo rol empleado. **[VERIFICAR]** si se hicieron. | Se acepta | El prototipo se reconstruye; se sustituyen por pruebas automatizadas (MERP-113, T-4) |
| F-08 | `ENVIRONMENT` y `LOG_LEVEL` están en la plantilla y ningún archivo del backend las lee. *Código.* | Requisito | RNF-024 |
| F-09 | El orden por urgencia, las estadísticas y la tendencia del dashboard se calculan en el navegador, sin pruebas. *Código.* | Requisito | MERP-103, P-3; RNF-020 |
| F-10 | El repositorio no tiene remoto ni integración continua. *Código.* | Requisito | RNF-014 |
| F-11 | El Panel lee todas las órdenes del dealer en cada consulta y filtra en memoria. *Código.* | Se acepta | Suficiente para el volumen del piloto. Revisar al crecer el historial |

## G. Datos de la base de desarrollo

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| G-01 | Dealer MYLSA: 62 órdenes, 1 carga, 1 usuario. **[VERIFICAR]** El número coincide con las 62 filas del archivo real; si se cargó con él, la base de desarrollo contiene datos personales de clientes reales. | Requisito | RNF-023. Si se confirma, hay que borrarlo de inmediato, sin esperar a la reconstrucción |
| G-02 | Dealer PRUEBA (8 órdenes), dealer General (vacío), dos ex-empleados inactivos, una cuenta de desarrollo inactiva y un usuario de prueba de M6 que sigue activo. | Se acepta | Son datos de prueba del prototipo; la versión nueva parte de una base vacía (DA-14) |
| G-03 | Limpieza de la base de desarrollo: no se hizo; falta decidir qué se borra. Borrar un dealer obliga a borrar antes sus usuarios, cargas y órdenes. | Decisión | DA-14 |

## H. Documentación del repositorio

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| H-01 | El README está desactualizado: describe la captura manual, remite a los documentos de la raíz, dice "Fases 0-9", lista "camiones" y cita un archivo que no está en el repositorio. | Decisión | DA-14 |
| H-02 | Hay dos juegos de documentos MERP: los de la raíz (versión 1) y `MERP-documentacion-completa/` (vigente). | Decisión | DA-14 |
| H-03 | Las 27 capturas de `docs-screenshots/` muestran pantallas que ya no existen. | Decisión | DA-14 |
| H-04 | `Lecciones-Aprendidas-MERP.md` y `MERP-Resumen-Ejecutivo.md` solo existen en un cuaderno personal de notas. **[VERIFICAR]** que MERP-115 no omita nada del original. | Se acepta | Las lecciones quedan en MERP-115 |
| H-05 | El plan de M8 propone añadir notas de vigencia a los documentos MERP; si se ejecuta, pisa esta serie. | Decisión | DA-14 |

## I. Futuro

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| I-01 | Seguridad de nivel producción. | Requisito | RNF-009 a RNF-018; MERP-116 |
| I-02 | Fusión con ATLAS. Sin diseño. | Decisión | DA-17 |
| I-03 | Notificaciones automáticas. Sin diseño. | Decisión | DA-17 |
| I-04 | Medir o avisar del cumplimiento del calendario de cargas (miércoles y viernes). Hoy es solo una convención. | Decisión | DA-17 |
| I-05 | Varios distribuidores. | Se acepta | El modelo ya separa por dealer; el dimensionamiento está en MERP-110 §8 |

## K. Perfiles de importación por distribuidor

Requisito añadido el 8 de octubre de 2026 (MERP-117 §4 bis).

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| K-01 | El prototipo tiene el formato del primer distribuidor fijo en el código del lector y solo acepta `.xlsx`. El archivo de un segundo distribuidor no se podría cargar. *Código.* | Requisito | RF-018; RN-021 |
| K-02 | Campos obligatorios definitivos del perfil. Propuesta: número de orden, VIN y fecha de entrada. Sin fecha de salida, las órdenes de ese dealer nunca se completan. | Decisión | DA-18 |
| K-03 | Si se aceptan `.xls` y reportes con encabezados en otra fila, o solo `.xlsx` limpio. | Decisión | DA-19 |
| K-04 | Si el Admin puede revertir un perfil editado por el distribuidor. | Decisión | DA-20 |
| K-05 | Si el asistente de mapeo entra en el piloto o después. | Decisión | DA-21 |
| K-06 | Qué significan `FechaFsct`, `Status` y `Tipo` en el archivo del segundo distribuidor. Pendiente de preguntarle. | Decisión | DA-22 |
| K-07 | **Riesgo: un mapeo incorrecto no falla, calcula mal.** Tomar una fecha de facturación como fecha de salida se carga sin errores y desvía el semáforo y las métricas. | Requisito | RN-025 (vista previa y tipos), RN-026 (versionado), RN-027 (registro), RN-029 (aviso), RN-024 (visibilidad del Admin) |
| K-08 | Corregir un perfil no corrige las órdenes ya cargadas con la versión equivocada: conservan los datos erróneos hasta que una carga posterior las traiga de nuevo, y las que ya no vengan no se corrigen. | Se acepta | Es consecuencia de la decisión de que un cambio de perfil no reescribe cargas anteriores (RN-026). Mitigación: volver a subir el archivo tras corregir |
| K-09 | El archivo del segundo distribuidor no se ha revisado para esta serie; su formato se describe según lo indicado. **[VERIFICAR]** | Requisito | MERP-115, RC-14: contrastar con el archivo real antes de configurar su perfil |
| K-10 | Una columna de estado del archivo (abierta o cerrada) choca con que MERP deriva el estado de la fecha de salida: una orden "cerrada" sin fecha de salida no se puede representar. | Decisión | DA-22; MERP-104, estado reportado |
| K-11 | El "no disponible" de RN-023 aún no está reflejado en la especificación del Panel ni en el contrato de la API. | Requisito | RN-023. Propagar a MERP-109 y MERP-106 al cerrar DA-18 |

## L. Prueba de carga del prototipo

Prueba ejecutada el 26 de octubre de 2026 en un entorno aislado, con el backend limitado a 2 CPU y 4 GB y la base a 1 CPU y 2 GB. Resultados parciales; el informe completo queda en `loadtest/RESULTADOS.md` del repositorio del prototipo.

| ID | Punto | Clasificación | Destino |
|---|---|---|---|
| L-01 | El backend corre con un solo proceso: llega a un núcleo completo y no usa el segundo aunque tenga 2 CPU. *Prueba.* | Requisito | RNF-027 |
| L-02 | El inicio de sesión tarda 1.1 s por Argon2, con solo 3 usuarios. *Prueba.* | Requisito | RNF-028 |
| L-03 | Un archivo de 2,000 filas tarda 5.1 s si es nuevo y 8.3 s si actualiza todo. El archivo real tiene unas 62 filas y tarda 0.2 a 0.3 s. *Prueba.* | Requisito | RNF-026. Si no se cumple en la versión nueva, revisar ADR-005 |
| L-04 | El equipo de la prueba tiene 4.8 GB de memoria para Docker y los límites suman 6 GB: la medición de memoria puede no ser confiable. *Prueba.* | Requisito | RNF-029, regla 2 |
| L-05 | El porcentaje de CPU sobre el límite del contenedor oculta la saturación de un solo núcleo. *Prueba.* | Requisito | RNF-029, regla 1 |
| L-06 | La prueba llegó a 20 distribuidores y 50 usuarios; el dato real es 25 distribuidores con crecimiento. Falta una segunda corrida con 25 y 50 distribuidores y 100 usuarios. | Requisito | RNF-025 |
| L-07 | La base de desarrollo no estaba en el estado del sembrado: 3 dealers, 5 usuarios, 105 órdenes y 5 cargas, frente a 2, 4, 47 y 2. Hay un dealer de origen desconocido con 58 órdenes y 3 cargas. **[VERIFICAR]** qué es y que no contenga datos reales. | Requisito | RNF-023; MERP-115, RC-31 |
| L-08 | No se pudo medir la generación de PDF ni el envío de correos: no existen en el prototipo. | Requisito | RNF-029: incluirlos en la prueba de la versión nueva |

## J. Cerrados

Se conservan para que no se reabran. La versión nueva debe mantenerlos cerrados.

| ID | Punto | Cómo se cerró | Regla que lo mantiene |
|---|---|---|---|
| J-01 | Al dar de baja un dealer, sus usuarios seguían entrando. | Commit `2df3629`. Probado el 8-oct-2026: tras la baja, inicio de sesión y sesiones abiertas se rechazan; al reactivar, vuelven. | RN-011 |
| J-02 | Un usuario no Admin sin dealer asignado podía operar. | Rechazo explícito por permisos (M6). | RN-014 |
| J-03 | Un dealer veía un instante los datos del usuario anterior en el mismo navegador. | Los datos en memoria del navegador se borran al iniciar y al cerrar sesión (M7). | MERP-103 §5 |
