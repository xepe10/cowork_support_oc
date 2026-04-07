# HOWTO – Cómo dar de baja manualmente a una persona (Persona + Planilla)

> **Importante:** Este procedimiento es excepcional. Debe usarse solo cuando el flujo normal de liquidación/baja **no se completó correctamente** y dejó inconsistencias (por ejemplo, persona activa en módulo Persona, pero con liquidación finalizada en planilla). Siempre que sea posible, preferir la corrección del flujo automático.

## 1. Verificaciones previas

Antes de hacer cambios manuales:

1. **Confirmar con el cliente**:
   - Que la persona efectivamente debe estar de baja.
   - Que la liquidación final fue revisada y es correcta.

2. **Revisar estado actual en Planilla**:
   - Verificar en módulo planilla que:
     - Exista una **liquidación finalizada** para la persona.

3. **Revisar estado actual en Persona (MongoDB)**:
   - Verificar en el módulo Persona si la persona sigue como **activa**.
   - Si hay dudas, consultar en BD (MongoDB) con alguien de IT para ver:
     - `persona`.
     - `silla`.
     - `registrotrayectoria`.
     - `baja`.
     - `logestadopersona`.

4. **Tomar backup puntual** (recomendado):
   - Exportar los documentos relevantes de la persona en MongoDB y registros de planilla/contrato/colaborador antes de tocar nada.

---

## 2. Qué significa "estar de baja" (referencia)

### 2.1. En módulo Persona (MongoDB)

Una persona está **de baja** cuando se cumple todo lo siguiente:

- **Tabla `persona`**
  - `estadopersona = "debaja"`.

- **Tabla `silla`**
  - La **última silla** de la persona está desactivada.
  - `fecha_fin` = fecha de baja.

- **Tabla `registrotrayectoria`**
  - El registro de trayectoria de la última silla tiene `fecha_fin` = fecha de baja.

- **Tabla `baja`**
  - Existe un registro de baja asociado:
    - a la persona,
    - y a la última silla.

- **Tabla `logestadopersona`**
  - Existe un registro con estado `debaja`, asociado a la baja.

### 2.2. En módulo Planilla

Una persona está **de baja** en planilla cuando:

- **Tabla `contrato`**
  - `fecha_fin` está seteada.
  - `pendiente = true`.

- **Tabla `colaborador`**
  - `fecha_baja` registrada.

- **Tabla `liquidacion`**
  - La liquidación asociada está en estado **finalizado**.

---

## 3. Pasos para dar de baja manualmente (orden recomendado)

> **Nota:** Estos pasos deben ejecutarse por alguien con conocimiento de la estructura de datos (IT) y, cuando sea posible, a través de herramientas internas/consultas preparadas en vez de tocar directamente desde consola.

### Paso 1 – Confirmar liquidación finalizada (Planilla)

1. En módulo planilla, localizar al colaborador.
2. Verificar que tiene una **liquidación en estado finalizado** para la fecha de baja correcta.
3. Si la liquidación no está finalizada o hay dudas, resolver primero ese punto.

### Paso 2 – Ajustar datos en Planilla

En la base de datos de planilla (según corresponda a tu entorno):

1. **Tabla `contrato`**
   - Localizar el contrato activo de la persona.
   - Setear:
     - `fecha_fin` = fecha de baja acordada.
     - `pendiente = true`.

2. **Tabla `colaborador`**
   - Localizar el registro del colaborador.
   - Setear:
     - `fecha_baja` = fecha de baja.

> Después de este paso, del lado de planilla, la persona debería verse como de baja según las reglas de negocio.

### Paso 3 – Ajustar datos en Persona (MongoDB)

En MongoDB, asegurarse de cumplir las condiciones de baja.

1. **En `persona`**
   - Encontrar el documento de la persona.
   - Actualizar `estadopersona` a `"debaja"`.

2. **En `silla`**
   - Identificar la **última silla** asociada a la persona.
   - Desactivarla y setear `fecha_fin` = fecha de baja.

3. **En `registrotrayectoria`**
   - Ubicar el registro de trayectoria de la última silla.
   - Actualizar `fecha_fin` al valor de fecha de baja.

4. **En `baja`**
   - Verificar si existe un registro de baja asociado a esa persona y silla:
     - Si no existe, crear un registro de baja con:
       - referencia a la persona,
       - referencia a la silla,
       - fecha de baja.

5. **En `logestadopersona`**
   - Insertar un nuevo log con:
     - estado = `"debaja"`,
     - referencia al registro de baja creado,
     - fecha/hora de la operación,
     - usuario que ejecuta el cambio.

> Al terminar este paso, en Mongo Persona la persona debería aparecer como de baja y los historiales de silla/trayectoria/baja/log coherentes entre sí.

### Paso 4 – Verificaciones finales

1. **En módulo Persona**
   - Confirmar que la persona ya no aparece como activa.
   - Revisar que la silla y la trayectoria muestren fecha de fin correcta.

2. **En módulo Planilla**
   - Confirmar que la persona aparece de baja (sin planilla futura activa).
   - Confirmar que la liquidación sigue en estado finalizado.

3. **Coherencia Persona–Planilla**
   - Verificar con la auditoría de consistencia (cuando esté activa) que:
     - No queden inconsistencias para esa persona.

---

## 4. Registro y aprendizaje

Después de ejecutar este procedimiento manual:

- Documentar el caso como incidente (si no lo está).
- Actualizar, si aplica, el runbook correspondiente (por ejemplo:
  - `planilla_eliminada_liquidaciones_perdidas.md`
  - `persona_activa_liquidacion_finalizada.md`).
- Revisar si el problema provino de:
  - falla en flujo automático,
  - error manual,
  - o falta de proceso/documentación.

Y, sobre todo, usar este procedimiento como **último recurso**; el objetivo del plan de estabilización es reforzar los flujos automáticos y procesos para que cada vez tengamos que recurrir menos a la baja manual.
