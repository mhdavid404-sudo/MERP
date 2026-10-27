# MERP-111 — Eliminación del rol Empleado (documento de cambio)
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.1 · **Fecha:** 8 de octubre de 2026 · **Estado:** histórico

> **HISTÓRICO — superado por MERP-100 §4 y MERP-102.** Este documento registra la decisión de eliminar el rol Empleado y cómo se aplicó en la fase M6. Se conserva sin cambios de contenido como registro. Los roles vigentes están en MERP-100 §4; las reglas, en MERP-102 (RN-005 y RN-006 retiradas; RN-012, RN-014 y RN-015 vigentes); el alta de usuarios Dealer, en MERP-105 (CU-013).

---

**Motivo:** Con el rediseño de MERP-108 (carga de Excel en vez de captura manual), el Dealer es quien sube el archivo — ya no se necesita un rol Empleado separado para operar el sistema.

---

## 1. Roles finales del sistema (reemplaza MERP-100 §5)

| Rol | Descripción |
|---|---|
| **Admin (Magma)** | Da de alta/baja/actualiza dealers. Autoriza nuevos admins. Ve dashboard de todos los dealers. Puede ver el Panel del Distribuidor (MERP-109) de cualquier dealer. |
| **Dealer** | Sube el archivo Excel (MERP-108). Ve el dashboard y el Panel del Distribuidor de su propio dealer. |

Ya no existe el rol **Empleado**.

## 2. Qué se elimina

- El rol `empleado` del enum `user_role` (MERP-107)
- CU-005 (alta de empleado) y CU-006 (baja de empleado) de MERP-105 — ya no aplican
- Endpoints `POST /users/empleados`, `GET /users/empleados`, `DELETE /users/empleados/{id}` de MERP-106
- La vista "Gestión de Empleados" del frontend (Fase 7 original)
- RN-005 y RN-006 (aislamiento de empleados por dealer) de MERP-102 — se simplifican, ya que ahora el aislamiento por dealer solo aplica al propio Dealer (un Dealer solo ve/opera lo suyo, ya no hay empleados de por medio)

## 3. Qué cambia (en vez de eliminarse)

- `POST /uploads` (MERP-108 §4): el rol permitido pasa de `dealer, empleado` a solo **`dealer`**
- El `CHECK` de la tabla `users` (MERP-107) se simplifica: `dealer_id` es NULL solo para admin, obligatorio para dealer (ya no hay tercer caso)

## 4. Migración de datos existentes

Si ya existen usuarios con rol `empleado` en la base de datos (del prototipo original), deben:
- Reasignarse como `dealer` si corresponde a una persona real que seguirá usando el sistema, o
- Darse de baja (lógica) si eran solo datos de prueba

Esto lo decide quien ejecute la migración — no hay una regla automática, ya que depende de si esos usuarios de prueba tienen sentido en el dealer real.

## 5. Aplicado en M6

Migración `c4d8f1a29b36` (down_revision `a7c3e91d5b20`): los empleados existentes pasaron a rol `dealer` con `activo=false` (sin excepción) y el enum `user_role` quedó solo con `admin` y `dealer`; el CHECK de `users` se recreó sin el tercer caso. El `downgrade` recrea el enum con `empleado` y el CHECK original, pero **no recupera quiénes eran empleados**: siguen como dealer inactivos. Los ex-empleados conservan su correo (UNIQUE): si alguien necesita acceso hay que usar otro correo o reactivarlo a mano en la base.

Se agregó `POST /users/dealers` (solo admin) para crear usuarios de dealer (MERP-106), y los casos de uso que filtran por dealer (GET /trucks, /trucks/{id}, /dashboard, /uploads, POST /uploads) responden 403 si el usuario no tiene dealer asignado.
