# MERP-115 — Lecciones aprendidas y restricciones de construcción
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.1 · **Fecha:** 26 de octubre de 2026 · **Estado:** vigente

> Cada lección del prototipo se convierte aquí en una **restricción de construcción**: una condición que la versión nueva debe cumplir desde el primer día, con la forma de comprobarla. Las lecciones 1 a 17 provienen del traspaso del proyecto; las 18 a 20 se añadieron al contrastar los documentos con el código. Es el único documento de la serie que describe el trabajo con el equipo de agentes (§4).

---

## 1. Restricciones de construcción

### 1.1 Entorno

| ID | Restricción | Cómo se comprueba | Lección de origen |
|---|---|---|---|
| RC-01 | Antes de escribir funcionalidad, el sistema completo (base, backend, frontend) debe levantarse desde cero con un comando y pasar una verificación de salud. Esa comprobación se repite en cada cambio. | Paso de la integración continua que levanta todo en limpio | 11, 17 |
| RC-02 | Las migraciones se aplican solas al arrancar en desarrollo, y se prueban desde una base vacía. | Ídem | 11 |
| RC-03 | Los puertos del equipo se eligen para no chocar con otros proyectos y se documentan. La API se direcciona con `127.0.0.1` explícito, no con `localhost`. | Tabla de puertos en MERP-113; revisión de la plantilla de configuración | 1, 5 |
| RC-04 | La carpeta de trabajo y la ubicación del repositorio se confirman antes de crear el proyecto. | Primera línea del plan de la primera fase | 6 |
| RC-05 | El montaje del código en los contenedores no puede tapar las dependencias instaladas dentro del contenedor. | RC-01 ejecutado en Windows | 9 |
| RC-06 | La recarga del frontend en desarrollo debe funcionar al montar carpetas desde Windows. | Un cambio en un archivo se refleja sin reiniciar | 10 |
| RC-07 | Servidor y base de datos corren en la zona horaria de la planta en **todos** los entornos, incluida la base administrada de producción. | Prueba que registra una carga y compara la hora; MERP-110, D-10 | 12 |
| RC-08 | El archivo principal de la aplicación y la ruta que usa la imagen coinciden desde el primer commit. | RC-01 | 3 |

### 1.2 Base de datos y backend

| ID | Restricción | Cómo se comprueba | Lección de origen |
|---|---|---|---|
| RC-09 | Toda migración que crea un tipo enumerado lo crea y lo elimina de forma explícita, y se prueba su reversión. | Aplicar y revertir todas las migraciones sobre una base vacía (MERP-107, B-6) | 2 |
| RC-10 | Para cambiar los valores de un tipo enumerado se planifica su recreación; no se asume que se puede quitar un valor. | Revisión de la migración | 2 |
| RC-11 | CORS se configura en la primera fase que expone la API, por variable de entorno. | RNF-010 | 4 |
| RC-12 | Los mensajes para el usuario se escriben aparte de los identificadores y códigos internos. | Ningún mensaje de error contiene un UUID ni un código de regla (RNF-003) | 8 |
| RC-13 | Seguridad y despliegue son parte de la primera entrega, no una fase final. | MERP-110 §2 cumplido antes de la primera demostración con datos reales | 4, 17 |

### 1.3 Datos y diseño

| ID | Restricción | Cómo se comprueba | Lección de origen |
|---|---|---|---|
| RC-14 | Ningún diseño que dependa de un archivo o dato externo se da por bueno sin contrastarlo con una muestra real. | Cada supuesto sobre el archivo cita la fila o columna real que lo respalda (MERP-112) | 14 |
| RC-15 | Los archivos con datos reales nunca entran al repositorio ni a una base que no sea la de producción. Para pruebas se usan copias anonimizadas. | RNF-006, RNF-023 | 15 |
| RC-16 | Los datos de demostración se equilibran a propósito: cada dealer con casos buenos y malos. | Revisión del sembrado | 7 |
| RC-17 | Al cambiar de usuario en un mismo navegador no queda ningún dato del anterior. | Prueba del frontend que inicia sesión con dos usuarios seguidos | 13 |

### 1.4 Documentación y proceso

| ID | Restricción | Cómo se comprueba | Lección de origen |
|---|---|---|---|
| RC-18 | La documentación vigente vive en una sola ubicación del repositorio, y el README apunta a ella. | Búsqueda de documentos duplicados sin resultados | 18 |
| RC-19 | Cuando un cambio supera una parte de un documento, ese documento se actualiza en la misma fase. Un documento de cambio es registro, no fuente vigente. | Criterio de cierre de cada fase | 19 |
| RC-20 | En los documentos, "construido" significa comprobado en el código. Lo pedido y no hecho se escribe como "planeado". Todo resumen se contrasta con el código antes de darlo por cierto. | Marcas de MERP-100 §1 | 20 |
| RC-21 | Cada fase tiene un plan escrito con criterios de aceptación, termina con pruebas en verde y es un solo commit. | Historial del repositorio | Práctica que funcionó |
| RC-22 | Cada requisito y cada regla nuevos entran con su criterio de aceptación y su prueba. | MERP-101 y MERP-102 sin entradas sin criterio | 20 |
| RC-23 | Las decisiones se registran en un único lugar (MERP-117), no repartidas entre el README, los planes y el código. | MERP-117 actualizado al cierre de cada fase | Carencia del prototipo |

### 1.5 Rendimiento y pruebas de carga

| ID | Restricción | Cómo se comprueba | Lección de origen |
|---|---|---|---|
| RC-29 | Desde F0, el arranque de producción del backend usa varios procesos de trabajo, al menos uno por núcleo. | RNF-027; monitoreo con carga | 21 |
| RC-30 | Los parámetros de Argon2 se fijan midiendo en el servidor real, con objetivo de unos 0.3 s por verificación, y se documentan. | RNF-028 | 22 |
| RC-31 | Antes de cada demostración o prueba, la base de desarrollo se reinicia al estado del sembrado y se comprueba con conteos. Ningún dato entra a desarrollo fuera del sembrado sin quedar anotado. | Conteos de dealers, usuarios, órdenes y cargas antes y después | 23 |
| RC-32 | Toda prueba de carga sigue las reglas de RNF-029: CPU real por contenedor, memoria suficiente, archivos que reescriben, sin datos reales, entorno aislado. | Informe de la prueba | 21, 24 |
| RC-33 | La prueba de carga se ejecuta antes de cotizar o rentar el servidor y se repite antes del piloto. El tamaño del servidor sale de la medición, no de una estimación. | Informe con fecha previo a la cotización | 24 |

## 2. Lecciones de origen

| # | Qué pasó |
|---|---|
| 1 | El puerto 5432 ya lo usaba el Postgres de ATLAS. |
| 2 | Al revertir una migración, los tipos enumerados de Postgres no se eliminaban; y para quitar un valor de uno hay que recrearlo. |
| 3 | En ATLAS, el archivo principal estaba fuera de la carpeta que esperaba la imagen. |
| 4 | CORS no estaba previsto y apareció al conectar el frontend. |
| 5 | El puerto 8000 lo usaba otro servicio y `localhost` resolvía a IPv6. |
| 6 | Había dos carpetas `dev` en el equipo y el proyecto se creó en la equivocada. |
| 7 | Los datos de demostración estaban sesgados: un dealer solo con casos buenos y otro solo con malos. |
| 8 | Los mensajes de error mostraban identificadores internos y códigos de regla. |
| 9 | En Windows, montar el proyecto en el contenedor tapaba el entorno de Python y los módulos de Node. |
| 10 | El servidor de desarrollo del frontend no detectaba cambios en archivos montados desde Windows. |
| 11 | Nadie había levantado todo desde cero: las migraciones nunca corrían solas en un equipo limpio. |
| 12 | Los contenedores corrían en UTC y el semáforo se adelantaba seis horas. |
| 13 | En un mismo navegador, un dealer veía un instante los datos de la sesión anterior. |
| 14 | El archivo real no coincidía con el diseño: la columna prevista para el tipo de ingreso (`THYP`) valía 0 en todas las filas; el dato real estaba en `TDS1`. |
| 15 | El archivo real contiene datos personales de clientes. |
| 16 | Los subagentes no aparecen si su definición no es válida, si la carpeta no es la del proyecto o si no se reinicia la sesión. |
| 17 | Casi todos los incidentes de entorno salieron tarde, por no probar el sistema completo de principio a fin. |
| 18 | Había dos juegos de documentos y solo uno se actualizaba; el README apuntaba al obsoleto. |
| 19 | Los documentos de cambio reemplazaban secciones de los documentos base sin marcarlos; seis documentos base describían un sistema que ya no existía. |
| 20 | El resumen de traspaso daba por construidas cosas que solo estaban pedidas (filtro por dealer, historial paginado) y nombraba mal una ruta. |
| 21 | En la prueba de carga, el backend del prototipo usaba un solo núcleo de los dos disponibles, y el porcentaje sobre el límite del contenedor (~50 %) ocultaba que ese núcleo estaba saturado. |
| 22 | El inicio de sesión tardaba 1.1 s por los parámetros de Argon2, sin haberlo medido nunca; con muchos usuarios a la vez sería un cuello de botella. |
| 23 | La base de desarrollo tenía datos que nadie recordaba haber cargado: un dealer extra con 58 órdenes. |
| 24 | El primer presupuesto del servidor se armó con una estimación; la prueba de carga llegó después y fue la que dio números reales. |

## 3. Patrón general

Los problemas vinieron de dos fuentes: no probar el sistema completo hasta tarde, y diseñar sobre supuestos sin contrastar con el dato real. Las dos se corrigen igual: verificar pronto contra lo real. RC-01, RC-14 y RC-20 son las tres restricciones que lo aseguran.

## 4. Trabajo con el equipo de agentes

El prototipo se construyó con Claude Code y un equipo de ocho subagentes. La reconstrucción usará el mismo método.

| Agente | Papel |
|---|---|
| Benja | Coordinador: plan de cada fase y reparto del trabajo |
| Isai | Arquitectura y diseño; revisa antes de que se escriba código |
| Valde | Backend |
| Leo | Frontend |
| Ramon | Seguridad (solo lectura) |
| Cofi | Pruebas y control de calidad |
| Royer | Docker y operación |
| Bueno | Nube / AWS |

**Cómo funcionan**

- Viven en `.claude/agents/` dentro del proyecto, un archivo por agente. La carpeta debe llamarse exactamente `agents`, en minúsculas.
- Cada archivo empieza con un bloque YAML con `name`, `description`, `tools` y `model`.
- Se cargan **solo al iniciar** la sesión. No se heredan de una carpeta `.claude` de nivel superior (lección 16).
- Desde M5, cada fase pasa primero por el coordinador, que escribe el plan; después revisa arquitectura y solo entonces se construye.
- Los planes y los informes quedan versionados en `.claude/plans/`: un plan por fase y una carpeta con los informes de cada agente.
- Los agentes de solo lectura no escriben archivos: quien dirige la sesión guarda su salida.

**Restricciones para la reconstrucción**

| ID | Restricción |
|---|---|
| RC-24 | Los agentes se crean y se comprueba que cargan antes de la primera fase. |
| RC-25 | El plan de cada fase cita los requisitos, reglas y decisiones de esta serie que cubre, por su identificador. |
| RC-26 | Ninguna fase empieza si depende de una decisión abierta de MERP-117 §5 sin cerrar. |
| RC-27 | La revisión de seguridad de solo lectura se hace al final de cada fase. En el prototipo encontró el hueco de la baja de dealer y la nota de privacidad de los campos de texto libre. |
| RC-28 | Los agentes no modifican esta serie de documentos fuera del criterio RC-19. |
