# MERP-107 — Base de datos
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`, migración `c4d8f1a29b36`.

> Reemplaza a la versión 1.0, cuyo esquema (tablas `trucks` y `work_records`, rol `empleado`) ya no existe. La Parte A fija lo que la base de la versión nueva debe garantizar; la Parte B describe el esquema real del prototipo.

---

# Parte A — Especificación

## 1. Principios

| # | Requisito | Criterio de aceptación |
|---|---|---|
| B-1 | Cada invariante del dominio (MERP-104 §3) está impuesta también por la base. | Insertar directo en la base un dato que viole una invariante falla |
| B-2 | Las claves primarias son identificadores generados, no datos de negocio. | Ninguna clave primaria es un número de orden, un correo o un nombre |
| B-3 | No hay borrado físico de dealers ni de usuarios (RN-008). | Las relaciones impiden borrar un dealer con usuarios, órdenes o cargas |
| B-4 | Los valores derivados no se almacenan como dato de origen. El color del semáforo nunca se guarda. | No existe columna de color |
| B-5 | Importes en decimal exacto con dos decimales. | Ninguna columna de importe es de coma flotante |
| B-6 | Todo cambio de esquema es una migración versionada, reversible y probada desde una base vacía. | Aplicar todas las migraciones sobre una base vacía y revertirlas termina sin error |
| B-7 | Cada tabla de negocio registra fecha de creación y de última modificación (RNF-019). | Las fechas existen y se actualizan sin intervención de la aplicación |
| B-8 | El tratamiento de las fechas respecto a la zona horaria es uno solo en toda la base [DA-12]. | Todas las columnas de fecha son del mismo tipo |

## 2. Tablas requeridas

| Tabla | Contenido | Restricciones obligatorias |
|---|---|---|
| Dealers | MERP-104, Dealer | Nombre no vacío. Unicidad del nombre según DA-11 |
| Usuarios | MERP-104, Usuario | Correo único **sin distinguir mayúsculas**. Admin sin dealer y Dealer con dealer. Relación con dealers que impide el borrado |
| Órdenes | MERP-104, Orden | (Dealer, número de orden) único. Salida ≥ entrada. Estado coherente con la salida. Relación con dealers que impide el borrado |
| Cargas | MERP-104, Carga | Relación con dealers y usuarios que impide el borrado. **Referencia obligatoria a la versión de perfil** con la que se leyó, que impide borrar esa versión |
| Perfiles de importación | MERP-104, Perfil de importación | Un perfil por dealer (único). Relación con dealers que impide el borrado |
| Versiones de perfil | MERP-104, Versión de perfil | (Perfil, número de versión) único. Hoja, fila de encabezados, mapeo, formato de fechas y equivalencias guardados **como datos estructurados**, no como código ni como texto libre. Autor y fecha obligatorios. Sin actualización ni borrado |

**Índices requeridos:** órdenes por dealer; órdenes por estado; cargas por dealer y fecha (para el historial paginado, DA-06); usuarios por dealer; versiones por perfil y número.

**Requisitos propios de los perfiles**

| # | Requisito | Criterio de aceptación |
|---|---|---|
| B-9 | Las versiones de perfil son inmutables también en la base. | Un intento de modificar o borrar una versión, escrito directo en la base, falla |
| B-10 | La carga y su versión de perfil pertenecen al mismo dealer. | No se puede guardar una carga con la versión de perfil de otro dealer |
| B-11 | El mapeo y las equivalencias tienen una estructura validada. | Un mapeo sin alguno de los campos obligatorios, o con un campo repetido, no se puede guardar |
| B-12 | Cambiar el perfil no toca filas de órdenes ni de cargas. | Tras crear una versión, ninguna orden ni carga anterior cambia su fecha de modificación |

**Columnas sujetas a decisión:** los cuatro datos del cliente final y el respaldo de columnas sin procesar [DA-03]. Si se decide no guardarlos, no deben existir en el esquema.

## 3. Cambios obligatorios respecto al prototipo

| Cambio | Origen |
|---|---|
| Fechas de creación y modificación en todas las tablas | RNF-019 |
| Unicidad del correo sin distinguir mayúsculas | RN-020 |
| Una sola migración inicial limpia: la versión nueva no arrastra las migraciones del prototipo | MERP-117 |
| Base de datos exclusiva para pruebas | RNF-020 |
| Tablas de perfiles de importación y de versiones, y referencia a la versión en cada carga | RF-018, RN-021 a RN-029 |

---

# Parte B — Prototipo

PostgreSQL 16. Esquema en la migración `c4d8f1a29b36`.

## 4. Tipos enumerados

| Tipo | Valores |
|---|---|
| `user_role` | `admin`, `dealer` |
| `tipo_ingreso` | `garantia`, `mantenimiento` |
| `order_estado` | `en_proceso`, `completado` |

## 5. Tablas

Equivalente en SQL de los modelos de `backend/app/infrastructure/persistence/models.py`. La fuente exacta es la cadena de migraciones.

```sql
CREATE TABLE dealers (
    id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre  VARCHAR(150) NOT NULL,
    activo  BOOLEAN NOT NULL
);

CREATE TABLE users (
    id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nombre         VARCHAR(150) NOT NULL,
    email          VARCHAR(150) NOT NULL UNIQUE,
    password_hash  VARCHAR(255) NOT NULL,
    rol            user_role NOT NULL,
    dealer_id      UUID REFERENCES dealers(id) ON DELETE RESTRICT,
    activo         BOOLEAN NOT NULL,
    CONSTRAINT ck_users_rol_dealer_id CHECK (
        (rol = 'admin' AND dealer_id IS NULL)
        OR (rol = 'dealer' AND dealer_id IS NOT NULL)
    )
);
CREATE INDEX idx_users_dealer ON users(dealer_id);

CREATE TABLE uploads (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    dealer_id       UUID NOT NULL REFERENCES dealers(id) ON DELETE RESTRICT,
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    nombre_archivo  VARCHAR(255) NOT NULL,
    fecha_carga     TIMESTAMP NOT NULL DEFAULT now(),
    filas_total     INTEGER NOT NULL,
    nuevas          INTEGER NOT NULL,
    actualizadas    INTEGER NOT NULL,
    rechazadas      INTEGER NOT NULL,
    errores         JSONB NOT NULL DEFAULT '[]'      -- [{"fila": 7, "motivo": "…"}]
);
CREATE INDEX idx_uploads_dealer ON uploads(dealer_id);

CREATE TABLE orders (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    dealer_id        UUID NOT NULL REFERENCES dealers(id) ON DELETE RESTRICT,
    orden_externa    VARCHAR(50) NOT NULL,
    vin              VARCHAR(50) NOT NULL,
    modelo           VARCHAR(100),
    anio             INTEGER,
    placas           VARCHAR(20),
    tipo_ingreso     tipo_ingreso,
    tipo_servicio    VARCHAR(50),
    fecha_entrada    TIMESTAMP NOT NULL,
    fecha_salida     TIMESTAMP,
    fecha_promesa    TIMESTAMP,
    asesor           VARCHAR(150),
    total            NUMERIC(12,2),
    mano_obra        NUMERIC(12,2),
    refacciones      NUMERIC(12,2),
    cliente          VARCHAR(200),
    contacto         VARCHAR(150),
    telefono         VARCHAR(100),
    email            VARCHAR(150),
    datos_crudos     JSONB NOT NULL DEFAULT '{}',
    estado           order_estado NOT NULL,
    ultima_carga_id  UUID REFERENCES uploads(id) ON DELETE SET NULL,
    CONSTRAINT uq_orders_dealer_orden_externa UNIQUE (dealer_id, orden_externa),
    CONSTRAINT ck_orders_fecha_salida_after_entrada
        CHECK (fecha_salida IS NULL OR fecha_salida >= fecha_entrada),
    CONSTRAINT ck_orders_estado_fecha_salida
        CHECK ((estado = 'completado') = (fecha_salida IS NOT NULL))
);
CREATE INDEX idx_orders_dealer ON orders(dealer_id);
CREATE INDEX idx_orders_estado ON orders(estado);
```

Notas:
- Todas las fechas son `TIMESTAMP` **sin zona horaria**, en hora local de planta.
- `estado` se guarda, aunque en el dominio es derivado, para poder filtrar y contar en SQL; la restricción impide que contradiga a `fecha_salida`.
- `rechazadas` se guarda; `sin_cambios` no existe como columna: se calcula.
- Ninguna tabla tiene fechas de creación o modificación.
- `dealers.nombre` no es único.
- `users.email` es único **distinguiendo** mayúsculas; la aplicación busca sin distinguirlas.
- No existe columna de color de semáforo.
- **No existen tablas de perfiles de importación.** La tabla `uploads` no guarda con qué reglas se leyó cada archivo.

## 6. Migraciones

Cadena lineal en `backend/migrations/versions/`:

| Revisión | Contenido | Anterior |
|---|---|---|
| `8e5ce008b64e` | Esquema inicial de la versión 1 (`dealers`, `users`, `trucks`, `work_records`) | — |
| `62f16403dd2f` | Tablas `orders` y `uploads` | `8e5ce008b64e` |
| `a7c3e91d5b20` | Elimina `trucks`, `work_records` y el tipo `truck_estado`. Los datos se descartaron | `62f16403dd2f` |
| `c4d8f1a29b36` | Elimina el rol `empleado`: los empleados pasan a `dealer` con `activo=false`; se recrea el tipo `user_role` y la restricción de `users`. **Actual** | `a7c3e91d5b20` |

Revertir la última migración recrea el rol `empleado`, pero no recupera quiénes lo tenían.

## 7. Conexión

- La aplicación lee la dirección de la base de `DATABASE_URL`.
- Una sesión de base de datos por petición: se confirma si todo sale bien y se revierte ante cualquier error.
- Las pruebas de integración usan la misma base de desarrollo, dentro de una transacción que se revierte (MERP-113).
