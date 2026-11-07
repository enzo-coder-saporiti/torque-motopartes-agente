# Torque Motopartes – Sistema Agéntico de Atención al Cliente

Proyecto integrador del curso de IA (cohorte 2026).
Autor: Enzo Saporiti

## Caso de negocio

Torque Motopartes es una tienda online (ficticia) de repuestos y accesorios para moto. El equipo de atención recibe consultas repetitivas sobre estado de pedidos, envíos, devoluciones y compatibilidad de productos.

Este proyecto construye, módulo a módulo, un sistema de agentes de IA en n8n que:

- Atiende consultas de clientes por chat, email y notas de voz en Telegram.
- Clasifica cada consulta en una taxonomía cerrada: `PEDIDOS_ENVIOS`, `DEVOLUCIONES_GARANTIAS`, `VENTAS` u `OTRO`.
- Consulta información real del negocio (pedidos y manual de políticas, con cita de la fuente).
- Pasa cada respuesta por un Juez de calidad antes de enviarla o registrarla en el CRM.
- Deriva a un asesor humano cuando corresponde.

Todo el sistema opera con guardrails de seguridad y trazabilidad: cada caso lleva un `traceId` que lo sigue por todos los sub-workflows, los logs y el CRM.

## Estructura del repositorio

Cada módulo parte del flujo del módulo anterior y lo extiende. No se rehace desde cero.

| Carpeta | Hito | Contenido | Estado |
|---|---|---|---|
| [`modulo1/`](modulo1/) | Agente base | Chat Trigger → AI Agent (Tools Agent) + tool de pedidos → Log en Slack | ✅ Entregado |
| [`modulo2/`](modulo2/) | Multi-agente | Manager + 2 Workers como sub-workflows (patrón Manager-Worker) | ✅ Entregado |
| [`modulo3/`](modulo3/) | Memoria | Memoria híbrida: Airtable por Session_ID + resumen automático en JSON | ✅ Entregado |
| [`modulo4/`](modulo4/) | Integraciones | Canal email: Gmail + HubSpot + Slack con guardrails (anti auto-reply, Look up, borradores HITL) | ✅ Entregado |
| [`modulo5/`](modulo5/) | RAG | Manual parseado con LlamaParse + vector store y Worker 3 de búsqueda (Top-K y Minimum Score) | ✅ Entregado |
| [`modulo6/`](modulo6/) | Voz | Canal de notas de voz en Telegram: Whisper (Groq) + agente RAG + ElevenLabs, con contención de costos y compliance | ✅ Entregado |
| [`modulo7/`](modulo7/) | Vertical de industria | Informe de consultoría: arquitectura multi-agente, permisos, KPIs, semáforo HITL y scorecards | ✅ Entregado |
| [`modulo8/`](modulo8/) | Supervisor QA | Juez AI-as-a-Judge (JSON Schema), Switch de 3 rutas con autocorrección y HITL en Slack, test A/B Haiku vs Sonnet | ✅ Entregado |
| [`modulo9/`](modulo9/) | Gobernanza y monitoreo | traceId en todos los sub-workflows, Logger central, dashboard en Airtable, TCO/ROI, keys por entorno, riesgos y ZDR | ✅ Entregado |
| [`modulo10/`](modulo10/) | Brochure técnico-comercial | Propuesta para comité: diagrama de valor, business case con payback, roadmap de fases 0 a 3, pricing híbrido con ventana de conversión de 7 días, plan de 30 días. Marca de origen en HubSpot (5 propiedades) escrita solo después del Juez | ✅ Entregado |
| `final/` | Proyecto Final | Sistema asíncrono, idempotente y supervisado + Documentación Maestra (DMSA) | Pendiente |

## Stack

| Herramienta | Rol |
|---|---|
| n8n Cloud | Orquestación de los flujos (Manager, Workers y Supervisor QA) |
| Anthropic (Claude) | Haiku 4.5 para clasificación, redacción, resúmenes y voz; Sonnet 4.5 (temperatura 0) para el Juez de calidad |
| Google Sheets | Base de pedidos (solo lectura) |
| Airtable | Memoria persistente de clientes, auditoría del Juez, logs de trazabilidad y dashboard operativo |
| Slack | Observabilidad (`#torque-ops-logs`) y aprobaciones humanas (`#torque-escalamientos`) |
| Gmail | Canal de email con borradores para revisión humana |
| HubSpot | CRM: contactos y marca de origen de cada interacción validada del agente |
| LlamaParse | Conversión del manual de políticas a Markdown limpio |
| Gemini | Embeddings del manual para la búsqueda documental |
| Telegram | Canal de voz y despacho de respuestas |
| Groq (Whisper) | Transcripción de notas de voz |
| ElevenLabs | Respuestas en audio |

## Seguridad

Los archivos `.json` exportados de n8n **no contienen API keys ni tokens**, solo los nombres de las credenciales. Para ejecutar los flujos, cada usuario debe crear sus propias credenciales en n8n (ver el README de cada módulo). Las keys de Anthropic están separadas por entorno (desarrollo y producción), cada una con su tope de gasto.
