# Modelo Ambiental: Lista de Acontecimientos — Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

El sistema administra el ciclo de vida completo de los reclamos de usuarios por **cortes de servicio**, **errores de facturación**, **fugas** y **fallas técnicas**. Recibe el reclamo, lo clasifica por **tipo de servicio (agua/luz)**, **categoría** y **urgencia**, aplica los **plazos regulatorios**, lo asigna a una **cuadrilla técnica** o al **área comercial**, y lo sigue hasta el **cierre** dentro de los plazos.

Este documento es la **fuente única** de la Lista de Acontecimientos: los demás diagramas la referencian, no la duplican.

## 1. Definición de Frontera

**Dentro del sistema:**
- Recepción y registro del reclamo (presencial, telefónico, web) con emisión de comprobante.
- Clasificación por servicio, categoría y urgencia.
- Aplicación y monitoreo de plazos regulatorios.
- Asignación a cuadrilla técnica o derivación al área comercial.
- Seguimiento: avances, resoluciones, escalamientos y cierre.
- Notificación al usuario y generación de reportes (operativo y regulatorio).

**Fuera del sistema (ejecutado por terceros que interactúan como Entidades Externas):**
- La ejecución física del trabajo de campo de la cuadrilla.
- La resolución material del caso comercial (anulación/ajuste de factura, etc.).
- La definición de la normativa: la entidad reguladora dicta los plazos; el sistema solo los recibe y los aplica.
- La prestación misma del servicio (agua/luz).

## 2. Entidades Externas (EE)

| Código | Entidad Externa | Rol | Flujos asociados |
|---|---|---|---|
| EE1 | Usuario | Presenta el reclamo, consulta estado, actualiza contactos, recibe comprobantes y notificaciones | F1, F2, F3, F4 |
| EE2 | Cuadrilla Técnica | Recibe órdenes de trabajo y avisos de plazo; reporta avances y resolución técnica | F5, F6 |
| EE3 | Área Comercial | Recibe casos derivados y avisos de plazo; reporta resolución comercial | F7 |
| EE4 | Entidad Reguladora | Suministra la normativa de plazos; recibe el reporte regulatorio mensual | F8 |

## 3. Lista de Acontecimientos

### 3.1 Flujo externo (F) — llegada de un dato desde una Entidad Externa

| ID | Evento | Fuente | Destino | Descripción |
|---|---|---|---|---|
| F1 | Solicitud de reclamo | Usuario | Sistema | Ingreso de un reclamo con datos del usuario, servicio, categoría y descripción (P1.1). |
| F2 | Consulta de estado del reclamo | Usuario | Sistema | Solicitud de información sobre el avance y estado actual de un reclamo (P1.3). |
| F3 | Actualización de datos de contacto | Usuario | Sistema | Cambio de teléfono/email del usuario asociado a un reclamo (P1.4). |
| F4 | Solicitud de reenvío de comprobante | Usuario | Sistema | El usuario pide nuevamente su comprobante de reclamo (P1.2). |
| F5 | Reporte de avance de trabajo | Cuadrilla Técnica | Sistema | La cuadrilla informa progreso sobre una orden de trabajo (P3.1). |
| F6 | Reporte de resolución técnica | Cuadrilla Técnica | Sistema | La cuadrilla declara resuelto el problema de campo (P3.2). |
| F7 | Resolución comercial | Área Comercial | Sistema | El área comercial informa la solución del caso (anulación, ajuste) (P3.3). |
| F8 | Normativa regulatoria de plazos | Entidad Reguladora | Sistema | Publicación/actualización de plazos máximos por servicio, categoría y urgencia (P2.5). |

### 3.2 Temporal (T) — ocurre por el paso del tiempo

| ID | Evento | Disparador | Destino | Descripción |
|---|---|---|---|---|
| T1 | Vencimiento próximo del plazo | Se alcanza el umbral configurable (ej. 80% del plazo) | Cuadrilla Técnica / Área Comercial | El sistema detecta reclamos al borde del plazo y emite aviso (P4.1). |
| T2 | Vencimiento del plazo excedido | Fecha límite superada sin cierre | Cuadrilla Técnica / Área Comercial | El sistema detecta reclamos vencidos y emite alerta (P4.2). |
| T3 | Generación de reporte operativo diario | Fin del día operativo | Sistema (AD: Reportes) | Resumen diario de reclamos, avances y situación de plazos (P5.1). |
| T4 | Generación de reporte regulatorio mensual | Cierre de mes | Entidad Reguladora | Reporte mensual de reclamos y cumplimiento de plazos (P5.2). |

### 3.3 Control (C) — condiciones de estado que el sistema decide

| ID | Evento | Disparador | Destino | Descripción |
|---|---|---|---|---|
| C1 | Alarma de reclamo crítico | Reclamo clasificado como urgencia crítica (riesgo vital o mayor, ej. fuga de alto caudal, corte prolongado) | Cuadrilla Técnica / Área Comercial | Escalamiento inmediato con máxima prioridad (P4.3). |
| C2 | Escalamiento por inacción | Reclamo sin avance registrado en un porcentaje del plazo | Sistema (actualiza estado del reclamo) | El sistema marca el reclamo como escalado y pide aceleración al responsable (P3.6 vía P4). |

> **Nota metodológica:** los eventos **no son** subfunciones ni procesos del DFD. Se disparan en procesos del Modelo de Comportamiento según las etiquetas entre paréntesis (P1.x, P2.x…), que no los reemplazan.

## 4. Requerimientos funcionales clave que sostienen los eventos

1. Recepción y registro del reclamo con comprobante único por canal (presencial, telefónico, web).
2. Clasificación por tipo de servicio (agua/luz), categoría (corte, facturación, fuga, falla) y urgencia.
3. Aplicación de plazos regulatorios según normativa vigente.
4. Asignación a cuadrilla técnica (trabajo de campo) o derivación al área comercial.
5. Seguimiento con avances registrados y monitoreo de vencimientos.
6. Cierre del caso, notificación al usuario y generación de reportes regulatorios.