
erDiagram
  CLIENTE ||--o{ SOLICITUD_RETIRO : "realiza"
  SOLICITUD_RETIRO ||--o| ASIGNACION_RETIRO : "tiene"
  VEHICULO ||--o{ ASIGNACION_RETIRO : "usa"
  CONDUCTOR ||--o{ ASIGNACION_RETIRO : "conduce"
  SOLICITUD_RETIRO ||--o{ DETALLE_RECICLAJE : "incluye"
  TIPO_ARTICULO ||--o{ DETALLE_RECICLAJE : "clasifica"

  CLIENTE {
    bigint id_cliente PK
    enum   tipo_doc
    varchar nro_doc
    varchar nombre
    varchar telefono
    varchar email
    varchar direccion
    varchar ciudad
    varchar comuna
    datetime created_at
    datetime updated_at
  }

  SOLICITUD_RETIRO {
    bigint id_solicitud PK
    bigint cliente_id FK
    date   fecha_solicitada
    enum   franja_horaria
    varchar direccion_retiro
    varchar ciudad
    enum   estado
    text   observaciones
    datetime created_at
  }

  ASIGNACION_RETIRO {
    bigint id_asignacion PK
    bigint solicitud_id FK 
    bigint vehiculo_id FK
    bigint conductor_id FK
    date   fecha_programada
    enum   franja_horaria
    enum   estado
    time   hora_inicio_estimada
    time   hora_fin_estimada
  }

  VEHICULO {
    bigint id_vehiculo PK
    varchar patente
    enum   tipo
    decimal capacidad_kg
    bool   activo
    text   notas
  }

  CONDUCTOR {
    bigint id_conductor PK
    varchar nombre
    varchar rut_dni
    varchar telefono
    enum   licencia_tipo
    varchar email
    bool   activo
  }

  TIPO_ARTICULO {
    bigint id_tipo PK
    varchar nombre
    enum   categoria
    bool   requiere_manejo_especial
    enum   unidad_medida
    decimal valor_estimado_por_kg
    text   comentarios
  }

  DETALLE_RECICLAJE {
    bigint id_detalle PK
    bigint solicitud_id FK
    bigint tipo_articulo_id FK
    int    cantidad
    decimal peso_kg
    enum   estado
    text   observaciones
  }
