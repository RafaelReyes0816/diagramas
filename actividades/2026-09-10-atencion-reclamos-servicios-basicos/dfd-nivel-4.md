# DFD Nivel 4: Procesos Primitivos — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Hojas del árbol de descomposición: cada proceso de este nivel es un **proceso primitivo** con especificación propia en [especificaciones.md](especificaciones.md). Solo se llega a Nivel 4 en la lógica que concentra reglas de negocio (urgencia, plazos, monitoreo, criticidad y reporte).

## P2.2.1 — Evaluar Criticidad

```mermaid
flowchart LR
    P2211((P2.2.1.1<br/>Verificar Riesgo<br/>Vital))
    P2212((P2.2.1.2<br/>Verificar Magnitud<br/>del Incidente))

    P2211 -->|Datos de Criticidad| P2212
    P2212 -->|Criticidad Evaluada| P222
    P222((P2.2.2<br/>Asignar Nivel<br/>de Urgencia))
```

## P2.2.2 — Asignar Nivel de Urgencia

```mermaid
flowchart LR
    P2221((P2.2.2.1<br/>Determinar<br/>Nivel))
    P2222((P2.2.2.2<br/>Persistir<br/>Urgencia))

    AD1[("[(AD: Reclamos)]")]

    P2221 -->|Nivel Obtenido| P2222
    P2222 --> AD1
    P2222 -->|Reclamo Priorizado| P23
    P23((P2.3<br/>Determinar Plazo<br/>Regulatorio))
```

## P2.3.2 — Calcular Fecha Límite

```mermaid
flowchart LR
    P2321((P2.3.2.1<br/>Aplicar Días<br/>Hábiles))
    P2322((P2.3.2.2<br/>Fijar<br/>Fecha Límite))

    AD1[("[(AD: Reclamos)]")]

    P2321 -->|Plazo en Días Hábiles| P2322
    P2322 --> AD1
    P2322 -->|Reclamo con Plazo| P24
    P24((P2.4<br/>Consolidar<br/>Clasificación))
```

## P4.2.1 — Verificar Días Restantes

```mermaid
flowchart LR
    P4211((P4.2.1.1<br/>Calcular Días<br/>Restantes))
    P4212((P4.2.1.2<br/>Comparar con<br/>Umbral))

    AD1[("[(AD: Reclamos)]")]

    AD1 --> P4211
    P4211 -->|Días Restantes| P4212
    P4212 -->|Resultado del Monitoreo| P422
    P422((P4.2.2<br/>Generar Alerta<br/>de Vencimiento))
```

## P4.2.2 — Generar Alerta de Vencimiento

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]

    P4221((P4.2.2.1<br/>Conformar<br/>Alerta))
    P4222((P4.2.2.2<br/>Enviar<br/>Alerta))

    P4221 -->|Alerta Conformada| P4222
    P4222 -->|Alerta de Vencimiento| CT1
```

## P4.3.1 — Aplicar Criterios de Criticidad

```mermaid
flowchart LR
    P4311((P4.3.1.1<br/>Evaluar Criticidad<br/>del Servicio))
    P4312((P4.3.1.2<br/>Evaluar Población<br/>Afectada))

    P4311 -->|Datos de Servicio| P4312
    P4312 -->|Caso Crítico Identificado| P432
    P432((P4.3.2<br/>Escalar<br/>Reclamo))
```

## P4.3.2 — Escalar Reclamo

```mermaid
flowchart LR
    CT1["[EE: Cuadrilla Técnica]"]

    P4321((P4.3.2.1<br/>Conformar<br/>Alarma))
    P4322((P4.3.2.2<br/>Notificar<br/>Escalamiento))

    P4321 -->|Alarma Conformada| P4322
    P4322 -->|Alarma de Reclamo Crítico| CT1
```

## P5.3.1 — Consolidar Datos del Reporte

```mermaid
flowchart LR
    P5311((P5.3.1.1<br/>Extraer Reclamos<br/>del Periodo))
    P5312((P5.3.1.2<br/>Calcular Métricas<br/>de Cumplimiento))

    AD1[("[(AD: Reclamos)]")]

    AD1 --> P5311
    P5311 -->|Reclamos Extraídos| P5312
    P5312 -->|Datos Consolidados| P532
    P532((P5.3.2<br/>Emitir<br/>Reporte))
```

📌 Leyenda del Diagrama
[EE: ...] = Entidad Externa (actor/fuente/destino fuera del sistema)
P2.2.2.1((...)) = Proceso (transformación o acción del sistema)
[(AD: ...)] = Almacén de Datos (datos en reposo)
-->|...| = Flujo de Datos (información en movimiento)

## Puntos Clave

- **Los procesos de este nivel son atómicos**: su lógica se describe en pseudocódigo estructurado en [especificaciones.md](especificaciones.md).
- **No se descompone más P1, P3 ni P5.1/P5.2**: su lógica es suficientemente simple y se especifica directamente como primitivo en el Nivel 3.
- **Cada bloque mantiene el balanceo**: los flujos E/S del proceso padre se conservan intactos en el nivel 4.