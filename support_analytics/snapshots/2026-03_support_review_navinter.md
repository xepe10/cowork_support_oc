# Snapshot de soporte – Marzo 2026 (casos Navinter)

## 1. Resumen ejecutivo

En los primeros ejercicios se identificaron dos temas principales de soporte en el cliente **Navinter** relacionados con el módulo de **Nómina/Vacaciones**:

1. Manejo de liquidaciones en **sustitución patronal**.
2. Diferencias entre "Días disponibles" y "Días pendientes" en el perfil de vacaciones.

Ambos casos evidencian:
- Decisiones de diseño temporales no suficientemente visibles para el usuario.
- Inconsistencias en la lógica de cálculo / validación de datos.
- Necesidad de reforzar documentación y validaciones para evitar tickets repetitivos.

## 2. Tema #1 – Liquidación anulada en sustitución patronal

- **Módulo:** Nómina / Liquidaciones.
- **Descripción:**
  - En escenarios de **sustitución patronal**, las liquidaciones asociadas aparecen con estado **"Anulada"**.
  - El comportamiento es intencional (diseño temporal) para evitar impacto en cálculos futuros tanto en el patrono anterior como en el nuevo.
- **Problema de soporte:**
  - El estado "Anulada" se interpreta como error o cancelación total.
  - No es evidente para el cliente que la liquidación sigue siendo consultable y que no afecta cálculos futuros.
- **Causa raíz:**
  - Falta de un estado/flujo específico para liquidaciones de sustitución patronal.
  - Comunicación/documentación insuficiente sobre la decisión de diseño temporal.
- **Impacto:**
  - Tickets de aclaración provenientes de HR.
  - Riesgo de desconfianza en el sistema cuando se ven liquidaciones anuladas sin contexto.

**Recomendaciones:**
- Producto:
  - Priorizar el desarrollo de un estado específico para **liquidación por sustitución patronal**.
  - Evaluar una etiqueta/indicador visual más claro que "Anulada" en estos casos temporales.
- Documentación/Soporte:
  - Mantener actualizado el runbook `kb/runbooks/liquidacion_sustitucion_patronal_anulada.md`.
  - Agregar una nota visible en UI o help center explicando este comportamiento mientras sea temporal.

## 3. Tema #2 – Diferencias en saldos de vacaciones

- **Módulo:** Vacaciones.
- **Descripción:**
  - En el perfil del colaborador, el widget de **"Días disponibles"** y el consolidado de **"Días pendientes" (Resumen Anual)** muestran saldos distintos.
- **Hallazgos técnicos:**
  - El modelo conceptual es correcto: créditos diarios por licencia menos permisos de vacaciones (días gozados).
  - Se encontraron permisos de vacaciones con **fechas anteriores a la fecha de alta** del colaborador.
  - Una parte del sistema filtraba correctamente los permisos posteriores al alta; otra parte no aplicaba ese filtro.
- **Causa raíz:**
  - Inconsistencia en la lógica de filtrado entre dos componentes.
  - Falta de validación al registrar permisos antes de la fecha de alta.
- **Impacto:**
  - Colaboradores ven saldos distintos según la vista.
  - Genera confusión y tickets de aclaración.
  - Riesgo de pérdida de confianza en la información de vacaciones.

**Recomendaciones:**
- Producto:
  - Unificar la lógica de cálculo de saldos en todos los widgets de vacaciones.
  - Implementar validación para impedir permisos de vacaciones con fecha < fecha de alta.
- Documentación/Soporte:
  - Mantener el artículo `kb/troubleshooting/diferencia_dias_vacaciones_disponibles_pendientes.md`.
  - Añadir tooltips o ayuda en la UI explicando qué es cada saldo.

## 4. Conclusiones generales y próximos pasos

- Los dos casos muestran que muchas incidencias se pueden reducir con:
  - Estados más expresivos en la UI (menos sobrecarga en la palabra "Anulada").
  - Validaciones de negocio más estrictas (fechas relativas a la fecha de alta).
  - Mejor documentación visible para HR y soporte.
- Próximos pasos sugeridos:
  1. Crear un backlog de mejoras en `support_analytics/roadmap/2026-03_improvement_backlog.md` con estos dos temas como ítems de alta prioridad.
  2. Definir, junto con Producto, la prioridad de:
     - Nuevo estado de sustitución patronal.
     - Unificación de lógica de vacaciones + validaciones.
  3. A partir de nuevos tickets, seguir llenando `kb/` y futuros snapshots mensuales.
