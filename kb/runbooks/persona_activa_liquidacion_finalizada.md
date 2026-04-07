# Runbook – Persona activa en módulo Persona con liquidación finalizada en planilla

## Resumen

Un colaborador aparece **activo** en el módulo Persona, pero ya tiene **liquidación finalizada** del lado de planilla. Esto revela una inconsistencia de estados entre Persona y Planilla/Liquidaciones.

## Contexto

- **Cliente:** Navinter (pero el problema puede presentarse en otros clientes).
- **Módulos:**
  - Personas (MongoDB)
  - Planillas / Liquidaciones
- **Síntoma:**
  - En módulo Persona, la persona figura como **activa**.
  - En módulo Planilla, la liquidación está **finalizada** y la persona debería estar de baja.

## Hallazgo

- Se encontró que el colaborador **no tiene registro de baja** en la tabla/colección de bajas en MongoDB.
  - Esto indica que, durante el proceso de liquidación, **no se creó el registro de baja**.
  - Por lo tanto, no tiene la información necesaria para que el módulo Persona lo identifique como de baja (estado, vínculo a silla, etc.).

- Antecedente:
  - Se había detectado previamente que en el **último paso de la liquidación** el formulario de baja **no cargaba correctamente**.
  - Como resultado:
    - Al finalizar la liquidación, la parte de baja no enviaba la información.
    - El sistema daba error en esa parte pero el proceso marcaba la liquidación como "exitosa".
  - Esto se intentó corregir, pero la reincidencia muestra que la **solución/mitigación anterior no fue efectiva**.

- Auditoría de información insuficiente:
  - Se dejó una consulta de auditoría para detectar inconsistencias, pero:
    - Solo cubría un criterio parcial (un tipo de caso).
    - No detectó este caso.
    - No tenía alertas de notificación activas, por lo que no levantó señal temprana.

## Responsables (por tarea)

- **Henry:**
  - Responsable de implementar una solución efectiva al problema de finalización de liquidación–baja.
  - La solución original no previno la reaparición del problema.

- **Brian:**
  - Responsable de la consulta de auditoría de información:
    - No aplicó correctamente el criterio para cubrir más escenarios.
    - No tenía activas notificaciones sobre resultados de auditoría.

> Nota: El objetivo no es culpar, sino dejar claro quién debía ajustar qué parte del sistema/proceso.

## Contención

- Se corrigió de forma manual el estado de la persona en los datos para que quede **de baja** en módulo planilla y en módulo Persona.
- Se alinea el estado actual de la persona con el hecho de que ya tiene una liquidación finalizada.

## Definiciones clave – Qué significa "estar de baja"

### 1. Estar de baja en módulo Persona (MongoDB)

Una persona se considera **de baja** en Persona cuando se cumplen TODAS estas condiciones:

- `estadopersona` = **debaja**.
- Su **última silla** está desactivada, con `fecha_fin` igual a la fecha de baja.
- El registro de **trayectoria** de la última silla está actualizado con `fecha_fin` correspondiente.
- Existe un **registro en la colección de bajas**:
  - Asociado a la persona.
  - Asociado a la última silla.
- Existe un registro en **logestadopersona** con estado `debaja`, asociado a la baja.

### 2. Estar de baja en módulo Planilla

Del lado de planilla, una persona se considera **de baja** cuando:

- En **contrato**:
  - `fecha_fin` está seteada.
  - `pendiente = true`.
- En **colaborador**:
  - Existe `fecha_baja` registrada.
- La **liquidación** asociada está en estado **finalizado**.

> Cualquier inconsistencia entre estas definiciones (por ejemplo, liquidación finalizada sin baja en Persona) debe ser tratada como incidente de datos.

## Acciones de prevención (CAPA)

### 1) Solución de raíz del flujo de liquidación

- Revisar el flujo completo de **finalización de liquidación** y asegurar que:
  - El formulario de baja **siempre cargue correctamente** en el último paso.
  - No se marque la liquidación como "exitosa" si:
    - no se completó el envío de datos de baja, o
    - falla la creación del registro de baja en MongoDB.
- Agregar logs y mensajes de error claros para estas fallas.
- Incluir pruebas (automatizadas o checklist) antes de desplegar cambios que toquen este flujo.

### 2) Auditoría de estados Persona–Planilla

- Reactivar y ampliar la auditoría de información para validar la **consistencia de estados** entre Persona y Planilla:
  - Detectar personas con liquidación finalizada pero sin baja en Persona.
  - Detectar personas con baja en planilla pero activas en Persona, y viceversa.
- Establecer ejecución periódica (por ejemplo, diaria) de esta auditoría.
- Configurar **alertas de notificación** cuando se detecten inconsistencias, para que soporte/IT actúe proactivamente.

### 3) Documentación de procesos

- Documentar el proceso estándar de **baja automática**:
  - Cómo el flujo de liquidación debe crear registros de baja en Mongo y actualizar estados en ambos módulos.
- Documentar un proceso para dar de baja **manual** cuando el flujo automático no se completó (HowTo separado):
  - Qué campos/tablas se deben ajustar en Persona (Mongo).
  - Qué campos/tablas se deben ajustar en Planilla.
  - Validaciones que deben hacerse después de los cambios.

## Pendiente

- Crear y mantener una guía HOWTO:
  - "Cómo dar de baja a alguien de manera manual" conforme a las definiciones de baja en Persona y Planilla.
- Conectar este runbook con los KR de Tecnología:
  - % de soportes con RCA + CAPA documentados.
  - Reducción de inconsistencias de estado Persona–Planilla.
