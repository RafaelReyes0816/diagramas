# DFD Nivel 3: Subfunciones / Eventos — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Descomposición de las funciones con lógica significativa. Quedan aquí los procesos que materializan los eventos (F, T, C) del Modelo Ambiental: registro, clasificación, urgencia, plazo, monitoreo, criticidad y reporte.

## P1.1 — Registrar Reclamo (evento F1)

```mermaid
flowchart LR
    U1["[EE: Usuario]"]

    P111((P1.1.1<br/>Validar Datos<br/>del Usuario))
    P112((P1.1.2<br/>Validar Datos<br/>del Reclamo))
    P113((P1.1.3<br/>Persistir<br/>Reclamo))

    AD1[("[(AD: Reclamos)]")]
    AD2[("[(AD: Usuarios)]")]

    U1 -->|Solicitud de Reclamo| P111
    P111 --> AD2
    P111 -->|Datos Validados| P112
    P112 -->|Reclamo Validado| P113
    P113 --> AD1
    P113 -->|Reclamo Registrado| P12
    P12((P1.2<br/>Emitir Comprobante))
```

## P2.1 — Clasificar por Tipo

```mermaid
flowchart LR
    P211((P2.1.1<br/>Identificar<br/>Servicio))
    P212((P2.1.2<br/>Determinar<br/>Categoría))

    AD3[("[(AD: Clasificaciones)]")]

    P211 --> AD3
    P211 -->|Servicio Identificado| P212
    P212 --> AD3
    P212 -->|Reclamo Clasificado por Tipo| P22
    P22((P2.2<br/>Evaluar Urgencia))
```

## P2.2 — Evaluar Urgencia

```mermaid
flowchart LR
    P221((P2.2.1<br/>Evaluar<br/>Criticidad))
    P222((P2.2.2<br/>Asignar Nivel<br/>de Urgencia))

    AD3[("[(AD: Clasificaciones)]")]

    P221 --> AD3
    P221 -->|Criticidad Evaluada| P222
    P222 -->|Nivel de Urgencia| AD1
    AD1[("[(AD: Reclamos)]")]
    P222 -->|Reclamo Priorizado| P23
    P23((P2.3<br/>Determinar Plazo<br/>Regulatorio))
```

## P2.3 — Determinar Plazo Regulatorio (evento F5)

```mermaid
flowchart LR
    ER1["[EE: Entidad Reguladora]"]

    P231((P2.3.1<br/>Buscar Norma<br/>Vigente))
    P232((P2.3.2<br/>Calcular<br/>Fecha Límite))

    AD1[("[(AD: Reclamos)]")]
    AD4[("[(AD: Normativas)]")]

    ER1 -->|Normativa y Plazos| P231
    P231 --> AD4
    P231 -->|Plazo Máximo| P232
    P232 --> AD1
    P232 -->|Reclamo con Plazo| P24
    P24((P2.4<br/>Consolidar<br/>Clasificación))
```

## P4.2 — Monitorear Plazos Regulatorios (eventos T1 / C1)

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]

    P421((P4.2.1<br/>Verificar Días<br/>Restantes))
    P422((P4.2.2<br/>Generar Alerta<br/>de Vencimiento))

    AD1[("[(AD: Reclamos)]")]
    AD4[("[(AD: Normativas)]")]

    AD1 --> P421
    AD4 --> P421
    P421 -->|Resultado del Monitoreo| P422
    P422 -->|Alerta de Vencimiento| CT1
```

## P4.3 — Detectar Reclamos Críticos (evento C2)

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]

    P431((P4.3.1<br/>Aplicar Criterios<br/>de Criticidad))
    P432((P4.3.2<br/>Escalar<br/>Reclamo))

    AD1[("[(AD: Reclamos)]")]

    AD1 --> P431
    P431 -->|Caso Crítico Identificado| P432
    P432 -->|Alarma de Reclamo Crítico| CT1
```

## P5.3 — Generar Reporte Regulatorio (evento T2)

```mermaid
flowchart LR
    ER1["[EE: Entidad Reguladora]"]

    P531((P5.3.1<br/>Consolidar<br/>Datos))
    P532((P5.3.2<br/>Emitir<br/>Reporte))

    AD1[("[(AD: Reclamos)]")]
    AD6[("[(AD: Ordenes de Trabajo)]")]

    AD1 --> P531
    AD6 --> P531
    P531 -->|Datos Consolidados| P532
    P532 -->|Reporte Regulatorio| ER1
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P2.1.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

## Puntos Clave

- **Los eventos del Modelo Ambiental quedan mapeados**: F1 (P1.1), F5 (P2.3), T1/C1 (P4.2), C2 (P4.3), T2 (P5.3).
- **Cada bloque conserva los flujos E/S de su proceso padre** (P2.2 recibe "Reclamo Clasificado por Tipo" y entrega "Reclamo Priorizado"; P4.3 recibe y produce lo mismo que P4.3 del Nivel 2).
- **P4.2 es indiferente a relojes del mundo**: dispara únicamente comparando fecha límite contra la fecha actual, según Yourdon (es un evento temporal puro).