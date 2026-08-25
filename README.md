# Sistema Agéntico de Sales Ops & Customer Success para E-commerce

Proyecto integrador del curso de **IA Automation Avanzada**.

El objetivo es construir de forma incremental un sistema agéntico empresarial capaz de recibir consultas de clientes, interpretar su intención, utilizar herramientas externas, mantener contexto, consultar conocimiento corporativo, ejecutar controles de calidad y escalar acciones sensibles a supervisión humana.

El proyecto evoluciona módulo a módulo hasta llegar a una arquitectura completa basada en el patrón **Manager-Worker**.

## Arquitectura actual — Módulo 1

La primera versión implementa un agente base con capacidad de razonamiento y uso autónomo de herramientas.

Flujo actual:

```text
Chat Trigger
↓
Normalización de entrada
↓
AI Agent Manager
├── OpenAI Chat Model
└── Gmail Tool
↓
Log de Observabilidad
```

## Funcionalidades implementadas

* Recepción de consultas mediante Chat Trigger.
* Normalización de variables de entrada.
* Generación de `trace_id` para trazabilidad.
* Conservación de `session_id` para futura memoria persistente.
* AI Agent configurado como Tools Agent.
* System Prompt estructurado con rol, ámbito, objetivos, reglas y escalamiento.
* Guardrail de máximo 5 iteraciones.
* Integración con OpenAI.
* Gmail conectado como Tool lateral del agente.
* Activación autónoma de Gmail únicamente cuando se requiere seguimiento humano.
* Log final de observabilidad mediante Gmail.
* Pruebas con casos donde la Tool se activa y casos donde no se utiliza.

## Variables principales

```json
{
  "trace_id": "identificador de ejecución",
  "session_id": "identificador de sesión",
  "channel": "chat",
  "user_name": "Usuario",
  "message": "mensaje recibido"
}
```

Estas variables se mantendrán durante la evolución del proyecto para facilitar la integración futura de memoria, sub-workflows y trazabilidad.

## Estructura del repositorio

```text
modulo1/
    checkpoint1_lucas_taranto.json

README.md
```

En los próximos checkpoints se incorporarán nuevos módulos manteniendo y extendiendo la arquitectura existente.

## Roadmap del proyecto

* Módulo 1: Agente base y Tools Agent.
* Módulo 2: Arquitectura Manager-Worker y sub-workflows.
* Módulo 3: Memoria persistente con Session_ID.
* Módulo 4: Integraciones empresariales.
* Módulo 5: Base documental y RAG.
* Módulo 6: Voice AI.
* Módulo 7: Especialización vertical de negocio.
* Módulo 8: Supervisor AI-as-a-Judge.
* Módulo 9: Gobernanza, métricas, trazabilidad y ROI.
* Módulo 10: Propuesta técnico-comercial.
* Módulo 11: Proyecto Final Integrador.

## Seguridad y control

La primera versión ya incorpora controles básicos que se mantendrán durante toda la evolución del sistema:

* límite máximo de iteraciones;
* restricciones explícitas dentro del System Prompt;
* uso de herramientas únicamente cuando existe una necesidad operativa;
* destinatarios de Gmail definidos previamente;
* separación entre razonamiento y ejecución de acciones;
* trazabilidad mediante `trace_id`;
* supervisión mediante logs.

Las credenciales, API Keys y tokens privados no se almacenan dentro de este repositorio.

## Checkpoint 1

Archivo entregable:

`modulo1/checkpoint1_lucas_taranto.json`

El workflow fue validado mediante pruebas manuales en n8n, incluyendo:

1. Un caso comercial que activa de forma autónoma la herramienta Gmail.
2. Un caso informativo donde el agente decide no utilizar la herramienta.
3. Envío correcto del log de observabilidad al finalizar la ejecución.
