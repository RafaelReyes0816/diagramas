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

| Código | Evento | Tipo |
|---|---|---|
| F1 | El usuario solicita un reclamo | F |
| F2 | El usuario consulta el estado del reclamo | F |
| F3 | El usuario actualiza sus datos de contacto | F |
| F4 | El usuario solicita el reenvío de su comprobante | F |
| F5 | La cuadrilla técnica reporta el avance del trabajo | F |
| F6 | La cuadrilla técnica reporta la resolución técnica | F |
| F7 | El área comercial reporta la resolución comercial | F |
| F8 | La entidad reguladora emite la normativa de plazos | F |
| T1 | El plazo del reclamo está próximo a vencer | T |
| T2 | El plazo del reclamo vence sin ser atendido | T |
| T3 | Se genera el reporte operativo diario | T |
| T4 | Se genera el reporte regulatorio mensual | T |
| C1 | El sistema dispara la alarma de reclamo crítico | C |
| C2 | El sistema escala el reclamo por inacción | C |

> **Nota metodológica:** `F` = flujo de dato (llegada de un dato desde una Entidad Externa), `T` = temporal (ocurre por el paso del tiempo), `C` = control (condición de estado que el sistema decide). Los eventos **no son** subfunciones ni procesos del DFD.

## 4. Requerimientos funcionales clave que sostienen los eventos

1. Recepción y registro del reclamo con comprobante único por canal (presencial, telefónico, web).
2. Clasificación por tipo de servicio (agua/luz), categoría (corte, facturación, fuga, falla) y urgencia.
3. Aplicación de plazos regulatorios según normativa vigente.
4. Asignación a cuadrilla técnica (trabajo de campo) o derivación al área comercial.
5. Seguimiento con avances registrados y monitoreo de vencimientos.
6. Cierre del caso, notificación al usuario y generación de reportes regulatorios.