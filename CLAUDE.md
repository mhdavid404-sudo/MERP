# MERP — Reglas del proyecto

Reconstrucción de MERP desde cero. El prototipo (`C:\Users\User\Desktop\dev\ERP`, commit `5d77e51`) está congelado: se puede leer como referencia, pero nunca se modifica ni se copia su código, sus datos ni sus archivos de Excel.

## Fuente de verdad

- La documentación vigente es `docs/MERP/` (MERP-100 a MERP-117 y `ANOTACIONES_RECONSTRUCCION.md`).
- Donde `docs/MERP/ANOTACIONES_RECONSTRUCCION.md` contradiga a los documentos MERP, manda ANOTACIONES. Cada contradicción que se aplique se anota en el informe de la fase.
- El diseño visual sigue `docs/DESIGN.md`. `docs/referencias/` es solo material de consulta, no fuente vigente.

## Trabajo por fases

El plan de fases está en `docs/MERP/ANOTACIONES_RECONSTRUCCION.md` §5 (F0 a F8). Cada fase sigue este orden:

1. **Plan** — Benja escribe el plan en `.claude/plans/` (`F<n>-plan.md`), con criterios de aceptación y los identificadores de requisitos, reglas y decisiones que cubre (RC-21, RC-25).
2. **Revisión del plan** — Isai lo revisa antes de escribir código.
3. **Construcción** — Valde (backend), Leo (frontend) y Royer (Docker, CI y despliegue).
4. **Pruebas** — Cofi.
5. **Revisión de seguridad** — Ramon, de solo lectura, al final de la fase (RC-27).
6. **Cierre** — un solo commit por fase, con pruebas, compilación y análisis estático en verde. El informe de la fase queda en `.claude/plans/` (`F<n>-informe.md`).

Reglas:

- No se construye nada fuera de la fase en curso.
- Si una tarea depende de una decisión abierta (MERP-117 §5, pendientes de ANOTACIONES §4), se detiene y se pregunta al dueño del proyecto. No se supone la respuesta (RC-26).
- Los agentes no modifican `docs/MERP/` fuera del criterio de RC-19 (RC-28).

## Restricciones obligatorias

- Se cumplen las restricciones de construcción de MERP-115 (RC-01 a RC-36).
- Se cumplen los requisitos no funcionales de MERP-101 (RNF).
- **Nunca se usan datos reales**: ni en el repositorio, ni en la base de desarrollo, ni en pruebas. Las pruebas usan datos sintéticos o copias anonimizadas en `tests/fixtures/` (RC-15, RC-31, RNF-023).
- Los textos visibles para el usuario van en español.

## Repositorio

- Remoto: https://github.com/mhdavid404-sudo/MERP.git (rama `main`).
- No se hace push sin confirmación del dueño del proyecto.
- Finales de línea LF (`.gitattributes`).
