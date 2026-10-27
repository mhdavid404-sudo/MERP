# MERP-100 — Documento Maestro
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 4.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Autor:** David Axel Manzano Hernández
**Prototipo de referencia:** commit `5d77e51` (M7), migración `c4d8f1a29b36`.

> **Cambio de enfoque (v4.0).** Si la junta de Magma aprueba el prototipo, MERP se reconstruirá desde cero. Esta serie deja de ser solo el registro del prototipo y pasa a ser **la especificación de esa reconstrucción**. Cada documento separa lo que el sistema debe hacer de cómo está hecho hoy.

---

## 1. Cómo leer la serie

Cada documento tiene dos partes:

| Parte | Qué contiene | Carácter |
|---|---|---|
| **Parte A — Especificación** | Qué debe hacer la versión nueva. No usa nombres internos del prototipo. | Normativa: es lo que se construye y se verifica |
| **Parte B — Prototipo** | Cómo está hecho hoy en `5d77e51`: rutas, archivos, tablas, comportamientos observados. | Informativa: evidencia y referencia, no obligación |

Marcas que aparecen en todos los documentos:

| Marca | Significado |
|---|---|
| **Construido** | Existe en el prototipo y se comprobó en el código |
| **Parcial** | Existe en el prototipo, pero no cumple del todo lo que pide la especificación |
| **No construido** | La especificación lo pide y el prototipo no lo tiene |
| **Planeado** | Se pidió para el prototipo y nunca se construyó |
| **[DA-nn]** | Depende de una decisión abierta (MERP-117 §5). No se construye hasta cerrarla |
| **[VERIFICAR]** | Dato que no se pudo comprobar del todo |

Cada requisito (MERP-101) y cada regla de negocio (MERP-102) lleva un **criterio de aceptación** verificable y, cuando existe, la prueba automatizada del prototipo que ya lo demuestra.

## 2. Propósito del sistema

MERP da a Grupo Magma Automotive visibilidad sobre las órdenes de taller de sus distribuidores (dealers):

1. El distribuidor **sube el Excel de órdenes** que exporta su propio sistema de taller. El sistema crea o actualiza una orden por fila.
2. Un **dashboard con semáforo** muestra qué tan a tiempo va cada orden.
3. Un **Panel del Distribuidor** devuelve al distribuidor métricas de su propia operación.

La unidad de trabajo es la **orden de taller**, no el camión: un mismo vehículo puede tener varias órdenes.

## 3. Alcance de la versión nueva

### 3.1 Se conserva del prototipo (construido)

- Inicio de sesión; dos roles: Admin y Dealer.
- Gestión de dealers: alta, edición, baja lógica y reactivación.
- Alta de usuarios Admin y de usuarios Dealer.
- Carga de Excel por el Dealer, con resumen de órdenes nuevas, actualizadas, sin cambios y rechazadas.
- Historial de cargas.
- Dashboard con semáforo, estadísticas y tendencia.
- Panel del Distribuidor con cuatro secciones.
- Corte de acceso a los usuarios de un dealer dado de baja.

### 3.2 Entra como requisito desde el inicio (no construido)

- Seguridad de producción: HTTPS, CORS configurable y cerrado, límite de intentos de inicio de sesión, respuesta de inicio de sesión que no delate qué correos existen (MERP-101, MERP-116).
- Despliegue: arranque de producción sin recarga automática ni datos de demostración, base de datos privada y cifrada, secretos fuera del repositorio, repositorio remoto e integración y despliegue continuos (MERP-110).
- Calidad: pruebas automatizadas también en el frontend, construcción reproducible, formato único de errores (MERP-101).
- Nomenclatura del dominio: "orden" en rutas, mensajes y pantallas (MERP-117).
- Perfiles de importación por distribuidor: un solo caso de uso de carga que lee el archivo de cada dealer según su perfil, guardado como dato (RF-018; MERP-117 §4 bis).

### 3.3 Pedido para el prototipo y nunca construido (planeado)

| Elemento | Decisión que lo define |
|---|---|
| Agrupar por distribuidor la tabla del dashboard del Admin | DA-07 |
| Filtro por dealer en el dashboard | DA-07 |
| Paginación del historial de cargas | DA-06 |

### 3.4 Depende de decisiones abiertas

Gestión de usuarios, sesión y contraseñas, datos del cliente final y retención, forma de contar el semáforo, clasificación del tipo de ingreso, validación estricta del Excel y perfiles de importación (campos obligatorios, formatos aceptados, asistente). Lista completa en MERP-117 §5.

### 3.5 Fuera de alcance

Notificaciones automáticas, integración con el Excel / Power BI interno de Magma y fusión con ATLAS (DA-17).

## 4. Roles

| Rol | Puede | No puede |
|---|---|---|
| **Admin (Magma)** | Gestionar dealers. Crear usuarios Admin y Dealer. Ver el dashboard, el historial de cargas y el Panel de cualquier dealer, incluidos los dados de baja. | Subir archivos de Excel. |
| **Dealer** | Subir el Excel de su dealer. Ver su dashboard, su historial y su Panel. | Ver datos de otro dealer. Gestionar dealers o usuarios. |

No hay más roles. El rol Empleado del diseño original se eliminó (MERP-111, histórico).

## 5. Semáforo

| Color | Días transcurridos desde la entrada |
|---|---|
| Verde | 0 a 4 |
| Amarillo | 5 a 7 |
| Rojo | Más de 7 |

En proceso, se cuenta contra el momento de la consulta; con fecha de salida, se cuenta contra esa fecha y queda congelado. Se calcula en el servidor y nunca se guarda. **Cómo se cuentan los días está abierto [DA-01]** (MERP-102, RN-002).

## 6. Documentos de la serie

| Documento | Contenido | Estado |
|---|---|---|
| MERP-100 | Documento maestro (este) | Vigente |
| MERP-101 | Requisitos con criterios de aceptación | Vigente |
| MERP-102 | Reglas de negocio con criterios de aceptación | Vigente |
| MERP-103 | Arquitectura | Vigente |
| MERP-104 | Modelo de datos (dominio) | Vigente |
| MERP-105 | Casos de uso | Vigente |
| MERP-106 | Contrato de la API | Vigente |
| MERP-107 | Base de datos | Vigente |
| MERP-108 | Rediseño: ingesta de Excel | **Histórico**, superado por MERP-112 |
| MERP-109 | Panel del Distribuidor | Vigente |
| MERP-110 | Despliegue: requisitos y plan | Vigente; **no ejecutado** |
| MERP-111 | Eliminación del rol Empleado | **Histórico**, superado por MERP-100 §4 y MERP-102 |
| MERP-112 | Especificación de la carga de Excel | Vigente |
| MERP-113 | Entorno, operación y pruebas | Vigente |
| MERP-114 | Pendientes, riesgos y deuda técnica, clasificados | Vigente |
| MERP-115 | Lecciones aprendidas y restricciones de construcción | Vigente |
| MERP-116 | Datos personales y seguridad | Vigente |
| MERP-117 | Decisiones de reconstrucción y decisiones abiertas | Vigente |

Los documentos viven en `MERP-documentacion-completa/`. Las copias MERP-100 a 107 de la raíz del repositorio son la versión 1 y están obsoletas.

**Orden de lectura para reconstruir:** 117 (qué se conserva, cambia y descarta; decisiones abiertas) → 101 y 102 (qué debe cumplir) → 104, 112 y 109 (dominio, carga, panel) → 106 y 107 (contrato y datos) → 103 y 115 (arquitectura y restricciones) → 110 y 116 (despliegue y seguridad).

## 7. Estado

| Etapa | Estado |
|---|---|
| Prototipo v1 (captura manual, tres roles) | Construido y retirado |
| Aprobación de los gerentes de Magma y cambio de alcance | Hecho |
| M0 a M7 (carga de Excel, retiro de la captura manual y del rol Empleado, RN-011, Panel) | Hecho. Prototipo de referencia `5d77e51` |
| M8 (cierre del prototipo: README, capturas, agrupado) | **No iniciada** |
| Aprobación de la junta de Magma | **Pendiente** |
| Reconstrucción | **No iniciada.** Requiere cerrar antes las decisiones de MERP-117 §5 |
| Despliegue en AWS | **No iniciado** |

Calidad del prototipo: 245 pruebas automatizadas de backend en verde; el frontend compila y pasa el análisis estático, sin pruebas automatizadas (MERP-113).

## 8. Historial de versiones

| Versión | Cambio |
|---|---|
| 1.0 | Prototipo: captura manual, tres roles. |
| 2.0 | Proyecto aprobado: nota de rediseño hacia la carga de Excel. |
| 3.0 (8-oct-2026) | Reescritura contra el código de `5d77e51`. |
| 4.0 (8-oct-2026) | La serie pasa a ser especificación de la reconstrucción: Parte A / Parte B, criterios de aceptación, MERP-117. |
| 4.1 (8-oct-2026) | Se añade el requisito de perfiles de importación por distribuidor (RF-018). |
