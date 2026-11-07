# Módulo 10 · Brochure técnico-comercial y trazabilidad en el CRM

**Proyecto:** Torque Motopartes, asistente de atención postventa supervisado (caso de estudio).
**Autor:** Enzo Saporiti · Curso AI Automation Advanced · Cohorte 2026.

## Qué se entrega

| Archivo | Contenido |
|---|---|
| `PreEntrega_Modulo10_EnzoSaporiti.pdf` | Brochure para el comité de dirección: narrativa ejecutiva y diagrama de valor, business case con baseline, ROI y payback, roadmap de las fases 0 a 3, pricing híbrido con ventana de conversión, plan de 30 días con responsables, trazabilidad de metadatos en el CRM y defensa ante tres objeciones. Anexos con evidencia y fórmulas. |
| `Torque_Supervisor_QA_M8.json` | Export del workflow Supervisor QA con la rama de registro en el CRM agregada en este módulo (sin credenciales: solo referencias por nombre). |
| `pe10_hubspot_contacto.png` | Contacto de prueba en HubSpot con las cinco propiedades de trazabilidad completas. |
| `pe10_logs_crm.png` | Registro del caso `trc-20261107-165603-392` en Airtable, de `inicio_caso` a `crm_registrado`. |
| `pe10_ejecucion_crm.png` | Ejecución en n8n: camino ACEPTADO → Telegram → HubSpot → log. |

## Qué se construyó en n8n

La consigna es sobre todo de redacción. La única pieza técnica nueva es la **marca de origen en el CRM**, que sostiene el cobro del success fee. Se agregó al Supervisor QA porque es el flujo que ya tiene al Juez antes del envío, así que HubSpot se escribe **solo después de una respuesta validada**.

```
Log: decisión final ─┐
Log: hitl aprobado  ─┼─> Datos para CRM -> ¿Tiene email? -> Registrar en CRM (HubSpot) ─┬─ éxito -> Log: CRM registrado
Log: hitl editado   ─┘                                                                 └─ error -> Alerta: falla CRM (Slack)
```

- **Datos para CRM** (Edit Fields): `email`, `traceId`, `intencion`, `validacion` (`JUEZ_ACEPTADO`, `HITL_APROBADO` o `HITL_EDITADO`, según la rama que corrió) y `fecha_interaccion`.
- **Registrar en CRM (HubSpot):** operación *Create or Update* (no duplica contactos), Retry On Fail con 3 intentos cada 2.000 ms, On Error con salida de error.
- **Log: CRM registrado:** llamada al Worker 5 (Logger Central) sin esperar, con el evento `crm_registrado`.
- **Alerta: falla CRM:** mensaje en `#torque-ops-logs` con el `traceId` y el rol responsable.
- El formulario **Caso de prueba** suma un campo `email` opcional; sin email no se escribe en el CRM.

## Credenciales y configuración

Se usan credenciales ya existentes en n8n: `HubSpot Torque` (OAuth2), `Slack Torque Motopartes`, `Telegram Torque Bot` y `Airtable Personal Access Token Curso AI ADV`. No hay keys escritas en nodos ni en textos.

Propiedades de contacto creadas en HubSpot (Settings → Data Management → Properties → Contact):

| Propiedad | Tipo | Valor |
|---|---|---|
| `torque_origen` | Single-line text | `agente_torque` |
| `torque_ultimo_traceid` | Single-line text | código de seguimiento del caso |
| `torque_validacion` | Single-line text | `JUEZ_ACEPTADO` / `HITL_APROBADO` / `HITL_EDITADO` |
| `torque_intencion` | Single-line text | tema de la consulta |
| `torque_ultima_interaccion` | Date picker | fecha de la última respuesta validada |

La fecha se calcula en el día de Argentina y se envía como medianoche UTC, que es lo que exige HubSpot:

```
{{ DateTime.fromISO($now.setZone('America/Argentina/Buenos_Aires').toISODate(), { zone: 'utc' }).toMillis() }}
```

En Airtable, el single select `evento` de `Logs_Trazabilidad` suma la opción `crm_registrado`.

## Cómo importar y probar

1. Crear las cinco propiedades en HubSpot.
2. Importar `Torque_Supervisor_QA_M8.json` en n8n y reasignar las credenciales en cada nodo que las pida.
3. Reingestar el vector store desde la rama manual del Worker 3 (se borra con cada reinicio de n8n).
4. Ejecutar el formulario con una consulta, una intención, la arquitectura `A_Haiku` y el email `cliente-a@example.com`.
5. Verificar el contacto en HubSpot (View all properties → buscar `torque`) y el registro en `Logs_Trazabilidad` filtrando por el `traceId`.

## Prueba y evidencia (7/11/2026)

| Caso | Intención | Veredicto | Envío | CRM | Validación |
|---|---|---|---|---|---|
| `trc-20261107-165603-392` | VENTAS | ACEPTADO (exactitud 5) | Telegram | Contacto creado con las 5 propiedades | `JUEZ_ACEPTADO` |

El mismo `traceId` aparece en los logs de W3, W2 y W4, en el envío y en HubSpot: es la correlación de punta a punta que usa la facturación.

## Decisiones comerciales (resumen)

- **Pricing:** setup de USD 1.500, retainer de USD 150 por mes desde la Fase 2 y success fee del 3% sobre ventas asistidas, con tope de USD 300 por mes.
- **Ventana de conversión:** 7 días corridos desde `torque_ultima_interaccion`.
- **Payback** (1.000 consultas por mes): 2,3 meses en el escenario piloto y 4,4 en el conservador.

## Limitaciones conocidas

- Si nadie responde en Slack en 4 horas, la rama "Editar" queda esperando sin límite. Se corrige en la entrega final con tres salidas: Aprobar, Editar y Sin respuesta.
- En el canal de email del Manager (M6), HubSpot todavía se escribe antes del Juez. Para la entrega final, el flujo productivo es el basado en el Supervisor QA.
- La sesión de prueba de `Memoria_Clientes` tiene un nombre de aspecto real; en la evidencia del PDF se ocultó y se va a reemplazar por "Cliente A".
