# MERP-106 — Contrato de la API
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 2.0 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.

> La Parte A define el contrato que la versión nueva debe ofrecer, con nombres del dominio. La Parte B documenta el contrato real del prototipo, verificado en el código, con sus nombres heredados.

---

# Parte A — Especificación

## 1. Convenciones

| Tema | Requisito |
|---|---|
| Versión | Todas las rutas bajo un prefijo de versión (`/api/v1`) |
| Formato | JSON, salvo la subida de archivo (formulario multiparte) |
| Autenticación | Todas las rutas la exigen, salvo el inicio de sesión y la verificación de salud. El mecanismo de sesión se cierra en DA-05 |
| Autorización | Por rol, comprobada en el servidor. El dealer del usuario nunca se toma de un dato enviado por el cliente (RN-013) |
| Nombres | Del dominio: órdenes, cargas, dealers, usuarios (RNF-022) |
| Fechas | Hora local de planta, sin zona, en formato ISO 8601 [DA-12] |
| Importes | Texto con dos decimales |
| Datos del cliente final | Ninguna respuesta los incluye |

## 2. Errores

Un único formato para **todos** los errores, incluida la validación de datos de entrada (RNF-017):

```json
{ "error": { "code": "CODIGO", "message": "Texto legible para el usuario" } }
```

| Situación | Estado HTTP | Código |
|---|---|---|
| Sin sesión, sesión inválida o expirada, usuario o dealer inactivo | 401 | `AUTHENTICATION_ERROR` |
| Rol sin permiso, o usuario sin dealer asignado | 403 | `AUTHORIZATION_ERROR` |
| Recurso inexistente, o ajeno al dealer del usuario | 404 | `NOT_FOUND` |
| Dato duplicado | 409 | `DUPLICATE_INPUT` |
| Regla de negocio incumplida | 422 | `BUSINESS_RULE_VIOLATION` |
| Dato de entrada inválido o faltante | 400 | `VALIDATION_ERROR` |
| Demasiados intentos de inicio de sesión (RNF-015) | 429 | Por definir |

**Criterios de aceptación**
1. Ninguna respuesta de error tiene otra estructura.
2. Ningún mensaje contiene identificadores internos ni códigos de regla.
3. Ninguna situación prevista produce un error 500: en particular, dos altas simultáneas con el mismo correo y una sesión con identificador de usuario mal formado.

## 3. Operaciones requeridas

Las rutas son la **propuesta** para la versión nueva. Lo obligatorio es la operación, sus permisos y sus resultados.

| Operación | Ruta propuesta | Rol | Resultado | Se verifica con |
|---|---|---|---|---|
| Verificación de salud | `GET /health` | Público | Estado del servicio | — |
| Iniciar sesión | `POST /auth/login` | Público | Sesión, rol y dealer | RF-008 |
| Cerrar sesión | `POST /auth/logout` | Autenticado | — | RF-016 [DA-05] |
| Cambiar la contraseña | Por definir | Autenticado | — | RF-016 [DA-05] |
| Listar dealers | `GET /dealers` | Admin | Dealers activos e inactivos | RF-001 |
| Crear dealer | `POST /dealers` | Admin | Dealer creado | RF-001 |
| Editar o reactivar dealer | `PATCH /dealers/{id}` | Admin | Dealer actualizado | RF-001 |
| Dar de baja dealer | `DELETE /dealers/{id}` | Admin | Dealer inactivo | RF-001, RF-014 |
| Crear usuario Admin | `POST /users/admins` | Admin | Usuario creado, sin contraseña ni hash | RF-002 |
| Crear usuario Dealer | `POST /users/dealers` | Admin | Usuario creado | RF-009 |
| Listar usuarios | `GET /users` | Por decidir | Usuarios | RF-015 [DA-04] |
| Dar de baja o reactivar usuario | `PATCH /users/{id}` | Por decidir | Usuario actualizado | RF-015 [DA-04] |
| Listar órdenes | `GET /orders` | Admin, Dealer | Órdenes del alcance | RF-012 |
| Consultar una orden | `GET /orders/{id}` | Admin, Dealer | Orden | RF-012, RN-012 |
| Dashboard | `GET /dashboard` | Admin, Dealer | Órdenes con días y color, bajo la clave `orders` | RF-007 |
| Subir Excel | `POST /uploads` | Dealer | Resumen de la carga | RF-010 |
| Historial de cargas | `GET /uploads` | Admin, Dealer | Cargas, con paginación [DA-06] | RF-011 |
| Panel del Distribuidor | `GET /panel` | Admin, Dealer | Las cuatro secciones | RF-013 |

Cambios obligatorios respecto al prototipo:
- Las órdenes dejan de llamarse `trucks` en rutas y en respuestas.
- El dashboard devuelve también el identificador del dealer de cada orden, necesario para agrupar sin depender del nombre.
- El orden de las filas del dashboard y sus agregados los entrega el servidor (MERP-103, P-3). El criterio de orden entre dealers se cierra en DA-07.

## 4. Contenido mínimo de las respuestas

| Recurso | Campos |
|---|---|
| Dealer | Identificador, nombre, activo |
| Usuario | Identificador, nombre, correo, rol, dealer (si aplica), activo |
| Orden | Identificador, dealer, número de orden, VIN, modelo, año, placas, tipo de ingreso, tipo de servicio, asesor, fechas de entrada, salida y promesa, estado |
| Fila del dashboard | Identificador, identificador y nombre del dealer, número de orden, modelo, año, tipo de ingreso, asesor, estado, días transcurridos, color, fechas de entrada y salida |
| Carga | Identificador, dealer, archivo, fecha, filas totales, nuevas, actualizadas, sin cambios, rechazadas, lista de (fila, motivo); en el historial, además nombre del dealer y de quien la subió |
| Panel | MERP-109 §8 |

---

# Parte B — Prototipo

Contrato real en `5d77e51`. Prefijo `/api/v1` en todas las rutas salvo `/health`.

## 5. Convenciones del prototipo

- Autenticación: `Authorization: Bearer <token>`. Token JWT firmado (HS256) con el identificador del usuario, su rol, su dealer y la expiración.
- En cada petición autenticada se relee al usuario y a su dealer en la base (RN-011).
- CORS abierto a cualquier origen, fijo en el código.
- Las altas de usuarios recortan los espacios de los textos.

## 6. Errores del prototipo

Los errores de dominio usan el formato del §2. **La validación automática de datos de entrada no**: responde `422` con el formato propio del marco de trabajo:

```json
{ "detail": [ { "loc": ["body", "email"], "msg": "…", "type": "…" } ] }
```

| Estado | Código | Cuándo |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Archivo no válido; Admin que pide el Panel sin dealer |
| 401 | `AUTHENTICATION_ERROR` | Sin token, token inválido o expirado, usuario inactivo, dealer dado de baja, credenciales incorrectas |
| 403 | `AUTHORIZATION_ERROR` | Rol insuficiente; usuario sin dealer; Dealer que pide el Panel de otro |
| 404 | `NOT_FOUND` | Dealer u orden inexistente; orden de otro dealer |
| 409 | `DUPLICATE_INPUT` | Correo ya registrado |
| 422 | `BUSINESS_RULE_VIOLATION` | Crear un usuario para un dealer dado de baja |
| 422 | Formato `detail` | Cuerpo o parámetros mal formados: correo inválido, contraseña corta, nombre en blanco, UUID mal formado, periodo inválido |
| 500 | — | Dos altas simultáneas con el mismo correo; token bien firmado con identificador de usuario mal formado (MERP-114, D-06 y D-07) |

## 7. Rutas del prototipo

### Salud y sesión

| Método | Ruta | Rol | Petición | Respuesta |
|---|---|---|---|---|
| GET | `/health` | Público | — | `200 {"status":"ok"}` |
| POST | `/auth/login` | Público | `{email, password}` | `200 {access_token, token_type:"bearer", rol, dealer_id}` · `401` |

### Dealers

| Método | Ruta | Rol | Petición | Respuesta |
|---|---|---|---|---|
| GET | `/dealers` | Admin | — | `200 [{id, nombre, activo}]`, ordenados por nombre |
| POST | `/dealers` | Admin | `{nombre}` (1 a 150 caracteres) | `201 {id, nombre, activo}` |
| PATCH | `/dealers/{id}` | Admin | `{nombre?, activo?}` | `200 {id, nombre, activo}` · `404` |
| DELETE | `/dealers/{id}` | Admin | — | `200 {id, nombre, activo:false}` (baja lógica) · `404` |

### Usuarios

| Método | Ruta | Rol | Petición | Respuesta |
|---|---|---|---|---|
| POST | `/users/admins` | Admin | `{nombre, email, password}` (contraseña ≥ 8) | `201 {id, nombre, email, rol, activo}` · `409` |
| POST | `/users/dealers` | Admin | `{nombre, email, password, dealer_id}` | `201 {id, nombre, email, rol:"dealer", dealer_id, activo}` · `404` · `422` · `409` |

No existen rutas para listar, editar ni dar de baja usuarios.

### Órdenes (nombre heredado: `trucks`)

| Método | Ruta | Rol | Respuesta |
|---|---|---|---|
| GET | `/trucks` | Admin, Dealer | `200 [Orden]`, por fecha de entrada descendente |
| GET | `/trucks/{id}` | Admin, Dealer | `200 Orden` · `404` si no existe o es de otro dealer |

`Orden`: `id`, `dealer_id`, `orden_externa`, `modelo`, `anio`, `vin`, `placas`, `tipo_ingreso`, `tipo_servicio`, `asesor`, `fecha_entrada`, `fecha_salida`, `fecha_promesa`, `estado`. Sin datos del cliente final.

### Dashboard

| Método | Ruta | Rol | Respuesta |
|---|---|---|---|
| GET | `/dashboard` | Admin, Dealer | `200 {trucks: [Fila]}` |

No acepta filtros. No trae el identificador del dealer. El orden es el del listado de órdenes (fecha de entrada descendente); el orden por urgencia lo aplica el navegador.

```json
{
  "trucks": [
    {
      "id": "uuid",
      "dealer": "Magma Toluca",
      "orden_externa": "T001271",
      "modelo": "MINIVAN",
      "anio": 2025,
      "tipo_ingreso": "mantenimiento",
      "asesor": "ASESOR A",
      "estado": "en_proceso",
      "dias_transcurridos": 3,
      "semaforo": "verde",
      "fecha_entrada": "2026-10-05T09:36:00",
      "fecha_salida": null
    }
  ]
}
```

`modelo`, `anio`, `tipo_ingreso` y `asesor` pueden ser nulos. Los valores del ejemplo son ilustrativos.

### Cargas

| Método | Ruta | Rol | Petición | Respuesta |
|---|---|---|---|---|
| POST | `/uploads` | Dealer | Formulario multiparte, campo `archivo` (`.xlsx`, ≤ 5 MB) | `201 Carga` · `400` archivo no válido · `403` Admin |
| GET | `/uploads` | Admin, Dealer | `?dealer_id=` (solo tiene efecto para el Admin) | `200 [Carga + dealer, subido_por]`, por fecha descendente, sin paginación |

`Carga`: `id`, `dealer_id`, `nombre_archivo`, `fecha_carga`, `filas_total`, `nuevas`, `actualizadas`, `sin_cambios`, `rechazadas`, `errores: [{fila, motivo}]`.

Mensajes de rechazo del archivo (`400`): "Solo se aceptan archivos de Excel (.xlsx)." · "El archivo está vacío." · "El archivo excede el tamaño máximo de 5 MB." · "El archivo no es un Excel (.xlsx) válido." · "No se encontraron las columnas obligatorias (ORDEN, SERIE, ALTA) en el encabezado del archivo."

### Panel del Distribuidor

| Método | Ruta | Rol | Parámetros | Respuesta |
|---|---|---|---|---|
| GET | `/panel` | Admin, Dealer | `periodo=semana\|mes` (por defecto `semana`); `dealer_id` (obligatorio para el Admin) | `200` · `400` · `401` · `403` · `404` · `422` |

```json
{
  "dealer": { "id": "uuid", "nombre": "Magma Toluca", "activo": true },
  "periodo": { "tipo": "mes", "desde": "2026-09-02T10:02:01", "hasta": "2026-10-02T10:02:01" },
  "ordenes_en_periodo": 47,
  "cumplimiento_promesa": { "evaluables": 35, "a_tiempo": 5, "porcentaje": 14.3, "completadas_sin_promesa": 0 },
  "asesores": [
    { "asesor": "ASESOR A", "volumen": 29, "completadas": 24, "tiempo_promedio_dias": 10.5 }
  ],
  "ingresos": {
    "ordenes_facturadas": 35, "ordenes_sin_importe": 0,
    "total": "328201.42", "mano_obra": "83889.58", "refacciones": "133246.34",
    "porcentaje_mano_obra": 38.6, "porcentaje_refacciones": 61.4
  },
  "mezcla": {
    "tipo_ingreso": [
      { "clave": "garantia", "ordenes": 0 },
      { "clave": "mantenimiento", "ordenes": 42 },
      { "clave": null, "ordenes": 5 }
    ],
    "tipo_servicio": [ { "clave": "PAQUETE", "ordenes": 47 } ]
  }
}
```

El ejemplo procede de los datos de desarrollo del prototipo. Notas del contrato:
- Los importes son texto con dos decimales y nunca nulos (`"0.00"` si no hay órdenes).
- Pueden ser nulos el porcentaje de cumplimiento, el tiempo promedio y los dos porcentajes del desglose.
- `asesor: null` es "Sin asesor"; `clave: null` es "Sin tipo" en tipo de servicio y "Sin clasificar" en tipo de ingreso.
- La respuesta no incluye datos de cliente, VIN, placas ni números de orden.

## 8. Diferencias entre el prototipo y la especificación

| Tema | Prototipo | Especificación |
|---|---|---|
| Nombre de las órdenes | `/trucks`, clave `trucks` | `/orders`, clave `orders` |
| Errores de validación | Formato `detail`, estado 422 | Formato único |
| Dashboard | Sin identificador de dealer; orden en el navegador | Con identificador; orden y agregados del servidor |
| Historial | Sin paginación | Con paginación [DA-06] |
| Usuarios | Solo altas | Gestión [DA-04] |
| Sesión | Sin cierre en el servidor | Según DA-05 |
| Errores 500 conocidos | Dos casos | Ninguno |
