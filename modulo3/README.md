# Checkpoint 3 – Memoria persistente y resumen automático

Documento de entrega: [`PreEntrega_Modulo3_EnzoSaporiti.pdf`](PreEntrega_Modulo3_EnzoSaporiti.pdf)

## Caso de uso

Cada ejecución de n8n es efímera. Sin memoria, el agente "olvida" al cliente entre un mensaje y el siguiente. Este módulo agrega una **memoria híbrida** a la arquitectura Manager-Worker del Módulo 2:

| Capa | Herramienta | Qué guarda |
|---|---|---|
| Corto plazo | Simple Memory de n8n (volátil) | Los últimos mensajes de la sesión, solo para generar el resumen |
| Largo plazo | Airtable, por Session_ID | Nombre verificado, estado del caso, resumen consolidado en JSON, datos clave y contador |

## Circuito

1. **Lectura:** Airtable Search filtra por `Session_ID`.
   - Si el cliente es **nuevo**, se crea su registro.
   - Si es **recurrente**, se carga su memoria.
2. **Inyección:** el Redactor (Worker 2, AI Agent en modo Tools Agent) recibe el nombre y el último resumen entre `[INICIO DE CONTEXTO COMPARTIDO]` y `[FIN DEL CONTEXTO COMPARTIDO]`.
3. **Resumen:** al superar los 5 mensajes, Claude Haiku 4.5 condensa el historial en `{ asunto_principal, puntos_clave, accion_requerida }`.
4. **Escritura:** se actualiza el mismo registro por su id (escritura idempotente). Nunca se guardan transcripciones completas.

## Archivos

| Archivo | Workflow |
|---|---|
| [`manager_modulo3_saporiti_enzo.json`](manager_modulo3_saporiti_enzo.json) | Torque – Manager (M3) |
| [`worker1_modulo3_saporiti_enzo.json`](worker1_modulo3_saporiti_enzo.json) | Torque – Worker 1 Consulta Pedidos (ahora devuelve el nombre verificado) |
| [`worker2_modulo3_saporiti_enzo.json`](worker2_modulo3_saporiti_enzo.json) | Torque – Worker 2 Redacción (con inyección de memoria) |

## Credenciales necesarias

Las del Módulo 2 (**Anthropic**, **Google Sheets** y **Slack**), más **Airtable Personal Access Token API**. El token necesita los scopes `data.records:read`, `data.records:write` y `schema.bases:read`, y acceso a la base `Torque_Memoria`.

## Cómo reproducirlo

1. **Creá la base en Airtable:** base `Torque_Memoria`, tabla `Memoria_Clientes`, con estas columnas:

   | Columna | Tipo |
   |---|---|
   | `Session_ID` | Single line text (campo primario) |
   | `Nombre del Cliente` | Single line text |
   | `Estado del Caso` | Single select: NUEVO, ATENDIDO, ESCALADO |
   | `Resumen Consolidado` | Long text |
   | `Datos Clave` | Long text |
   | `Cantidad de Mensajes` | Number |
   | `Fecha de Actualización` | Last modified time |

2. **Importá y publicá los Workers:** importá los dos Workers, asigná sus credenciales y publicalos con **Publish**.
3. **Importá el Manager:** asigná las credenciales y seleccioná la base y la tabla en los nodos `Leer memoria`, `Registrar cliente nuevo` y `Persistir memoria`.
4. **Reconectá los Workers:** en `Delegar a Worker 1` y `Delegar a Worker 2`, volvé a seleccionar los Workers desde la lista.

## Prueba sugerida

Enviá al menos 6 mensajes en la misma sesión del chat. El primero debería ser:

```
Hola, quiero saber dónde está mi pedido TM-1003, mi mail es martin.diaz@example.com
```

Desde el sexto mensaje, `Resumen Consolidado` se completa con el JSON. En el mensaje siguiente, el agente retoma la conversación usando ese resumen.

## Evidencia

**Lienzo general del Manager M3:**

![Manager M3](manager_m3_lienzo.png)

**Zona 1 – Memoria de entrada:**

![Zona 1](m3_zona1_memoria_entrada.png)

**Zona 2 – Enrutamiento:**

![Zona 2](m3_zona2_enrutamiento.png)

**Zona 3 – Memoria de salida:**

![Zona 3](m3_zona3_memoria_salida.png)

**Configuración del resumen:**

![Generar resumen](generar_resumen_config.png)

**Esquema de Airtable:**

![Esquema](airtable_esquema.png)

**Registros con el resumen en JSON:**

![Registros](airtable_registros.png)

**Activación del resumen (ejecución #168):**

![Resumen](ejecucion_resumen.png)

**Contexto inyectado en el agente:**

![Inyección](inyeccion_contexto.png)

**Log de Slack con memoria:**

![Slack](slack_memoria.png)
