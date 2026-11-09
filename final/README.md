# Proyecto Final · Sistema agéntico asíncrono, idempotente y supervisado

**Proyecto:** Torque Motopartes, atención postventa (caso de estudio).
**Autor:** Enzo Saporiti · Curso AI Automation Advanced · Cohorte 2026.

## Qué se entrega

| Archivo | Contenido |
|---|---|
| `Saporiti_Enzo_ProyectoFinal_AI_Experto.pdf` | Documentación Maestra del Sistema Agéntico (DMSA): resumen ejecutivo, arquitectura con capturas del lienzo, reporte de auditoría con 3 ejemplos reales en JSON, ROI y TCO. Anexos con la evidencia de las pruebas, los System Prompts literales, los contratos JSON y la deuda técnica. |
| `Torque_Atencion_Asincrona_Final.json` | Orquestador (Manager): webhook, idempotencia, coordinación de workers, veredicto, HITL, despacho, CRM y alertas. |
| `Torque_Worker_2_Redaccion.json` | Redactor (Haiku 4.5) con la herramienta de búsqueda documental. |
| `Torque_Worker_3_Busqueda_Documental.json` | RAG sobre el manual de políticas (ingesta manual y recuperación). |
| `Torque_Worker_4_Juez_de_Calidad.json` | Juez (Sonnet 4.5, temperatura 0, JSON Schema) con controles deterministas y logs del veredicto. |
| `Torque_Worker_5_Logger_Central.json` | Registro de trazabilidad en Airtable. |
| `Torque_Alertas_de_Error.json` | Error Workflow global: Error Trigger → Slack. |
| `Torque_Simulador_de_Cliente.json` | Envía una solicitud y su reenvío al webhook para probar asincronía e idempotencia. |
| `capturas/` | Evidencia de las pruebas del 9/11/2026 (las mismas del Anexo A de la DMSA). |

Los `.json` no contienen API keys ni tokens: solo los nombres de las credenciales.

## Cómo funciona

```
Canal externo ──POST + X-Torque-Key──> Webhook
   │  valida campos (400 si faltan)
   │  busca el requestId en Solicitudes (200 "duplicada" si ya existe)
   │  registra la solicitud y responde 202 con el traceId   ← antes de cualquier IA
   ▼
Orquestador (segundo plano)
   memoria del cliente → W2 Redactor (↔ W3 manual) → W4 Juez → Auditoria_QA (upsert)
   ├─ ACEPTADO  → Telegram → HubSpot (solo validado)
   ├─ CORREGIR  → 1 reintento del Redactor con la crítica
   └─ RECHAZADO → Slack: Aprobar / Editar / Sin respuesta (4 h → aviso seguro, sin CRM)
Errores: salidas de error en workers, Telegram y HubSpot + Error Workflow global → Slack
Trazabilidad: el mismo traceId en la respuesta 202, Solicitudes, Auditoria_QA, logs, Slack y HubSpot
```

## Credenciales necesarias en n8n

| Nombre | Tipo | Uso |
|---|---|---|
| `Anthropic DEV` / `Anthropic PROD` | API key | Redactor y Juez (claves separadas por entorno) |
| `Torque Webhook Key` | Header Auth (`X-Torque-Key`) | Autenticación del webhook y del Simulador |
| `Airtable Personal Access Token Curso AI ADV` | Token | Memoria, auditoría, logs y solicitudes |
| `Slack Torque Motopartes` | Bot (chat:write, channels:read) | HITL y alertas |
| `Telegram Torque Bot` | Token del bot | Despacho al cliente |
| `HubSpot Torque` | OAuth2 | CRM |
| `Gemini Embeddings Torque` | API key | Embeddings del manual |

## Tablas de Airtable (base `Torque_Memoria`)

- **Solicitudes:** `requestId` (primario), `traceId`, `Estado`.
- **Memoria_Clientes:** `Session_ID`, `Nombre del Cliente`, `Estado del Caso`, `Resumen Consolidado`, `Datos Clave`, `Cantidad de Mensajes`, `Fecha de Actualización`.
- **Auditoria_QA:** `executionId`, `Fecha`, `Arquitectura`, `Intento`, `Consulta`, `Respuesta_Propuesta`, `Exactitud_Factual`, `Estado_Veredicto`, `Categoria_Error`, `Critica`, `Tokens_Entrada`, `Tokens_Salida`, `Costo_USD`, `Lote`, `traceId`, `Es_Aceptado`, `Es_HITL`, `Clave` (clave del upsert: `traceId-iN`).
- **Logs_Trazabilidad:** `traceId`, `timestamp`, `workflow`, `nodeName`, `evento`, `status`, `toolOutput`, `systemPromptSnapshot`, `confidenceScore`.

HubSpot necesita cinco propiedades de contacto: `torque_origen`, `torque_ultimo_traceid`, `torque_validacion`, `torque_intencion` y `torque_ultima_interaccion` (Date picker). Ver `modulo10/`.

## Cómo importar y probar

1. Crear las credenciales y las tablas.
2. Importar los workers (W2 a W5) y el workflow de alertas; después el orquestador y el Simulador. Reasignar las credenciales y los sub-workflows en cada nodo que lo pida.
3. Asignar `Torque – Alertas de Error` como Error Workflow del orquestador, W2, W3 y W4 (Settings → Error Workflow).
4. Ejecutar la rama de ingesta manual del W3 (el vector store se borra cuando n8n reinicia).
5. Publicar el orquestador y copiar la Production URL del webhook en los dos HTTP Request del Simulador.
6. Completar `Solicitud de prueba` en el Simulador con un `chat_id` de Telegram propio y ejecutarlo. Resultado esperado: 202 en el primer envío y 200 "duplicada" en el segundo.

### Contrato de la solicitud

```json
{
  "requestId": "req-20261109-125611",
  "session_id": "<Session_ID de Memoria_Clientes>",
  "chat_id": "<chat de Telegram>",
  "email": "cliente-a@example.com",
  "consulta": "¿Cuánto tarda en llegar un paquete a Mendoza?",
  "intencion": "VENTAS",
  "arquitectura": "A_Haiku",
  "respuesta_forzada": ""
}
```

Obligatorios: `requestId`, `consulta` y `chat_id`. `respuesta_forzada` solo se usa en pruebas para auditar al Juez con una respuesta defectuosa.

## Pruebas y evidencia (9/11/2026)

| # | Escenario | traceId | Resultado |
|---|---|---|---|
| 1 | Solicitud incompleta | — | 400 en 398 ms, sin consumir IA |
| 2 | Aceptada + reenvío | `trc-20261109-125612-410` | 202 en 3,6 s y 200 "duplicada" con el mismo traceId; Juez 5/5. Telegram respondió Gateway Timeout pero entregó el mensaje: no se reintentó y se alertó |
| 3 | HITL con edición | `trc-20261109-132944-422` | RECHAZADO 1/5 → Editar → Telegram → HubSpot `HITL_EDITADO` |
| 4 | HITL sin respuesta | `trc-20261109-135135-432` | Aviso de derivación y escalamiento al Supervisor, sin CRM |
| 5 | Falla del CRM | `trc-20261109-141257-441` | HubSpot rechazó el email: 3 intentos → alerta en `#torque-ops-logs` |

Las cinco corridas reenviaron la misma solicitud: 0 duplicados de procesamiento, envío o registro.

## Limitaciones conocidas

- **Críticas del Juez:** en las pruebas 3 y 4 el veredicto fue correcto, pero la crítica citó la sección 4.3 (fallas de fábrica) cuando correspondía la 4.1 (derecho de arrepentimiento). El revisor debe verificar contra el manual, no contra la crítica.
- **Textos editados:** se envían sin volver a pasar por el Juez.
- **Canal de email (Módulo 6):** escribe en HubSpot antes del Juez; queda fuera del alcance productivo hasta reencaminarlo por este flujo.
- **Pedidos puntuales (W1) y clasificación de intención:** viven en la capa de canales de los módulos 2 a 6; la solicitud asíncrona recibe la intención ya clasificada.
- **Idempotencia:** dos solicitudes idénticas en el mismo segundo podrían pasar ambas el control (Airtable no tiene restricción de unicidad).
- **Vector store en memoria:** hay que reingestar el manual después de cada reinicio de n8n.
