# Diccionario de Datos (DD): Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Define la **estructura algebraica** de los flujos y almacenes usados en los DFD (02–05). Cobertura completa ha de leerse junto al DER (07-erd.md) y las Especificaciones (09-especificaciones.md).

**Notación algebraica:** `=` compuesto de · `+` concatenación ("Y") · `[ | ]` selección ("O") · `{ }` iteración (ej. `1{ x }5`) · `( )` opcional · `@` identificador/clave primaria · `* *` comentario.

## 1. Flujos externos

`solicitud de reclamo = usuario_reclamo + fecha_recepcion + canal + servicio + categoria + urgencia + descripcion`
- `usuario_reclamo = nombre + documento + telefono + (email) + direccion` *Datos del reclamante*
- `canal = [ presencial | telefonico | web ]`
- `servicio = [ agua | luz ]`
- `categoria = [ corte | facturacion | fuga | falla_tecnica ]`
- `urgencia = [ programada | normal | alta | critica ]`

`comprobante de reclamo = @id_reclamo + fecha_recepcion + canal + servicio + categoria + descripcion + fecha_tope`

`comprobante reenviado = comprobante de reclamo` *Mismo formato, nueva emisión*

`solicitud de reenvío de comprobante = @id_reclamo + (datos_identificacion)`

`consulta de estado del reclamo = @id_reclamo + datos_identificacion`
- `datos_identificacion = [ documento | @id_reclamo ]`

`estado del reclamo = @id_reclamo + estado + fecha_tope + (ultimo_avance)`

`datos de contacto actualizados = @id_reclamo + telefono + email`

`confirmación de actualización = @id_reclamo + resultado_actualizacion`
- `resultado_actualizacion = [ confirmado | rechazado ]`

`orden de trabajo = @id_orden + @id_reclamo + servicio + categoria + urgencia + cuadrilla + fecha_asignacion + fecha_tope`

`caso derivado al área comercial = @id_reclamo + fecha_derivacion + area_comercial + resumen_del_caso`
- `area_comercial = [ facturacion | cobranza ]`

`reporte de avance de trabajo = @id_orden + @id_reclamo + fecha_avance + descripcion + estado_parcial`

`reporte de resolución técnica = @id_orden + @id_reclamo + fecha_resolucion + resultado + (detalle)`
- `resultado = [ resuelto | descartado | derivado ]`

`resolución comercial = @id_reclamo + fecha_resolucion + resultado_comercial + (detalle)`
- `resultado_comercial = [ anulacion_factura | ajuste_factura | reclamacion_infundada | otro ]`

`normativa de plazos = normativa`
- `normativa = @id_normativa + servicio + categoria + urgencia + plazo_maximo_dias + vigencia_desde`

`reporte regulatorio mensual = periodo + { 1{ reclamo_reportado }N } + indicadores`
- `periodo = AAAA `+` MM`
- `reclamo_reportado = @id_reclamo + servicio + categoria + fecha_recepcion + estado + fecha_cierre + (dias_atencion)`
- `dias_atencion = fecha_cierre − fecha_recepcion`
- `indicadores = total_ingresados + total_cerrados + total_vencidos + porcentaje_cumplimiento`

`aviso de vencimiento próximo = @id_reclamo + fecha_tope + dias_restantes`

`alerta de reclamo vencido = @id_reclamo + fecha_tope + exceso_dias`

`alarma de reclamo crítico = @id_reclamo + servicio + categoria + descripcion + urgencia`

`notificación de resolución = @id_reclamo + resultado + fecha_cierre + (detalle)`

## 2. Flujos internos (entre procesos)

`reclamo registrado = @id_reclamo + fecha_recepcion + canal + caratula`
- `caratula = numero_reclamo + serie`

`solicitud válida = solicitud de reclamo` *Datos que superaron la validación*

`reclamo registrado sin carátula = @id_reclamo + fecha_recepcion + canal + servicio + categoria + descripcion`

`reclamo clasificado = @id_reclamo + servicio + categoria + urgencia + via_atencion`
- `via_atencion = [ tecnica | comercial ]`

`requerimiento de atención técnica = @id_reclamo + servicio + categoria + urgencia`

`requerimiento de atención comercial = @id_reclamo + servicio + categoria + urgencia`

`fecha límite determinada = @id_reclamo + fecha_tope`

`reclamo resuelto = @id_reclamo + fecha_resolucion + resultado`

`reclamo apto para cierre = @id_reclamo + verificacion_cierre`
- `verificacion_cierre = resuelto + avances_completos + sin_observaciones`

`reclamo cerrado = @id_reclamo + fecha_cierre + resultado`

## 3. Almacenes de Datos (AD)

`AD: Usuarios = { 1{ usuario }N }`
- `usuario = @id_usuario + nombre + documento + telefono + (email) + direccion`

`AD: Reclamos = { 1{ reclamo }N }`
- `reclamo = @id_reclamo + id_usuario + fecha_recepcion + canal + servicio + categoria + urgencia + descripcion + estado + fecha_tope + (fecha_cierre) + (resultado)`
- `estado = [ registrado | clasificado | en_atencion_tecnica | en_atencion_comercial | resuelto | cerrado | escalado ]`

`AD: Normativa Regulatoria = { 1{ normativa }N }`
- `normativa = @id_normativa + servicio + categoria + urgencia + plazo_maximo_dias + vigencia_desde`

`AD: Órdenes de Trabajo = { 1{ orden_trabajo }N }`
- `orden_trabajo = @id_orden + @id_reclamo + cuadrilla + fecha_asignacion + estado_orden`
- `estado_orden = [ asignada | en_curso | resuelta ]`

`AD: Avances de Trabajo = { 1{ avance }N }`
- `avance = @id_avance + @id_orden + fecha_avance + descripcion + estado_parcial`

`AD: Reportes = { 1{ reporte }N }`
- `reporte = @id_reporte + tipo_reporte + periodo + fecha_generacion + contenido`
- `tipo_reporte = [ operativo_diario | regulatorio_mensual ]`

## 4. Reglas de integridad (referenciales)

- `AD: Reclamos.id_usuario` referencia `AD: Usuarios.@id_usuario` (FK).
- `AD: Órdenes de Trabajo.id_reclamo` referencia `AD: Reclamos.@id_reclamo` (FK).
- `AD: Avances de Trabajo.id_orden` referencia `AD: Órdenes de Trabajo.@id_orden` (FK).
- `AD: Reclamos.fecha_tope` se deriva por `normativa` (servicio + categoria + urgencia) de `AD: Normativa Regulatoria`.
- Un `reclamo` en estado comercial no posee `orden_trabajo`; uno técnico no posee `derivacion_comercial` (excluyencia documentada en 09-especificaciones.md, P2.1.3).