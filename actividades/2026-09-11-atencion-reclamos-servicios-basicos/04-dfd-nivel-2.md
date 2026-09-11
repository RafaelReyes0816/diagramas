# DFD Nivel 2 (Funciones): Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Descomposición de cada subsistema del Nivel 1 (03-dfd-nivel-1.md) en sus **funciones** (P1.x–P5.x). Cada diagrama respeta el **balanceo estricto**: todo flujo de entrada/salida del proceso padre aparece en el diagrama hijo, y los Almacenes de Datos conservan el mismo nombre en todos los niveles.

## Diagrama 1: P1 — Registrar y Consultar Reclamos

```mermaid
flowchart LR
    U[EE: Usuario]

    P1_1((P1.1<br/>Registrar Reclamo))
    P1_2((P1.2<br/>Emitir Comprobante))
    P1_3((P1.3<br/>Atender Consulta de Estado))
    P1_4((P1.4<br/>Actualizar Datos de Contacto))

    ADU[(AD: Usuarios)]
    ADR[(AD: Reclamos)]

    U -->|solicitud de reclamo| P1_1
    P1_1 <-->|registro| ADU
    P1_1 <-->|registro| ADR
    P1_1 -->|reclamo registrado| P1_2

    U -->|solicitud de reenvío de comprobante| P1_2
    P1_2 <-->|consulta| ADR
    P1_2 -->|comprobante de reclamo| U
    P1_2 -->|comprobante reenviado| U

    U -->|consulta de estado del reclamo| P1_3
    P1_3 <-->|consulta| ADR
    P1_3 -->|estado del reclamo| U

    U -->|datos de contacto actualizados| P1_4
    P1_4 <-->|consulta| ADR
    P1_4 <-->|actualización| ADU
    P1_4 -->|confirmación de actualización| U
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P1.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

### Puntos Clave (P1)

- Balanceo con Nivel 1: todas las entradas del Usuario (fechas F1–F4) y sus salidas (comprobante, estado, confirmación) están presentes.
- El registro y la emisión del comprobante están separados: P1.1 produce el *reclamo registrado* y P1.2 lo convierte en comprobante (lectura sobre `AD: Reclamos`).
- El reenvío de comprobante (F4) reutiliza el mismo proceso P1.2, sin duplicar lógica.

## Diagrama 2: P2 — Clasificar y Asignar Reclamos

```mermaid
flowchart LR
    C[EE: Cuadrilla Técnica]
    A[EE: Área Comercial]
    R[EE: Entidad Reguladora]

    P2_1((P2.1<br/>Clasificar Reclamo))
    P2_2((P2.2<br/>Determinar Plazo<br/>Regulatorio))
    P2_3((P2.3<br/>Asignar a Cuadrilla<br/>Técnica))
    P2_4((P2.4<br/>Derivar a Área<br/>Comercial))
    P2_5((P2.5<br/>Actualizar Normativa<br/>Regulatoria))

    ADR[(AD: Reclamos)]
    ADN[(AD: Normativa Regulatoria)]
    ADO[(AD: Órdenes de Trabajo)]

    P2_1 <-->|consulta/actualización| ADR
    P2_1 -->|reclamo clasificado| P2_2
    P2_1 -->|requerimiento de atención técnica| P2_3
    P2_1 -->|requerimiento de atención comercial| P2_4

    P2_2 <-->|consulta| ADN
    P2_2 <-->|actualización| ADR
    P2_2 -->|fecha límite determinada| P2_3
    P2_2 -->|fecha límite determinada| P2_4

    P2_3 <-->|registro| ADO
    P2_3 -->|orden de trabajo| C

    P2_4 <-->|actualización de estado| ADR
    P2_4 -->|caso derivado al área comercial| A

    R -->|normativa de plazos| P2_5
    P2_5 <-->|registro| ADN
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P2.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

### Puntos Clave (P2)

- La clasificación (P2.1) es previa y **condiciona la vía de atención**: técnica (corte, fuga, falla) o comercial (facturación).
- El plazo regulatorio (P2.2) se determina leyendo `AD: Normativa Regulatoria` y se persiste en `AD: Reclamos` para el posterior monitoreo temporal de P4.
- **Asignación (P2.3)** y **derivación (P2.4)** son mutuamente excluyentes: un reclamo sigue una única vía; de ahí que las relaciones en el DER sean 0..1.
- P2.5 recibe la normativa (evento F8) y la incorpora al almacén; las actualizaciones solo afectan reclamos nuevos.

## Diagrama 3: P3 — Seguir y Cerrar Reclamos

```mermaid
flowchart LR
    C[EE: Cuadrilla Técnica]
    A[EE: Área Comercial]
    U[EE: Usuario]

    P3_1((P3.1<br/>Registrar Avance<br/>de Trabajo))
    P3_2((P3.2<br/>Registrar Resolución<br/>Técnica))
    P3_3((P3.3<br/>Registrar Resolución<br/>Comercial))
    P3_4((P3.4<br/>Cerrar Reclamo))
    P3_5((P3.5<br/>Notificar al Usuario))

    ADR[(AD: Reclamos)]
    ADU[(AD: Usuarios)]
    ADO[(AD: Órdenes de Trabajo)]
    ADA[(AD: Avances de Trabajo)]

    C -->|reporte de avance de trabajo| P3_1
    P3_1 <-->|registro| ADA
    P3_1 <-->|actualización| ADO
    P3_1 <-->|actualización de estado| ADR

    C -->|reporte de resolución técnica| P3_2
    P3_2 <-->|consulta/actualización| ADO
    P3_2 <-->|actualización de estado| ADR
    P3_2 -->|reclamo resuelto| P3_4

    A -->|resolución comercial| P3_3
    P3_3 <-->|consulta/actualización| ADR
    P3_3 -->|reclamo resuelto| P3_4

    P3_4 <-->|consulta/actualización| ADR
    P3_4 -->|reclamo cerrado| P3_5

    P3_5 <-->|consulta| ADR
    P3_5 <-->|consulta de contacto| ADU
    P3_5 -->|notificación de resolución| U
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P3.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

### Puntos Clave (P3)

- Los avances (P3.1) se acumulan sobre la orden técnica (`AD: Avances de Trabajo`) y reflejan el estado parcial en `AD: Reclamos`.
- La **resolución** (técnica P3.2 o comercial P3.3) alimenta el **cierre** (P3.4), que es un único proceso de estado en el reclamo.
- La **notificación** (P3.5) lee el contacto del usuario desde `AD: Usuarios`, separando la comunicación de la lógica de cierre.

## Diagrama 4: P4 — Vigilar Plazos Regulatorios

```mermaid
flowchart LR
    C[EE: Cuadrilla Técnica]
    A[EE: Área Comercial]

    P4_1((P4.1<br/>Detectar Vencimiento<br/>Próximo))
    P4_2((P4.2<br/>Detectar Reclamo<br/>Vencido))
    P4_3((P4.3<br/>Detectar Reclamo<br/>Crítico))

    ADR[(AD: Reclamos)]
    ADN[(AD: Normativa Regulatoria)]

    P4_1 <-->|consulta| ADR
    P4_1 <-->|consulta de plazos| ADN
    P4_1 -->|aviso de vencimiento próximo| C
    P4_1 -->|aviso de vencimiento próximo| A

    P4_2 <-->|consulta| ADR
    P4_2 -->|alerta de reclamo vencido| C
    P4_2 -->|alerta de reclamo vencido| A

    P4_3 <-->|consulta| ADR
    P4_3 -->|alarma de reclamo crítico| C
    P4_3 -->|alarma de reclamo crítico| A
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P4.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

### Puntos Clave (P4)

- Es la respuesta a los **eventos temporales y de control** (T1, T2, C1 de 01-lista-eventos.md): no espera entrada externa, se dispara por el paso del tiempo.
- Todo reclamo es evaluado contra su `fecha_tope` (almacenada por P2.2) y la normativa vigente (`AD: Normativa Regulatoria`).
- Los avisos se dirigen a la **entidad responsable del caso** (cuadrilla o área comercial), nunca fuera de la frontera definida.

## Diagrama 5: P5 — Generar Reportes

```mermaid
flowchart LR
    R[EE: Entidad Reguladora]

    P5_1((P5.1<br/>Generar Reporte<br/>Operativo Diario))
    P5_2((P5.2<br/>Generar Reporte<br/>Regulatorio Mensual))

    ADR[(AD: Reclamos)]
    ADA[(AD: Avances de Trabajo)]
    ADO[(AD: Órdenes de Trabajo)]
    ADP[(AD: Reportes)]

    P5_1 <-->|consulta| ADR
    P5_1 <-->|consulta| ADA
    P5_1 <-->|registro| ADP

    P5_2 <-->|consulta| ADR
    P5_2 <-->|consulta| ADO
    P5_2 <-->|registro| ADP
    P5_2 -->|reporte regulatorio mensual| R
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P5.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

### Puntos Clave (P5)

- Procesos temporales puros (T3, T4): se ejecutan en horario programado y leen exclusivamente de los almacenes operativos; no reciben flujos de las demás funciones.
- El **reporte operativo diario** (T3) queda archivado en `AD: Reportes` para consumo interno; el **reporte regulatorio mensual** (T4) se entrega a la Entidad Reguladora.
- Ninguna función escribe sobre los almacenes operativos (solo leen), preservando la integridad del registro de reclamos.