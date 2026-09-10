# DFD Nivel 1: Subsistemas — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

El proceso de contexto P0 se descompone en cinco subsistemas con responsabilidades separadas por fase del ciclo de vida del reclamo. Las Entidades Externas permanecen y todos los flujos del nivel padre se conservan (balanceo estricto). Los almacenes de datos compartidos se muestran con el mismo nombre en todos los niveles.

## Diagrama

```mermaid
flowchart LR
    U1["[EE: Usuario]"]
    U2["[EE: Usuario*]"]
    CT1["[EE: Cuadrilla Técnica]"]
    AC1["[EE: Área Comercial]"]
    ER1["[EE: Entidad Reguladora]"]

    P1((P1<br/>Gestionar Reclamos))
    P2((P2<br/>Clasificar y Priorizar))
    P3((P3<br/>Asignar y Coordinar))
    P4((P4<br/>Ejecutar y Dar<br/>Seguimiento))
    P5((P5<br/>Cerrar y Reportar))

    AD1[("[(AD: Reclamos)]")]
    AD2[("[(AD: Usuarios)]")]
    AD3[("[(AD: Clasificaciones)]")]
    AD4[("[(AD: Normativas)]")]
    AD5[("[(AD: Cuadrillas)]")]
    AD6[("[(AD: Ordenes de Trabajo)]")]
    AD7[("[(AD: Seguimiento)]")]
    AD8[("[(AD: Notificaciones)]")]

    U1 -->|Solicitud de Reclamo| P1
    P1 -->|Comprobante de Reclamo| U2
    U2 -->|Consulta de Estado| P1
    P1 -->|Estado del Reclamo| U2

    P1 -->|Reclamo Registrado| P2
    P2 -->|Reclamo Clasificado| P3
    P3 -->|Reclamo Asignado| P4
    P4 -->|Reclamo Resuelto| P5

    ER1 -->|Normativa y Plazos| P2
    P5 -->|Notificación de Resolución| U2
    P5 -->|Reporte Regulatorio| ER1

    P3 -->|Orden de Trabajo| CT1
    CT1 -->|Reporte de Avance| P4
    P3 -->|Derivación Comercial| AC1
    AC1 -->|Resolución Comercial| P4

    P1 --> AD1
    P2 --> AD1
    P1 --> AD2
    P2 --> AD3
    P2 --> AD4
    P3 --> AD5
    P3 --> AD6
    P4 --> AD7
    P4 --> AD1
    P4 --> AD6
    P5 --> AD1
    P5 --> AD8
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

## Descomposición funcional

| Proceso | Responsabilidad Yourdon | Salidas |
|---------|-------------------------|---------|
| P1 Gestionar Reclamos | Captura la frontera: recepción, registro, comprobante y consultas del usuario | Reclamo Registrado, Comprobante, Estado del Reclamo |
| P2 Clasificar y Priorizar | Transforma el reclamo en hechos clasificados: tipo, urgencia y plazo | Reclamo Clasificado |
| P3 Asignar y Coordinar | Convierte el reclamo en compromisos externos: cuadrilla o área comercial | Reclamo Asignado, Orden de Trabajo, Derivación Comercial |
| P4 Ejecutar y Dar Seguimiento | Consume los reportes de ejecución y vigila vencimientos y críticos | Reclamo Resuelto |
| P5 Cerrar y Reportar | Cierra el caso, notifica al usuario y emite reportes reguladores | Notificación, Reporte Regulatorio |

## Puntos Clave

- **El flujo padre→hijo se conserva**: los 12 flujos del contexto aparecen en este nivel (verificación de balanceo).
- **Los comprobantes y estados de consulta no alteran el caso**: son respuestas puramente de P1, ajenas a la clasificación posterior.
- **Los almacenes nacen en este nivel** y permanecen con nombre idéntico en niveles inferiores, conforme a Yourdon.
- El desarrollo de reclamos asíncrono (trabajo de campo de una cuadrilla) **requiere el almacén intermedio AD: Ordenes de Trabajo** para no romper el flujo sincrónico.