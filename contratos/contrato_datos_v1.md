
# Contrato de Datos — ECO-ELECTRÓNICOS (v1.0)

**Responsable:** Persona A  
**Ámbito:** Modelo relacional para gestión de clientes, solicitudes, asignaciones y reciclaje.  
**Nota de proceso:** Cualquier cambio posterior **debe** reflejarse aquí y anunciar diff.

---

## 1. Convenciones de nombres
- Tablas: `snake_case` en **plural** (ej. `solicitudes_retiro`, `detalles_reciclaje`).
- PK: `id_<entidad>` (`BIGINT UNSIGNED AUTO_INCREMENT`).
- FK: `<entidad>_id` usando mismo tipo que la PK referida.
- Timestamps: `created_at`, `updated_at` (`DATETIME`).
- Booleanos: `BOOLEAN` (`0`/`1`).
- Dominios controlados: `ENUM` para estados, categorías, franjas y tipos.

## 2. Dominios y enumeraciones (canónicos)
- `cliente.tipo_doc`: `RUT`, `DNI`, `PASAPORTE`
- `solicitudes_retiro.franja_horaria` y `asignaciones_retiro.franja_horaria`:
  - `AM(08-12)`, `PM(12-16)`, `NOCHE(16-20)`
- `solicitudes_retiro.estado`:
  - `pendiente`, `programada`, `en_ruta`, `completada`, `cancelada`
- `asignaciones_retiro.estado`:
  - `asignada`, `en_ruta`, `finalizada`, `cancelada`
- `tipos_articulo.categoria`:
  - `Computo`, `Telefonia`, `AudioVideo`, `Electrodomestico`, `Bateria`, `Otro`
- `tipos_articulo.unidad_medida`:
  - `kg`, `unidad`
- `vehiculos.tipo`:
  - `Camioneta`, `Camión`, `Furgón`, `Moto`
- `conductores.licencia_tipo`:
  - `A1`, `A2`, `A3`, `B`, `C`, `D`, `E`
- `detalles_reciclaje.estado`:
  - `reportado`, `recibido`, `rechazado`

> Si estos dominios cambian con frecuencia o requieren multi-idioma, migrar a **tablas catálogo** con FK.

## 3. Esquema canónico (tablas y claves)

### 3.1 `clientes`
- **PK:** `id_cliente`
- **Únicos:** `nro_doc`, `email`
- **Campos:**
  - `tipo_doc` (ENUM) **NOT NULL**
  - `nro_doc` (VARCHAR(20)) **NOT NULL**
  - `nombre` (VARCHAR(120)) **NOT NULL**
  - `telefono` (VARCHAR(20)) **NOT NULL**
  - `email` (VARCHAR(120)) **NOT NULL**
  - `direccion` (VARCHAR(200)) **NOT NULL**
  - `ciudad` (VARCHAR(100)) **NOT NULL**
  - `comuna` (VARCHAR(100)) NULL
  - `created_at`, `updated_at`
- **Relación:** CLIENTE 1—N SOLICITUD_RETIRO

### 3.2 `solicitudes_retiro`
- **PK:** `id_solicitud`
- **FK:** `cliente_id` → `clientes.id_cliente`
- **Campos:**
  - `fecha_solicitada` (DATE) **NOT NULL**
  - `franja_horaria` (ENUM) **NOT NULL**
  - `direccion_retiro` (VARCHAR(200)) **NOT NULL**
  - `ciudad` (VARCHAR(100)) **NOT NULL**
  - `estado` (ENUM) **NOT NULL** default `pendiente`
  - `observaciones` (TEXT) NULL
  - `created_at` (DATETIME) **NOT NULL**
- **Índices:** `idx_sol_estado(estado)`, `idx_sol_fecha(fecha_solicitada)`
- **Relaciones:** 1—0..1 ASIGNACION_RETIRO; 1—N DETALLE_RECICLAJE

### 3.3 `asignaciones_retiro`
- **PK:** `id_asignacion`
- **FK:** `solicitud_id` → `solicitudes_retiro.id_solicitud` (**UNIQUE**)
- **FK:** `vehiculo_id` → `vehiculos.id_vehiculo`
- **FK:** `conductor_id` → `conductores.id_conductor`
- **Campos:**
  - `fecha_programada` (DATE) **NOT NULL**
  - `franja_horaria` (ENUM) **NOT NULL**
  - `estado` (ENUM) **NOT NULL** default `asignada`
  - `hora_inicio_estimada` (TIME) NULL
  - `hora_fin_estimada` (TIME) NULL
- **Restricciones anti-solape:**
  - **UNIQUE**(`vehiculo_id`, `fecha_programada`, `franja_horaria`)
  - **UNIQUE**(`conductor_id`, `fecha_programada`, `franja_horaria`)

### 3.4 `vehiculos`
- **PK:** `id_vehiculo`
- **Únicos:** `patente`
- **Campos:**
  - `patente` (VARCHAR(12)) **NOT NULL**
  - `tipo` (ENUM) **NOT NULL**
  - `capacidad_kg` (DECIMAL(10,2)) **NOT NULL** (>0)
  - `activo` (BOOLEAN) **NOT NULL** default `1`
  - `notas` (TEXT) NULL

### 3.5 `conductores`
- **PK:** `id_conductor`
- **Únicos:** `rut_dni`, `email` (NULLable)
- **Campos:**
  - `nombre` (VARCHAR(120)) **NOT NULL**
  - `rut_dni` (VARCHAR(20)) **NOT NULL**
  - `telefono` (VARCHAR(20)) **NOT NULL**
  - `licencia_tipo` (ENUM) **NOT NULL**
  - `email` (VARCHAR(120)) NULL
  - `activo` (BOOLEAN) **NOT NULL** default `1`

### 3.6 `tipos_articulo`
- **PK:** `id_tipo`
- **Únicos:** `nombre`
- **Campos:**
  - `nombre` (VARCHAR(80)) **NOT NULL**
  - `categoria` (ENUM) **NOT NULL**
  - `requiere_manejo_especial` (BOOLEAN) **NOT NULL** default `0`
  - `unidad_medida` (ENUM) **NOT NULL** default `kg`
  - `valor_estimado_por_kg` (DECIMAL(10,2)) NULL
  - `comentarios` (TEXT) NULL

### 3.7 `detalles_reciclaje`
- **PK:** `id_detalle`
- **FK:** `solicitud_id` → `solicitudes_retiro.id_solicitud` (DELETE CASCADE)
- **FK:** `tipo_articulo_id` → `tipos_articulo.id_tipo`
- **Únicos:** **UNIQUE**(`solicitud_id`, `tipo_articulo_id`)
- **Campos:**
  - `cantidad` (INT) **NOT NULL** ≥ 1
  - `peso_kg` (DECIMAL(10,2)) NULL ≥ 0
  - `estado` (ENUM) **NOT NULL** default `reportado`
  - `observaciones` (TEXT) NULL

## 4. Reglas de negocio clave
1. Una solicitud tiene **a lo sumo una** asignación: `UNIQUE(solicitud_id)` en `asignaciones_retiro`.
2. **No hay doble booking** por franja: unicidad por (`vehiculo_id`, `fecha_programada`, `franja_horaria`) y por (`conductor_id`, `fecha_programada`, `franja_horaria`).
3. En `detalles_reciclaje`, cada (`solicitud_id`, `tipo_articulo_id`) es único para evitar duplicados de línea (ajustable según negocio).
4. `clientes.email`, `clientes.nro_doc`, `vehiculos.patente`, `conductores.rut_dni` son **únicos**.

## 5. Ciclos de vida (máquinas de estado)
**Solicitud:**  
`pendiente` → (`programada` | `cancelada`) → (`en_ruta` | `cancelada`) → `completada`  

**Asignación:**  
`asignada` → `en_ruta` → `finalizada` (o `cancelada` en cualquier etapa previa)  

**Detalle:**  
`reportado` → (`recibido` | `rechazado`)

## 6. Índices sugeridos
- `solicitudes_retiro(estado)`, `solicitudes_retiro(fecha_solicitada)`
- `asignaciones_retiro(fecha_programada, franja_horaria)`
- `detalles_reciclaje(solicitud_id)`

## 7. Normalización
- Esquema en **3FN**. Dominios controlados por `ENUM`; `tipos_articulo` separado como catálogo.
- Sin dependencias parciales ni transitivas en tablas transaccionales.
- No se almacenan agregados derivables (ej.: total de peso por solicitud).

## 8. Roadmap de cambios
- **v1.0:** Borrador estable para trabajo de B y C.
- **v1.1 (propuesto):** Localidades normalizadas (ciudad/comuna/region) y migrar ENUMs a catálogos si requieren administración dinámica.
