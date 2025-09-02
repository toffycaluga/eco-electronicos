# Contrato de Datos — v1

**Ámbito:** Plataforma de alquiler de automóviles  
**Responsable:** Persona A (Diseño conceptual)  
**Estado:** v1 publicado (borrador estable). Cambios posteriores se anunciarán como *diff*.

---

## 1. Nomenclatura oficial

- Entidades (conceptual): **Pascal Case** singular (Cliente, Vehiculo, Alquiler, Pago).
- Atributos (canónicos): **snake_case**.
- Claves:
  - Primaria: `id_<entidad>` (p. ej., `id_cliente`).
  - Foránea: `id_<entidad_referida>` (p. ej., `id_vehiculo`).
- Fechas: `fecha_inicio`, `fecha_fin`, `fecha_pago`.
- Dinero: `precio_dia`, `monto`.
- Unicidad: `email` único por cliente.

> Nota: Persona B usará los **mismos nombres canónicos** en SQL (MySQL 8).

---

## 2. Entidades y atributos (canónicos)

### 2.1 Cliente
- **PK**: `id_cliente`
- `nombre` — texto corto; nombre completo visible.
- `telefono` — texto; formato libre (normalización futura).
- `email` — texto; **único** (regla de negocio).
- `direccion` — texto.

### 2.2 Vehiculo
- **PK**: `id_vehiculo`
- `marca` — texto.
- `modelo` — texto.
- `anio` — entero (año calendario).
- `precio_dia` — decimal; **no negativo**.

### 2.3 Alquiler
- **PK**: `id_alquiler`
- **FK**: `id_cliente` → Cliente
- **FK**: `id_vehiculo` → Vehiculo
- `fecha_inicio` — fecha.
- `fecha_fin` — fecha; **>= `fecha_inicio`**.
- `estado_alquiler` *(futuro_uso, opcional en v1)*: `{reservado, en_curso, finalizado, cancelado}`.

### 2.4 Pago
- **PK**: `id_pago`
- **FK**: `id_alquiler` → Alquiler
- `monto` — decimal; **no negativo**.
- `fecha_pago` — fecha.
- `metodo_pago` *(futuro_uso)*: `{efectivo, tarjeta, transferencia}`.

---

## 3. Relaciones y cardinalidades

- **Cliente 1 — N Alquiler**: un cliente puede tener múltiples alquileres; un alquiler pertenece a un único cliente.
- **Vehiculo 1 — N Alquiler**: un vehículo puede participar en múltiples alquileres (no superpuestos temporalmente); un alquiler corresponde a un único vehículo.
- **Alquiler 1 — N Pago**: un alquiler puede tener **0..N** pagos; todos los pagos se asocian a un único alquiler.

---

## 4. Reglas de negocio (vinculantes)

1. **Fechas válidas:** `fecha_fin >= fecha_inicio`.
2. **No negativos:** `precio_dia >= 0`, `monto >= 0`.
3. **Email único:** no pueden existir dos clientes con el mismo `email`.
4. **Días de arriendo (definición canónica):**  
   `dias_alquilados = fecha_fin - fecha_inicio + 1` (cómputo **inclusivo**).
5. **Alquiler activo (hoy):**  
   `activo_si = (CURRENT_DATE BETWEEN fecha_inicio AND fecha_fin)`.
6. **“Alquilado en un mes dado” (solapamiento temporal):**  
   Un alquiler cuenta para un mes **si se superpone** con la ventana `[primer_dia_mes, primer_dia_mes_siguiente)`.
7. **Disponibilidad de vehículo en fecha X:**  
   Disponible si **no existe** alquiler tal que `X BETWEEN fecha_inicio AND fecha_fin`.

---

## 5. Enumeraciones y dominios (futuro_uso)

- `estado_alquiler`: `{reservado, en_curso, finalizado, cancelado}`  
  - Transiciones válidas: `reservado → en_curso → finalizado`; en cualquier momento `→ cancelado` (si procede).
- `metodo_pago`: `{efectivo, tarjeta, transferencia}`.

*(Persona B puede diferir su implementación; mantener en contrato para consistencia semántica.)*

---

## 6. Vistas/derivados (definiciones semánticas, no físicas)

- **Total a pagar por alquiler (sin descuentos/recargos):**  
  `total_arriendo = dias_alquilados * precio_dia`.
- **Total pagado por cliente:** suma de `monto` de todos los pagos de sus alquileres.
- **Promedio de pago por cliente:** promedio de `monto` (0 si no hay pagos).

---

## 7. Gestión de cambios (diffs)

- Cualquier ajuste (nombre, atributo, estado) se publicará como **changelog** al final de este documento:
  - `YYYY-MM-DD — cambio — motivo — impacto en B/C`.

---

## 8. Apéndice — Tabla de nombres canónicos (resumen)

| Tipo       | Nombre | Atributos clave |
|------------|--------|------------------|
| Entidad    | Cliente | id_cliente, nombre, telefono, email, direccion |
| Entidad    | Vehiculo | id_vehiculo, marca, modelo, anio, precio_dia |
| Entidad    | Alquiler | id_alquiler, id_cliente, id_vehiculo, fecha_inicio, fecha_fin, (estado_alquiler) |
| Entidad    | Pago | id_pago, id_alquiler, monto, fecha_pago, (metodo_pago) |

