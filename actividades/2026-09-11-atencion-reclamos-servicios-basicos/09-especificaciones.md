# Especificaciones de Procesos Primitivos: Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Lógica interna mínima (`minispecs`) de todos los **procesos primitivos** identificados tras la descomposición top-down (04-dfd-nivel-2.md y 05-dfd-nivel-3.md). Cada especificación describe: objetivo, entradas (flujos), salidas (flujos), almacenes accedidos y reglas de transformación en lenguaje estructurado (sin código de programación).

## P1.1.1 — Validar Datos del Reclamo

**Objetivo:** garantizar que la solicitud de reclamo cumpla las reglas mínimas antes del registro.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `solicitud de reclamo` | `solicitud válida` o rechazo | — |

**Reglas:**
1. Verificar datos obligatorios: documento, nombre, dirección, descripción.
2. Validar catálogos: `servicio` ∈ {agua, luz} y `categoria` ∈ {corte, facturación, fuga, falla_técnica}.
3. Si la `categoria` es `fuga` o `corte` y no hay dirección de suministro → rechazar.
4. Si el teléfono o email no son válidos de formato → rechazar.
5. Si toda la validación pasa → emitir `solicitud válida`; en caso contrario devolver el motivo (no se registra).

## P1.1.2 — Registrar Usuario y Reclamo

**Objetivo:** incorporar el reclamo y su autor al modelo de información.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `solicitud válida` | `reclamo registrado sin carátula` | AD: Usuarios, AD: Reclamos |

**Reglas:**
1. Buscar usuario por `documento` en AD: Usuarios.
2. Si no existe → crear registro y devolver `id_usuario`; si existe → actualizar datos de contacto.
3. Crear el reclamo en AD: Reclamos con estado `registrado`, fecha de recepción actual y `id_normativa` pendiente (se fija en P2.2).
4. Emitir `reclamo registrado sin carátula`.

## P1.1.3 — Asignar Carátula y Canal

**Objetivo:** completar la identificación única del reclamo.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo registrado sin carátula` | `reclamo registrado` | AD: Reclamos |

**Reglas:**
1. Generar `numero_reclamo` correlativo anual y `serie` por canal.
2. Fijar `canal` = {presencial, telefónico, web} según el medio de ingreso.
3. Actualizar AD: Reclamos con la carátula.
4. Emitir `reclamo registrado` hacia P1.2 (comprobante).

## P1.2 — Emitir Comprobante

**Objetivo:** entregar el comprobante del reclamo (emisión inicial o reenvío).

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo registrado` / `solicitud de reenvío de comprobante` | `comprobante de reclamo` / `comprobante reenviado` | AD: Reclamos (consulta) |

**Reglas:**
1. Consultar el reclamo por `@id_reclamo`.
2. Conformar el comprobante con datos del reclamo y `fecha_tope` (si ya fue fijada por P2.2).
3. Si es reenvío (F4) y el reclamo existe → emitir `comprobante reenviado`; si no existe → notificar error.

## P1.3 — Atender Consulta de Estado

**Objetivo:** informar al usuario el estado vigente de su reclamo.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `consulta de estado del reclamo` | `estado del reclamo` | AD: Reclamos (consulta) |

**Reglas:**
1. Localizar el reclamo por `@id_reclamo` o `documento`.
2. Devolver estado, `fecha_tope` y (si existe) el último avance registrado.
3. Si no existe → responder "no encontrado".

## P1.4 — Actualizar Datos de Contacto

**Objetivo:** mantener actualizado el medio de contacto del reclamante.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `datos de contacto actualizados` | `confirmación de actualización` | AD: Reclamos (consulta), AD: Usuarios (actualización) |

**Reglas:**
1. Verificar que el `@id_reclamo` exista (AD: Reclamos).
2. Actualizar `telefono` y `email` del usuario en AD: Usuarios.
3. Emitir `confirmación de actualización` con resultado `confirmado`.

## P2.1.1 — Determinar Servicio y Categoría

**Objetivo:** clasificar el reclamo por el tipo de servicio y la categoría del problema.

| Entradas | Salidas | Almacenes |
|---|---|---|
| reclamo sin clasificar (AD) | `servicio y categoría` | AD: Reclamos (consulta/actualización) |

**Reglas:**
1. Leer el reclamo más antiguo en estado `registrado` (primero en entrar, primero en clasificarse — FIFO).
2. Asignar `servicio` y `categoria` a partir de la solicitud.
3. Actualizar el reclamo en AD: Reclamos (estado `clasificado`).

## P2.1.2 — Asignar Nivel de Urgencia

**Objetivo:** determinar la prioridad de atención mediante una matriz servicio × categoría × contexto.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `servicio y categoría` | `clasificación con urgencia` | — |

**Reglas (matriz de urgencia):**

| Servicio | Categoría | Condición de contexto | Urgencia |
|---|---|---|---|
| agua | fuga | caudal alto / riesgo vital | crítica |
| luz | corte | prolongado (≥ umbral) o zona crítica | crítica |
| agua | corte | sin servicio prolongado | alta |
| luz | falla técnica | falla general | alta |
| agua | fuga | caudal normal | normal |
| luz | corte | breve o programado | programada |
| ambos | facturación | cualquier | normal |

1. Aplicar la matriz; el caso `crítica` se marca para la alarma C1 (P4.3).

## P2.1.3 — Determinar Vía de Atención

**Objetivo:** decidir si el caso se atiende por vía técnica o comercial.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `clasificación con urgencia` | `requerimiento de atención técnica`, `requerimiento de atención comercial` | — |

**Reglas:**
1. `via_atencion = tecnica` si `categoria` ∈ {corte (operativo), fuga, falla_técnica}.
2. `via_atencion = comercial` si `categoria` = facturación (o corte por cobranza/administrativo).
3. Emitir el requerimiento correspondiente a P2.3 o P2.4; nunca ambos (excluyencia reflejada en 07-erd.md).

## P2.2.1 — Consultar Normativa Aplicable

**Objetivo:** recuperar la regla de plazo vigente para la clasificación del reclamo.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo clasificado` | `normativa aplicable` | AD: Normativa Regulatoria, AD: Reclamos (consulta) |

**Reglas:**
1. Seleccionar AD: Normativa Regulatoria por `servicio + categoria + urgencia`.
2. Filtrar por vigencia: `vigencia_desde ≤ fecha_recepcion` y mayor `vigencia_desde` de las aplicables.
3. Si no existe regla → usar plazo por defecto de la configuración del sistema (parametrizable).

## P2.2.2 — Calcular Fecha Límite

**Objetivo:** determinar la fecha límite de atención.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `normativa aplicable` | `fecha límite calculada` | — |

**Reglas:**
1. `fecha_tope = fecha_recepcion + plazo_maximo_dias`.
2. Considerar días hábiles si la normativa lo define; en caso contrario días corridos.
3. Para urgencia `crítica`, la `fecha_tope` no podrá exceder el plazo especial de urgencia (mínimo regulatorio).

## P2.2.3 — Registrar Fecha Límite

**Objetivo:** persistir el plazo para el monitoreo de P4.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `fecha límite calculada` | `fecha límite determinada` | AD: Reclamos (actualización) |

**Reglas:**
1. Actualizar `fecha_tope` e `id_normativa` en AD: Reclamos.
2. Emitir `fecha límite determinada` hacia P2.3 (técnica) o P2.4 (comercial) según la vía.

## P2.3 — Asignar a Cuadrilla Técnica

**Objetivo:** emitir la orden de trabajo a la cuadrilla.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `requerimiento de atención técnica`, `fecha límite determinada` | `orden de trabajo` | AD: Órdenes de Trabajo (registro) |

**Reglas:**
1. Seleccionar la cuadrilla con menor carga y capacidad para el servicio (agua/luz).
2. Crear `orden_trabajo` en AD: Órdenes de Trabajo con estado `asignada`.
3. Actualizar AD: Reclamos a `en_atencion_tecnica`.
4. Emitir la `orden de trabajo` a la EE Cuadrilla Técnica.

## P2.4 — Derivar a Área Comercial

**Objetivo:** transferir el caso al área comercial correspondiente.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `requerimiento de atención comercial`, `fecha límite determinada` | `caso derivado al área comercial` | AD: Reclamos (actualización) |

**Reglas:**
1. Determinar `area_comercial` = {facturación, cobranza} según el origen del error.
2. Actualizar AD: Reclamos a `en_atencion_comercial` con `fecha_derivacion`.
3. Emitir `caso derivado al área comercial` a la EE Área Comercial.

## P2.5 — Actualizar Normativa Regulatoria

**Objetivo:** incorporar los cambios de plazos publicados por el regulador.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `normativa de plazos` | — | AD: Normativa Regulatoria (registro) |

**Reglas:**
1. Validar la publicación (período y reglas).
2. Insertar o superponer vigencia (`vigencia_desde`).
3. Los reclamos ya clasificados **no** recalculan su plazo; solo los nuevos (regla de inmutabilidad del plazo).

## P3.1 — Registrar Avance de Trabajo

**Objetivo:** capturar el progreso reportado por la cuadrilla.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reporte de avance de trabajo` | — | AD: Órdenes de Trabajo, AD: Reclamos, AD: Avances de Trabajo |

**Reglas:**
1. Verificar que la `@id_orden` exista y esté `asignada` o `en_curso`.
2. Registrar `avance` en AD: Avances de Trabajo.
3. Actualizar la orden a `en_curso` y el reclamo con su último avance (para P1.3).
4. El avance resetea el contador usado por el escalamiento por inacción (evento C2 en 01-lista-eventos.md).

## P3.2 — Registrar Resolución Técnica

**Objetivo:** incorporar la resolución del caso técnico.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reporte de resolución técnica` | `reclamo resuelto` | AD: Órdenes de Trabajo, AD: Reclamos |

**Reglas:**
1. Validar `@id_orden` y su correspondencia con el `@id_reclamo`.
2. Actualizar la orden a `resuelta` y el reclamo a `resuelto` con `resultado`.
3. Emitir `reclamo resuelto` hacia P3.4 (cierre).

## P3.3 — Registrar Resolución Comercial

**Objetivo:** incorporar la resolución del caso comercial.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `resolución comercial` | `reclamo resuelto` | AD: Reclamos |

**Reglas:**
1. Validar que el reclamo esté `en_atencion_comercial`.
2. Actualizar el reclamo a `resuelto` con `resultado_comercial` y `fecha_resolucion`.
3. Emitir `reclamo resuelto` hacia P3.4 (cierre).

## P3.4.1 — Verificar Condiciones de Cierre

**Objetivo:** comprobar que el caso está apto para cerrarse.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo resuelto` | `reclamo apto para cierre` | AD: Reclamos (consulta) |

**Reglas:**
1. Verificar `estado = resuelto`.
2. Si es técnico: validar avances completos (AD: Avances de Trabajo) y orden `resuelta`.
3. Validar ausencia de observaciones o reclamos duplicados pendientes.
4. Apta → emitir `reclamo apto para cierre`.

## P3.4.2 — Registrar Cierre del Reclamo

**Objetivo:** fijar el cierre y la fecha efectiva del caso.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo apto para cierre` | `reclamo cerrado` | AD: Reclamos (actualización) |

**Reglas:**
1. Fijar `estado = cerrado` y `fecha_cierre = fecha actual`.
2. Determinar `resultado` final del caso.
3. Emitir `reclamo cerrado` hacia P3.5 (notificación) y habilitar su inclusión en reportes (P5).

## P3.5 — Notificar al Usuario

**Objetivo:** comunicar la resolución del caso al reclamante.

| Entradas | Salidas | Almacenes |
|---|---|---|
| `reclamo cerrado` | `notificación de resolución` | AD: Reclamos (consulta), AD: Usuarios (consulta de contacto) |

**Reglas:**
1. Obtener el contacto del usuario (telefono/email) desde AD: Usuarios.
2. Conformar `notificación de resolución` con resultado y `fecha_cierre`.
3. Enviar por el canal preferido del usuario; registrar intento de envío.

## P4.1 — Detectar Vencimiento Próximo

**Objetivo:** avisar antes del vencimiento del plazo (evento T1).

| Entradas | Salidas | Almacenes |
|---|---|---|
| temporizador (periódico) | `aviso de vencimiento próximo` | AD: Reclamos, AD: Normativa Regulatoria (consulta) |

**Reglas:**
1. Calcular `dias_restantes = fecha_tope − fecha_actual`.
2. Si `0 < dias_restantes ≤ umbral` (umbral = 20% del plazo, configurable) y el reclamo no está `cerrado` → emitir `aviso de vencimiento próximo` a la entidad responsable (cuadrilla o área comercial).
3. Evitar duplicados: no reavisar el mismo reclamo en el mismo día.

## P4.2 — Detectar Reclamo Vencido

**Objetivo:** alertar cuando el plazo ya fue superado (evento T2).

| Entradas | Salidas | Almacenes |
|---|---|---|
| temporizador (periódico) | `alerta de reclamo vencido` | AD: Reclamos (consulta) |

**Reglas:**
1. Emitir alerta para todo reclamo con `fecha_tope < fecha_actual` y `estado ≠ cerrado`.
2. Calcular `exceso_dias = fecha_actual − fecha_tope` y adjuntarlo a la alerta.
3. El reclamo se marca `escalado` para reportes y seguimiento especial.

## P4.3 — Detectar Reclamo Crítico

**Objetivo:** escalar con máxima prioridad los casos de urgencia crítica (evento C1).

| Entradas | Salidas | Almacenes |
|---|---|---|
| temporizador (periódico) | `alarma de reclamo crítico` | AD: Reclamos (consulta) |

**Reglas:**
1. Recuperar reclamos con `urgencia = crítica`, no atendidos ni cerrados.
2. Para agua: fuga de caudal alto / riesgo vital. Para luz: corte prolongado o zona crítica.
3. Ante la alarma, P3 (seguimiento) podrá reasignar a la cuadrilla de máxima disponibilidad y recortar el plazo efectivo.

## P5.1 — Generar Reporte Operativo Diario

**Objetivo:** resumir el estado operativo diario (evento T3).

| Entradas | Salidas | Almacenes |
|---|---|---|
| temporizador (fin de día) | — | AD: Reclamos, AD: Avances de Trabajo (consulta), AD: Reportes (registro) |

**Reglas:**
1. Computar ingresados, en atención, próximos a vencer, vencidos y escalados del día.
2. Registrar el `reporte` (tipo `operativo_diario`) en AD: Reportes.

## P5.2 — Generar Reporte Regulatorio Mensual

**Objetivo:** entregar la información regulatoria mensual (evento T4).

| Entradas | Salidas | Almacenes |
|---|---|---|
| temporizador (cierre de mes) | `reporte regulatorio mensual` | AD: Reclamos, AD: Órdenes de Trabajo (consulta), AD: Reportes (registro) |

**Reglas:**
1. Agrupar reclamos del período y calcular indicadores: `total_ingresados`, `total_cerrados`, `total_vencidos`, `porcentaje_cumplimiento`.
2. Incluir los `dias_atencion` por reclamo cerrado.
3. Registrar en AD: Reportes (tipo `regulatorio_mensual`) y emitir el `reporte regulatorio mensual` a la EE Entidad Reguladora.