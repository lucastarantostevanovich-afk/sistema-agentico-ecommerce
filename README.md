# Sistema Agéntico de Sales Ops & Customer Success para E-commerce

Proyecto integrador del curso de **IA Automation Avanzada**.

El objetivo es desarrollar de forma incremental una arquitectura agéntica empresarial capaz de recibir consultas, interpretar intenciones, delegar tareas a especialistas, aplicar controles de seguridad, mantener trazabilidad y conservar memoria persistente entre ejecuciones.

---

## Arquitectura actual — Módulo 3

La solución evolucionó desde el agente base del Módulo 1 hacia una arquitectura distribuida basada en el patrón **Manager-Worker**, incorporando en el Módulo 3 una capa de **memoria híbrida y persistencia de contexto**.

Flujo general:

```text
Chat Trigger
↓
Normalización
↓
Guardrails
↓
Búsqueda de memoria en Airtable
↓
IF - Usuario nuevo / recurrente
↓
Preparación de contexto
↓
Incremento de contador
↓
IF - Supera 5 mensajes
├── NO → AI Agent Manager
└── SÍ → Leer historial
         ↓
         GPT-4o-mini Summarizer
         ↓
         Parsear JSON
         ↓
         Actualizar Airtable
         ↓
         AI Agent Manager
↓
Switch de enrutamiento
├── SALES → Worker Sales
├── CUSTOMER_SUCCESS → Worker Customer Success
└── HUMAN_REVIEW → Revisión humana
↓
Logs
↓
Google Sheets
Funcionalidades implementadas
Arquitectura multi-agente
Workflow Manager como orquestador principal.
Patrón Manager-Worker.
Worker Sales independiente.
Worker Customer Success independiente.
Execute Workflow Trigger en ambos Workers.
Wait for child workflow to finish.
Contratos JSON de entrada y salida.
Manejo estructurado de errores.
Reintentos limitados.
Clasificación y enrutamiento

Taxonomía cerrada:

SALES
CUSTOMER_SUCCESS
HUMAN_REVIEW

El Manager utiliza:

AI Agent.
Structured Output Parser.
Switch de enrutamiento.
Reglas explícitas para evitar clasificación por palabras clave aisladas.
HUMAN_REVIEW como ruta segura ante ambigüedad o sensibilidad.
Seguridad

La solución incorpora:

Guardrail de datos sensibles.
Detección de números de tarjeta.
Jailbreak Detection.
Topical Alignment.
Human Review.
Taxonomía cerrada.
Manejo estructurado de errores.
Trazabilidad mediante trace_id.
Logs de ejecución.
Persistencia de auditoría mediante Google Sheets.

Las credenciales, API Keys y tokens privados no se almacenan en el repositorio.

Módulo 3 — Memoria Persistente y Contexto Híbrido

En el Módulo 3 se incorporó una capa de memoria persistente sobre la arquitectura multi-agente existente.

El objetivo es evitar la pérdida de contexto entre ejecuciones efímeras de n8n y permitir que el Manager pueda interpretar nuevas consultas utilizando información histórica de la misma sesión.

Memoria de corto plazo

Se utiliza Simple Memory como memoria dinámica reciente.

Características:

Clave de aislamiento mediante Session_ID.
Ventana de contexto de 6 interacciones.
Historial reciente disponible para el Manager.
Separación entre sesiones para evitar cruces de contexto.
Memoria de largo plazo

La memoria persistente se implementa mediante Airtable.

Base:

BASE CODER IA AVANZADA

Tabla:

Memoria_Agente

Campos principales:

Session_ID
Fecha_Actualizacion
User_Name
Resumen_Consolidado
Estado_Caso
Datos_Clave
Cantidad_Mensajes

Cada registro se encuentra asociado a un Session_ID único.

Circuito de Persistencia

El circuito de memoria funciona mediante:

Search Records
↓
Filtro exacto por Session_ID
↓
IF - Existe Memoria
├── FALSE → Crear registro inicial
└── TRUE  → Recuperar contexto existente
↓
Preparar Contexto
↓
Actualizar memoria

Para usuarios nuevos se crea un registro inicial limpio.

Para usuarios recurrentes se recuperan:

nombre del usuario;
resumen consolidado;
estado del caso;
datos clave;
contador de mensajes.
Summarization automática

Cuando:

Cantidad_Mensajes > 5

se activa una rama de summarization.

Flujo:

Leer Historial Memoria
↓
GPT-4o-mini
↓
Parsear Resumen
↓
Actualizar Airtable

El modelo genera exclusivamente un objeto JSON con la estructura:

{
  "asunto_principal": "string",
  "puntos_clave": [
    "string"
  ],
  "accion_requerida": "string"
}

El objeto se almacena en Resumen_Consolidado y se actualiza sobre el mismo registro de Airtable de forma idempotente.

Los puntos relevantes también se conservan en Datos_Clave.

No se almacenan:

transcripciones completas;
payloads HTML;
logs técnicos;
prompts internos;
metadatos innecesarios.
Context Engineering

La memoria persistente recuperada se inyecta en el System Prompt del Manager mediante delimitadores rígidos:

[INICIO DE CONTEXTO COMPARTIDO]

Usuario:
...

Asunto principal:
...

Puntos clave:
...

Acción requerida:
...

Estado del caso:
...

[FIN DEL CONTEXTO COMPARTIDO]

El contenido recuperado se utiliza únicamente como contexto histórico.

La memoria no puede modificar:

el rol del agente;
la taxonomía;
las reglas;
los límites;
los criterios de clasificación;
las políticas de escalamiento.

Ante contradicción entre la memoria histórica y el mensaje actual, el Manager prioriza el mensaje actual.

Contrato Manager-Worker

Datos enviados a los Workers:

{
  "trace_id": "identificador de ejecución",
  "session_id": "identificador de sesión",
  "user_name": "Usuario",
  "message": "mensaje recibido",
  "intent": "SALES | CUSTOMER_SUCCESS"
}

Salida estándar:

{
  "status": "success | error",
  "worker": "sales | customer_success",
  "respuesta_propuesta": "",
  "contexto_utilizado": "",
  "fuente": "",
  "message": ""
}
Estructura del repositorio
modulo1/
    checkpoint1_lucas_taranto.json

modulo2/
    manager_modulo2_taranto_lucas.json
    worker1_modulo2_taranto_lucas.json
    worker2_modulo2_taranto_lucas.json

modulo3/
    manager_modulo2_taranto_lucas.json
    worker1_modulo2_taranto_lucas.json
    worker2_modulo2_taranto_lucas.json
    PreEntrega_Modulo3_LucasTaranto.pdf

README.md
Checkpoint 1

Implementación inicial del agente base con:

Chat Trigger.
Normalización.
AI Agent.
OpenAI.
Gmail como Tool.
Trazabilidad mediante trace_id.
Logs de observabilidad.
Checkpoint 2

Evolución hacia arquitectura multi-agente distribuida con:

Manager-Worker.
Dos sub-workflows independientes.
Contratos de datos.
Guardrails.
Human Review.
Manejo de errores.
Logs.
Persistencia en Google Sheets.

Las rutas SALES, CUSTOMER_SUCCESS, HUMAN_REVIEW y los Guardrails fueron validadas mediante pruebas manuales.

Checkpoint 3

Incorporación de memoria persistente y contexto híbrido:

Memoria de corto plazo mediante Simple Memory.
Persistencia de largo plazo mediante Airtable.
Aislamiento mediante Session_ID.
Search Records por sesión.
Rama para usuarios nuevos y recurrentes.
Contador de mensajes.
Summarization automática al superar 5 mensajes.
GPT-4o-mini como modelo de resumen.
Salida JSON estructurada.
Update idempotente del registro.
Inyección protegida de contexto en el Manager.
Delimitadores rígidos de memoria.
Prevención de amnesia cruzada.
Persistencia únicamente de información relevante y accionable.

La documentación técnica del checkpoint se encuentra disponible en:

modulo3/PreEntrega_Modulo3_LucasTaranto.pdf

Repositorio del Módulo 3:

https://github.com/lucastarantostevanovich-afk/sistema-agentico-ecommerce/tree/main/modulo3

Roadmap
✅ Módulo 1: Agente base.
✅ Módulo 2: Manager-Worker y sub-workflows.
✅ Módulo 3: Memoria persistente y summarization.
⏳ Módulo 4: Integraciones empresariales.
⏳ Módulo 5: RAG y base documental.
⏳ Módulo 6: Voice AI.
⏳ Módulo 7: Especialización vertical.
⏳ Módulo 8: AI-as-a-Judge.
⏳ Módulo 9: Gobernanza, métricas y ROI.
⏳ Módulo 10: Propuesta técnico-comercial.
⏳ Módulo 11: Proyecto Final Integrador.
