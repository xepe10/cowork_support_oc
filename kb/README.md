# HCM Support Knowledge Base

Este directorio será la base de conocimiento compartida por todos los agentes (L1, técnicos y analista).

## Estructura propuesta

- `kb/`
  - `index.md` — Índice general y taxonomía.
  - `runbooks/` — Pasos detallados para resolver incidencias recurrentes.
  - `howto/` — Guías "cómo hacer X" para admins / soporte.
  - `troubleshooting/` — Árboles de diagnóstico por módulo.
  - `reference/` — Definiciones, modelos de datos, contratos de API, etc.

Dentro de cada carpeta:

- Archivos en Markdown (`.md`).
- Nombres de archivo en inglés o snake_case, con prefijos por módulo cuando aplique, por ejemplo:
  - `runbooks/payroll_missing_employees.md`
  - `troubleshooting/attendance_sync_delays.md`
  - `howto/create_new_leave_policy.md`

Cada artículo debe incluir:

- Frontmatter simple (opcional):
  - Módulo, severidad, tipo de problema, tags.
- Secciones estándar:
  - **Problem**
  - **Symptoms**
  - **Root cause**
  - **Resolution steps**
  - **Checks & validation**
  - **Notes / Variants**

Este archivo es solo una descripción; el contenido real vive en los subdirectorios.
