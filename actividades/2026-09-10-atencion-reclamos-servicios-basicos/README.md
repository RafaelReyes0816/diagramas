# Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto

La empresa de servicios básicos (agua y electricidad) necesita un sistema que gestione el ciclo de vida completo de los reclamos de sus usuarios. El flujo inicia cuando un usuario presenta un reclamo por **corte de servicio**, **error de facturación**, **fuga** o **falla técnica**, continúa con la **clasificación por tipo y urgencia**, la **asignación a cuadrillas técnicas o al área comercial**, y termina con el **seguimiento y cierre** del caso dentro de los **plazos regulatorios**.

Las cuadrillas técnicas y el área comercial operan fuera del sistema de información; interactúan con él mediante reportes de avance y órdenes de trabajo. La entidad reguladora define los plazos máximos de atención y recibe reportes periódicos.

## Lista de Eventos

### Flujo externo (F)
| ID | Evento | Fuente | Destino |
|----|--------|--------|---------|
| F1 | Solicitud de Reclamo | Usuario | Sistema |
| F2 | Consulta de Estado | Usuario | Sistema |
| F3 | Reporte de Avance de Trabajo | Cuadrilla Técnica | Sistema |
| F4 | Resolución Comercial | Área Comercial | Sistema |
| F5 | Regulación y Plazos (normativa vigente) | Entidad Reguladora | Sistema |
| F6 | Recepción de dato de contacto actualizado | Usuario | Sistema |

### Temporal (T)
| ID | Evento | Descripción |
|----|--------|-------------|
| T1 | Vencimiento de plazo regulatorio | Al detectar que un reclamo alcanza el umbral de días, el sistema genera la alerta antes del vencimiento |
| T2 | Emisión de reportes periódicos | Generación automática de reportes de reclamos (diario operativo, mensual regulatorio) |

### Control (C)
| ID | Evento | Descripción |
|----|--------|-------------|
| C1 | Alerta de vencimiento próximo | Aviso interno antes de exceder el plazo (umbral configurable, ej. 20% del plazo) |
| C2 | Alarma de reclamo crítico | Reclamo con riesgo vital o mayor (fuga de agua de alto caudal, corte prolongado) que exige escalamiento inmediato |
| C3 | Reenvío/reimpresión de comprobante | El usuario solicita nuevamente su comprobante de reclamo |

## Requerimientos funcionales clave

1. Recepción y registro del reclamo con comprobante único por canal (presencial, telefónico, web).
2. Clasificación por tipo de servicio (agua/luz), categoría (corte, facturación, fuga, falla) y urgencia.
3. Aplicación de plazos regulatorios según normativa vigente.
4. Asignación a cuadrilla técnica (trabajo de campo) o derivación al área comercial.
5. Seguimiento con avances registrados y monitoreo de vencimientos.
6. Cierre del caso, notificación al usuario y generación de reportes regulatorios.

## Índice de diagramas

| Diagrama | Archivo |
|----------|---------|
| DFD Nivel 0 (Contexto) | [dfd-nivel-0.md](dfd-nivel-0.md) |
| DFD Nivel 1 (Subsistemas) | [dfd-nivel-1.md](dfd-nivel-1.md) |
| DFD Nivel 2 (Funciones) | [dfd-nivel-2.md](dfd-nivel-2.md) |
| DFD Nivel 3 (Subfunciones/Eventos) | [dfd-nivel-3.md](dfd-nivel-3.md) |
| DFD Nivel 4 (Procesos primitivos) | [dfd-nivel-4.md](dfd-nivel-4.md) |
| DER (Entidad-Relación) | [erd.md](erd.md) |
| Diccionario de Datos | [diccionario-datos.md](diccionario-datos.md) |
| Especificaciones de primitivos | [especificaciones.md](especificaciones.md) |