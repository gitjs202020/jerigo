# JeriGo

Repositorio para organizar los workflows de n8n y la documentación del asistente JeriGo.

## Estructura

```text
workflows/
  README.md
  chatbot/             # Workflow conversacional del bot
  dashboards/          # Paneles de monitoreo y supervisión
  rag-ingestion/       # Ingesta e indexación documental
  error-detector/      # Detección y notificación de errores
jerigo-test/            # Diseño y pruebas aisladas de workflows
docs/                  # Documentación técnica y operativa
```

Los workflows se guardan como exportaciones JSON de n8n, separados por responsabilidad. No incluyas credenciales, tokens ni secretos en el repositorio. Antes de importar un workflow, configura sus credenciales y verifica las dependencias de n8n y de la base de datos.

El inventario de tablas y funciones que los workflows esperan encontrar está en [`docs/supabase.md`](docs/supabase.md). Es una referencia derivada de los exports, no una inspección del esquema desplegado.

La migración inicial está en `supabase/migrations/`. Revisa `docs/supabase.md` y aplícala primero en un proyecto de testing; no modifica tablas existentes.

## Estado de los archivos fuente

- Chatbot: `workflows/chatbot/JeriGo - Chat WhatsApp.json`.
- Dashboards: `workflows/dashboards/` (estado de n8n y supervisión comercial).
- Ingesta SharePoint/RAG: `workflows/rag-ingestion/JeriGo - Ingesta SharePoint.json`.
- Monitoreo de errores: incorporado en `workflows/error-detector/jerigo-error-monitoring.json`.

Antes de importar los workflows en n8n, configura sus credenciales y verifica las dependencias externas.
