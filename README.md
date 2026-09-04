# Sistema Agéntico de Sales Ops & Customer Success para E-commerce

Proyecto integrador del curso de **IA Automation Avanzada**.

El objetivo es desarrollar de forma incremental una arquitectura agéntica empresarial capaz de recibir consultas, interpretar intenciones, delegar tareas a especialistas, aplicar controles de seguridad, mantener trazabilidad, conservar memoria persistente entre ejecuciones e integrarse con herramientas reales del negocio.

---

# Arquitectura actual — Módulo 4

La solución evolucionó desde el agente base del Módulo 1 hacia una arquitectura distribuida basada en el patrón **Manager-Worker**.

En el Módulo 3 se incorporó una capa de **memoria híbrida y persistencia de contexto**.

En el Módulo 4 se sumó una capa de **integraciones empresariales reales**, incorporando:

- Gmail como casilla de soporte.
- Airtable como CRM ligero / fuente operativa de clientes.
- Slack como canal de operaciones.
- Controles anti auto-respuesta.
- Validación de emails.
- Lookup previo a creación de contactos.
- Creación de borradores con Human-in-the-loop.
- Limpieza de payload antes de conectores externos.

---

# Flujo general

```text
Gmail Trigger / Chat Trigger
↓
IF Anti Auto Reply
├── TRUE → Cortar Bucle / No Responder
└── FALSE
     ↓
Normalización de entrada
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
├── SALES
│    ↓
│    Worker Sales
│    ↓
│    Preparar CRM
│    ↓
│    IF Email válido
│    ↓
│    Buscar contacto
│    ↓
│    IF Contacto existe
│    ├── TRUE → Actualizar contacto
│    └── FALSE → Crear contacto
│    ↓
│    Gmail Create Draft
│    ↓
│    Logs
│
├── CUSTOMER_SUCCESS
│    ↓
│    Worker Customer Success
│    ↓
│    Preparar CRM
│    ↓
│    IF Email válido
│    ↓
│    Buscar contacto
│    ↓
│    IF Contacto existe
│    ├── TRUE → Actualizar contacto
│    │           ↓
│    │           Gmail Create Draft
│    │
│    └── FALSE → Human Review
│                ↓
│                Limpiar Payload Slack
│                ↓
│                Slack Ops
│
└── HUMAN_REVIEW
     ↓
     Preparar Human Review
     ↓
     Limpiar Payload Slack
     ↓
     Slack Ops

↓
Logs
↓
Google Sheets
Funcionalidades implementadas
Arquitectura multi-agente

La solución utiliza una arquitectura distribuida basada en el patrón Manager-Worker.

Componentes principales:

Workflow Manager como orquestador principal.
Worker Sales independiente.
Worker Customer Success independiente.
Execute Workflow Trigger en ambos Workers.
Wait for child workflow to finish.
Contratos JSON de entrada y salida.
Manejo estructurado de errores.
Reintentos limitados.
Human Review como ruta segura.
Clasificación y enrutamiento

La taxonomía del Manager es cerrada:

SALES
CUSTOMER_SUCCESS
HUMAN_REVIEW

El Manager utiliza:

AI Agent.
Structured Output Parser.
Switch de enrutamiento.
Reglas explícitas de clasificación.
Human Review ante ambigüedad o sensibilidad.

La clasificación no depende únicamente de palabras clave aisladas.

Seguridad y Guardrails

La solución incorpora controles preventivos antes de ejecutar acciones externas.

Guardrails implementados
Guardrail de datos sensibles.
Detección de números de tarjeta.
Jailbreak Detection.
Topical Alignment.
Human Review.
Taxonomía cerrada.
Trazabilidad mediante trace_id.
Manejo estructurado de errores.
Logs de ejecución.
Persistencia de auditoría mediante Google Sheets.

Las credenciales, API Keys y tokens privados no se almacenan en el repositorio.

Módulo 3 — Memoria Persistente y Contexto Híbrido

En el Módulo 3 se incorporó una capa de memoria persistente sobre la arquitectura multi-agente existente.

El objetivo es evitar la pérdida de contexto entre ejecuciones efímeras de n8n y permitir que el Manager interprete nuevas consultas utilizando información histórica de la misma sesión.

Memoria de corto plazo

Se utiliza Simple Memory como memoria dinámica reciente.

Características:

Clave de aislamiento mediante Session_ID.
Ventana de contexto de 6 interacciones.
Historial reciente disponible para el Manager.
Separación entre sesiones.
Prevención de cruces de contexto entre usuarios.
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

Circuito de persistencia
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

El objeto se almacena en Resumen_Consolidado.

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
Módulo 4 — Integraciones Avanzadas e Interconexión de Sistemas

En el Módulo 4 se conectó el sistema agéntico con herramientas externas reales del ecosistema operativo.

El objetivo es integrar el cerebro agéntico con una fuente operativa de clientes, una casilla de soporte y un canal interno, incorporando controles visuales preventivos para evitar errores operativos.

Gmail — Casilla de soporte

Se incorporó un Gmail Trigger como segundo canal de entrada.

Flujo:

Gmail Trigger
↓
IF Anti Auto Reply
├── TRUE → Cortar Bucle / No Responder
└── FALSE → Normalizar Email

El sistema detecta e ignora automáticamente correos cuyos asuntos o remitentes contengan:

Auto-reply
Out of office
Undeliverable
no-reply@

También se incorporaron controles adicionales para evitar que mensajes generados por el propio sistema vuelvan a ingresar al workflow.

Prevención de bucles infinitos

El nodo:

M4 - IF - Anti Auto Reply

se encuentra inmediatamente después del Gmail Trigger.

Si detecta un mensaje automático:

TRUE
↓
M4 - Cortar Bucle - No Responder

El flujo se detiene antes de ejecutar agentes o conectores externos.

Normalización y limpieza del correo

El nodo:

M4 - Set - Normalizar Email

reduce el payload original del correo a los campos operativos necesarios:

from
subject
body_text
message_id
thread_id
channel
session_id
message
user_name

Esto evita transportar objetos innecesarios o payloads pesados hacia el resto de la arquitectura.

CRM ligero / Fuente Operativa de Clientes

Para este checkpoint se utiliza Airtable como CRM ligero y fuente operativa de clientes.

Base:

BASE CODER IA AVANZADA

Tabla:

CRM_Contactos

Campos principales:

Email
Nombre
Session_ID
Estado
Ultima_Interaccion
Origen
Lookup antes de Create

Antes de crear un contacto, el sistema realiza una búsqueda por email.

Flujo:

Preparar CRM
↓
IF Email válido
↓
Buscar Contacto
↓
IF Contacto Existe
├── TRUE → Actualizar contacto
└── FALSE → Crear contacto

Este diseño evita duplicados y garantiza que la creación solo ocurra cuando el contacto no existe previamente.

Validación de email

Antes de consultar o escribir en el CRM se valida que el email no esté vacío.

Se utilizan los nodos:

M4 - IF - Email Valido
M4 - IF - Email Valido CS

Esto evita operaciones sobre registros incompletos y reduce errores de payload.

Gmail Create Draft — Human-in-the-loop

Las respuestas generadas por los Workers no se envían automáticamente al cliente.

Se utilizan:

M4 - Gmail - Crear Draft Sales
M4 - Gmail - Crear Draft CS

Configuración:

Resource: Draft
Operation: Create

El mensaje queda almacenado en la carpeta de borradores de Gmail para revisión humana.

Este mecanismo implementa una barrera de seguridad Human-in-the-loop antes de la emisión final.

Customer Success — Escalamiento controlado

Si una consulta de Customer Success corresponde a un contacto existente:

CUSTOMER_SUCCESS
↓
Worker CS
↓
Buscar Contacto
↓
Actualizar Contacto
↓
Gmail Create Draft

Si el contacto no existe:

CUSTOMER_SUCCESS
↓
Buscar Contacto
↓
Contacto Existe = FALSE
↓
Preparar Human Review CS
↓
Limpiar Payload Slack
↓
Slack Ops

No se crea automáticamente un contacto de soporte sin validación.

Human Review

El sistema cuenta con una ruta específica para casos ambiguos o sensibles:

HUMAN_REVIEW
↓
Preparar Human Review
↓
Limpiar Payload Slack
↓
Slack Ops

Esta ruta permite derivar casos que requieren intervención humana antes de ejecutar una acción externa.

Slack — Canal de operaciones

Slack se utiliza como canal interno del equipo operativo.

Nodo principal:

M4 - Slack - Notificar Ops

Antes de enviar información se utiliza:

M4 - Set - Limpiar Payload Slack

El payload se reduce a:

tipo_evento
trace_id
session_id
email
motivo
requiere_revision_humana
channel

Include Other Input Fields se mantiene desactivado.

Esto evita transportar objetos innecesarios, payloads masivos o información técnica irrelevante.

Principio de Mínimo Privilegio

Las integraciones se configuraron limitando las operaciones a las funciones necesarias.

El sistema requiere únicamente:

lectura de mensajes entrantes;
creación de borradores;
búsqueda de contactos;
actualización y creación controlada de registros;
envío de alertas internas a Slack.

Las respuestas finales al cliente no se envían automáticamente.

Pruebas de regresión realizadas

Se realizaron pruebas manuales utilizando Test Step y Execute Workflow.

Casos validados:

Gmail normal.
Correo automático.
Anti auto-reply.
Corte de bucle.
Clasificación SALES.
Clasificación CUSTOMER_SUCCESS.
Clasificación HUMAN_REVIEW.
Contacto existente.
Contacto inexistente.
Lookup antes de Create.
Update de contacto.
Create de contacto.
Validación de email.
Gmail Draft Sales.
Gmail Draft Customer Success.
Human Review.
Escalamiento de CS sin contacto.
Limpieza de payload.
Notificación Slack.

Durante las pruebas se corrigieron referencias cruzadas entre ramas y se validó el comportamiento completo de punta a punta.

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

modulo4/
    checkpoint4_lucas_taranto.json

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

Las rutas:

SALES
CUSTOMER_SUCCESS
HUMAN_REVIEW

y los Guardrails fueron validados mediante pruebas manuales.

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

Checkpoint 4

Incorporación de integraciones avanzadas e interconexión con sistemas externos:

Gmail como casilla de soporte.
Airtable como CRM ligero / fuente operativa de clientes.
Slack como canal operativo.
OAuth2 en Gmail y Slack.
IF anti auto-reply inmediatamente posterior al trigger.
Protección contra bucles infinitos.
Normalización del payload del correo.
Validación de emails.
Lookup antes de Create.
Prevención de duplicados.
Update/Create de contactos.
Create Draft para Sales.
Create Draft para Customer Success.
Human-in-the-loop.
Escalamiento de Customer Success sin contacto.
Set de limpieza antes de Slack.
Notificaciones internas.
Tests de regresión de punta a punta.

Archivo de entrega:

modulo4/checkpoint4_lucas_taranto.json

Repositorio del Módulo 4:

https://github.com/lucastarantostevanovich-afk/sistema-agentico-ecommerce/tree/main/modulo4

Roadmap

✅ Módulo 1: Agente base.
✅ Módulo 2: Manager-Worker y sub-workflows.
✅ Módulo 3: Memoria persistente y summarization.
✅ Módulo 4: Integraciones empresariales, CRM, Gmail, Slack y HITL.
⏳ Módulo 5: RAG y base documental.
⏳ Módulo 6: Voice AI.
⏳ Módulo 7: Especialización vertical.
⏳ Módulo 8: AI-as-a-Judge.
⏳ Módulo 9: Gobernanza, métricas y ROI.
⏳ Módulo 10: Propuesta técnico-comercial.
⏳ Módulo 11: Proyecto Final Integrador.
