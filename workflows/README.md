# Workflows de n8n

Cada workflow debe quedar en su propia carpeta con el nombre descriptivo y la exportación JSON original:

```text
workflows/
  chatbot/
    JeriGo - Chat WhatsApp.json
  dashboards/
    JeriGo - Dashboard de estado.json
    JeriGo - Panel de supervisión comercial.json
  rag-ingestion/
    JeriGo - Ingesta SharePoint.json
  error-detector/
    jerigo-error-monitoring.json
```

Mantener los archivos JSON como exportaciones de n8n facilita su importación y evita mezclar la lógica de ejecución con la documentación. Si se agregan notas operativas, ubicarlas en `docs/` o junto al workflow, sin editar la estructura del export.

## Workflows incorporados

- `chatbot/JeriGo - Chat WhatsApp.json`: chatbot de 31 nodos. Aunque el nombre dice WhatsApp, el export tiene un `Telegram Trigger`.
- `dashboards/JeriGo - Dashboard de estado.json`: panel web de estado y ejecuciones de n8n, 9 nodos; inactivo en el export.
- `dashboards/JeriGo - Panel de supervisión comercial.json`: panel de consultas y vendedores, 4 nodos; inactivo en el export.
- `rag-ingestion/JeriGo - Ingesta SharePoint.json`: ingesta e indexación documental de 35 nodos; está inactivo en el export.
- `error-detector/jerigo-error-monitoring.json`: monitoreo y notificación de errores, 5 nodos.

Configura las credenciales requeridas directamente en tu instancia n8n antes de activar o ejecutar los workflows. Los archivos conservan las referencias de credenciales del export original.

Las tablas, operaciones y funciones de base de datos referenciadas por los exports están inventariadas en `../docs/supabase.md`. Verifica ese inventario contra el esquema desplegado antes de importar; no contiene credenciales ni reemplaza una migración SQL.

Para experimentar con cambios sin tocar estos exports, importa los cuatro JSON de `../jerigo-test/workflows/rag-ingestion/` en un proyecto n8n separado y sigue las pautas de `../jerigo-test/README.md`.
