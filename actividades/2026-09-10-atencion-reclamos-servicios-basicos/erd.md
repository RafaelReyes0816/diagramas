# DER: Diagrama Entidad-Relación — Sistema de Atención de Reclamos de Servicios Básicos

## Contexto

Modelo de datos para los almacenes comprendidos del sistema. Cubre el registro del usuario, el ciclo de vida del reclamo, la clasificación y normativa, la ejecución (técnica y comercial) y el seguimiento hasta el cierre.

## Diagrama

```mermaid
erDiagram
    USUARIO ||--o{ RECLAMO : "presenta"
    RECLAMO }o--|| CLASIFICACION : "recibe criterios"
    NORMATIVA_PLAZOS ||--o{ RECLAMO : "fija plazo"
    RECLAMO ||--o{ ORDEN_TRABAJO : "genera"
    CUADRILLA ||--o{ ORDEN_TRABAJO : "ejecuta"
    RECLAMO ||--o{ DERIVACION_COMERCIAL : "deriva"
    RECLAMO ||--o{ SEGUIMIENTO : "registra"
    RECLAMO ||--o{ NOTIFICACION : "emite"

    USUARIO {
        number id_usuario PK "identificador único"
        string nombre "nombre completo"
        string documento "ID documento"
        string telefono "tel. de contacto"
        string email "correo"
        string direccion "domicilio"
        string contrato "cuenta/contrato del servicio"
    }

    RECLAMO {
        number id_reclamo PK "número de reclamo"
        number id_usuario FK "usuario que reclama"
        number id_clasificacion FK "clasificación aplicada"
        number id_norma FK "norma de plazo aplicada"
        date fecha_recepcion "fecha de ingreso"
        string tipo_servicio "agua | luz"
        string categoria "corte | facturacion | fuga | falla"
        string descripcion "detalle del reclamo"
        string urgencia "alta | media | baja"
        date fecha_limite "plazo regulatorio"
        string estado "recibido | clasificado | asignado | en_atencion | resuelto | cerrado | vencido"
        string canal "presencial | telefonico | web"
        date fecha_cierre "cierre real del caso"
    }

    CLASIFICACION {
        number id_clasificacion PK "identificador"
        string tipo_servicio "agua | luz"
        string categoria "corte | facturacion | fuga | falla"
        string nivel_urgencia "alto | medio | bajo"
        string criterio "descripción del criterio"
    }

    NORMATIVA_PLAZOS {
        number id_norma PK "identificador"
        string tipo_servicio "agua | luz"
        string categoria "corte | facturacion | fuga | falla"
        number plazo_maximo "días hábiles límite"
        string vigencia "periodo de vigencia"
        string resolucion "referencia regulatoria"
    }

    CUADRILLA {
        number id_cuadrilla PK "identificador"
        string nombre "nombre de la cuadrilla"
        string especialidad "electricidad | hidráulica"
        string zona "zona de cobertura"
        string disponibilidad "disponible | ocupada"
    }

    ORDEN_TRABAJO {
        number id_orden PK "identificador"
        number id_reclamo FK "reclamo asociado"
        number id_cuadrilla FK "cuadrilla asignada"
        date fecha_asignacion "fecha de emisión"
        string tipo_trabajo "reparación | verificación | reinstalación"
        string estado "pendiente | en_proceso | finalizado"
        date fecha_fin "cierre del trabajo de campo"
    }

    DERIVACION_COMERCIAL {
        number id_derivacion PK "identificador"
        number id_reclamo FK "reclamo derivado"
        date fecha_derivacion "fecha de envío"
        string motivo "adjus facturación | error cargo | contrato"
        string estado "pendiente | atendido"
        date fecha_atencion "cierre del área comercial"
    }

    SEGUIMIENTO {
        number id_seguimiento PK "identificador"
        number id_reclamo FK "reclamo seguido"
        date fecha "fecha del avance"
        string avance "descripción del avance"
        string estado_previo "estado antes"
        string estado_nuevo "estado después"
        string responsable "origen del reporte"
    }

    NOTIFICACION {
        number id_notificacion PK "identificador"
        number id_reclamo FK "reclamo notificado"
        date fecha_envio "fecha de envío"
        string canal "sms | email | app | carta"
        string contenido "texto de la notificación"
        string estado_envio "enviado | fallido"
    }
```

## Puntos Clave

- **RECLAMO es la entidad núcleo**: agrega la clasificación y el plazo (FK a CLASIFICACION y NORMATIVA_PLAZOS), y **alterna** entre la ejecución técnica (ORDEN_TRABAJO) y la comercial (DERIVACION_COMERCIAL), reflejando la regla de asignación exclusiva del dominio.
- **CUADRILLA y NORMATIVA_PLAZOS son datos de referencia** que existen con independencia del caso; el resto del modelo depende de RECLAMO.
- El **SEGUIMIENTO mantiene el historial inmutable** (estado_previo/estado_nuevo) para la trazabilidad regulatoria.
- Correspondencia con almacenes: USUARIO → AD: Usuarios · RECLAMO/CLASIFICACION/NORMATIVA → AD: Reclamos/Clasificaciones/Normativas · ORDEN_TRABAJO/CUADRILLA → AD: Ordenes de Trabajo/Cuadrillas · SEGUIMIENTO → AD: Seguimiento · NOTIFICACION → AD: Notificaciones.