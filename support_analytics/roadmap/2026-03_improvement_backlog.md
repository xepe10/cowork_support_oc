# Backlog de mejoras – Marzo 2026

## Ítem 1 – Estado específico para liquidaciones de sustitución patronal

- **Categoría:** Producto / UX
- **Módulo:** Nómina / Liquidaciones
- **Descripción:**
  Actualmente, las liquidaciones asociadas a sustitución patronal se marcan como **"Anulada"** para evitar impacto en cálculos futuros, pero esto genera confusión en HR.
- **Problema que resuelve:**
  - HR interpreta "Anulada" como error definitivo.
  - Cliente espera un estado que refleje la realidad del movimiento.
- **Propuesta:**
  - Crear un tipo/estado específico para **liquidación por sustitución patronal** (ej. "Sustitución patronal" o similar).
  - Ajustar UI y cálculos para que este estado no impacte nómina futura, pero mantenga la liquidación consultable.
- **Impacto esperado:**
  - Menor volumen de tickets de aclaración.
  - Mayor claridad para HR y auditoría.
- **Prioridad sugerida:** Alta
- **Dueño sugerido:** Producto + Ingeniería Nómina

---

## Ítem 2 – Unificación de lógica y validaciones en vacaciones

- **Categoría:** Producto / Calidad de datos
- **Módulo:** Vacaciones
- **Descripción:**
  Existen diferencias de saldo entre el widget de **"Días disponibles"** y el **"Resumen Anual"** debido a que no todos los componentes aplican el mismo filtro de fechas de permisos vs fecha de alta.
- **Problema que resuelve:**
  - Saldos diferentes según vista → confusión para colaboradores y HR.
  - Permisos con fecha anterior a la fecha de alta causan cálculos inconsistentes.
- **Propuesta:**
  - Alinear la lógica de cálculo en todos los widgets:
    - Siempre considerar solo permisos de vacaciones con fecha **≥ fecha de alta**.
  - Implementar validación en UI/API para bloquear registro de permisos de vacaciones antes de la fecha de alta.
  - Opcional: script de data-fix para identificar y corregir permisos históricos inválidos.
- **Impacto esperado:**
  - Saldos de vacaciones consistentes en toda la aplicación.
  - Menos tickets de aclaración sobre diferencias de días.
- **Prioridad sugerida:** Alta
- **Dueño sugerido:** Producto + Ingeniería Vacaciones

---

## Ítem 3 – Documentación visible para HR y colaboradores

- **Categoría:** Documentación / UX
- **Módulo:** Nómina, Vacaciones
- **Descripción:**
  Actualmente el conocimiento sobre decisiones de diseño (sustitución patronal, cálculo de vacaciones) está principalmente en la KB interna.
- **Problema que resuelve:**
  - HR y usuarios finales no siempre tienen contexto, lo que genera tickets de "explicación".
- **Propuesta:**
  - Crear artículos de centro de ayuda públicos/para clientes basados en:
    - `kb/runbooks/liquidacion_sustitucion_patronal_anulada.md`
    - `kb/troubleshooting/diferencia_dias_vacaciones_disponibles_pendientes.md`
  - Añadir tooltips / ayudas contextuales en la UI en los puntos clave:
    - Estado de liquidaciones en sustitución patronal.
    - Significado de "Días disponibles" vs "Días pendientes".
- **Impacto esperado:**
  - Reducción de tickets de tipo "how‑to / aclaración".
  - Mejor comprensión del sistema por parte de HR.
- **Prioridad sugerida:** Media
- **Dueño sugerido:** Producto + Documentación / Customer Success
