# MERP-110 — Despliegue: requisitos y plan
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 3.0 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente como especificación y plan — **NO ejecutado**
**Prototipo de referencia:** commit `5d77e51`.

> No existe cuenta de AWS, servidor, base administrada ni flujo de despliegue. Nada de lo que describe este documento está construido.
>
> En la versión nueva el despliegue **es un requisito desde el primer día**, no una fase final: la primera entrega ya debe poder desplegarse por el flujo automático. La Parte A fija los requisitos y el plan; la Parte B, lo que el prototipo tiene y por qué no se puede desplegar tal cual.

---

# Parte A — Especificación

## 1. Decisiones tomadas

| Decisión | Valor |
|---|---|
| Nivel | Simple y barato: piloto con un solo distribuidor |
| Proveedor | AWS, en una cuenta nueva y exclusiva del proyecto |
| Despliegue | Automático en cada cambio a la rama principal |
| Crecimiento | Si el piloto se aprueba, migración gradual a algo más robusto (§8) |

## 2. Requisitos de despliegue

| # | Requisito | Criterio de aceptación | Requisito general |
|---|---|---|---|
| D-1 | El repositorio tiene un remoto y un flujo que, en cada cambio, ejecuta pruebas y análisis estático. | Un cambio que rompe una prueba no llega a producción | RNF-014 |
| D-2 | El despliegue es automático y repetible desde el flujo; nadie copia archivos a mano. | Desplegar dos veces el mismo commit deja el mismo resultado | RNF-014 |
| D-3 | Todo el tráfico va por HTTPS con certificado válido y renovación automática. | HTTP redirige a HTTPS | RNF-009 |
| D-4 | CORS limitado al dominio del frontend, por configuración. | Un origen no listado es rechazado; cambiarlo no toca código | RNF-010 |
| D-5 | El servidor arranca en modo producción: sin recarga automática y sin sembrar datos. | En un despliegue nuevo no existe ningún usuario de demostración | RNF-011 |
| D-6 | El primer Admin se crea por un procedimiento documentado, con contraseña propia. | El procedimiento está escrito y probado | RNF-011 |
| D-7 | El frontend se sirve compilado, no con un servidor de desarrollo. | La imagen de producción no contiene herramientas de desarrollo | RNF-011 |
| D-8 | La base de datos no es alcanzable desde internet y está cifrada. | Un intento de conexión desde fuera de la red del servidor falla | RNF-012 |
| D-9 | Los secretos están fuera del repositorio y de las imágenes; los de producción son propios. | Búsqueda de secretos en el repositorio sin resultados | RNF-013 |
| D-10 | Servidor y base de datos en `America/Mexico_City`. | Una carga a las 10:00 queda registrada a las 10:00 | RNF-005 [DA-12] |
| D-11 | Respaldo automático diario, con una restauración probada antes del piloto. | Fecha y resultado de la restauración registrados | RNF-018 |
| D-12 | La cuenta de AWS tiene verificación en dos pasos y una alerta de costo. | Alerta configurada con un tope mensual | MERP-116 |
| D-13 | El acceso de administración al servidor está limitado a direcciones conocidas. | El puerto de administración no responde desde otra dirección | — |
| D-14 | Las migraciones se aplican en el despliegue y un fallo lo detiene de forma visible. | Un despliegue con una migración rota no deja el servicio a medias | RNF-011 |
| D-15 | Las construcciones son reproducibles. | RNF-021 | RNF-021 |

**Condición de salida a piloto:** D-1 a D-14 cumplidos y registrados (RNF-001).

## 3. Arquitectura del piloto

```
┌──────────────────────────────────────────────┐
│              Repositorio remoto               │
│   (cambio en main → pruebas → despliegue)     │
└───────────────────┬──────────────────────────┘
                    ▼
┌──────────────────────────────────────────────┐
│        EC2 — 1 instancia (t3.micro)           │
│  ┌──────────────┐  ┌───────────────────────┐  │
│  │   Backend    │  │  Frontend compilado,  │  │
│  │              │  │  servido por nginx    │  │
│  └──────┬───────┘  └───────────────────────┘  │
└─────────┼────────────────────────────────────┘
          ▼
┌──────────────────────────────┐   ┌──────────────────────────────┐
│ RDS PostgreSQL (db.t3.micro) │   │ S3 (solo si se decide        │
│ privada y cifrada            │   │ guardar el archivo) [DA-13]  │
└──────────────────────────────┘   └──────────────────────────────┘
```

**Por qué una base administrada y no Postgres dentro del servidor:** separar la base evita perder los datos si hay que reconstruir el servidor. En el piloto serán datos reales.

| Servicio | Para qué |
|---|---|
| EC2 (1 × t3.micro) | Backend y frontend en contenedores |
| RDS PostgreSQL (db.t3.micro, una zona) | Base de datos |
| S3 (1 bucket), solo si DA-13 lo decide | Respaldo del archivo subido: privado, cifrado y con borrado automático |
| Dirección IPv4 pública fija | Para que la dirección no cambie al reiniciar |
| Grupo de seguridad | Administración solo desde direcciones conocidas; 80 y 443 públicos; base sin acceso público |

## 4. Flujo de integración y despliegue

Proveedor previsto: GitHub Actions.

1. Cambio en la rama principal.
2. Pruebas de backend y frontend, compilación y análisis estático.
3. Construcción de las imágenes de producción.
4. Despliegue en el servidor y aplicación de migraciones.
5. Verificación de salud; si falla, el despliegue se marca como fallido.

| Secreto del flujo | Contenido |
|---|---|
| Dirección del servidor | Dirección pública |
| Llave de acceso | Para la conexión de despliegue |
| Usuario del servidor | |

## 5. Configuración de producción

Variables del servidor, fuera del control de versiones (lista general en MERP-113 §2):

| Variable | Valor en producción |
|---|---|
| Dirección de la base | La de la base administrada |
| Secreto de firma de sesiones | Propio de producción |
| Duración de la sesión | Según DA-05 |
| Orígenes permitidos | Solo el dominio del frontend |
| Entorno | Producción |
| Dirección pública de la API | Para compilar el frontend |
| Zona horaria | `America/Mexico_City` |

**Zona horaria en la base administrada:** RDS usa UTC por defecto y no lee la configuración de los contenedores. Hace falta un grupo de parámetros propio con `timezone = America/Mexico_City`.

## 6. Costo

**Corrección respecto a la v1.0.** La v1 estimaba "$0 durante el primer año" por la capa gratuita de 12 meses. Ya no aplica a las cuentas nuevas: reciben hasta USD 200 en créditos y un plan gratuito de 6 meses, y después se paga. **[VERIFICAR]** las condiciones vigentes en la página oficial de AWS antes de abrir la cuenta.

Estimación mensual, **aproximada y a confirmar con la calculadora oficial de AWS**:

| Concepto | USD al mes (aprox.) |
|---|---|
| EC2 t3.micro | 7.5 |
| RDS db.t3.micro, una zona | 13 |
| Almacenamiento | 4 |
| Dirección IPv4 pública | 3.6 |
| S3, gestor de secretos y DNS | 1 a 2 |
| **Total** | **29 a 30** (unos 350 al año) |

Los primeros 6 meses los créditos pueden cubrir casi todo. El dominio se paga aparte (unos USD 12 al año). Desglose y escenarios en MERP-116 §6.

## 7. HTTPS y dominio · [DA-15]

HTTPS es obligatorio (D-3) y requiere un dominio. Dos caminos:

| Opción | Costo adicional | Notas |
|---|---|---|
| Certificado gratuito (Let's Encrypt) en el propio servidor | Ninguno | Suficiente para el piloto |
| Balanceador con certificado administrado | Unos USD 16 a 22 al mes | Para la etapa de crecimiento |

Falta decidir el nombre de dominio y quién lo registra.

## 8. Ruta de crecimiento

Si el piloto se aprueba:

1. Contenedores administrados (ECS / Fargate) en lugar de un servidor único.
2. Balanceador con certificado administrado.
3. Secretos en un gestor de secretos.
4. Más de un distribuidor: redimensionar. Estimación sin cálculo detallado: USD 60 a 120 al mes.
5. Base en varias zonas (duplica el costo de la base) y plan de recuperación ante desastres.
6. Retomar la integración con el Excel / Power BI interno de Magma [DA-17].

---

# Parte B — Prototipo

## 9. Por qué el prototipo no se puede desplegar tal cual

Comprobado en el código de `5d77e51`:

| # | Situación | Requisito que incumple |
|---|---|---|
| 1 | **CORS fijo en el código**: `allow_origins=["*"]` en `backend/app/main.py`. No hay variable que lo controle; cerrarlo exige cambiar código. | D-4 |
| 2 | **Arranque de desarrollo**: `docker-entrypoint.sh` ejecuta `uvicorn --reload`. | D-5 |
| 3 | **El arranque siembra datos de demostración**, con usuarios y contraseñas conocidos, e ignora el fallo del sembrado (`\|\| true`). | D-5, D-6 |
| 4 | **El frontend corre el servidor de desarrollo de Vite** (`npm run dev`). No hay compilación de producción ni configuración de nginx. | D-7 |
| 5 | **`docker-compose.yml` es de desarrollo**: incluye la base en un contenedor, monta el código del equipo y publica 5433, 8000 y 5173. | D-8 |
| 6 | **Sin remoto y sin flujo automático**: no hay remoto de git ni carpeta `.github/`. | D-1, D-2 |
| 7 | **Zona horaria resuelta solo en local**, en `docker-compose.yml`. | D-10 |
| 8 | El secreto y la duración de la sesión se leen en cada petición, no al arrancar; si faltan, el fallo aparece al usar el sistema. | MERP-113, E-6 |
| 9 | La imagen del backend instala una herramienta con la etiqueta `latest`; la del frontend usa otra versión de Node que el equipo. | D-15 |
| 10 | No hay código que guarde el archivo subido. | Solo relevante si DA-13 decide guardarlo |

## 10. Nombres de variables del prototipo

La v1 de este documento usaba nombres que no coinciden con el código:

| v1 de este documento | Código del prototipo |
|---|---|
| `JWT_SECRET` | `JWT_SECRET_KEY` |
| `CORS_ORIGINS` | **No existe** |
| — | `JWT_EXPIRATION_MINUTES`, obligatoria |
