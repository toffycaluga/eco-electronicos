# Plataforma de Alquiler de Automóviles

**Contexto académico:** Trabajo **grupal** del **Bootcamp Python de Skillnest**.  

---
---

## 🎯 Objetivos cubiertos
1. **Identificación de entidades y relaciones** necesarias para la gestión de clientes, solicitudes de retiro, vehículos, conductores y artículos reciclados.  
2. **Representación gráfica** del modelo conceptual en un diagrama E-R.  
3. **Definición de atributos, claves y restricciones** en un contrato de datos formal (`contrato_datos_v1.md`).  
4. **Estandarización de nomenclatura y estados** para facilitar consistencia en todo el equipo.  
5. **Mini-glosario** con definiciones claras de los términos centrales.

---

## 🖼️ Diagrama Entidad–Relación
El diagrama preliminar se encuentra en:

- `01_ER/Diagrama_ER.md` (código en Mermaid para exportar a PNG/PDF).  

Permite visualizar entidades fuertes, entidad débil (**detalle_reciclaje**) y las cardinalidades principales:
- Cliente 1—N Solicitudes  
- Solicitud 1—0..1 Asignación  
- Solicitud 1—N Detalles  
- Tipo de artículo 1—N Detalles  
- Vehículo / Conductor 1—N Asignaciones  

---

## 📑 Contrato de Datos v1
El contrato oficial se encuentra en:

- `contratos/contrato_datos_v1.md`

Incluye:
- **Convenciones de nombres** (tablas en plural, snake_case, claves canónicas).  
- **Enumeraciones oficiales** (ej.: estados de solicitudes, franjas horarias, tipos de vehículos, licencias de conductor, etc.).  
- **Esquema canónico** con atributos, tipos de datos, PK, FK y restricciones de unicidad.  
- **Reglas de negocio clave**, como:
  - Una solicitud tiene **máx. una asignación**.  
  - No hay doble booking de vehículo/conductor en la misma fecha y franja.  
  - Evitar duplicar líneas del mismo tipo de artículo en una solicitud.  
- **Ciclos de vida** de Solicitud, Asignación y Detalle.  
- **Nivel de normalización:** hasta **3FN**.

---

## 📖 Mini-glosario
Disponible en:

- `contratos/mini_glosario.md`

Define términos clave como **Cliente**, **Solicitud de Retiro**, **Asignación de Retiro**, **Detalle de Reciclaje**, **Franja Horaria** y **Estado**, para que todo el equipo maneje un mismo vocabulario.

---

## 🚦 Próximos pasos
- Personas B y C pueden avanzar con:  
  - **Transformación al modelo relacional** (SQL DDL).  
  - **Normalización explícita** hasta 3FN.  
  - **Diccionario de datos con ejemplos**.  
- Si se realizan cambios en entidades, atributos o estados, se actualizará **únicamente** el contrato (`contrato_datos_v1.md`) y se comunicará un *diff*.

---

## ✅ Estado de entrega (Persona A)
- [x] Borrador de Diagrama E-R publicado.  
- [x] Contrato de Datos v1 redactado.  
- [x] Mini-glosario entregado.  

## ✅ Estado de entrega (Persona B)
- [x] Transformación al modelo relacional (SQL DDL)

## Integrantes 

- **Moises Ortega**
- **Tatu Vergara**
- **Abraham Lillo**
- **Leonardo V**
