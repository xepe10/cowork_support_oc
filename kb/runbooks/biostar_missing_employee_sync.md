# EPC → BioStar missing employee sync

**Module:** Integraciones biométricas  
**Aplica a:** Cualquier cliente con sincronización activa EPC ↔ BioStar (Panavision es el ejemplo más frecuente)  
**Última actualización:** 2026-03-10

## Problem
Soporte reporta que, tras dar de alta a un colaborador en EPC, la persona no aparece en el sistema BioStar del cliente que tiene la integración activa.

## Symptoms
- El cliente indica “el colaborador no existe en BioStar”.
- En n8n, el flujo de sincronización de colaborador muestra la última ejecución con `success=false` y un `HTTP 500` (o diferente de 200) en el nodo que llama al servicio `Integration Biostar2 / users`.
- El envío de correo de confirmación tampoco se dispara porque el flujo terminó con error.

## Root cause
La creación del colaborador en BioStar falló al invocar el servicio `Integration Biostar2` (método `users`). El error exacto se registra en la colección `LogIntegration` de MongoDB. Ejemplo observado: `errorCode: 20 (Permission denied)` cuando BioStar rechazó la creación (p.ej. por un User Group inválido).

## Resolution steps
1. **Abrir el flujo en n8n**
   - Abre el workflow de sincronización correspondiente al cliente (p.ej. `Panavision – Sincronización de colaboradores BioStar`).
   - Ir a la pestaña **Executions**.
   - Filtrar por la fecha/hora reportada. Identifica la ejecución fallida (`success = false`).

2. **Revisar el nodo de integración**
   - Abre la ejecución y revisa el nodo que llama a `Integration Biostar2 / users`.
   - Si la respuesta no es 200, anota el `status code`, el cuerpo de respuesta y el payload enviado.

3. **Consultar LogIntegration en Mongo**
   - Conecta a la base Mongo donde se guardan las integraciones.
   - Ejecuta algo como:
     ```javascript
     db.LogIntegration
       .find({ service: "IntegrationBiostar2", method: "users" })
       .sort({ createdAt: -1 })
       .limit(5)
     ```
   - Ubica el registro correspondiente (mismo `createdAt`/`employeeId`).
   - Revisa `errorCode`, `errorMessage` y el `payload` para identificar la causa (ej.: `20 Permission denied`, `userGroupId inexistente`, etc.). Corrige el dato o coordina con BioStar si es un permiso/credencial.

4. **Reprocesar la alta manualmente** (una vez corregida la causa)
   - En n8n abre el webhook de reintento del flujo (nodo `Webhook alta`).
   - Copia el JSON del request original (desde la ejecución fallida o desde `LogIntegration.payload`) y pégalo en el nodo correspondiente del flujo (modo edición).
   - Guarda el flujo.
   - Copia la URL del webhook y ejecuta un **POST** desde Postman/cURL con el mismo cuerpo.
   - Verifica que se cree una nueva ejecución en n8n y que `success=true`.

5. **Validar**
   - Confirma en BioStar que el colaborador aparece en el User Group correcto.
   - Asegúrate de que exista un nuevo registro exitoso en `LogIntegration` (sin `errorCode`).
   - Notifica al cliente que la sincronización fue reintentada y completada.

## Checks & validation
- Ejecutar búsqueda del colaborador en BioStar y validar acceso/área asignada.
- Revisar que la última ejecución n8n aparezca como `success` y que el nodo de correo haya corrido.
- Revisar `LogIntegration` para confirmar que el registro más reciente tiene `status` o `errorCode` vacío.

## Notes / Variants
- El mismo procedimiento aplica para otros objetos sincronizados (áreas, puestos). Basta con abrir el flujo específico (p.ej. “Nueva área BioStar”) y repetir la revisión en **Executions** + `LogIntegration`.
- Errores comunes:
  - `500 Request failed`: revisar disponibilidad del API de BioStar o credenciales.
  - `20 Permission denied`: el User Group destino no existe o el API key perdió permisos.
- Si el webhook manual también falla, conserva el `requestId` y escala a Integraciones para revisar el servicio `Integration Biostar2`.
