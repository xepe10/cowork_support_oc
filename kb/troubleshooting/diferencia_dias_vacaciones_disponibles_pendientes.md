# Diferencia entre "Días disponibles" y "Días pendientes" en los widgets de vacaciones

## Problema

En el perfil del colaborador existen dos widgets relacionados a vacaciones que pueden mostrar valores distintos, lo cual genera confusión:

- Panel izquierdo: **"Vacaciones" → DÍAS DISPONIBLES** (ejemplo: 21.6)
- Panel derecho: **"Resumen Anual de Vacaciones" → DÍAS PENDIENTES (total)** (ejemplo: 22.6)

Pregunta del cliente: _"¿Por qué hay diferencia entre las dos opciones de vacaciones disponibles en el perfil? Esto causa duda en el personal."_

## Modelo conceptual

Hay dos flujos de datos principales:

1. **Créditos (días acreditados)**
   - Derivan de la **licencia/política de vacaciones** asignada al colaborador.
   - El sistema acredita días **diariamente** según esa licencia.
   - Con el tiempo se obtiene el total de **días acreditados** desde la fecha de alta del colaborador.

2. **Permisos / solicitudes de vacaciones**
   - Las vacaciones se registran como un tipo específico de **permiso** vinculado a la licencia de vacaciones.
   - Cada solicitud cubre uno o más días; el sistema la traduce a una **cantidad de días gozados**.
   - Esos días gozados se **restan** del saldo acreditado.

En concepto, el saldo debería ser:

> **Saldo (disponible/pendiente) = Días acreditados totales − Días gozados (permisos)**

## Significado previsto de cada widget

- **DÍAS DISPONIBLES (panel izquierdo)**
  - Objetivo: mostrar el **saldo actualmente utilizable** por el colaborador.
  - Debe considerar:
    - Días acreditados por la licencia.
    - Permisos de vacaciones válidos **posteriores a la fecha de alta** del colaborador.

- **DÍAS PENDIENTES (consolidado en el resumen anual)**
  - Objetivo: mostrar el **total de días pendientes por periodo** (año, periodo de política, etc.).
  - También se calcula a partir de días acreditados menos días gozados, pero con una **agrupación por periodos**.

## Bug específico observado en este caso

En el caso de Navinter (captura proporcionada), el análisis mostró:

- Los créditos (días acreditados) estaban **correctos**.
- Existían permisos de vacaciones con **fecha anterior a la fecha de alta** del colaborador.
- Una parte del sistema **sí filtraba** los permisos para contar solo los posteriores a la fecha de alta.
- Otra parte **no aplicaba** ese filtro e incluía un permiso con fecha **previa al alta**.

Resultado:

- Un widget (por ejemplo, DÍAS DISPONIBLES) usaba el conteo filtrado → **menos días gozados** → saldo **ligeramente mayor**.
- El otro widget (DÍAS PENDIENTES del resumen anual) incluía el permiso previo al alta → **más días gozados** → saldo diferente.

Esto generaba una discrepancia (por ejemplo, 21.6 vs 22.6) y confusión en los colaboradores.

## Pasos de resolución (para soporte/técnico)

1. **Confirmar colaborador y periodos**
   - Identificar al colaborador afectado y los periodos donde se observa la discrepancia.

2. **Revisar créditos**
   - Verificar los **días acreditados** según la licencia de vacaciones.
   - Confirmar que concuerdan con la configuración de la licencia y la fecha de alta.

3. **Revisar permisos (vacaciones)**
   - Listar todos los permisos de tipo vacaciones para el colaborador.
   - Poner especial atención en:
     - Fechas **relativas a la fecha de alta** del colaborador.
     - Cualquier permiso registrado **antes de la fecha de alta**.

4. **Corregir permisos inválidos**
   - Si existen permisos con fecha anterior a la fecha de alta:
     - Ajustar/migrar esas fechas a un rango válido.
     - O, si fueron errores de registro, anular/eliminar según la política del cliente.

5. **Recalcular / refrescar vistas**
   - Verificar que ambos widgets usen la misma regla de filtrado de permisos (solo posteriores a la fecha de alta).
   - Si actualmente solo uno aplica el filtro, registrar un bug/tarea técnica para alinear ambos cálculos.

6. **Validar resultado**
   - Comparar nuevamente **DÍAS DISPONIBLES** vs **DÍAS PENDIENTES** consolidados.
   - Deben ser consistentes (salvo diferencias esperadas como redondeos o solicitudes futuras ya aprobadas, si aplica).

## Explicación sugerida para L1 al cliente (mientras el bug exista)

> En el perfil se muestran dos vistas de vacaciones:
> 
> - **Días disponibles**: el saldo que la persona puede usar hoy, calculado en base a los días que se le han acreditado por su licencia de vacaciones menos los permisos que ya ha gozado.
> - **Días pendientes (Resumen Anual)**: el saldo por periodo, usando la misma lógica de créditos menos días gozados.
> 
> En su caso encontramos que había uno o más permisos de vacaciones registrados con fecha **anterior a la fecha de alta** de la persona. En una parte del sistema esos permisos se filtraban correctamente (solo se contaban permisos posteriores al alta), pero en otra vista no se estaba aplicando ese filtro, lo que generaba la diferencia entre los dos saldos.
> 
> Ya ajustamos las fechas de esos permisos para que coincidan con la fecha de alta y el cálculo sea consistente. Vamos a alinear también ambos cálculos en el sistema para evitar que esto vuelva a ocurrir.
> 
> Si detectan alguna otra discrepancia en el futuro, por favor indíquennos el colaborador y haremos la misma revisión de créditos y permisos.

## Prevención / Notas de producto

- Al registrar o importar permisos de vacaciones, se debe validar que ninguna fecha sea **anterior a la fecha de alta** del colaborador.
- Todas las vistas/widgets de vacaciones deben usar la misma regla de filtrado de permisos (solo posteriores a la fecha de alta) para evitar saldos inconsistentes.
- Puede ser útil agregar un tooltip o ayuda contextual en la UI que explique:
  - Qué representa "Días disponibles".
  - Qué representa "Días pendientes" en el resumen anual.
