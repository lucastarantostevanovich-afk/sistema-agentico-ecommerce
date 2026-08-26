# Sistema Agéntico de Sales Ops & Customer Success para E-commerce

Proyecto integrador del curso de **IA Automation Avanzada**.

El objetivo es desarrollar de forma incremental una arquitectura agéntica empresarial capaz de recibir consultas, interpretar intenciones, delegar tareas a especialistas, aplicar controles de seguridad y mantener trazabilidad.

---

## Arquitectura actual — Módulo 2

La solución evolucionó desde el agente base del Módulo 1 hacia una arquitectura distribuida basada en el patrón **Manager-Worker**.

Flujo general:

```text
Chat Trigger
↓
Normalización
↓
Guardrails
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
Workflow Manager como orquestador principal.
Taxonomía cerrada:
SALES
CUSTOMER_SUCCESS
HUMAN_REVIEW
Structured Output Parser.
Switch de enrutamiento.
Worker Sales independiente.
Worker Customer Success independiente.
Execute Workflow Trigger en ambos Workers.
Contratos JSON de entrada y salida.
Set/Edit Fields para evitar Data Stuffing.
Wait for child to finish.
Manejo estructurado de errores.
Reintentos limitados.
Human Review.
Guardrail de datos sensibles.
Jailbreak Detection.
Topical Alignment.
Logs mediante Gmail.
Persistencia de trazabilidad mediante Google Sheets.
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
Logs y persistencia en Google Sheets.

Las rutas SALES, CUSTOMER_SUCCESS, HUMAN_REVIEW y los Guardrails fueron validadas mediante pruebas manuales.

Roadmap
Módulo 1: Agente base.
Módulo 2: Manager-Worker y sub-workflows.
Módulo 3: Memoria persistente.
Módulo 4: Integraciones empresariales.
Módulo 5: RAG y base documental.
Módulo 6: Voice AI.
Módulo 7: Especialización vertical.
Módulo 8: AI-as-a-Judge.
Módulo 9: Gobernanza, métricas y ROI.
Módulo 10: Propuesta técnico-comercial.
Módulo 11: Proyecto Final Integrador.
Seguridad

La solución incorpora:

guardrails preventivos;
taxonomía cerrada;
Human Review;
manejo estructurado de errores;
trazabilidad mediante trace_id;
logs;
persistencia de auditoría.

Las credenciales, API Keys y tokens privados no se almacenan en el repositorio.
