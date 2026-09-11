# DER (Modelo de Datos): Sistema de Atención de Reclamos de Servicios Básicos (Agua y Luz)

## Contexto y Análisis Previo

Modelo de **Información** independiente del DFD (Modelo de Comportamiento). Se deriva de los Almacenes de Datos estabilizados en 03-dfd-nivel-1.md: `AD: Usuarios`, `AD: Reclamos`, `AD: Normativa Regulatoria`, `AD: Órdenes de Trabajo`, `AD: Avances de Trabajo` y `AD: Reportes`.

Decisiones de diseño:
- Un reclamo es **técnico o comercial**, no ambos: la asignación a cuadrilla (`ORDEN_TRABAJO`) y la derivación comercial (`DERIVACION_COMERCIAL`) son **alternativas mutuamente excluyentes** (0..1 en ambos casos).
- La normativa es un catálogo vigente que **establece el plazo** de cada reclamo según servicio + categoría + urgencia.
- Los avances solo existen vinculados a una orden de trabajo técnica.

## Diagrama

```mermaid
erDiagram
    USUARIO {
        int id_usuario PK "Identificador único"
        string nombre "Nombre completo"
        string documento "DNI/RUC"
        string telefono "Contacto telefónico"
        string email "Correo electrónico"
        string direccion "Dirección de suministro"
    }

    RECLAMO {
        int id_reclamo PK "Número de reclamo"
        int id_usuario FK "Usuario que reclama"
        int id_normativa FK "Plazo aplicado"
        date fecha_recepcion "Fecha de ingreso"
        string canal "Presencial, telefónico, web"
        string servicio "Agua o luz"
        string categoria "Corte, facturación, fuga, falla"
        string urgencia "Programada, normal, alta, crítica"
        string descripcion "Relato del usuario"
        string estado "Registrado, clasificado, en atención, resuelto, cerrado"
        date fecha_tope "Plazo regulatorio calculado"
        date fecha_cierre "Fecha de cierre efectivo"
        string resultado "Resultado final del caso"
    }

    NORMATIVA_PLAZO {
        int id_normativa PK "Identificador de la regla"
        string servicio "Agua o luz"
        string categoria "Corte, facturación, fuga, falla"
        string urgencia "Programada, normal, alta, crítica"
        int plazo_maximo_dias "Máximo regulatorio en días"
        date vigencia_desde "Inicio de vigencia"
    }

    ORDEN_TRABAJO {
        int id_orden PK "Identificador de la orden"
        int id_reclamo FK "Reclamo atendido"
        string cuadrilla "Cuadrilla designada"
        date fecha_asignacion "Fecha de asignación"
        string estado_orden "Asignada, en curso, resuelta"
    }

    AVANCE {
        int id_avance PK "Identificador del avance"
        int id_orden FK "Orden asociada"
        date fecha_avance "Fecha del reporte"
        string descripcion "Detalle del avance"
        string estado_parcial "Progreso parcial"
    }

    DERIVACION_COMERCIAL {
        int id_derivacion PK "Identificador de la derivación"
        int id_reclamo FK "Reclamo derivado"
        date fecha_derivacion "Fecha de derivación"
        string area_comercial "Área responsable (facturación/cobranza)"
        string estado_derivacion "Derivada, resuelta"
    }

    USUARIO ||--o{ RECLAMO : presenta
    NORMATIVA_PLAZO ||--o{ RECLAMO : establece_plazo
    RECLAMO o|--o| ORDEN_TRABAJO : genera
    RECLAMO o|--o| DERIVACION_COMERCIAL : deriva
    ORDEN_TRABAJO ||--o{ AVANCE : registra
```

📌 Leyenda del Diagrama
USUARIO = Entidad (tabla/almacén de datos)
||--o{ ∪ = Cardinalidad 1:N (uno a muchos)
o|--o| = Cardinalidad 0..1:0..1 (opcional)
PK = Clave primaria · FK = Clave foránea

## Puntos Clave

- **Modelo independiente:** el DER no es un nivel del DFD; describe la estructura estática de los almacenes identificados en el comportamiento.
- La excluyencia técnica/comercial se captura con dos relaciones opcionales desde `RECLAMO` y una regla de negocio documentada en 09-especificaciones.md (P2.1.3).
- `NORMATIVA_PLAZO` se relaciona con `RECLAMO` (1:N) porque cada reclamo recibe exactamente una regla vigente al momento de su clasificación.
- `AVANCE` depende existencialmente de `ORDEN_TRABAJO` (composición 1:N): sin orden técnica no existen avances.
- La cardinalidad 0..1 en `DERIVACION_COMERCIAL` permite que el cierre del reclamo se evalúe sobre el propio estado del registro sin entidades intermedias adicionales.