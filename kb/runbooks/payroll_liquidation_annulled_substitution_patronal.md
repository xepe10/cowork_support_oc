# Payroll liquidation shown as **Annulled** after substitution patronal

## Problem

Customer sees a payroll liquidation for an employee involved in a **sustitución patronal** (employer substitution) showing status **“Anulada”** instead of the expected **“Aprobada”**.

Example: Employee *Sheyla* at customer *Navinter*; liquidation appears as annulled after substitution patronal.

## Context

- Module: Payroll / Liquidaciones.
- Scenario: **Sustitución patronal** between two employers (old and new).
- Current product limitation: the system does **not yet fully support** a dedicated liquidation flow/state for substitution patronal cases.

## Symptoms

- Liquidation generated as part of the substitution patronal process.
- Status displayed: **Anulada**.
- Customer expectation (based on prior communication): liquidation should remain in **Aprobada** state.

## Root cause / Design decision (current behavior)

- The system still lacks full support for a specific **substitution patronal liquidation** type/state.
- To avoid these liquidations impacting payroll calculations either in the **previous employer** or the **new employer**, the current workaround is:
  - The liquidation is left in **Anulada** status.
  - The calculation details and reports remain **consultable**.
  - The liquidation **does not affect future calculations** in either employer.

This is a **temporary design** until native support for substitution patronal liquidations is implemented.

## Resolution (how to handle the ticket)

1. **Confirm the scenario**
   - Validate that the employee’s movement is indeed a **sustitución patronal**.
   - Confirm that the liquidation in question belongs to that movement.

2. **Explain the current system limitation**
   - The product does not yet have a dedicated liquidation type/state for substitution patronal.
   - As an interim solution, these liquidations are marked as **Anulada** intentionally.

3. **Clarify impact to the customer**
   - Although the status shows as **Anulada**, the liquidation:
     - Sigue disponible para **consulta** de reportería y revisión de cálculos.
     - **No afecta** cálculos futuros de nómina ni en el patrono anterior ni en el nuevo.

4. **Set expectations about future behavior**
   - Indicate that once the system includes full support for substitution patronal liquidations:
     - These cases will move to a specific **“sustitución patronal”** state (or equivalent).
     - They will stop appearing as annulled while keeping the intended behavior of not impacting future calculations.
   - The customer will be notified when this enhancement is deployed.

## Suggested L1 response (template)

> Buenos días, [nombre].
>
> En el caso de Sheyla, al tratarse de una **sustitución patronal**, la liquidación aparece actualmente como **“Anulada”** de forma intencional. Esto se debe a que, por ahora, el sistema todavía no cuenta con un tipo de liquidación específico para sustitución patronal.
>
> Para evitar que esa liquidación afecte cálculos en el patrono anterior o en el nuevo, se deja en estado **Anulada**, pero **sigue disponible para consulta** de reportes y de los cálculos que se realizaron en su momento. Lo importante es que no impacta ningún cálculo futuro de nómina.
>
> Tenemos planificada la mejora para soportar este tipo de movimientos de forma nativa. Cuando esté disponible, estas liquidaciones dejarán de verse como “Anulada” y pasarán a un estado específico de **sustitución patronal**, sin afectar el funcionamiento del sistema. En ese momento les avisaremos.
>
> Quedamos atentos si necesitan que revisemos algún caso adicional o detalle específico de la liquidación.

## Prevention / Notes

- Until the dedicated substitution patronal liquidation feature is released:
  - This “Annulled but consultable” behavior is **expected** for substitution cases.
  - L1 should **not** attempt to "re-approve" these liquidations manually.
- Once the feature is available, this runbook should be **updated** to:
  - Describe the new status/behavior.
  - Include any migration notes (e.g., historical cases auto-mapped from "Anulada" to the new state).
