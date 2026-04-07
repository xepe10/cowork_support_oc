# Runbook – Eliminación de planilla finalizada borró liquidaciones asociadas

## Resumen

Un cliente reporta que **no aparecen algunas liquidaciones que ya estaban finalizadas**. Se identifica que, al eliminar una planilla de diciembre para hacer ajustes, también se eliminaron las liquidaciones asociadas debido a una relación `DELETE CASCADE` en base de datos.

## Contexto

- **Cliente:** _(completar)_
- **Módulo:** EPC – Planillas / Liquidaciones
- **Periodo:** Planilla de diciembre, acción ejecutada en marzo.
- **Síntoma:** Faltan liquidaciones que antes aparecían como finalizadas.

## Línea de tiempo (simplificada)

1. Cliente solicita eliminar una **planilla de diciembre** para realizar ajustes.
2. Geycow ejecuta la eliminación de la planilla.
3. Después de la eliminación, el cliente detecta que **desaparecieron liquidaciones asociadas** a esa planilla.
4. Se investiga y se encuentra que la relación planilla → liquidación en BD estaba configurada con `DELETE CASCADE`.

## Causa raíz (RCA)

### Técnica

- La relación entre tablas de **planilla** y **liquidación** estaba definida con `DELETE CASCADE`.
- Al eliminar la planilla, la base de datos eliminó automáticamente todas las liquidaciones asociadas.

### De proceso

- No existía un **procedimiento estandarizado y documentado** para eliminar planillas, especialmente planillas **finalizadas**.
- No se habían definido claramente:
  - En qué estados se puede eliminar una planilla.
  - Qué dependencias revisar antes de eliminar (liquidaciones, asistencia, horas extras, ISR, correlativos, etc.).

### De producto / reglas de negocio

- El sistema **permitía eliminar una planilla con liquidaciones asociadas**, sin validación previa.
- No existía una regla que bloqueara la eliminación o que guíe al usuario sobre cómo tratar las liquidaciones (desasociar, regenerar, re‑asociar).

## Mitigación aplicada

1. **Restauración de datos**
   - Se restauraron los registros de liquidaciones desde un **backup de staging**.

2. **Reconstrucción de la planilla de diciembre**
   - Se generó nuevamente la planilla de diciembre.
   - Se reingresaron los montos que tenía la planilla original eliminada.

3. **Reasociación de liquidaciones restauradas**
   - Las liquidaciones restauradas se asociaron a la nueva planilla de diciembre.

4. **Ajuste de la relación en base de datos**
   - Se cambió la relación planilla → liquidación de `DELETE CASCADE` a `DELETE RESTRICT`.
   - A partir de ahora, la BD **bloquea la eliminación** de una planilla si tiene liquidaciones asociadas.

## Acciones de prevención (CAPA)

### 1) Endurecer reglas de negocio

- Mantener `DELETE RESTRICT` en planilla → liquidación.
- Agregar validación en la capa de aplicación para:
  - Bloquear la eliminación de planillas con liquidaciones asociadas.
  - Mostrar un mensaje claro al usuario:
    > "No se puede eliminar la planilla porque tiene liquidaciones asociadas. Debes gestionar primero las liquidaciones."

### 2) Definir política y proceso para eliminar planillas

- Documentar un proceso formal que responda a:
  - ¿Se puede eliminar una planilla finalizada? ¿En qué casos sí/no?
  - ¿Qué pasos seguir si hay liquidaciones asociadas (desasociar, regenerar cálculos, re‑asociar)?
  - ¿Qué impacto tiene en:
    - cálculos de asistencia,
    - horas extras,
    - ISR,
    - correlativos y reportes?
- Establecer que **no se elimina una planilla finalizada con liquidaciones asociadas** sin un procedimiento especial y supervisado.

### 3) Mejoras en la aplicación (futuro)

- Antes de permitir eliminar una planilla, el sistema debe:
  - Consultar si existen liquidaciones asociadas.
  - Mostrar un resumen del impacto (nº de liquidaciones, períodos, estados).
  - En caso de planillas finalizadas, exigir un flujo guiado o un rol con permisos especiales.

### 4) Checklist operativo y capacitación

- Incluir en los checklists de soporte/implementación:
  - Revisar si la planilla tiene liquidaciones asociadas.
  - Confirmar el estado de la planilla (borrador, en edición, finalizada).
  - Evaluar impactos antes de eliminar.
- Capacitar a Geycow y a cualquier operador que pueda eliminar planillas sobre:
  - El nuevo proceso.
  - Cuándo escalar antes de borrar.

## Pendiente

- Incorporar el **procedimiento correcto para eliminar una planilla** en las guías HOWTO y vincularlo a este runbook.
- Conectar este caso con los KR del área de Tecnología:
  - % de soportes con RCA + CAPA documentados.
  - Reducción de incidentes recurrentes en funcionalidades críticas (planillas/liquidaciones).
