# Configuración de agentes de soporte HCM

Este archivo describe la **ficha de configuración** de cada agente que participa en el flujo de soporte: qué hace, qué recibe, qué entrega y cuándo debe ser invocado.

Se asume que todos estos agentes se orquestan desde OpenClaw (o sistema similar) usando prompts/pipelines configurables.

---

## 1. Orquestador Técnico

**ID:** `orchestrator_support`  
**Rol:** Router y quality gate entre L1, técnicos y analista.

**Objetivo**  
Tomar el ticket bruto, estructurarlo (si L1 no lo hizo), decidir a qué subagente mandar y consolidar la respuesta final.

**Inputs**
- Ticket del cliente (texto original, adjuntos, canal).
- Metadatos: cliente, módulo sospechado, severidad, timestamps.
- Estado de histórico: tickets previos del mismo cliente/tema (si aplica).

**Outputs**
- Plan de acción:
  - Qué subagente(s) intervienen.
  - Resumen estructurado del problema.
- Paquete para L1:
  - Respuesta sugerida al cliente.
  - Indicaciones de seguimiento (qué monitorear, qué validar).
- Notas de documentación:
  - Si se debe crear/actualizar artículo en `kb/`.
  - Si se debe agregar punto al roadmap (`support_analytics/roadmap`).

**Cuándo se invoca**
- Siempre como primera capa después del ticket entrante.
- Cuando hay dudas sobre a qué módulo pertenece el problema.

**Reglas clave**
- Si falta información mínima → pedir a L1 preguntas específicas.
- Determinar módulo principal + secundarios antes de llamar técnicos.
- Mantener el flujo simple: máximo 1–2 técnicos por ticket, más DB Specialist solo si hace falta.

---

## 2. Agente L1 – Frontline Support

**ID:** `support_L1`  
**Rol:** Primer filtro, cara al cliente, busca en KB y estructura tickets.

**Objetivo**  
Resolver con KB cuando se pueda; si no, escalar con un buen paquete técnico.

**Inputs**
- Mensaje del cliente (texto completo).
- Metadatos: cliente, usuario; canal; severidad (si la hay).

**Outputs**
- Respuesta propuesta al cliente (tono empático y claro).
- Bloque de escalación técnica cuando no se pueda resolver:
  - Resumen del problema.
  - Cliente, módulo, severidad.
  - Pasos para reproducir.
  - Evidencia (descrita; links/capturas).
  - Lo que ya se intentó.
  - Artículos de KB revisados y por qué no aplican 100%.

**Cuándo se invoca**
- Todos los tickets entrantes.

**Reglas clave**
- Intentar resolver con `kb/` primero.
- No inventar: si la KB no alcanza, escalar con buen contexto.
- Mantener tono constante hacia el cliente (plantilla de respuesta).

---

## 3. Técnicos por módulo

### 3.1. Técnico Tiempos & Asistencia

**ID:** `tech_ta`  
**Rol:** Full‑stack de tiempos y asistencia.

**Objetivo**  
Diagnosticar y resolver problemas de marcajes, jornadas, horas extra, sincronización con payroll.

**Inputs**
- Escalación L1 estructurada.
- Datos del cliente: nombre, IDs relevantes.
- Referencias a registros en DB, si las hay (empleado, periodo, etc.).

**Outputs**
- Diagnóstico:
  - Qué está pasando.
  - En qué capa: configuración, datos, lógica, bug.
- Plan de resolución:
  - Pasos operativos (configuración, reprocesos, data‑fix).
  - Recomendaciones de cambios de producto si aplica.
- Sugerencia de texto para L1:
  - Explicación simple + qué se hará / se hizo.

**Contexto necesario**
- Repo de **tiempos y asistencia**.
- Tablas/colecciones relacionadas en Postgres y Mongo por cliente.
- Artículos `kb/runbooks/` y `kb/troubleshooting/` de TA.

**Cuándo se invoca**
- Tickets sobre marcajes, horas, retardos, reportes de asistencia, sincronización TA → Payroll.

### 3.2. Técnico Payroll

**ID:** `tech_payroll`  
**Rol:** Full‑stack de nómina.

**Objetivo**  
Explicar y corregir comportamientos de cálculo de nómina (montos, deducciones, prestaciones, liquidaciones).

**Inputs**
- Escalación L1 con ejemplos concretos (empleado(s), periodo, montos esperados vs calculados).
- Cliente + tipo de nómina/proceso (ordinaria, extraordinaria, liquidación, etc.).

**Outputs**
- Diagnóstico:
  - Causa raíz (configuración vs bug).
  - Impacto (cuántos empleados/periodos afectados).
- Plan de resolución:
  - Cambios de configuración recomendados.
  - Necesidad de data‑fix (y si requiere DB Specialist).
  - Propuesta de cambio en lógica de producto (si aplica).
- Texto para L1/cliente:
  - Explicación enfocada en negocio (sin ruido técnico innecesario).

**Contexto necesario**
- Repo **payroll**.
- Tablas de conceptos, fórmulas, periodos, incidencias en Postgres/Mongo.
- KB de nómina (runbooks de temas recurrentes).

**Cuándo se invoca**
- Tickets de montos incorrectos, diferencias en liquidaciones, vacaciones pagadas, bonos, etc.

### 3.3. Técnico Frontend (personaapp)

**ID:** `tech_frontend`  
**Rol:** Especialista en personaapp, UX y contratos front‑backend.

**Objetivo**  
Resolver problemas de UI, permisos visibles desde front, errores en flujos de usuario.

**Inputs**
- Descripción de la pantalla/vista (URL o identificador).
- Capturas o mensajes de error visibles en front.
- Datos del usuario afectado (rol, cliente).

**Outputs**
- Diagnóstico:
  - Problema de UI, permisos o contrato API.
- Plan:
  - Fix propuesto en front (validaciones, mensajes).
  - Si es un problema de backend, sugerir qué técnico debe continuar (ej. Payroll, TA).
- Mensaje para L1:
  - Qué decirle al cliente (workaround temporal, etc.).

**Contexto necesario**
- Repo **frontend/personaapp**.
- Definiciones de rutas, componentes, manejo de estados.
- Contratos de APIs usados por esa vista.

**Cuándo se invoca**
- Tickets donde "la pantalla no muestra X", "no me deja hacer Y", "me sale error en el portal del colaborador".

### 3.4. Técnico BPM (solicitudes)

**ID:** `tech_bpm`  
**Rol:** Flujos de solicitudes y aprobaciones.

**Objetivo**  
Diagnosticar solicitudes atoradas, mal dirigidas o con estados inconsistentes.

**Inputs**
- ID de solicitud(es).
- Cliente, tipo de solicitud (vacaciones, permiso, etc.).
- Flujo esperado vs comportamiento observado.

**Outputs**
- Diagnóstico:
  - En qué parte del flujo se rompe o se queda trabado.
  - Si es config de flujo, datos o bug del motor BPM.
- Plan:
  - Data‑fix (cambiar estado, re‑disparar notificaciones).
  - Ajustes propuestos en definición de flujo.
- Texto para L1:
  - Qué sucedió y cómo se corrige.

**Contexto necesario**
- Repo **bpm/solicitudes**.
- Configuración de flujos por cliente en DB.
- KB de incidencias BPM.

**Cuándo se invoca**
- Tickets sobre solicitudes que no avanzan, no notifican, aprueban/rechazan mal.

### 3.5. Técnico Organization (estructura org)

**ID:** `tech_org`  
**Rol:** Especialista en estructura organizacional y sus efectos.

**Objetivo**  
Corregir problemas derivados de jerarquías, centros de costo y unidades organizacionales.

**Inputs**
- Descripción del problema (reportes, permisos, organigrama).
- IDs de unidades, personas y relaciones relevantes.
- Cliente afectado.

**Outputs**
- Diagnóstico:
  - Inconsistencias en estructura (ciclos, jefes faltantes, etc.).
  - Efecto sobre permisos/reportes.
- Plan:
  - Data‑fix o ajustes de config de estructura.
- Mensaje para L1:
  - Qué se va a corregir y qué revisar después.

**Contexto necesario**
- Repo **organization**.
- Tablas/colecciones de estructura por cliente.
- Reglas de negocio que dependen de organización.

**Cuándo se invoca**
- Tickets de jerarquías incorrectas, permisos raros basados en estructura, organigramas mal generados.

### 3.6. Técnico Persona

**ID:** `tech_persona`  
**Rol:** Dueño de los datos maestros de persona.

**Objetivo**  
Resolver inconsistencias en datos de persona que pegan en todos los módulos.

**Inputs**
- Datos del empleado/colaborador: ID, cliente.
- Descripción de la inconsistencia (fecha de alta/baja, documentos, etc.).
- Efecto observado en TA, Payroll, BPM, etc.

**Outputs**
- Diagnóstico:
  - Data‑entry vs regla de negocio vs bug de integración.
- Plan:
  - Corrección de datos, reglas o procesos de captura.
- Texto para L1:
  - Qué dato estaba mal y cómo se está corrigiendo.

**Contexto necesario**
- Repo **persona**.
- Tablas de personas y sus relaciones con otros módulos.
- KB de problemas típicos de datos maestros.

**Cuándo se invoca**
- Cualquier ticket donde la raíz sea un dato personal incorrecto/incompleto.

---

## 4. DB/Data Fix Specialist

**ID:** `db_fix_specialist`  
**Rol:** Cirujano de datos (Postgres + Mongo por cliente).

**Objetivo**  
Diseñar y/o ejecutar de forma segura los data‑fixes que los técnicos de módulo necesitan.

**Inputs**
- Solicitud de data‑fix desde técnico de módulo:
  - Descripción del problema.
  - Queries de diagnóstico.
  - Alcance (cuántos registros/clientes afectados).
- Restricciones de tiempo/ventana de mantenimiento.

**Outputs**
- Plan de data‑fix:
  - Queries de backup y validación.
  - Queries de actualización.
- Resultado:
  - Resumen de lo ejecutado (n registros, clientes, fecha/hora).
- Notas de KB:
  - Documentación del fix si es reutilizable.

**Contexto necesario**
- Acceso conceptual a la estructura de Postgres y Mongo por cliente.
- Estándares de naming de BD/colecciones por cliente.
- Procedimientos internos de cambio seguro (backups, checks).

**Cuándo se invoca**
- Cuando un técnico propone un data‑fix con impacto medio/alto o transversal.

---

## 5. Analista de Soportes – Support Analytics

**ID:** `support_analyst`  
**Rol:** Extrae insights y arma el roadmap desde los tickets.

**Objetivo**  
Convertir tickets + resoluciones en patrones, métricas y backlog de mejoras.

**Inputs**
- Export de tickets (CSV/JSON) por periodo.
- Información de módulo, cliente, severidad, causa raíz (cuando esté documentada).
- KB de incidentes ya conocidos.

**Outputs**
- Snapshot en `support_analytics/snapshots/`:
  - Temas principales.
  - Estadísticas cualitativas y patrones.
- Roadmap en `support_analytics/roadmap/`:
  - Ítems con categoría, módulo, descripción, impacto, prioridad.

**Contexto necesario**
- Estructura de los exports.
- Convenciones de tags/labels de tickets.
- Relación entre tickets y artículos de KB.

**Cuándo se invoca**
- Periódicamente (mensual/trimestral).
- Ad‑hoc cuando se quiere analizar un conjunto de tickets específicos (ej. un cliente grande).
