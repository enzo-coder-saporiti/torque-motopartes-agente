# Torque Motopartes – Sistema Agéntico de Atención al Cliente

Proyecto integrador del curso de IA (cohorte 2026).
Autor: Enzo Saporiti

## Caso de negocio

Torque Motopartes es una tienda online (ficticia) de repuestos y accesorios para moto. El equipo de atención recibe consultas repetitivas sobre estado de pedidos, envíos, devoluciones y compatibilidad de productos.

Este proyecto construye, módulo a módulo, un sistema de agentes de IA en n8n que:

- Atiende consultas de clientes.
- Clasifica cada consulta en una taxonomía cerrada: `PEDIDOS_ENVIOS`, `DEVOLUCIONES_GARANTIAS`, `VENTAS` u `OTRO`.
- Consulta información real del negocio.
- Deriva a un asesor humano cuando corresponde.

Todo el sistema opera con guardrails de seguridad y trazabilidad.

## Estructura del repositorio

Cada módulo parte del flujo del módulo anterior y lo extiende. No se rehace desde cero.

| Carpeta | Hito | Contenido | Estado |
|---|---|---|---|
| [`modulo1/`](modulo1/) | Agente base | Chat Trigger → AI Agent (Tools Agent) + tool de pedidos → Log en Slack | ✅ Entregado |
| [`modulo2/`](modulo2/) | Multi-agente | Manager + 2 Workers como sub-workflows (patrón Manager-Worker) | ✅ Entregado |
| [`modulo3/`](modulo3/) | Memoria | Memoria híbrida: Airtable por Session_ID + resumen automático en JSON | ✅ Entregado |
| `modulo4/` | Integraciones | HubSpot + Gmail + Slack vía OAuth2 | Pendiente |
| `modulo5/` | RAG | Base documental en LlamaCloud | Pendiente |
| `modulo6/` | Voz | STT / TTS vía Telegram | Pendiente |
| ... | ... | ... | ... |
| `final/` | Proyecto Final | Sistema completo supervisado + documentación | Pendiente |

## Stack

| Herramienta | Rol |
|---|---|
| n8n Cloud | Orquestación de los flujos |
| Anthropic (Claude) | Modelo de lenguaje del agente |
| Google Sheets | Base de pedidos (solo lectura) |
| Slack | Canal de observabilidad del equipo de operaciones |
| Airtable | Memoria persistente y auditoría (desde el Módulo 3) |
| HubSpot, Gmail, LlamaCloud, Telegram, ElevenLabs | Se incorporan en los módulos siguientes |

## Seguridad

Los archivos `.json` exportados de n8n **no contienen API keys ni tokens**, solo los nombres de las credenciales. Para ejecutar los flujos, cada usuario debe crear sus propias credenciales en n8n (ver el README de cada módulo).
