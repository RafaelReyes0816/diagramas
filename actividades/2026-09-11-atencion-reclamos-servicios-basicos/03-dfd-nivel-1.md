# DFD Nivel 1 (Subsistemas): Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Descomposición del contexto (02-dfd-nivel-0.md) en **cinco subsistemas** que procesan los eventos de 01-lista-eventos.md. Los subsistemas se acoplan **exclusivamente mediante Almacenes de Datos** (regla de conservación de datos de Yourdon: flujos asíncronos requieren almacén intermedio).

- **P1** Registra/consulta reclamos y emite comprobantes (F1–F4).
- **P2** Clasifica, determina plazos y asigna/deriva (F8 + respuesta a F1).
- **P3** Registra avances, resoluciones, cierra y notifica (F5–F7).
- **P4** Vigila plazos y emite avisos/alarmas (T1, T2, C1).
- **P5** Genera reportes operativo y regulatorio (T3, T4).

## Diagrama

```mermaid
flowchart LR
    U[EE: Usuario]
    C[EE: Cuadrilla Técnica]
    A[EE: Área Comercial]
    R[EE: Entidad Reguladora]

    P1((P1<br/>Registrar y Consultar<br/>Reclamos))
    P2((P2<br/>Clasificar y Asignar<br/>Reclamos))
    P3((P3<br/>Seguir y Cerrar<br/>Reclamos))
    P4((P4<br/>Vigilar Plazos<br/>Regulatorios))
    P5((P5<br/>Generar Reportes))

    ADU[(AD: Usuarios)]
    ADR[(AD: Reclamos)]
    ADN[(AD: Normativa<br/>Regulatoria)]
    ADO[(AD: Órdenes<br/>de Trabajo)]
    ADA[(AD: Avances<br/>de Trabajo)]
    ADP[(AD: Reportes)]

    U -->|solicitud de reclamo| P1
    U -->|solicitud de reenvío de comprobante| P1
    U -->|consulta de estado del reclamo| P1
    U -->|datos de contacto actualizados| P1
    P1 -->|comprobante de reclamo| U
    P1 -->|comprobante reenviado| U
    P1 -->|estado del reclamo| U
    P1 -->|confirmación de actualización| U

    P1 <-->|consulta/registro| ADU
    P1 <-->|consulta/registro| ADR

    P2 <-->|consulta/actualización| ADR
    P2 <-->|consulta| ADN
    P2 <-->|registro| ADO
    P2 -->|orden de trabajo| C
    P2 -->|caso derivado al área comercial| A
    R -->|normativa de plazos| P2

    C -->|reporte de avance de trabajo| P3
    C -->|reporte de resolución técnica| P3
    A -->|resolución comercial| P3
    P3 <-->|consulta/actualización| ADR
    P3 <-->|consulta/registro| ADO
    P3 <-->|registro| ADA
    P3 <-->|consulta| ADU
    P3 -->|notificación de resolución| U

    P4 <-->|consulta| ADR
    P4 <-->|consulta| ADN
    P4 -->|aviso de vencimiento próximo| C
    P4 -->|alerta de reclamo vencido| C
    P4 -->|alarma de reclamo crítico| C
    P4 -->|aviso de vencimiento próximo| A
    P4 -->|alerta de reclamo vencido| A
    P4 -->|alarma de reclamo crítico| A

    P5 <-->|consulta| ADR
    P5 <-->|consulta| ADA
    P5 <-->|consulta| ADO
    P5 <-->|registro| ADP
    P5 -->|reporte regulatorio mensual| R
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

## Puntos Clave

- **Balanceo con Nivel 0:** todos los flujos externos del contexto aparecen asignados a su subsistema (P1–P5); ninguna Entidad Externa se descompone.
- Los subsistemas **no intercambian flujos directos entre sí**: la separación es asíncrona mediante `AD: Reclamos` (P1 registra → P2/P3/P4/P5 consultan), `AD: Órdenes de Trabajo` (P2 asigna → P3 da seguimiento) y `AD: Normativa Regulatoria` (P2 mantiene → P4 consulta).
- **P4** materializa los eventos temporales/control (T1, T2, C1) como salidas automáticas hacia la entidad responsable del caso.
- **P5** produce el reporte regulatorio (T4) hacia la Entidad Reguladora; el reporte operativo (T3) queda archivado en `AD: Reportes`.