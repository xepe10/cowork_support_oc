# Difference between "Días disponibles" and "Días pendientes" in vacation widgets

## Problem

In the employee profile there are two vacation-related widgets that can show different values, which confuses employees:

- Left panel: **"Vacaciones" → DÍAS DISPONIBLES** (e.g. 21.6)
- Right panel: **"Resumen Anual de Vacaciones" → DÍAS PENDIENTES (total)** (e.g. 22.6)

Customer question: _"Why is there a difference between the two vacation options available in the profile? It causes confusion to staff."_

## Conceptual model

There are two core data streams:

1. **Credits (días acreditados)**
   - Based on the employee's **vacation license/policy**.
   - The system accrues days **daily** according to that license.
   - Over time this produces total **credited days** since the employee's start date (alta).

2. **Permissions / vacation requests (permisos)**
   - Vacations are treated as a specific type of **permission** linked to the vacation license.
   - Each request covers one or more dates; the system converts this into a **number of days taken**.
   - These days taken are **subtracted** from the credited balance.

Conceptually the balance should be:

> **Available / Pending = Total credited days − Total days taken (permissions)**

## Intended meaning of each widget

- **DÍAS DISPONIBLES (left panel)**
  - Intent: show the **current usable balance** for the employee.
  - Should consider:
    - Days credited from the license.
    - Vacation permissions that are valid for the employee **after their hire date (fecha de alta)**.

- **DÍAS PENDIENTES (consolidated in annual summary)**
  - Intent: show the **total pending days by period** (year, policy period, etc.).
  - Also derived from credited days minus days taken, but based on the **period aggregation**.

## Specific bug observed in this case

In the Navinter case (screenshot provided), investigation showed:

- Credits (días acreditados) were **correct**.
- Some vacation permissions (permisos) had a **date earlier than the employee's hire date (fecha de alta)**.
- One part of the system **correctly filtered** permissions to only count those **after the hire date**.
- Another part **did not apply** this filter and was including a permission dated **before** the hire date.

Result:

- One widget (e.g. DÍAS DISPONIBLES) used the filtered count → **lower days taken** → slightly **higher available**.
- The other widget (DÍAS PENDIENTES in the annual summary) included the pre-hire permission → **higher days taken** → different balance.

This created a discrepancy (e.g. 21.6 vs 22.6) and confusion for employees.

## Resolution steps (for support/technical)

1. **Confirm the employee and periods**
   - Identify the employee and the periods where the discrepancy appears.

2. **Review credits**
   - Check **días acreditados** generated from the vacation license.
   - Confirm they match the license configuration and hire date.

3. **Review permissions (vacation requests)**
   - List all vacation-type permissions for the employee.
   - Pay special attention to:
     - Dates **relative to the hire date (fecha de alta)**.
     - Any permissions accidentally recorded **before the hire date**.

4. **Correct invalid permissions**
   - If permissions exist with dates earlier than the hire date:
     - Adjust/migrate those permissions to valid dates.
     - Or, if they are erroneous, remove/cancel them according to policy.

5. **Recalculate / refresh views**
   - Ensure both widgets use the same filtering rules (post-hire permissions only).
   - If the product currently filters only on one side, log a bug/technical task to align both calculations.

6. **Validate**
   - Compare **DÍAS DISPONIBLES** vs consolidated **DÍAS PENDIENTES** again.
   - They should now be consistent (apart from any known intentional differences such as rounding or future-dated approved requests, if applicable).

## Suggested L1 explanation to customer (while bug exists)

> En el perfil se muestran dos vistas de vacaciones:
> 
> - **Días disponibles**: el saldo que la persona puede usar hoy, calculado en base a los días acreditados por su licencia y los permisos de vacaciones que ha tomado.
> - **Días pendientes (Resumen Anual)**: el saldo por periodo, usando la misma lógica de créditos menos días gozados.
> 
> En su caso encontramos que había uno o más permisos de vacaciones registrados con fecha **anterior a la fecha de alta** de la persona. En una parte del sistema esos permisos se filtraban correctamente (solo se contaban permisos posteriores al alta), pero en otra vista no se estaba aplicando ese filtro, lo que generaba la diferencia entre los dos saldos.
> 
> Ya ajustamos las fechas de esos permisos para que coincidan con la fecha de alta y el cálculo sea consistente. Vamos a alinear también ambos cálculos en el sistema para evitar que esto vuelva a ocurrir.
> 
> Si detectan alguna otra discrepancia en el futuro, por favor indíquennos el colaborador y haremos la misma revisión de créditos y permisos.

## Prevention / Product notes

- When recording or importing vacation permissions, enforce validation so that no permission can be dated **before the employee's hire date**.
- Ensure **all** vacation widgets use the same filtering rule on permissions (post-hire only) to avoid inconsistent balances.
- Consider adding a help tooltip in the UI explaining:
  - What "Días disponibles" represents.
  - What "Días pendientes" in the annual summary represents.
