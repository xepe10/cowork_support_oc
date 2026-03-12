# Error al generar planilla: "No existe ninguna compensación configurada para este Grupo y para este concepto de pago"

**Módulo:** Payroll  
**Aplica a:** Cualquier cliente  
**Última actualización:** 2026-03-12

## Problem
Al intentar generar una nueva planilla, el sistema muestra el mensaje:

> "La planilla no pudo ser generada debido a: Error al generar planilla: No existe ninguna compensación configurada para este Grupo y para este concepto de pago"

La planilla no se crea.

## Symptoms
- En la pantalla de **Planillas** → **Nueva planilla**, luego de seleccionar un **Grupo de planilla** y completar fechas, al presionar **Guardar** aparece el cuadro de error anterior.
- No se genera el registro de planilla.
- El mensaje hace referencia a que no hay compensaciones configuradas para el **Grupo de planilla** y el **concepto de pago** asociado.

## Root cause
Para generar una planilla, el motor de payroll busca **colaboradores que tengan compensaciones activas** para:

- El **grupo de planilla** seleccionado.
- El **concepto de pago** que define la planilla (por ejemplo, "Nómina de pago mensual", "Viáticos", etc.).

Si **no existe ningún colaborador** con una compensación configurada que coincida con ese grupo + concepto, el sistema lanza el error y no genera la planilla.

En resumen: el error suele indicar que, efectivamente, **no hay nadie configurado para pagar en esa planilla**.

> Nota: Si sí existen colaboradores con esa configuración y el error igualmente aparece, puede tratarse de otro problema (bug, filtro adicional, fechas, estado de compensación). En ese caso hay que escalar a Técnico Payroll.

## Resolution steps

1. **Confirmar el Grupo de planilla y el concepto de pago**
   - Identificar el **Grupo de planilla** que se está usando al crear la planilla (por ejemplo, "Nómina de pago mensual", "Viáticos", etc.).
   - Identificar el **concepto de pago** principal asociado a esa planilla.

2. **Ir a la pantalla de Compensaciones**
   - Navegar a **Procesos → Compensaciones**.

3. **Filtrar por Grupo de planilla y concepto de pago**
   - En la pantalla de compensaciones, usar los filtros para:
     - Seleccionar el **Grupo de planilla** en cuestión.
     - Seleccionar el **concepto de pago** correspondiente a la planilla que se está intentando generar.

4. **Revisar si existen colaboradores con compensaciones activas**
   - Observar el resultado del filtro:
     - **Si no aparece ningún colaborador:**
       - Esta es la causa del error. No hay compensaciones configuradas para ese grupo + concepto.
       - Se debe configurar al menos una compensación para los colaboradores que deban pagarse en esa planilla.
     - **Si sí aparecen colaboradores:**
       - La causa no es la ausencia de compensaciones. Puede haber otro problema (fechas, estado de compensación, filtros adicionales o bug).
       - Ver sección "Escalada".

5. **Configurar compensaciones si faltan**
   - Para cada colaborador que deba incluirse en la planilla:
     - Crear o editar la **compensación** para asignarle:
       - El **concepto de pago** correcto.
       - El **Grupo de planilla** correcto.
       - Fechas de vigencia adecuadas (que cubran el periodo de la planilla).
   - Guardar los cambios.

6. **Reintentar generación de planilla**
   - Volver a **Planillas → Nueva planilla**.
   - Seleccionar el mismo Grupo de planilla y fechas.
   - Guardar y verificar que esta vez se genere la planilla sin el error.

## Escalada a Técnico Payroll

Escalar a `tech_payroll` cuando:

- Sí existen uno o más colaboradores con compensaciones activas para el grupo + concepto, pero el error sigue apareciendo.
- O se detecta que las compensaciones están bien configuradas y aún así la planilla no se genera.

Al escalar, incluir:

- Capturas de:
  - El mensaje de error en la pantalla de nueva planilla.
  - La pantalla de compensaciones filtrada por grupo + concepto donde se vean los colaboradores.
- Datos:
  - Cliente, grupo de planilla, concepto de pago.
  - Periodo de la planilla (inicio de periodo, fecha de generación).

## Checks & validation

- Tras configurar las compensaciones, generar nuevamente la planilla y verificar que:
  - La planilla se crea correctamente.
  - El número de colaboradores y montos coincide con lo esperado.
- Verificar que las fechas de vigencia de las compensaciones cubran el periodo de la planilla.

## Notes / Variants

- El mismo patrón puede aplicar a otros tipos de planilla por concepto (viáticos, aguinaldo, bonos, etc.): siempre hay que validar que existen colaboradores con el concepto de pago y grupo de planilla configurados.
- Si una planilla se genera correctamente para un grupo + concepto en un periodo, pero falla en otro, revisar:
  - Cambios recientes en compensaciones (altas/bajas, cambios de grupo).
  - Cambios en configuración de conceptos o grupos de planilla.
