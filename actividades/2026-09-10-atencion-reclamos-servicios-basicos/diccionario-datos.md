# Diccionario de Datos — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Definición algebraica de los flujos y almacenes del modelo, conforme a la notación Yourdon: `=` compuesto de · `+` concatenación · `[|]` selección · `{}` iteración · `()` opcional · `@` identificador · `* *` comentario.

## Flujos externos (Modelo Ambiental)

| Flujo | Definición |
|-------|-------------|
| SOLICITUD_RECLAMO | = CANAL + TIPO_SERVICIO + CATEGORIA + DESCRIPCION + DATOS_USUARIO |
| COMPROBANTE_RECLAMO | = @NUMERO_COMPROBANTE + FECHA_REGISTRO + DATOS_RECLAMO + DATOS_USUARIO |
| CONSULTA_ESTADO | = @NUMERO_COMPROBANTE + (DATOS_USUARIO) |
| ESTADO_RECLAMO | = DATOS_RECLAMO + ESTADO + 1{AVANCE_REGISTRO}N |
| NOTIFICACION_RESOLUCION | = @ID_RECLAMO + RESULTADO + FECHA_CIERRE + CANAL |
| COMPROBANTE_REENVIADO | = COMPROBANTE_RECLAMO |
| DATO_CONTACTO_ACTUALIZADO | = @ID_USUARIO + (TELEFONO) + (EMAIL) + (DIRECCION) |

## Flujos internos entre subsistemas

| Flujo | Definición |
|-------|-------------|
| RECLAMO_REGISTRADO | = DATOS_RECLAMO + FECHA_RECEPCION |
| RECLAMO_CLASIFICADO | = RECLAMO_REGISTRADO + CATEGORIA + NIVEL_URGENCIA + FECHA_LIMITE |
| RECLAMO_ASIGNADO | = RECLAMO_CLASIFICADO + (ID_ORDEN) + (ID_DERIVACION) |
| RECLAMO_RESUELTO | = @ID_RECLAMO + RESULTADO + FECHA_RESOLUCION + 1{AVANCE_REGISTRO}N |

* (ID_ORDEN) y (ID_DERIVACION) son **mutuamente excluyentes**: `[ID_ORDEN | ID_DERIVACION]`.

## Flujos con Entidades Externas

| Flujo | Definición |
|-------|-------------|
| ORDEN_TRABAJO | = @ID_ORDEN + DATOS_CUADRILLA + DATOS_RECLAMO + FECHA_LIMITE |
| REPORTE_AVANCE | = @ID_ORDEN + FECHA + ESTADO + OBSERVACIONES + (FECHA_FIN) |
| DERIVACION_COMERCIAL | = @ID_RECLAMO + MOTIVO_DERIVACION + DATOS_RECLAMO |
| RESOLUCION_COMERCIAL | = @ID_RECLAMO + RESULTADO + FECHA_ATENCION + OBSERVACIONES |
| NORMATIVA_Y_PLAZOS | = 1{RESOLUCION_NORMATIVA}N |
| RESOLUCION_NORMATIVA | = @ID_NORMA + TIPO_SERVICIO + CATEGORIA + PLAZO_MAXIMO + VIGENCIA |
| REPORTE_REGULATORIO | = PERIODO + 1{DATOS_RECLAMO + TIEMPO_ATENCION + ESTADO}N + METRICAS_CUMPLIMIENTO |

## Flujos de control (eventos T/C)

| Flujo | Definición |
|-------|-------------|
| ALERTA_VENCIMIENTO | = @ID_RECLAMO + FECHA_LIMITE + DIAS_RESTANTES + URGENCIA |
| ALARMA_RECLAMO_CRITICO | = @ID_RECLAMO + CRITERIO_CRITICO + ZONA + POBLACION_AFECTADA |

## Estructuras reutilizadas

| Estructura | Definición |
|------------|-------------|
| DATOS_USUARIO | = @ID_USUARIO + NOMBRE + DOCUMENTO + TELEFONO + (EMAIL) + (DIRECCION) + CONTRATO |
| DATOS_RECLAMO | = @ID_RECLAMO + FECHA_RECEPCION + TIPO_SERVICIO + CATEGORIA + DESCRIPCION + ESTADO |
| DATOS_CUADRILLA | = @ID_CUADRILLA + NOMBRE + ESPECIALIDAD + ZONA + DISPONIBILIDAD |
| AVANCE_REGISTRO | = FECHA + DESCRIPCION_AVANCE + ESTADO_PREVIO + ESTADO_NUEVO + RESPONSABLE |
| CANAL | = [presencial | telefonico | web] |
| TIPO_SERVICIO | = [agua | luz] |
| CATEGORIA | = [corte | facturacion | fuga | falla] |
| NIVEL_URGENCIA | = [alta | media | baja] |
| ESTADO | = [recibido | clasificado | asignado | en_atencion | resuelto | cerrado | vencido] |

## Almacenes de datos

| Almacén (AD) | Contenido |
|--------------|-----------|
| AD: Usuarios | = 1{DATOS_USUARIO}N; **@ = ID_USUARIO** |
| AD: Reclamos | = 1{DATOS_RECLAMO + NIVEL_URGENCIA + FECHA_LIMITE}N; **@ = ID_RECLAMO** |
| AD: Clasificaciones | = 1{ID_CLASIFICACION + TIPO_SERVICIO + CATEGORIA + NIVEL_URGENCIA + CRITERIO}N; **@ = ID_CLASIFICACION** |
| AD: Normativas | = 1{RESOLUCION_NORMATIVA}N; **@ = ID_NORMA** |
| AD: Cuadrillas | = 1{DATOS_CUADRILLA}N; **@ = ID_CUADRILLA** |
| AD: Ordenes de Trabajo | = 1{ORDEN_TRABAJO}N; **@ = ID_ORDEN** |
| AD: Seguimiento | = 1{ID_SEGUIMIENTO + @ID_RECLAMO + AVANCE_REGISTRO}N; **@ = ID_SEGUIMIENTO** |
| AD: Notificaciones | = 1{ID_NOTIFICACION + @ID_RECLAMO + FECHA_ENVIO + CANAL + CONTENIDO + ESTADO_ENVIO}N; **@ = ID_NOTIFICACION** |

## Puntos Clave

- **La alternativa `[ID_ORDEN | ID_DERIVACION]`** explicita la exclusividad técnica/comercial de la asignación, regla central del dominio.
- **Los plazos se expresan en días hábiles** (ver NORMATIVA_Y_PLAZOS), lo que obliga al cálculo de fecha límite en el primitivo P2.3.2.1.
- **Todos los flujos de control referencian @ID_RECLAMO**: la alerta y la alarma siempre se vinculan a un caso rastreable.