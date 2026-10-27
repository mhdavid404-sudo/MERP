# MERP-116 — Datos personales y seguridad
## Sistema de trazabilidad de órdenes de taller — Grupo Magma Automotive

**Versión:** 1.0 · **Fecha:** 8 de octubre de 2026 · **Estado:** vigente
**Prototipo de referencia:** commit `5d77e51`.
**Destinatario:** Grupo Magma Automotive.

> Este documento explica qué información sensible maneja MERP, qué protecciones tiene ya el prototipo, cuáles faltan, qué se exigirá a la versión de producción y cuánto cuesta operarla. Las secciones 2 y 3 describen el prototipo tal como está; las secciones 4 y 5 son requisitos y propuestas para la versión nueva. Todo lo afirmado sobre el prototipo se comprobó en su código.
>
> Este documento no interpreta la ley. El aviso de privacidad, el responsable de los datos y las obligaciones legales los define el área legal de Magma.

---

## 1. Qué información sensible maneja el sistema

| Información | Dónde está | Quién la ve hoy |
|---|---|---|
| **Datos del cliente final del distribuidor**: nombre del cliente, contacto, teléfono y correo | Se guardan con cada orden, tomados del archivo del distribuidor | **Nadie.** Ninguna pantalla ni respuesta del sistema los muestra |
| **Otras columnas del archivo** que el sistema no usa (20 columnas, por ejemplo `FACTURA`, `KIL` o `COLOR`) | Se guardan con cada orden como respaldo sin procesar | Nadie. **[VERIFICAR]** con Magma si alguna contiene datos personales |
| **VIN y placas** del vehículo | Con cada orden | El distribuidor dueño de la orden y el Admin, a través de la consulta de órdenes |
| **Nombre del asesor** y **tipo de servicio** | Con cada orden. Son texto libre que escribe el distribuidor | Quien vea el Panel de ese distribuidor. Si el distribuidor escribe ahí datos de una persona, quedan a la vista |
| **Importes**: total, mano de obra, refacciones | Con cada orden | Sumados, en el Panel de ese distribuidor |
| **Usuarios del sistema**: nombre, correo y contraseña | Tabla de usuarios. La contraseña se guarda solo como hash (Argon2); el sistema no puede leerla | El correo propio, en pantalla. El hash no sale en ninguna respuesta |
| **El archivo de Excel original** | **No se guarda.** Solo se guardan las órdenes y el registro de la carga | — |

Si en el futuro se decide conservar el archivo original (por ejemplo en S3), pasa a ser el elemento más sensible del sistema, porque contiene todo lo anterior junto.

**Punto a confirmar. [VERIFICAR]** La base de datos de desarrollo tiene un distribuidor de prueba con 62 órdenes, el mismo número de filas que el archivo real. Si esa carga se hizo con el archivo real, hay datos personales de clientes en un equipo de desarrollo y deben borrarse.

## 2. Protecciones que ya existen en el prototipo

| Protección | Cómo funciona | Comprobado en |
|---|---|---|
| Contraseñas con hash Argon2 | La contraseña nunca se guarda en claro ni se devuelve | Código; 3 pruebas |
| Sesión firmada | Tras iniciar sesión se emite un token firmado con el rol y el distribuidor del usuario, con caducidad | Código; 3 pruebas |
| Revalidación en cada petición | En cada petición se vuelve a consultar si el usuario y su distribuidor siguen activos; no basta con tener un token | Código; pruebas de RN-011 |
| Solo dos roles | Admin (Magma) y Dealer (distribuidor). Sin roles intermedios | Código |
| Aislamiento entre distribuidores | Un distribuidor solo ve lo suyo. Pedir una orden ajena responde igual que una inexistente; pedir el Panel ajeno se rechaza sin revelar si existe | Pruebas de aislamiento por cada consulta |
| El distribuidor de cada orden no viene del archivo | Se toma del usuario que sube. Un archivo no puede escribir en otro distribuidor | Código; 2 pruebas |
| Bajas lógicas | Dar de baja un distribuidor no borra nada | Código; pruebas |
| Corte de acceso al dar de baja un distribuidor | Sus usuarios dejan de entrar y sus sesiones abiertas dejan de servir. Probado en vivo el 8 de octubre de 2026 | RN-011; pruebas |
| Mensaje único en el inicio de sesión | Correo inexistente, contraseña incorrecta y usuario inactivo responden lo mismo | Código; pruebas |
| Validación del archivo | Solo `.xlsx`, máximo 5 MB, columnas obligatorias. Una fila inválida se rechaza sola con su motivo | Código; pruebas |
| Respuestas sin datos del cliente final | Ni la consulta de órdenes ni el Panel los incluyen | Código; 1 prueba específica del Panel |
| Limpieza del navegador entre sesiones | Al iniciar y al cerrar sesión se borran los datos en memoria | Código |
| El archivo real no está en el repositorio | Está excluido del control de versiones; las pruebas usan una copia anonimizada | Configuración del repositorio |
| Reglas reforzadas en la base de datos | La base impide órdenes duplicadas, fechas incoherentes y usuarios sin distribuidor | Código; pruebas |

## 3. Lo que NO existe todavía

El prototipo corre en un equipo local. Para exponerlo en internet con datos reales falta:

| Carencia | Riesgo | Se resuelve con |
|---|---|---|
| **Sin HTTPS** | Contraseñas y datos viajarían sin cifrar | RNF-009 |
| **CORS abierto a cualquier origen**, fijo en el código | Cualquier sitio web podría llamar a la API desde el navegador de un usuario | RNF-010 |
| **Sin límite de intentos de inicio de sesión** | Se pueden probar contraseñas sin freno | RNF-015 |
| **El tiempo de respuesta del inicio de sesión delata qué correos existen** | Facilita localizar cuentas válidas | RNF-016 |
| **No se puede dar de baja a un usuario ni cambiar una contraseña** | No hay forma de retirar el acceso a una persona sin dar de baja a todo su distribuidor | RF-015, RF-016 |
| **Sesión**: caducidad configurable (la plantilla propone 60 minutos; **[VERIFICAR]** el valor en uso), sin cierre en el servidor, guardada en el almacenamiento local del navegador | Un token robado sirve hasta que caduca | DA-05 |
| **Política de contraseñas mínima**: solo 8 caracteres | Contraseñas débiles | DA-05 |
| **Usuarios y contraseñas de demostración conocidos**, creados al arrancar | En producción serían una puerta abierta | RNF-011 |
| **Sin registro de cambios** | No se sabe cuándo ni quién modificó un dato | RNF-019, DA-16 |
| **Sin política de retención** | Los datos se acumulan sin límite | DA-03 |
| **Sin respaldos probados** | Una pérdida sería definitiva | RNF-018 |
| **Dos errores internos conocidos** en casos límite | Bajo; se corrigen en la versión nueva | MERP-114, D-06 y D-07 |

La lista completa, con su clasificación, está en MERP-114.

## 4. Requisitos para producción

Son requisitos de la versión nueva desde el inicio, no mejoras posteriores. El sistema no recibe datos reales hasta cumplirlos (RNF-001).

| # | Requisito | Referencia |
|---|---|---|
| 1 | Todo el tráfico por HTTPS | RNF-009; MERP-110, D-3 |
| 2 | CORS limitado al dominio del sistema, por configuración | RNF-010 |
| 3 | Base de datos sin acceso desde internet y cifrada | RNF-012 |
| 4 | Secretos en un gestor de secretos o, en el piloto, en el servidor fuera del repositorio; distintos de los de desarrollo | RNF-013 |
| 5 | Respaldos automáticos con una restauración probada | RNF-018 |
| 6 | Cuenta de AWS con verificación en dos pasos y alerta de costo | MERP-110, D-12 |
| 7 | Límite de intentos de inicio de sesión | RNF-015 |
| 8 | Inicio de sesión que no delate qué correos existen | RNF-016 |
| 9 | Arranque de producción sin datos ni usuarios de demostración | RNF-011 |
| 10 | Gestión de usuarios: poder retirar el acceso a una persona | RF-015 |
| 11 | Si se guarda el archivo original: almacenamiento privado, cifrado y con borrado automático | DA-13 |
| 12 | Ningún dato real fuera de producción | RNF-023 |
| 13 | Una revisión final de seguridad antes del piloto, con resultado registrado | RNF-001 |

## 5. Información que no se utiliza — propuesta para Magma

Hoy el sistema guarda el nombre, el contacto, el teléfono y el correo del cliente final de cada orden, y **ninguna pantalla los usa**. Guardar datos personales sin un uso concreto es un riesgo sin beneficio.

**Propuesta, sujeta a la aprobación de Magma. No es un cambio hecho.**

| # | Propuesta | Efecto |
|---|---|---|
| 1 | No guardar los cuatro datos del cliente final mientras no exista un caso de uso que los necesite. | El sistema deja de contener datos de contacto de clientes |
| 2 | Del respaldo de columnas sin procesar, conservar solo lo necesario: `TDS1` y `THYP`, que permiten reclasificar el tipo de ingreso. Eliminar las columnas que contengan datos personales. | El respaldo deja de ser una copia casi completa del archivo |
| 3 | Documentar para los distribuidores que los campos de asesor y de tipo de servicio no deben llevar datos personales de clientes. | Reduce el riesgo del texto libre en el Panel |
| 4 | Definir una política de retención: cuánto tiempo se conservan las órdenes y las cargas, y qué se hace después. Hoy no existe. | Los datos no se acumulan indefinidamente |
| 5 | Aviso de privacidad y responsable de los datos: a definir por el área legal de Magma. | — |

Estas propuestas corresponden a la decisión abierta **DA-03** de MERP-117. Si Magma decide conservar los datos del cliente final, deberá indicar para qué se usarán y quién podrá verlos.

## 6. Presupuesto mensual estimado

**Todas las cifras son aproximadas y deben confirmarse con la calculadora oficial de AWS antes de comprometer un presupuesto.**

### 6.1 Piloto con un distribuidor

| Concepto | USD al mes (aprox.) |
|---|---|
| Servidor (EC2 t3.micro) | 7.5 |
| Base de datos administrada (RDS db.t3.micro, una zona) | 13 |
| Almacenamiento | 4 |
| Dirección IPv4 pública | 3.6 |
| S3, gestor de secretos y DNS (Route 53) | 1 a 2 |
| **Total** | **29 a 30** |
| **Al año** | **Unos 350** |

- Las cuentas nuevas de AWS reciben hasta USD 200 en créditos y un plan gratuito de 6 meses; durante esos meses los créditos pueden cubrir casi todo. Después se paga. **[VERIFICAR]** las condiciones vigentes al abrir la cuenta.
- El dominio se paga aparte: unos USD 12 al año.
- HTTPS no tiene costo si se usa un certificado gratuito (Let's Encrypt) en el propio servidor.

### 6.2 Opciones que aumentan el costo

| Opción | Efecto aproximado |
|---|---|
| Balanceador de carga con certificado administrado | Suma USD 16 a 22 al mes |
| Base de datos en varias zonas | Duplica el costo de la base |
| Varios distribuidores | USD 60 a 120 al mes. **Estimación sin cálculo detallado** |

El plan técnico de despliegue está en MERP-110.
