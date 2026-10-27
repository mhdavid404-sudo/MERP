# MERP-113 — Entorno, operación y pruebas
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.1 · **Fecha:** 26 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> La Parte A fija lo que el entorno de la versión nueva debe cumplir. La Parte B describe el entorno real del prototipo: contenedores, variables, sembrado y pruebas.

---

# Parte A — Especificación

## 1. Entornos

| Entorno | Para qué | Datos |
|---|---|---|
| Desarrollo | Construir y demostrar en un equipo local | De demostración, sembrados |
| Pruebas | Ejecutar las pruebas automatizadas | Base exclusiva, vacía, creada y destruida por las pruebas |
| Producción | Piloto con el distribuidor | Reales |

| # | Requisito | Criterio de aceptación | Origen |
|---|---|---|---|
| E-1 | Los tres entornos se distinguen solo por configuración; el código es el mismo. | Ningún archivo de código cambia entre entornos | RNF-024 |
| E-2 | Desarrollo se levanta con un comando en un equipo sin estado previo. | Verificado en un equipo limpio, y de forma automática en la integración continua | RNF-008 |
| E-3 | Las pruebas no usan la base de desarrollo. | Ejecutar las pruebas no lee ni escribe datos de desarrollo | RNF-020 |
| E-4 | Producción no siembra datos ni arranca con recarga automática. | RNF-011 | RNF-011 |
| E-5 | Ningún entorno distinto de producción contiene datos reales. | RNF-023 | RNF-023 |
| E-6 | Las variables obligatorias se validan al arrancar. | Falta una → el arranque falla nombrándola | RNF-024 |
| E-7 | Las versiones de lenguajes, dependencias y herramientas están fijadas y son las mismas en el equipo, en los contenedores y en la integración continua. | Dos construcciones del mismo commit son idénticas | RNF-021 |
| E-8 | Servidor y base de datos en la zona horaria de la planta, en todos los entornos. | RNF-005 | RNF-005 [DA-12] |

## 2. Variables de configuración requeridas

| Variable | Contenido | Obligatoria |
|---|---|---|
| Dirección de la base de datos | Conexión a PostgreSQL | Sí |
| Secreto de firma de sesiones | Distinto por entorno | Sí |
| Duración de la sesión | Según DA-05 | Sí |
| Orígenes permitidos (CORS) | Lista de dominios del frontend | Sí en producción (RNF-010) |
| Entorno | Desarrollo, pruebas o producción; decide si se siembra y si se recarga | Sí |
| Dirección pública de la API | Para compilar el frontend | Sí |
| Zona horaria | `America/Mexico_City` | Sí |

Toda variable declarada en la plantilla debe ser leída por el código, y toda variable leída debe estar en la plantilla.

## 3. Pruebas

| # | Requisito | Criterio de aceptación |
|---|---|---|
| T-1 | Cada regla de MERP-102 tiene al menos una prueba unitaria sin base de datos. | Tabla de correspondencia regla → prueba, completa |
| T-2 | Cada operación de la API tiene pruebas de integración de éxito, de permisos y de aislamiento entre dealers. | RNF-007 |
| T-3 | La carga de Excel se prueba con un archivo de ejemplo anonimizado y con un archivo de casos límite. | MERP-112 §9 |
| T-4 | El frontend tiene pruebas de la lógica que conserve y de los flujos principales. | Existe un comando de pruebas del frontend y pasa |
| T-5 | Las pruebas, la compilación y el análisis estático corren en cada cambio. | RNF-014 |
| T-6 | Todo cálculo dependiente del tiempo se prueba con la hora fijada. | MERP-103, P-6 |
| T-7 | Hay una prueba de carga repetible, versionada en el repositorio, que cubre carga de archivos, consultas, su mezcla, archivos grandes, generación de PDF y envío de correos. | RNF-025, RNF-026, RNF-029 |

Las 245 pruebas del prototipo son la base de partida: describen el comportamiento esperado y se reutilizan como criterios de aceptación en MERP-101, 102, 109 y 112.

## 4. Datos de demostración

El sembrado de desarrollo debe dejar: dos dealers, al menos un Admin, un usuario por dealer y dos cargas de un archivo anonimizado que muestren una actualización entre cargas. Debe poder repetirse sin duplicar. Un fallo del sembrado debe ser visible.

---

# Parte B — Prototipo

## 5. Contenedores

`docker-compose.yml`, tres servicios:

| Servicio | Contenedor | Imagen | Puerto equipo → contenedor | Notas |
|---|---|---|---|---|
| `db` | `merp_db` | `postgres:16` | 5433 → 5432 | Zona horaria con `TZ` y `PGTZ`; volumen `merp_pgdata`; comprobación de salud |
| `backend` | `merp_backend` | Construida de `./backend` (Python 3.13) | 8000 → 8000 | Lee `.env`; `TZ` fijada; monta el código del equipo; espera a que la base esté sana |
| `frontend` | `merp_frontend` | Construida de `./frontend` (Node 22) | 5173 → 5173 | Servidor de desarrollo de Vite; monta el código del equipo |

- El puerto 5433 evita el choque con otro Postgres del equipo en el 5432.
- Los contenedores protegen con un volumen propio el entorno de Python y los módulos de Node, para que el montaje del código no los tape.
- El servidor de desarrollo del frontend detecta cambios por sondeo, necesario al montar carpetas desde Windows.

| Dirección | Servicio |
|---|---|
| `http://localhost:5173` | Frontend |
| `http://localhost:8000` | API |
| `http://localhost:8000/docs` | Documentación interactiva de la API |
| `http://localhost:8000/health` | Verificación de salud |
| `localhost:5433` | PostgreSQL |

## 6. Arranque del backend

`backend/docker-entrypoint.sh` hace, en este orden:

1. Aplica las migraciones.
2. Ejecuta el sembrado de demostración. Si falla, el fallo se ignora (`|| true`).
3. Arranca el servidor con recarga automática (`--reload`).

Es un arranque de desarrollo. No hay arranque de producción.

## 7. Variables de entorno

Plantilla en `.env.example`. Los archivos `.env` no se versionan.

| Variable | La lee | Notas |
|---|---|---|
| `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` | El contenedor de la base | |
| `DATABASE_URL` | Backend, migraciones y pruebas | Dentro de Docker apunta a `db:5432`; las pruebas fuera de Docker la reescriben a `127.0.0.1:5433` |
| `JWT_SECRET_KEY` | Backend | Se lee en cada petición, no al arrancar |
| `JWT_EXPIRATION_MINUTES` | Backend | La plantilla propone 60. **[VERIFICAR]** el valor del `.env` real |
| `VITE_API_BASE_URL` | Frontend | `http://127.0.0.1:8000/api/v1`, con `127.0.0.1` explícito |
| `VITE_APP_ENV` | No comprobado | En la plantilla. **[VERIFICAR]** si el frontend la usa |
| `ENVIRONMENT`, `LOG_LEVEL` | **Nadie** | Están en la plantilla y ningún archivo del backend las lee |

No hay variable para CORS: está fijo en el código.

## 8. Zona horaria

Las fechas se guardan sin zona, en hora local de planta. Los contenedores corren en `America/Mexico_City`. Antes de fijarlo, corrían en UTC y el semáforo se adelantaba seis horas.

## 9. Sembrado de demostración

`backend/scripts/seed_demo_data.py`. Sobre una base vacía crea:

| Usuario | Rol | Dealer |
|---|---|---|
| `admin@magma.com` | Admin | — |
| `sistemas@magma.com` | Admin | — |
| `dealer.toluca@magma.com` | Dealer | Magma Toluca |
| `dealer.gdl@magma.com` | Dealer | Magma Guadalajara |

Las contraseñas de demostración están en el script y en el README; no se repiten aquí.

- Dos cargas del archivo anonimizado, ambas como Magma Toluca, fechadas el miércoles y el viernes más recientes: **37 nuevas**; después **10 nuevas, 6 actualizadas y 31 sin cambios**. Resultado: 47 órdenes.
- **Magma Guadalajara queda sin órdenes**, para mostrar el aislamiento y el estado vacío.
- Las fechas del archivo se recorren semanas completas para que la demostración se vea igual sin importar cuándo se reinicie.
- Todo se crea llamando a la propia API, salvo el primer Admin y la fecha de cada carga, que se escriben directo en la base.
- Es repetible por partes: no siembra usuarios si ya hay usuarios, ni cargas si ya hay cargas.

Para volver al estado inicial: eliminar el volumen de la base y levantar de nuevo.

## 10. Datos de la base de desarrollo que no vienen del sembrado

Consultados el 8 de octubre de 2026:

| Dealer | Órdenes | Cargas | Usuarios |
|---|---|---|---|
| Magma Toluca | 47 | 2 | 4 |
| Magma Guadalajara | 0 | 0 | 2 |
| MYLSA | 62 | 1 | 1 |
| PRUEBA | 8 | 1 | 1 |
| General | 0 | 0 | 0 |

Además del sembrado hay: dos ex-empleados inactivos, una cuenta de desarrollo inactiva, un usuario de prueba de M6 activo, y los dealers MYLSA, PRUEBA y General con sus usuarios. **[VERIFICAR]** si las 62 órdenes de MYLSA proceden del archivo real (MERP-114, G-01).

## 11. Pruebas del backend

245 pruebas, todas en verde el 8 de octubre de 2026 (32 segundos).

| Grupo | Archivo | Pruebas |
|---|---|---|
| — | `test_health.py` | 1 |
| Integración | `test_auth_authorization.py` | 9 |
| | `test_auth_login.py` | 5 |
| | `test_dashboard_endpoint.py` | 4 |
| | `test_dealers_endpoint.py` | 14 |
| | `test_order_repository.py` | 9 |
| | `test_panel_endpoint.py` | 16 |
| | `test_trucks_read_endpoint.py` | 6 |
| | `test_uploads_endpoint.py` | 13 |
| | `test_users_admin_endpoint.py` | 4 |
| | `test_users_dealers_endpoint.py` | 20 |
| Unitarias, aplicación | `test_access_policy.py` | 8 |
| | `test_authenticate_user.py` | 8 |
| | `test_create_admin.py` | 2 |
| | `test_create_dealer_user.py` | 5 |
| | `test_dealer_id_requerido.py` | 6 |
| | `test_dealer_use_cases.py` | 7 |
| | `test_get_dashboard.py` | 3 |
| | `test_get_panel_distribuidor.py` | 10 |
| | `test_list_get_trucks.py` | 5 |
| | `test_process_upload.py` | 14 |
| Unitarias, dominio | `test_order.py` | 10 |
| | `test_panel_distribuidor.py` | 29 |
| | `test_semaforo_calculator.py` | 8 |
| Unitarias, infraestructura | `test_argon2_password_hasher.py` | 3 |
| | `test_jwt_token_service.py` | 3 |
| | `test_xlsx_order_reader.py` | 23 |

Algunos archivos tienen más pruebas que funciones porque varias funciones se ejecutan con varios juegos de datos.

- Las pruebas de integración usan **la base de desarrollo**, cada una dentro de una transacción que se revierte.
- Requieren la base levantada en `127.0.0.1:5433`.

| Comando | Qué comprueba | Resultado el 8-oct-2026 |
|---|---|---|
| `uv run pytest` (en `backend/`) | Las 245 pruebas | 245 en verde |
| `npx tsc -b` (en `frontend/`) | Tipos | Sin errores |
| `npm run build` | Compilación | Correcta |
| `npm run lint` | Análisis estático | Sin avisos |

## 12. Pruebas del frontend

No hay. No existe comando de pruebas ni archivos de prueba. Solo se verifica con compilación y análisis estático. La lógica de orden por urgencia, estadísticas y tendencia del dashboard no está cubierta.

## 13. Dependencias

Versiones instaladas en el equipo de desarrollo el 8 de octubre de 2026.

| Backend (Python 3.13.14, gestionado con uv) | Versión |
|---|---|
| fastapi | 0.141.1 |
| starlette | 1.6.0 |
| uvicorn | 0.52.4 |
| sqlalchemy | 2.0.52 |
| alembic | 1.20.0 |
| psycopg2-binary | 2.9.13 |
| pydantic | 2.13.5 |
| pyjwt | 2.14.0 |
| argon2-cffi | 25.1.0 |
| openpyxl | 3.1.5 |
| python-multipart | 0.0.32 |
| pytest (desarrollo) | 9.1.1 |
| httpx (desarrollo) | 0.28.1 |

| Frontend (Node 24.18.0 en el equipo) | Versión |
|---|---|
| react, react-dom | 19.3.0 |
| react-router-dom | 7.18.3 |
| @tanstack/react-query | 5.102.8 |
| vite | 8.3.0 |
| @vitejs/plugin-react | 6.1.1 |
| typescript | 6.0.3 |
| eslint | 10.10.0 |
| typescript-eslint | 8.70.0 |

Diferencias de reproducibilidad observadas (MERP-114, grupo F):
- `pyproject.toml` declara versiones mínimas (`>=`), no exactas; la exactitud depende del archivo de bloqueo.
- La imagen del backend instala `uv` con la etiqueta `latest`.
- La imagen del frontend usa Node 22; el equipo, Node 24. Además instala con `npm install` copiando solo `package.json`.
- En una construcción, `uv sync` cambió tres versiones respecto a las esperadas.

## 14. Repositorio

- Rama `main`, 14 commits; el último, `5d77e51`.
- **Sin remoto configurado** y sin integración continua.
- Sin confirmar en el repositorio a la fecha: el plan de la fase M8, el inventario técnico, el estado verificado y esta reescritura de documentos.
