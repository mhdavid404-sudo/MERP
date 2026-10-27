# MERP-103 — Arquitectura
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.0 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> La Parte A fija los principios que la versión nueva debe respetar. La Parte B describe la estructura real del prototipo, verificada en el código.

---

# Parte A — Especificación

## 1. Vista general

```
┌────────────────────────────────────────────────┐
│                Frontend (web)                   │
│  Inicio de sesión · Dashboard · Panel ·         │
│  Cargas · Dealers y usuarios                    │
└───────────────────────┬────────────────────────┘
                        │ HTTPS / JSON
┌───────────────────────▼────────────────────────┐
│                    Backend                      │
│  ┌───────────────────────────────────────────┐  │
│  │ Presentación: rutas, esquemas, permisos   │  │
│  ├───────────────────────────────────────────┤  │
│  │ Aplicación: casos de uso, puertos         │  │
│  ├───────────────────────────────────────────┤  │
│  │ Dominio: entidades, reglas, cálculos      │  │
│  ├───────────────────────────────────────────┤  │
│  │ Infraestructura: base de datos, lector de │  │
│  │ Excel, contraseñas, sesiones              │  │
│  └───────────────────────────────────────────┘  │
└───────────────────────┬────────────────────────┘
                        │
                ┌───────▼────────┐
                │   PostgreSQL   │
                └────────────────┘
```

El Excel entra por el frontend, lo recibe la presentación, lo interpreta un lector de infraestructura y lo procesa un caso de uso de aplicación que aplica las reglas del dominio.

## 2. Principios obligatorios

| # | Principio | Cómo se verifica |
|---|---|---|
| P-1 | **Dependencias hacia el dominio.** El dominio no importa nada de aplicación, infraestructura ni presentación. La aplicación no importa infraestructura: usa puertos. | Comprobación automática de importaciones en la integración continua |
| P-2 | **Las reglas de negocio viven en el dominio**, como entidades y funciones puras sin entrada ni salida. | Cada regla de MERP-102 tiene una prueba unitaria que no necesita base de datos |
| P-3 | **El servidor calcula; el frontend presenta.** Semáforo, días, métricas del Panel, orden de urgencia y agregados del dashboard se calculan en el servidor. | Ninguna regla de MERP-102 está implementada solo en el frontend |
| P-4 | **El alcance por rol se decide en un solo lugar** por operación, en la capa de aplicación, y nunca se confía en un identificador de dealer enviado por el cliente. | Prueba de aislamiento por cada operación (RNF-007) |
| P-5 | **Una transacción por caso de uso.** O se guarda todo o nada. | Prueba de fallo a mitad de una carga (RN-017, criterio 4) |
| P-6 | **El tiempo es inyectable.** Todo cálculo que dependa de "ahora" recibe el reloj como parámetro. | Las pruebas de semáforo y Panel fijan la hora |
| P-7 | **La identidad se revalida en cada petición** contra la base de datos; el contenido de la sesión no basta para autorizar. | RN-011, criterio 2 |
| P-8 | **Configuración por entorno, validada al arrancar** (RNF-024). | Arrancar sin una variable obligatoria falla nombrándola |
| P-9 | **Errores con un único formato** (RNF-017), generados en un solo punto a partir de errores de dominio. | Prueba de formato por cada tipo de error |
| P-10 | **Importes en decimal exacto**, nunca en coma flotante, y transmitidos como texto. | Prueba de suma y redondeo del Panel |

## 3. Decisiones de arquitectura (ADR)

| ADR | Decisión | Situación para la versión nueva |
|---|---|---|
| ADR-001 | Aplicación web con frontend propio. | Se conserva |
| ADR-002 | Base de datos real (PostgreSQL) desde el inicio, también en pruebas. | Se conserva, con una base exclusiva para pruebas (RNF-020) |
| ADR-003 | Autenticación simple de prototipo: usuario, contraseña y token, sin capas adicionales. De aquí salió también el CORS abierto. | **Sustituida** por los requisitos de seguridad RNF-009 a RNF-016 y por DA-05 |
| ADR-004 | El semáforo se calcula en el servidor. | Se conserva y se amplía a todo cálculo de negocio (P-3) |
| ADR-005 | Sin colas ni procesos en segundo plano. | Se conserva con otra justificación: el archivo se procesa en la misma petición porque es pequeño (el de ejemplo, 62 órdenes, pesa unos 25 KB; el límite es 5 MB). Revisar si el volumen crece |
| ADR-006 | Fechas sin zona horaria, en hora local de planta. | **Abierta [DA-12]** |
| ADR-007 | La carga más reciente es la foto vigente de cada orden (RN-016). | Se conserva; el caso de las órdenes ausentes queda abierto [DA-08] |
| ADR-008 | La baja de un dealer retira el acceso por regla, sin modificar a los usuarios (RN-011). | Se conserva |
| ADR-009 | Las métricas del Panel son funciones puras sobre una ventana móvil (MERP-109). | Se conserva |
| ADR-010 | Importes en decimal y como texto en la API. | Se conserva (P-10) |

La justificación original de ADR-005 ("este sistema no procesa archivos Excel") dejó de ser cierta al cambiar a la carga de Excel; la decisión se sostiene por el tamaño del archivo.

## 4. Stack

Mismo stack que el prototipo, **a confirmar en DA-14**: Python con FastAPI, SQLAlchemy y Alembic; PostgreSQL; React con TypeScript y Vite; Docker. Las versiones se fijan al iniciar la reconstrucción (RNF-021).

## 5. Frontend

- Una pantalla por función: inicio de sesión, dashboard, Panel, cargas, dealers y usuarios.
- Los datos remotos se gestionan con una caché de consultas que **se vacía al iniciar y al cerrar sesión**, y cuyas claves incluyen la identidad del usuario.
- La protección de rutas por rol en el frontend es de comodidad; la autorización real es del servidor.
- Sin lógica de negocio (P-3).
- Con pruebas automatizadas (RNF-020).

---

# Parte B — Prototipo

## 6. Estructura del backend

```
backend/
├── app/
│   ├── main.py              Aplicación, CORS, manejadores de error, 7 enrutadores bajo /api/v1, /health
│   ├── domain/
│   │   ├── entities/        dealer, order, upload, user
│   │   ├── value_objects/   order_estado, periodo_panel, semaforo, tipo_ingreso, user_role
│   │   ├── services/        semaforo_calculator, panel_distribuidor
│   │   └── exceptions.py    6 errores de dominio
│   ├── application/
│   │   ├── access_policy.py Regla de acceso RN-011
│   │   ├── dto/             Objetos de transferencia
│   │   ├── ports/           4 repositorios, lector de archivo, hasher, servicio de tokens
│   │   └── use_cases/       13 casos de uso
│   ├── infrastructure/
│   │   ├── excel/           Lector de Excel (openpyxl)
│   │   ├── persistence/     Modelos, sesión, 4 repositorios SQLAlchemy
│   │   └── security/        Argon2 y JWT (HS256)
│   └── api/
│       ├── dependencies/    Autenticación, permisos, armado de casos de uso
│       ├── routers/         auth, dashboard, dealers, panel, trucks, uploads, users
│       ├── schemas/         Esquemas Pydantic
│       └── exception_handlers.py
├── migrations/versions/     4 migraciones
├── scripts/                 Sembrado de demostración; anonimizador de Excel
└── tests/                   245 pruebas
```

La capa de presentación se llama `api/` en el prototipo.

## 7. Estructura del frontend

```
frontend/src/
├── main.tsx, index.css, types.ts, labels.ts, format.ts
├── app/         Enrutador, ruta protegida, plantilla, sesión, almacenamiento de sesión, caché de consultas
├── api/         Cliente HTTP y un módulo por recurso
├── components/  Insignia de semáforo, modal, gráficas de barras y de línea en SVG propio
└── features/    auth, dashboard, panel, cargas, dealers
```

Rutas de pantalla: `/login`, `/` (dashboard), `/panel`, `/cargas`, `/dealers` (solo Admin). Sin librería de componentes ni de gráficas.

## 8. Cómo cumple el prototipo los principios

| Principio | Situación | Detalle |
|---|---|---|
| P-1 | Construido, sin comprobación automática | Se respeta por convención |
| P-2 | Construido | Semáforo y Panel son funciones puras con pruebas unitarias |
| P-3 | **Parcial** | El servidor calcula días, color y Panel. El **orden por urgencia, el promedio de días, las estadísticas y las series de tendencia del dashboard se calculan en el navegador**, sin pruebas |
| P-4 | Construido | El alcance del dashboard reutiliza el del listado de órdenes |
| P-5 | Construido | Una sesión de base de datos por petición, con confirmación o reversión |
| P-6 | **Parcial** | El Panel recibe el reloj; el dashboard llama a la hora del sistema directamente |
| P-7 | Construido | Cada petición autenticada relee al usuario y a su dealer |
| P-8 | **Parcial** | Tres variables se leen al usarse; el secreto y la duración de la sesión se leen en cada petición |
| P-9 | **Parcial** | Los errores de dominio usan un formato; la validación automática de entrada usa otro |
| P-10 | Construido | Decimal con redondeo de la mitad hacia arriba; importes del Panel como texto |

Otras características del prototipo que la versión nueva debe cambiar: CORS abierto y fijo en el código, un único arranque que recarga y siembra, y nombres heredados de "camión" (MERP-117 §3).
