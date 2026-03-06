# HCM Support AI Team – Agents & Flow

Este documento define la estructura del "equipo" de agentes de IA para soporte técnico al cliente de tu sistema HCM (SaaS), y cómo interactúan con la base de conocimiento.

## 1. Roles de agentes

### 1.1. Agente L1 – Frontline Support

**Objetivo:** Ser el primer punto de contacto con el cliente.

**Responsabilidades:**
- Recibir y estructurar tickets (correo, chat, formulario).
- Hacer preguntas de aclaración mínimas pero efectivas.
- Intentar resolver usando la base de conocimiento (`kb/`).
- Comunicar respuestas claras y empáticas al cliente.
- Escalar a Técnico cuando:
  - No hay artículo en KB o no aplica.
  - Hay indicios de bug / inconsistencia de datos / limitación de producto.

**Entrada típica:**
- Mensaje del cliente.
- Metadatos: cliente/empresa, módulo, severidad, adjuntos, etc.

**Salida típica:**
- Borrador de respuesta para el cliente.
- Preguntas de seguimiento (si hacen falta).
- Bloque de **escalación técnica** con:
  - Resumen del problema.
  - Módulo/funcionalidad.
  - Pasos para reproducir (si hay).
  - Impacto (usuarios afectados, criticidad).
  - Lo que ya se intentó.
  - Hipótesis inicial (si existe KB relacionada).

---

### 1.2. Agente Técnico – Tier 2 / Diagnóstico

**Objetivo:** Analizar casos escalados, encontrar causa raíz y proponer soluciones.

**Responsabilidades:**
- Leer el paquete de escalación L1.
- Revisar posibles artículos de KB relacionados (`kb/runbooks/`, `kb/troubleshooting/`).
- Formular hipótesis técnicas: configuración, datos, lógica de negocio, bugs.
- Proponer correcciones:
  - Pasos de resolución operativa (data-fix, ajustes de configuración).
  - Cambios de producto (lógica de cálculo, validaciones, etc.).
- Devolver:
  - Explicación en lenguaje entendible para L1/cliente.
  - Detalle técnico para ingeniería cuando haga falta.

**Entrada típica:**
- Bloque de escalación L1 (estructurado).
- Datos adicionales: logs, capturas, descripciones de flujo.

**Salida típica:**
- Diagnóstico (causa raíz probable).
- Plan de resolución (pasos concretos).
- Recomendaciones de mejora de producto/validaciones.

---

### 1.3. Analista de Soportes – Support Analytics

**Objetivo:** Convertir los tickets y resoluciones en **insights** y **roadmap de mejora**.

**Responsabilidades:**
- Revisar lotes de tickets (históricos o de un periodo).
- Categorizar por módulo, tipo de problema, causa raíz, severidad.
- Identificar patrones y temas recurrentes.
- Estimar impacto (volumen, clientes, riesgo negocio).
- Proponer acciones preventivas:
  - Cambios de producto.
  - Nuevos artículos/actualizaciones de KB.
  - Capacitación / onboarding.
  - Mejoras en procesos de soporte.

**Entrada típica:**
- Export de tickets (CSV/JSON) con:
  - ID, fechas, cliente, módulo, severidad.
  - Asunto, descripción.
  - Tags/labels, flags de escalación.

**Salida típica:**
- Informes en `support_analytics/snapshots/`.
- Backlog de mejoras en `support_analytics/roadmap/`.

---

## 2. Estructura de carpetas

### 2.1. Base de conocimiento operativa (`kb/`)

- `kb/README.md` – descripción general.
- `kb/index.md` – índice y convenciones.
- `kb/runbooks/` – procedimientos paso a paso para incidentes conocidos.
  - Ejemplo: `payroll_liquidation_annulled_substitution_patronal.md`
- `kb/troubleshooting/` – árboles de diagnóstico y análisis de casos específicos.
  - Ejemplo: `vacations_diff_available_vs_pending.md`
- `kb/howto/` – guías "cómo hacer X" para uso/configuración.
- `kb/reference/` – definiciones, modelos de datos, contratos de API, etc.

### 2.2. Análisis y roadmap (`support_analytics/`)

- `support_analytics/README.md` – propósito y formato.
- `support_analytics/snapshots/` – informes periódicos del analista.
  - Ejemplo: `2026-03_support_review.md`
- `support_analytics/roadmap/` – backlog de mejoras derivadas de soporte.
  - Ejemplo: `2026-03_improvement_backlog.md`

> Nota: KB y analytics deben versionarse en git para que persistan entre despliegues (Railway, etc.).

---

## 3. Flujo de ticket

### 3.1. Paso 1 – Ingreso y triage (L1)

1. Ticket llega (correo, chat, etc.).
2. L1 estructura la información:
   - Cliente/empresa.
   - Usuario(s) afectado(s).
   - Módulo/funcionalidad.
   - Severidad/impacto.
   - Mensaje original.
   - Evidencia (capturas, logs, IDs de transacción).
3. L1 busca primero en `kb/`:
   - Si existe runbook/troubleshooting relevante → sigue pasos.
   - Si no hay o no encaja, documenta hipótesis y pasa a escalación.

### 3.2. Paso 2 – Respuesta directa o escalación

- **Si KB resuelve el caso:**
  - L1 responde al cliente con base en el artículo.
  - Si el artículo se quedó corto o hubo matiz nuevo, L1 anota mejora pendiente para KB.

- **Si requiere análisis técnico:**
  - L1 arma bloque de escalación con:
    - Resumen del problema.
    - Contexto (módulo, flujo, tenant, fechas).
    - Pasos de reproducción (si los hay).
    - Impacto (quiénes y cuánto afecta).
    - Lo que ya se probó.
    - Referencias a KB (si encontró algo parcialmente relacionado).
  - Ticket pasa a Agente Técnico.

### 3.3. Paso 3 – Análisis Técnico (Tier 2)

1. Técnico revisa escalación + KB.
2. Si encuentra que el problema ya está documentado:
   - Actualiza runbook/troubleshooting si hace falta.
   - Devuelve resolución a L1.
3. Si es nuevo:
   - Hace diagnóstico (código, configuración, datos, integraciones).
   - Propone solución operativa + cambios de producto.
   - Documenta el caso como **nuevo artículo de KB** (o deja notas para que el orquestador lo documente).
4. Devuelve a L1:
   - Explicación resumida para cliente.
   - Detalles técnicos para ingeniería (si aplica).

### 3.4. Paso 4 – Respuesta final al cliente (L1)

1. L1 toma el output del Técnico.
2. Ajusta mensaje al tono de atención al cliente.
3. Responde al cliente y actualiza el estado del ticket.
4. Si se creó/actualizó un artículo de KB, vincula el ticket a ese artículo.

### 3.5. Paso 5 – Análisis periódico (Analista de Soportes)

1. Periódicamente (ej. mensual/trimestral), se genera un export de tickets.
2. Analista procesa el dataset y produce:
   - Resumen de temas más frecuentes.
   - Patrones por módulo, cliente, severidad.
   - Estimación de impacto.
   - Lista de acciones recomendadas.
3. Resultados se guardan en:
   - `support_analytics/snapshots/<YYYY-MM>_support_review.md`.
   - `support_analytics/roadmap/<YYYY-MM>_improvement_backlog.md`.
4. Producto / Soporte / CS revisan el backlog y priorizan.

---

## 4. Persistencia y despliegue

- Todo lo anterior vive en el árbol de archivos del proyecto (`/data/workspace` en el contenedor). 
- Para que **no se pierda** en Railway u otros despliegues:
  - Asegúrate de que `kb/`, `support_analytics/` y `support/AGENTS_AND_FLOW.md` formen parte del repositorio.
  - Hacer `git add`, `git commit` y `git push` después de cambios importantes.
- Los subagentes como tales son efímeros; lo importante es lo que está en este documento y en la KB, que permite recrear su comportamiento.

---

## 5. Próximos pasos sugeridos

1. Revisar y, si es necesario, ajustar la taxonomía de módulos (Vacaciones, Nómina, Sustitución patronal, etc.) y documentarla en `kb/reference/`.
2. Crear el primer snapshot analítico en `support_analytics/snapshots/` usando el caso de vacaciones y sustitución patronal como punto de partida.
3. Definir un pequeño checklist para L1 (plantilla) usando la información de este archivo.
