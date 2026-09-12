# Diccionario de Datos (DD): Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Define la **estructura algebraica** de los flujos y almacenes usados en los DFD (02–05). Siguiendo la convención de clase, cada estructura se identifica con **letras** para mantener las ecuaciones cortas; un esquema de identificación mapea la letra con su nombre, y la **tabla de catálogos** agrupa los valores simples reutilizados.

**Notación algebraica:** `=` compuesto de · `+` concatenación ("Y") · `[ | ]` selección ("O") · `{ }` iteración (ej. `1{ x }N`) · `( )` opcional · `@` identificador/clave primaria · `* *` comentario.

## 1. Esquema de identificación (letra = estructura)

| Letra | Estructura | Letra | Estructura |
|---|---|---|---|
| A | Solicitud de reclamo | N | Normativa de plazos |
| B | Usuario (datos del reclamante) | O | Reporte regulatorio mensual |
| C | Comprobante de reclamo | P | Aviso de vencimiento próximo |
| D | Solicitud de reenvío de comprobante | Q | Alerta de reclamo vencido |
| E | Consulta de estado del reclamo | R | Alarma de reclamo crítico |
| F | Estado del reclamo | S | Notificación de resolución |
| G | Datos de contacto actualizados | T | Reclamo registrado (con carátula) |
| H | Confirmación de actualización | U | Reclamo clasificado |
| I | Orden de trabajo | V | Requerimiento de atención técnica |
| J | Reporte de avance de trabajo | W | Requerimiento de atención comercial |
| K | Reporte de resolución técnica | X | Fecha límite determinada |
| L | Caso derivado al área comercial | Y | Reclamo resuelto |
| M | Resolución comercial | AE | Reclamo registrado sin carátula |
| Z | Reclamo apto para cierre | — | — |
| — | — | AA | Reclamo cerrado |
| — | — | AB | Avance (registro almacenado) |
| — | — | AC | Reporte (registro almacenado) |
| — | — | AR | Reclamo (registro almacenado) |

## 2. Ecuaciones de flujos externos

`A = B + fecha_recepcion + canal + servicio + categoria + urgencia + descripcion`

`B = nombre + documento + telefono + (email) + direccion`

`C = @id_reclamo + fecha_recepcion + canal + servicio + categoria + descripcion + fecha_tope`

*Alias:* `comprobante reenviado = C` *Mismo formato, nueva emisión*

`D = @id_reclamo + (datos_identificacion)`

`E = @id_reclamo + datos_identificacion`
- `datos_identificacion = [ documento | @id_reclamo ]`

`F = @id_reclamo + estado + fecha_tope + (ultimo_avance)`

`G = @id_reclamo + telefono + email`

`H = @id_reclamo + resultado_actualizacion`
- `resultado_actualizacion = [ confirmado | rechazado ]`

`I = @id_orden + @id_reclamo + servicio + categoria + urgencia + cuadrilla + fecha_asignacion + fecha_tope`

`J = @id_orden + @id_reclamo + fecha_avance + descripcion + estado_parcial`

`K = @id_orden + @id_reclamo + fecha_resolucion + resultado + (detalle)`

`L = @id_reclamo + fecha_derivacion + area_comercial + resumen_del_caso`
- `area_comercial = [ facturacion | cobranza ]`

`M = @id_reclamo + fecha_resolucion + resultado_comercial + (detalle)`
- `resultado_comercial = [ anulacion_factura | ajuste_factura | reclamacion_infundada | otro ]`

`N = @id_normativa + servicio + categoria + urgencia + plazo_maximo_dias + vigencia_desde`

`O = periodo + { 1{ reclamo_reportado }N } + indicadores`
- `periodo = AAAA + MM`
- `reclamo_reportado = @id_reclamo + servicio + categoria + fecha_recepcion + estado + fecha_cierre + (dias_atencion)`
- `dias_atencion = fecha_cierre − fecha_recepcion`
- `indicadores = total_ingresados + total_cerrados + total_vencidos + porcentaje_cumplimiento`

`P = @id_reclamo + fecha_tope + dias_restantes`

`Q = @id_reclamo + fecha_tope + exceso_dias`

`R = @id_reclamo + servicio + categoria + descripcion + urgencia`

`S = @id_reclamo + resultado + fecha_cierre + (detalle)`

## 3. Ecuaciones de flujos internos

`T = @id_reclamo + fecha_recepcion + canal + caratula`
- `caratula = numero_reclamo + serie`

*Alias:* `solicitud válida = A` *Datos que superaron la validación*

`AE = @id_reclamo + fecha_recepcion + canal + servicio + categoria + descripcion`
- *Reclamo registrado sin carátula, antes de asignar carátula (P1.2 → P2.1)*

`U = @id_reclamo + servicio + categoria + urgencia + via_atencion`
- `via_atencion = [ tecnica | comercial ]`

`V = @id_reclamo + servicio + categoria + urgencia`

`W = @id_reclamo + servicio + categoria + urgencia`

`X = @id_reclamo + fecha_tope`

`Y = @id_reclamo + fecha_resolucion + resultado`

`Z = @id_reclamo + verificacion_cierre`
- `verificacion_cierre = resuelto + avances_completos + sin_observaciones`

`AA = @id_reclamo + fecha_cierre + resultado`

## 4. Almacenes de Datos (AD)

`AD: Usuarios = { 1{ B }N }`

`AD: Reclamos = { 1{ AR }N }`
- `AR = @id_reclamo + id_usuario + fecha_recepcion + canal + servicio + categoria + urgencia + descripcion + estado + fecha_tope + (fecha_cierre) + (resultado)`

`AD: Normativa Regulatoria = { 1{ N }N }`

`AD: Órdenes de Trabajo = { 1{ I }N }`
- `I = @id_orden + @id_reclamo + cuadrilla + fecha_asignacion + estado_orden`

`AD: Avances de Trabajo = { 1{ AB }N }`
- `AB = @id_avance + @id_orden + fecha_avance + descripcion + estado_parcial`

`AD: Reportes = { 1{ AC }N }`
- `AC = @id_reporte + tipo_reporte + periodo + fecha_generacion + contenido`
- `tipo_reporte = [ operativo_diario | regulatorio_mensual ]`

## 5. Tabla de catálogos (valores simples reutilizados)

| Catálogo | Valores |
|---|---|
| `canal` | `[ presencial \| telefonico \| web ]` |
| `servicio` | `[ agua \| luz ]` |
| `categoria` | `[ corte \| facturacion \| fuga \| falla_tecnica ]` |
| `urgencia` | `[ programada \| normal \| alta \| critica ]` |
| `estado` | `[ registrado \| clasificado \| en_atencion_tecnica \| en_atencion_comercial \| resuelto \| cerrado \| escalado ]` |
| `resultado` | `[ resuelto \| descartado \| derivado ]` |
| `estado_orden` | `[ asignada \| en_curso \| resuelta ]` |
| `estado_parcial` | `[ iniciado \| en_proceso \| verificado ]` |
| `datos_identificacion` | `[ documento \| @id_reclamo ]` |
| `area_comercial` | `[ facturacion \| cobranza ]` |
| `via_atencion` | `[ tecnica \| comercial ]` |
| `resultado_comercial` | `[ anulacion_factura \| ajuste_factura \| reclamacion_infundada \| otro ]` |

## 6. Reglas de integridad (referenciales)

- `AD: Reclamos.id_usuario` referencia `AD: Usuarios.B.@id_usuario` (FK).
- `AD: Órdenes de Trabajo.id_reclamo` referencia `AD: Reclamos.AR.@id_reclamo` (FK).
- `AD: Avances de Trabajo.id_orden` referencia `AD: Órdenes de Trabajo.I.@id_orden` (FK).
- `AD: Reclamos.fecha_tope` se deriva por `N` (servicio + categoria + urgencia) de `AD: Normativa Regulatoria`.
- Un `reclamo` en estado comercial no posee `orden_trabajo`; uno técnico no posee `derivacion_comercial` (excluyencia documentada en 09-especificaciones.md, P2.1.3).