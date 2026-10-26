# Checkpoint 8 – Supervisor QA con AI-as-a-Judge, test A/B y dashboard

Documento de entrega: [`Checkpoint_Modulo8_Saporiti_Enzo.pdf`](Checkpoint_Modulo8_Saporiti_Enzo.pdf)

## Arquitectura

```
[Formulario de prueba] → [Contexto del cliente (Airtable)] → [Agente (Worker 2)] → [Juez (Worker 4 · JSON Schema)]
      → [Registrar veredicto (Airtable)] → [Switch]
           ├ ACEPTADO (exactitud ≥ 4) → envío real (Telegram)
           ├ CORREGIR → la crítica vuelve al agente (máximo 1 reintento) → si no aprueba → humano
           └ RECHAZADO / fallback → pausa + Slack [Aprobar] [Editar]
```

| Archivo | Workflow |
|---|---|
| [`supervisor_qa_modulo8_saporiti_enzo.json`](supervisor_qa_modulo8_saporiti_enzo.json) | Torque – Supervisor QA (M8) |
| [`worker4_juez_modulo8_saporiti_enzo.json`](worker4_juez_modulo8_saporiti_enzo.json) | Torque – Worker 4 Juez de Calidad (Claude Sonnet 4.5, temperatura 0) |
| [`worker2_modulo8_saporiti_enzo.json`](worker2_modulo8_saporiti_enzo.json) | Torque – Worker 2 Redacción (modelo según la arquitectura del A/B y crítica del Juez) |
| [`auditoria_qa.csv`](auditoria_qa.csv) | Registro externo de todos los veredictos (tabla `Auditoria_QA` de Airtable) |

## Salida del Juez (JSON Schema)

- `exactitud_factual`: entero de 1 a 5.
- `estado_veredicto`: `ACEPTADO`, `CORREGIR` o `RECHAZADO`.
- `categoria_error` y `critica`: campos de soporte para la autocorrección y el dashboard.

## Resultados del A/B (Lote 1, 10 corridas)

| Indicador | A · Haiku 4.5 | B · Sonnet 4.5 |
|---|---|---|
| Exactitud media del intento 1 | 4,2 | 4,0 |
| Resueltos sin humano | 4/5 | 4/5 |
| Costo medio del agente por caso | USD 0,0056 | USD 0,0133 |
| Costo de IA cada 1.000 casos (agente + Juez) | USD 18,87 | USD 24,64 |

**Recomendación:** Arquitectura A, más controles deterministas en el Juez para las reglas literales. El detalle está en el PDF.

## Evidencia

**Supervisor QA, vista general:**

![Supervisor](m8_gral.png)

**Zona 1:**

![Zona 1](m8_zona1.png)

**Zona 2:**

![Zona 2](m8_zona2.png)

**Worker 4 (Juez):**

![Juez](w4_lienzo.png)

**Configuración del Juez:**

![Config](juez_config.png)

**Switch de 3 rutas:**

![Switch](switch_config.png)

**Ruta ACEPTADO:**

![Aceptado](ejecucion_aceptado.png)

**Ruta CORREGIR:**

![Corregir](ejecucion_corregir.png)

**Ruta RECHAZADO:**

![Rechazado](ejecucion_rechazado.png)

**Escalamiento en Slack:**

![Slack](slack_botones.png)

**Tabla de auditoría en Airtable:**

![Airtable](airtable_auditoria.png)
