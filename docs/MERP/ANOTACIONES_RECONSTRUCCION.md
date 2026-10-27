# Anotaciones para la documentación de la reconstrucción de MERP

**Fecha:** 26 de octubre de 2026
**Uso:** notas de trabajo, no documento oficial. Todo lo de aquí se pasa a la serie MERP-100 a MERP-117 cuando se actualice la documentación.

---

## 1. Decisiones cerradas

| ID | Tema | Decisión |
|---|---|---|
| DA-01 | Semáforo | Días de calendario |
| DA-02 | `diag`, `repme`, `THYP` | Sin clasificar hasta que Magma confirme; correspondencias editables y reclasificación posible |
| DA-03 | Datos del cliente final | Se guardan **cifrados** (llave en Azure Key Vault); ninguna pantalla, reporte, PDF ni correo los muestra. Uso futuro solo con un caso de uso nuevo y documentado. Debe ser cifrado real, no codificación |
| DA-04 | Gestión de usuarios | Solo el Admin: crea, da de baja, reactiva y manda restablecimientos |
| DA-05 | Sesión y contraseñas | Invitación por correo al crear usuario; el Admin nunca escribe contraseñas. "Olvidé mi contraseña" para cualquier usuario. Mínimo 10 caracteres y al menos un carácter especial. Enlace de un solo uso, caduca (1 h restablecer, 72 h invitación). Respuesta igual exista o no el correo. Al cambiar contraseña se cierran las sesiones. Usuarios o dealers dados de baja no recuperan acceso. Límite de solicitudes y de intentos. Registro de cada envío |
| DA-06 | Historial de cargas | Paginado, con filtro por dealer para el Admin |
| DA-07 | Dashboard | Agrupado por dealer, orden alfabético entre grupos, urgencia dentro de cada grupo; filtro por dealer; orden y agregados los calcula el servidor |
| DA-08 | Órdenes que dejan de venir | Se marcan "no reportada"; no se inventa un cierre |
| DA-09 | Datos ilegibles en el Excel | Se carga la fila con aviso; se rechaza solo si falta un obligatorio o la fecha de entrada |
| DA-10 | VIN | Mayúsculas; un cambio entre cargas se permite con aviso |
| DA-11 | Nombre de dealer | Único, sin distinguir mayúsculas |
| DA-12 | Fechas | Con zona horaria; el archivo se interpreta como hora de planta |
| DA-13 | Excel original | No se guarda |
| DA-14 | Punto de partida | Python + FastAPI, PostgreSQL, React + TypeScript, Docker. Sin PHP. Repo nuevo con remoto. Base vacía. Prototipo congelado en `5d77e51`, sin ejecutar M8 |
| DA-15 | Dominio y HTTPS | Certificado gratuito; nombre de dominio pendiente (sistemas) |
| DA-16 | Bitácora | Altas y bajas de dealers y usuarios |
| DA-17 | Fuera de alcance | Integración con Power BI y fusión con ATLAS. El correo entra |
| DA-18 | Campos obligatorios | Orden, VIN y fecha de entrada (autorizados por Magma). Advertencia si un perfil no asigna columna de fecha de cierre |
| DA-19 | Formatos de archivo | `.xlsx` con encabezado en cualquier fila |
| DA-20 | Revertir perfil | El Admin revierte a una versión anterior (crea versión nueva); no edita |
| DA-21 | Asistente del diccionario | Al llegar el segundo distribuidor; en el piloto, perfil cargado a mano |
| DA-22 | Archivo del segundo distribuidor | Pendiente de preguntarle; no se configura hasta saberlo |

## 2. Decisiones del peloteo

### Infraestructura
- Nube: **Azure**, por el convenio con Microsoft (sustituye a AWS en MERP-110 y MERP-116).
- Correo: servicio de Azure o Microsoft 365 de Magma; dirección remitente pendiente (sistemas).
- Presupuesto enviado en tres planes: piloto ~60-65 USD/mes, producción ~140-150, alta disponibilidad ~290-300. Precios de lista, aproximados.
- Observación: la serie B de Azure baja el rendimiento con carga sostenida; para varios distribuidores evaluar serie D.
- El tamaño del servidor se fija con la prueba de carga, no con estimaciones.
- Estimación de datos: con 10 distribuidores, la base crece menos de 1 GB al año.
- **Dimensionamiento confirmado:** 25 distribuidores al inicio, con crecimiento; unas 50 personas usándolo. Con 25 dealers la base crece unos 225 MB al año.
- **Plan base: plan 2 (producción)** con servidor de rendimiento constante (serie D), no serie B.
- **Disponibilidad:** no crítico, pero que no se caiga. Sin duplicar todo (no plan 3): reinicio automático de contenedores, monitoreo con alertas, respaldos diarios con restauración probada.
- **Convenio con Microsoft:** hay descuento; falta el porcentaje (sistemas). Cotizar a precio de lista y aplicar el descuento.
- **Prueba de carga:** se corrió con el prompt original (hasta 20 dealers y 50 usuarios). Falta una segunda corrida con 25 y 50 dealers, y 100 usuarios, para cubrir el crecimiento.
- Pendiente: lineamientos de marca de Magma (colores, logotipo, tipografía) para crear el DESIGN.md de MERP; Refero Styles solo como inspiración.

### Datos y carga
- El Excel **sí traerá órdenes abiertas** (confirmado). El archivo real recibido solo trae cerradas: hace falta un archivo de muestra con abiertas antes de F3.
- Campos estándar de MERP = solo los usados del primer distribuidor: orden (`ORDEN`), VIN (`SERIE`), fecha de entrada (`ALTA`), fecha de cierre (`CIERRE`), promesa (`PROMESA` + `HRA.PROM`), modelo (`UNIDAD`), año (`MOD`), placas (`PLACA`), asesor (`ASESOR`), tipo de servicio (`TIPO SERV`), tipo de ingreso (`TDS1`), total, mano de obra y refacciones (`TOTAL`, `M.OBRA`, `REF`).
- Las demás columnas no se guardan, salvo `TDS1` y `THYP` en crudo.
- Diccionario = perfil de importación (ya documentado: RF-018, RN-021 a RN-029).

### Correo de carga
- Cada vez que un Dealer sube, correo a **todos los Admin**.
- Contenido: dealer, quién subió, fecha, nuevas, actualizadas, sin cambios, rechazadas, si cambió el perfil, enlace al detalle. **Sin adjunto.**

### Reportes PDF
- **PDF de la carga:** se genera solo en cada carga, queda en el historial; el correo enlaza a él.
- **PDF semanal por dealer:** resumen de las cargas de la semana. Semana lunes a domingo; se genera el domingo en la noche. Con una carga, sale con esa; sin cargas, sale marcando "sin cargas esta semana".
- Lo descargan el Dealer (el suyo) y el Admin (cualquiera).
- Sin datos del cliente final.
- Secciones: encabezado; resumen del semáforo; las 10 más atrasadas; cumplimiento de promesa; facturación (aclarando que el total incluye IVA); desempeño por asesor; última carga. El semanal agrega: estado al final de la semana, qué pasó entre cargas (cerradas, nuevas, pasaron de amarillo a rojo), las cargas lado a lado y comparación con la semana anterior.
- **Resumen del semáforo (aprobado):** fila de cifras (en proceso, verde, amarillo, rojo, completadas en la semana), comparación con la semana anterior, barra horizontal por color y una línea de lectura generada sola.

### Gráficas
- Lista de las unidades en proceso con más tiempo: VIN, modelo, orden, dealer, días.
- **Unidades paradas (solo Admin):** todas las unidades en proceso por antigüedad. Barras apiladas por dealer, ordenadas de más a menos. Tramos: 0-4, 5-7, 8-15, 16-30, más de 30 días. Clic en un tramo muestra la lista.
- Colores: verde, amarillo y rojo reservados al semáforo; el resto de gráficas con otra paleta.

### Predicciones
- Facturación esperada y cuántas órdenes se liberan la próxima semana.
- Se diseña desde el inicio para guardar la historia; se activan con 3 a 6 meses de cargas.

## 3. Hallazgos del Excel real del primer distribuidor

- 40 columnas, una hoja, 62 órdenes, **todas cerradas**, todas cerradas en septiembre.
- `TOTAL` = `M.OBRA` + `REF` + `TTF` + `OTR` + `IVA` en las 62 filas; IVA del 16 %. El total incluye IVA.
- Cumplimiento de promesa: 7 de 62 a tiempo. Confirmar qué significa `PROMESA`.
- `ASESOR` trae nombres completos de empleados del distribuidor.
- `TELS` y `CONTACTO` vacíos; `CLIENTE` y `EMAIL` llenos.
- Un VIN de 9 caracteres; `UNIDAD` numérica en 4 filas; `P&D`, `REEQ` y `PK` vacías.
- Semáforo calendario contra 24 h: 9 órdenes cambian de conteo, 1 cambia de color.

## 4. Pendientes

| Pendiente | De quién |
|---|---|
| Archivo de muestra con órdenes abiertas | Distribuidor |
| Qué cubre el convenio de Azure | Sistemas |
| Servidor de pruebas o resultados antes de rentar | Sistemas |
| Porcentaje de descuento del convenio de Azure | Sistemas |
| Lineamientos de marca (colores, logotipo, tipografía) | Magma |
| Dirección remitente de los correos | Sistemas |
| Nombre de dominio | Sistemas |
| Significado de `FechaFsct`, `Status`, `Tipo` (DA-22) | Segundo distribuidor |
| Significado de `diag`, `repme`, `THYP`, `PROMESA`, `TTF` | Magma |
| Retención de datos y aviso de privacidad | Magma, área legal |
| Resultados de la prueba de carga del prototipo | Dueño del proyecto |
| Avisar a Claude Code que no ejecute el paso de documentos de M8 | Dueño del proyecto |

## 5. Plan de construcción (borrador)

| Fase | Contenido | Depende de |
|---|---|---|
| F0 | Repo nuevo, Docker desarrollo y producción, configuración validada, CI | — |
| F1 | Inicio de sesión, roles, invitación, recuperación, contraseñas, límite de intentos, correo | — |
| F2 | Dealers y gestión de usuarios | — |
| F3 | Perfiles de importación (diccionario) y carga | Archivo con órdenes abiertas |
| F4 | Dashboard, semáforo, gráficas nuevas | F3 |
| F5 | Panel del Distribuidor | F3 |
| F6 | Reportes PDF y correo de carga | F4, F5 |
| F7 | Pruebas de carga y despliegue en Azure | Servidor rentado |
| F8 | Predicciones | 3 a 6 meses de datos |

## 6. Documentos que habrá que actualizar

MERP-100, 101, 102, 103, 104, 105, 106, 107, 109, 110 (reescribir para Azure), 112, 113, 114, 116 (presupuesto y cifrado) y 117.

## 7. Ya pasado a la documentación (26-oct-2026)

Hallazgos de la prueba de carga del prototipo:
- MERP-101: RNF-025 (25 distribuidores y crecimiento), RNF-026 (tiempos de respuesta), RNF-027 (varios procesos en producción), RNF-028 (Argon2 ajustado, inicio de sesión < 0.5 s), RNF-029 (prueba de carga antes del piloto y reglas para medir bien).
- MERP-113: T-7 (prueba de carga repetible y versionada).
- MERP-114: grupo L (L-01 a L-08).
- MERP-115: lecciones 21 a 24 y restricciones RC-29 a RC-33.
- Informe completo de la prueba (26-oct-2026, noche): MERP-101 RNF-030 (dashboard sin historia completa), RNF-031 (conexiones a la base), RNF-032 (éxito solo después de guardar) y cifras actualizadas en RNF-026; MERP-114 L-09 a L-14; MERP-115 lecciones 25 a 27 y RC-34 a RC-36.

El resto de este archivo sigue pendiente de pasar a la documentación.
