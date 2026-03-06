# Liquidación anulada en casos de sustitución patronal

## Problema

El cliente ve una **liquidación de nómina** para una persona involucrada en una **sustitución patronal** que aparece con estado **"Anulada"** en lugar del estado **"Aprobada"** que esperaban.

Ejemplo: colaboradora *Sheyla* en el cliente *Navinter*; la liquidación aparece como anulada luego de la sustitución patronal.

## Contexto

- Módulo: Nómina / Liquidaciones.
- Escenario: **Sustitución patronal** entre dos patronos (anterior y nuevo).
- Limitación actual del producto: el sistema **todavía no soporta completamente** un flujo/estado específico de liquidación para casos de sustitución patronal.

## Síntomas

- Se genera una liquidación como parte del proceso de sustitución patronal.
- En la UI el estado se muestra como **"Anulada"**.
- El cliente, según indicaciones previas, esperaba que la liquidación quedara en estado **"Aprobada"** o similar.

## Causa raíz / Decisión de diseño (comportamiento actual)

- Aún no existe en el sistema un tipo/estado de liquidación específico para **sustitución patronal**.
- Para evitar que estas liquidaciones afecten cálculos de nómina tanto en el **patrono anterior** como en el **patrono nuevo**, se tomó la siguiente decisión temporal:
  - La liquidación se deja con estado **"Anulada"** de forma intencional.
  - Los detalles de cálculo y la reportería quedan **disponibles para consulta**.
  - La liquidación **no impacta cálculos futuros** en ninguno de los dos patronos.

Este es un **diseño temporal** hasta que exista soporte nativo para liquidaciones de sustitución patronal.

## Resolución (cómo manejar el ticket)

1. **Confirmar el escenario**
   - Verificar que el movimiento de la persona sea efectivamente una **sustitución patronal**.
   - Confirmar que la liquidación consultada está asociada a ese movimiento.

2. **Explicar la limitación actual del sistema**
   - Indicar que el producto todavía no cuenta con un tipo/estado de liquidación específico para sustitución patronal.
   - Aclarar que, como solución temporal, estas liquidaciones se marcan como **"Anulada"** de forma deliberada.

3. **Aclarar el impacto para el cliente**
   - Aunque el estado muestre **"Anulada"**, la liquidación:
     - Sigue disponible para **consulta de reportes** y revisión de cálculos históricos.
     - **No afecta** cálculos futuros de nómina ni en el patrono anterior ni en el nuevo.

4. **Definir expectativas sobre el comportamiento futuro**
   - Indicar que, cuando el sistema incluya soporte completo para liquidaciones de sustitución patronal:
     - Estos casos pasarán a un estado específico de **"sustitución patronal"** (o equivalente).
     - Dejarán de aparecer como anuladas, manteniendo el mismo objetivo de no impactar cálculos futuros.
   - Aclarar que se notificará al cliente cuando esta mejora sea desplegada.

## Respuesta sugerida para L1 (plantilla)

> Buenos días, [nombre].
>
> En el caso de Sheyla, al tratarse de una **sustitución patronal**, la liquidación aparece actualmente como **"Anulada"** de forma intencional. Esto se debe a que, por ahora, el sistema todavía no cuenta con un tipo de liquidación específico para sustitución patronal.
>
> Para evitar que esa liquidación afecte cálculos en el patrono anterior o en el nuevo, se deja en estado **"Anulada"**, pero **sigue disponible para consulta** de reportes y de los cálculos que se realizaron en su momento. Lo importante es que no impacta ningún cálculo futuro de nómina.
>
> Tenemos planificada la mejora para soportar este tipo de movimientos de forma nativa. Cuando esté disponible, estas liquidaciones dejarán de verse como "Anulada" y pasarán a un estado específico de **sustitución patronal**, sin afectar el funcionamiento del sistema. En ese momento les avisaremos.
>
> Quedamos atentos si necesitan que revisemos algún caso adicional o detalle específico de la liquidación.

## Prevención / Notas

- Mientras no se libere la funcionalidad específica de liquidación por sustitución patronal:
  - Este comportamiento de **"anulada pero consultable"** es **esperado** para estos casos.
  - L1 **no debe** intentar "re-aprobar" manualmente estas liquidaciones.
- Una vez que la funcionalidad esté disponible, este runbook debe **actualizarse** para:
  - Describir el nuevo estado/comportamiento.
  - Incluir notas de migración (por ejemplo, casos históricos que pasen automáticamente de "Anulada" al nuevo estado).
