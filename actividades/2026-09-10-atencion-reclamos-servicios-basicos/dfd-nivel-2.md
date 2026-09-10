# DFD Nivel 2: Funciones — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Cada subsistema del Nivel 1 se descompone en funciones (P1.1, P2.1, …). Los flujos externos del nivel padre se conservan asociados a la función que los toca (balanceo estricto).

## P1 — Gestionar Reclamos

```mermaid
flowchart LR
    U1["[EE: Usuario]"]
    U2["[EE: Usuario*]"]

    P11((P1.1<br/>Registrar Reclamo))
    P12((P1.2<br/>Emitir Comprobante))
    P13((P1.3<br/>Atender Consulta<br/>de Estado))
    P14((P1.4<br/>Actualizar Datos<br/>de Contacto))

    AD1[("[(AD: Reclamos)]")]
    AD2[("[(AD: Usuarios)]")]

    U1 -->|Solicitud de Reclamo| P11
    P11 --> AD1
    P11 <--> AD2
    P11 -->|Reclamo Registrado| P12
    P12 -->|Comprobante de Reclamo| U2
    U2 -->|Consulta de Estado| P13
    P13 --> AD1
    P13 -->|Estado del Reclamo| U2
    U1 -->|Dato de Contacto Actualizado| P14
    P14 --> AD2
```

## P2 — Clasificar y Priorizar

```mermaid
flowchart LR
    ER1["[EE: Entidad Reguladora]"]
    P1S((P1<br/>Gestionar Reclamos))

    P21((P2.1<br/>Clasificar por Tipo))

    P1S -->|Reclamo Registrado| P21
    P22((P2.2<br/>Evaluar Urgencia))
    P23((P2.3<br/>Determinar Plazo<br/>Regulatorio))
    P24((P2.4<br/>Consolidar<br/>Clasificación))

    AD1[("[(AD: Reclamos)]")]
    AD3[("[(AD: Clasificaciones)]")]
    AD4[("[(AD: Normativas)]")]

    P21 -->|Reclamo Clasificado por Tipo| P22
    P21 --> AD1
    P21 --> AD3
    P22 -->|Reclamo Priorizado| P23
    P22 --> AD3
    ER1 -->|Normativa y Plazos| P23
    P23 --> AD4
    P23 -->|Reclamo con Plazo| P24
    P24 --> AD1
    P24 -->|Reclamo Clasificado| P3
    P3((P3<br/>Asignar y Coordinar))
```

## P3 — Asignar y Coordinar

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]
    AC1["[EE: Área Comercial]"]
    P2S((P2<br/>Clasificar y Priorizar))

    P31((P3.1<br/>Seleccionar Cuadrilla<br/>Disponible))
    P32((P3.2<br/>Emitir Orden<br/>de Trabajo))
    P33((P3.3<br/>Derivar a Área<br/>Comercial))
    P34((P3.4<br/>Reasignar Reclamo))

    AD1[("[(AD: Reclamos)]")]
    AD5[("[(AD: Cuadrillas)]")]
    AD6[("[(AD: Ordenes de Trabajo)]")]

    P2S -->|Reclamo Clasificado| P31

    P31 --> AD5
    P31 -->|Cuadrilla Asignada| P32
    P32 --> AD6
    P32 -->|Orden de Trabajo| CT1
    P33 --> AD1
    P33 -->|Derivación Comercial| AC1
    P34 --> AD6
    P34 --> AD1
    P34 -->|Reasignación| P31
```

## P4 — Ejecutar y Dar Seguimiento

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]
    AC1["[EE: Área Comercial]"]
    P3S((P3<br/>Asignar y Coordinar))

    P41((P4.1<br/>Registrar Avance<br/>de Trabajo))
    P42((P4.2<br/>Monitorear Plazos<br/>Regulatorios))
    P43((P4.3<br/>Detectar Reclamos<br/>Críticos))
    P44((P4.4<br/>Consolidar<br/>Resolución))

    AD1[("[(AD: Reclamos)]")]
    AD4[("[(AD: Normativas)]")]
    AD6[("[(AD: Ordenes de Trabajo)]")]
    AD7[("[(AD: Seguimiento)]")]

    P3S -->|Reclamo Asignado| P41

    CT1 -->|Reporte de Avance| P41
    P41 --> AD7
    P41 --> AD6
    P42 --> AD4
    P42 --> AD1
    P42 -->|Alerta de Vencimiento| CT1
    P43 --> AD1
    P43 -->|Alarma de Reclamo Crítico| CT1
    AC1 -->|Resolución Comercial| P44
    P44 --> AD1
    P44 -->|Reclamo Resuelto| P5
    P5((P5<br/>Cerrar y Reportar))
```

## P5 — Cerrar y Reportar

```mermaid
flowchart LR
    U1["[EE: Usuario]"]
    ER1["[EE: Entidad Reguladora]"]
    P4S((P4<br/>Ejecutar y Dar<br/>Seguimiento))

    P51((P5.1<br/>Validar Cierre))
    P52((P5.2<br/>Notificar al<br/>Usuario))
    P53((P5.3<br/>Generar Reporte<br/>Regulatorio))
    P54((P5.4<br/>Reenviar<br/>Comprobante))

    AD1[("[(AD: Reclamos)]")]
    AD8[("[(AD: Notificaciones)]")]

    P4S -->|Reclamo Resuelto| P51

    P51 --> AD1
    P51 -->|Caso Valido| P52
    P52 --> AD8
    P52 -->|Notificación de Resolución| U1
    P53 --> AD1
    P53 -->|Reporte Regulatorio| ER1
    U1 -->|Solicitud de Reenvío| P54
    P54 --> AD1
    P54 -->|Comprobante Reenviado| U1
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P1.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

## Puntos Clave

- **P2.3 depende de la Entidad Reguladora** para convertir normativa en plazo concreto; sin la normativa no se fija fecha límite.
- **P4.2 y P4.3 son los eventos temporales y de control del modelo**: leen el repositorio y disparan la alerta/ alarma hacia la cuadrilla sin intervención del usuario.
- **P5.4** atiende el evento de control C3 (reimpresión de comprobante) sin alterar el caso.
- La reasignación (P3.4) realimenta la selección de cuadrilla, manual o automática, según disponibilidad.