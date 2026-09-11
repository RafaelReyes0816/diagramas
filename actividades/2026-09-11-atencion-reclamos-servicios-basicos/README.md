# Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto

La empresa de servicios básicos (agua y electricidad) requiere un sistema que gestione el ciclo de vida completo de los reclamos de sus usuarios. El flujo inicia cuando un usuario presenta un reclamo por **corte de servicio**, **error de facturación**, **fuga** o **falla técnica**, continúa con la **clasificación por tipo y urgencia**, la **asignación a cuadrillas técnicas o al área comercial**, y termina con el **seguimiento y cierre** del caso dentro de los **plazos regulatorios**.

Las cuadrillas técnicas y el área comercial operan fuera del sistema de información; interactúan mediante reportes de avance y órdenes de trabajo. La entidad reguladora define los plazos máximos de atención y recibe reportes periódicos.

## Índice de modelos (Metodología Yourdon, estándar SI-III)

| # | Modelo | Archivo |
|---|--------|---------|
| 01 | Modelo Ambiental: Lista de Acontecimientos (F/T/C) — **fuente única de eventos** | [01-lista-eventos.md](01-lista-eventos.md) |
| 02 | DFD Nivel 0 (Diagrama de Contexto / Frontera) | [02-dfd-nivel-0.md](02-dfd-nivel-0.md) |
| 03 | DFD Nivel 1 (Subsistemas P1–P5) | [03-dfd-nivel-1.md](03-dfd-nivel-1.md) |
| 04 | DFD Nivel 2 (Funciones P1.x–P5.x) | [04-dfd-nivel-2.md](04-dfd-nivel-2.md) |
| 05 | DFD Nivel 3 (Subfunciones P1.1, P2.1, P2.2, P3.4) | [05-dfd-nivel-3.md](05-dfd-nivel-3.md) |
| 06 | Procesos primitivos | **No aplica** — la descomposición llegó a procesos atómicos en el Nivel 3; los primitivos se consolidan y especifican en el `09`. |
| 07 | Modelo de Datos: DER | [07-erd.md](07-erd.md) |
| 08 | Diccionario de Datos (DD) | [08-diccionario-datos.md](08-diccionario-datos.md) |
| 09 | Especificaciones de procesos primitivos | [09-especificaciones.md](09-especificaciones.md) |

## Resumen de la arquitectura (Nivel 1)

| Subsistema | Responsabilidad | Eventos |
|---|---|---|
| P1 — Registrar y Consultar Reclamos | Registro, comprobante, consulta de estado, contacto | F1–F4 |
| P2 — Clasificar y Asignar Reclamos | Clasificación, plazos, asignación/derivación, normativa | F8 |
| P3 — Seguir y Cerrar Reclamos | Avances, resoluciones, cierre, notificación | F5–F7 |
| P4 — Vigilar Plazos Regulatorios | Avisos de vencimiento, alertas, alarmas críticas | T1, T2, C1 |
| P5 — Generar Reportes | Reporte operativo diario y regulatorio mensual | T3, T4 |

## Checklist metodológico

- [x] `01-lista-eventos.md` generado antes de cualquier DFD y no duplicado en los archivos de los DFD.
- [x] Frontera y Entidades Externas definidas en el Modelo Ambiental.
- [x] Descomposición top-down rigurosa P1 → P1.x → P1.1.x; sin forzar Nivel 4.
- [x] Balanceo de flujos verificado entre nivel padre e hijo.
- [x] Separación asíncrona mediante Almacenes de Datos entre subsistemas.
- [x] DER y DD generados como modelo de información independiente (no son niveles del DFD).
- [x] Notación algebraica del DD correcta (`=`, `+`, `[|]`, `{}`, `()`).