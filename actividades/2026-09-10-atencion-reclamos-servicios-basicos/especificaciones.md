# Especificaciones — Procesos Primitivos — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Lógica estructurada de cada proceso primitivo del DFD Nivel 4 y de los procesos de Nivel 3 que no requieren descomposición adicional (P3.x, P5.1, P5.2). Formato: pseudocódigo estructurado.

## P1.1.1 — Validar Datos del Usuario

```
LEER Solicitud de Reclamo
SI documento o contrato NO existen en AD: Usuarios ENTONCES
    CREAR registro usuario con DATOS_USUARIO
SINO
    VERIFICAR coincidencia de nombre y documento
FINSI
SI datos obligatorios faltantes ENTONCES
    DEVOLVER error de validación (sin registro)
SINO
    DEVOLVER DATOS_USUARIO validados
FINSI
```

## P1.1.2 — Validar Datos del Reclamo

```
LEER DATOS_USUARIO validados + TIPO_SERVICIO + CATEGORIA + DESCRIPCION
SI TIPO_SERVICIO ∉ [agua, luz] ENTONCES RECHAZAR
SI CATEGORIA ∉ [corte, facturacion, fuga, falla] ENTONCES RECHAZAR
SI DESCRIPCION está vacía ENTONCES RECHAZAR
DEVOLVER Reclamo Validado
```

## P1.1.3 — Persistir Reclamo

```
GENERAR @ID_RECLAMO
FECHA_RECEPCION := fecha y hora actual
ESTADO := "recibido"
GUARDAR registro en AD: Reclamos
DEVOLVER RECLAMO_REGISTRADO
```

## P2.2.1.1 — Verificar Riesgo Vital

```
LEER CATEGORIA + DESCRIPCION + ZONA
SI corte afecta servicio de salud, hospital o zona de riesgo declarada ENTONCES
    RIESGO_VITAL := cierto
SINO
    RIESGO_VITAL := falso
FIN
DEVOLVER RIESGO_VITAL
```

## P2.2.1.2 — Verificar Magnitud del Incidente

```
LEER CATEGORIA + DATOS de zona/población
SI fuga con caudal alto O corte masivo (población > umbral) ENTONCES
    MAGNITUD := "mayor"
SINO
    MAGNITUD := "menor"
FIN
DEVOLVER DATOS_DE_CRITICIDAD (RIESGO_VITAL, MAGNITUD)
```

## P2.2.2.1 — Determinar Nivel

```
SI RIESGO_VITAL = cierto O MAGNITUD = "mayor" ENTONCES
    NIVEL_URGENCIA := "alta"
SINO SI categoría = corte individual O falla sin riesgo ENTONCES
    NIVEL_URGENCIA := "media"
SINO
    NIVEL_URGENCIA := "baja"   (* facturación, correcciones comerciales *)
FIN
DEVOLVER NIVEL_URGENCIA
```

## P2.2.2.2 — Persistir Urgencia

```
ACTUALIZAR AD: Reclamos set urgencia = NIVEL_URGENCIA
   where id_reclamo = @ID_RECLAMO
ESTADO := "clasificado"
DEVOLVER Reclamo Priorizado
```

## P2.3.2.1 — Aplicar Días Hábiles

```
LEER PLAZO_MAXIMO (AD: Normativas) + FECHA_RECEPCION
dias_corridos := 0
FECHA := FECHA_RECEPCION
MIENTRAS dias_corridos <> PLAZO_MAXIMO
    FECHA := FECHA + 1 día
    SI FECHA es día hábil (lun–vie, no feriado) ENTONCES
        dias_corridos := dias_corridos + 1
    FINSI
FINMIENTRAS
DEVOLVER FECHA_LIMITE candidata
```

## P2.3.2.2 — Fijar Fecha Límite

```
ACTUALIZAR AD: Reclamos set fecha_limite = FECHA_LIMITE candidata
   where id_reclamo = @ID_RECLAMO
DEVOLVER Reclamo con Plazo
```

## P4.2.1.1 — Calcular Días Restantes

```
LEER FECHA_LIMITE (AD: Reclamos) de cada reclamo en estado ≠ cerrado
DIAS_RESTANTES := FECHA_LIMITE − FECHA_ACTUAL  (* días corridos, redondeo *)FIN
DEVOLVER DIAS_RESTANTES por reclamo
```

## P4.2.1.2 — Comparar con Umbral

```
LEER umbral de alerta (configuración, ej. 20% del PLAZO_MAXIMO)
POR CADA reclamo activo
    SI DIAS_RESTANTES ≤ umbral Y DIAS_RESTANTES > 0 ENTONCES
        MARCADO := "alerta"
    SI DIAS_RESTANTES < 0 ENTONCES
        ESTADO := "vencido" en AD: Reclamos
        MARCADO := "alerta vencido"
    FINSI
FINPOR
DEVOLVER Resultado del Monitoreo
```

## P4.2.2.1 — Conformar Alerta

```
PARA cada reclamo marcado
    CONFORMAR ALERTA_VENCIMIENTO (@ID_RECLAMO, FECHA_LIMITE, DIAS_RESTANTES, URGENCIA)
FINPARA
DEVOLVER Alerta Conformada
```

## P4.2.2.2 — Enviar Alerta

```
POR CADA Alerta Conformada
    ENVIAR a Cuadrilla Técnica responsable (zona de cobertura)
    REGISTRAR envío en AD: Notificaciones
FINPOR
```

## P4.3.1.1 — Evaluar Criticidad del Servicio

```
LEER DATOS_RECLAMO activos
SI TIPO_SERVICIO = "agua" Y CATEGORIA = "fuga" DE ALTO CAUDAL ENTONCES
    CRITICO := cierto
SI TIPO_SERVICIO = "luz" Y CATEGORIA = "corte" MASIVO ENTONCES
    CRITICO := cierto
SINO
    CRITICO := falso
FIN
```

## P4.3.1.2 — Evaluar Población Afectada

```
ESTIMAR POBLACION_AFECTADA por zona de cobertura
SI POBLACION_AFECTADA > umbral_crítico ENTONCES
    CONFIRMAR Caso Crítico
SI NO EXISTE (P4.3.1.1 = cierto) Y POBLACION ≤ umbral ENTONCES
    DESCARTAR escalamiento
FIN
DEVOLVER Caso Crítico Identificado (si aplica)
```

## P4.3.2.1 — Conformar Alarma

```
CONFORMAR ALARMA_RECLAMO_CRITICO (@ID_RECLAMO, CRITERIO, ZONA, POBLACION_AFECTADA)
DEVOLVER Alarma Conformada
```

## P4.3.2.2 — Notificar Escalamiento

```
ENVIAR Alarma Conformada a Cuadrilla Técnica y supervisor de zona
REGISTRAR en AD: Notificaciones
PROMOVER estado del reclamo a "en_atencion" (prioridad absoluta)
```

## P5.1 — Validar Cierre

```
LEER RECLAMO_RESUELTO
SI existe Orden de Trabajo finalizada O Derivación Comercial atendida
   Y avances registrados ENTONCES
    CIERRE := válido
SINO
    CIERRE := inválido (devuelve a P4)
FIN
SI FECHA_CIERRE real ≤ FECHA_LIMITE ENTONCES
    cumplimiento := "dentro de plazo"
SINO
    cumplimiento := "fuera de plazo"
FIN
DEVOLVER Caso Validado
```

## P5.2 — Notificar al Usuario

```
LEER RESULTADO de la resolución
SELECCIONAR CANAL preferido del usuario (sms > email > app > carta)
CONFORMAR NOTIFICACION_RESOLUCION
ENVIAR y REGISTRAR en AD: Notificaciones
ACTUALIZAR AD: Reclamos set estado = "cerrado", fecha_cierre
DEVOLVER Notificación de Resolución
```

## P5.3.1.1 — Extraer Reclamos del Periodo

```
DELIMITAR periodo (desde/hasta) solicitado por la normativa
EXTRAER de AD: Reclamos los casos con fecha_recepcion ∈ periodo
ACOMPAÑAR con FECHA_LIMITE y FECHA_CIERRE de cada caso
DEVOLVER Reclamos Extraídos
```

## P5.3.1.2 — Calcular Métricas de Cumplimiento

```
POR CADA reclamo extraído
    TIEMPO_ATENCION := FECHA_CIERRE − FECHA_RECEPCION
    SI FECHA_CIERRE ≤ FECHA_LIMITE ENTONCES cumplido += 1
FINPOR
METRICAS_CUMPLIMIENTO := (cumplidos/total) por categoría y tipo de servicio
DEVOLVER Datos Consolidados
```

📌 Leyenda de Notación
`:=` asignación · `[|]` selección · `∉` no pertenece · `( )` opcional · `* *` comentario · los identificadores `@` referencian claves del DD

## Puntos Clave

- **Las reglas de urgencia son tabulares y replicables** (P2.2.2.1): alto riesgo/magnitud → alta; corte o falla menor → media; comercial → baja.
- **El cálculo de plazo trabaja en días hábiles** (P2.3.2.1), alineado a la normativa regulatoria y al DD.
- **El monitoreo no detiene el caso**: solo alerta antes de vencer y marca "vencido" al superarlo (P4.2.1.2), dejando el registro inmutable como exige la trazabilidad.
- **La criticidad distingue "crítico" (alarma/post C2) de "urgencia" (prioridad de atención)**: ambos comparten criterios pero activan flujos distintos (escalamiento vs. secuencia normal).