# Dependencias de base de datos

Este inventario se obtuvo de los exports de n8n del repositorio. Describe los nombres de tablas, columnas y funciones que aparecen en los nodos; **no es una introspección de la instancia Supabase ni un DDL verificado**. Los tipos marcados como visibles son los que el export usa o declara en el mapeo de n8n, y deben contrastarse con el esquema real antes de migrar o desplegar.

No guardar aquí credenciales, URLs con claves, tokens, datos de usuarios ni volcados de filas. Los nombres de credenciales de n8n son referencias locales, no valores incluidos en esta documentación.

## Migración MVP

`supabase/migrations/20261026180000_jerigo_mvp.sql` crea los objetos inferibles de los exports: control de documentos (producción y prueba), tablas vectoriales y sus funciones de búsqueda, logs, feedback y runtime. Usa `CREATE ... IF NOT EXISTS` y no modifica tablas existentes; si ya existen objetos con esos nombres, verifica que su definición coincida porque la migración no los altera. Aplicala primero en un proyecto Supabase de testing, desde Supabase CLI o SQL Editor, después de revisar el archivo.

La dimensión `1536` coincide con la configuración explícita del embedding OpenAI de producción. Para los procesadores de prueba, el export omite el modelo y la dimensión; la migración asume el modelo predeterminado de 1536 dimensiones. Verifica que el modelo configurado en n8n produzca vectores de esa dimensión antes de indexar. Los índices usan similitud coseno.

Las tablas quedan con RLS habilitado y sin políticas para `anon`/`authenticated`; se otorgan permisos a `service_role`, como credencial de API de los workflows Supabase. Los nodos PostgreSQL deben conectarse con un rol autorizado que pueda leer y escribir estas tablas. No uses la clave `service_role` en clientes ni en código público. La migración asume que las operaciones PostgreSQL de los workflows y las tablas Supabase se aprovisionan en el mismo proyecto/base; si usan bases distintas, ejecuta el DDL PostgreSQL en la base correspondiente.

La migración **no crea ni altera** `jerigo_usuarios` ni `documents`: los exports no aportan su esquema completo y pueden existir datos/tablas previos. El chatbot, el panel comercial y algunos nodos del workflow principal dependen de esos recursos; confirmá su esquema antes de usar esos workflows. El nodo Postgres Chat Memory puede inicializar su propia tabla al ejecutarse; no se inventa una tabla de memoria aquí.

## Supabase: tablas y búsqueda vectorial

| Entorno/uso | Tabla | Columnas observadas | Operación o dependencia |
|---|---|---|---|
| Chatbot y documentos principales | `jerigo_usuarios` | `telegram_chat_id`, `estado`, `canal`; el panel también lee `nombre` y `area` | Busca usuarios activos del canal Telegram; el panel lista usuarios de ese canal. |
| Ingesta principal | `document_control` | `google_file_id`, `file_name`, `hash` | Consulta por ID de Drive, registra archivos y actualiza el hash. |
| Ingesta principal | `document_embeddings_openai` | No detalladas por el export | Vector store de inserción y recuperación. |
| Ingesta principal | `match_documents` | No detalladas por el export | Nombre de función de búsqueda configurado en el vector store; verificar la firma SQL y la dimensión del vector. |
| Ingesta de prueba | `document_control_test` | `google_file_id`, `file_name`, `hash` | Tabla de control independiente para la copia de prueba. |
| Ingesta de prueba | `document_embeddings_test` | No detalladas por el export | Vector store de inserción de los tres procesadores de prueba. |
| Ingesta de prueba | `match_documents_test` | No detalladas por el export | Nombre de función de búsqueda configurado para el vector store de prueba; verificar la firma SQL y la dimensión del vector. |
| Ingesta principal | `documents` | No detalladas por el export | Hay nodos Supabase de actualización y eliminación que apuntan a esta tabla; verificar su conexión y uso efectivo en la instancia. |

Los exports de vector store fijan los nombres de tabla y función, pero no describen el esquema completo de embeddings, el tipo/dimensión del vector, índices, políticas RLS ni firma de RPC. No deducirlos únicamente de este inventario.

## PostgreSQL: tablas usadas por los workflows

Los nodos PostgreSQL de n8n referencian estas tablas. El export no demuestra si sus credenciales apuntan al mismo proyecto Supabase que los nodos Supabase; el MVP asume una sola base y esto debe verificarse.

| Tabla | Columnas observadas | Uso |
|---|---|---|
| `public.chat_logs` | `id`, `session_id`, `pregunta`, `respuesta`, `documentos_rag`, `tokens_input`, `tokens_output`, `modelo`, `fecha` | Registra preguntas y respuestas; el panel comercial consulta los últimos 30 días y enlaza feedback por `chat_log_id`. |
| `public.feedback` | `id`, `session_id`, `pregunta`, `respuesta`, `calificacion`, `comentario`, `fecha`, `chat_log_id`, `esperando_comentario` | Guarda calificación/comentario y permite buscar feedback pendiente por sesión. |
| `public.workflow_runtime` | `id`, `chat_id`, `mensaje`, `workflow_name`, `created_at` | Guarda datos de ejecución; el detector de errores obtiene el `chat_id` más reciente. |

Los tipos de las columnas PostgreSQL se deben confirmar en la base real. El mapeo de n8n muestra algunos tipos esperados: por ejemplo, `id` y `chat_log_id` numéricos, `esperando_comentario` booleano y `documentos_rag` objeto. El SQL del panel además ordena por `feedback.id`, filtra por `chat_logs.fecha` y usa `feedback.chat_log_id`.

El chatbot también usa el nodo **Postgres Chat Memory**, con el ID de sesión derivado del usuario de Telegram y una ventana de contexto de 1. El nombre y el esquema de almacenamiento de ese nodo no están definidos en el export y no se documentan aquí como una tabla propia.

## Separación de entornos y limitaciones

- Mantener `document_control` / `document_embeddings_openai` / `match_documents` separados de `document_control_test` / `document_embeddings_test` / `match_documents_test`.
- La documentación de prueba indica limpiar manualmente los vectores de un documento antes de reingestarlo si cambió. El hash evita reprocesar archivos sin cambios, pero no elimina por sí solo los chunks de la versión anterior.
- El dashboard de estado consulta la API de n8n y no requiere tablas Supabase según su export. El panel de supervisión comercial sí ejecuta SQL PostgreSQL de solo lectura sobre `jerigo_usuarios`, `feedback` y `chat_logs`.
- Antes de habilitar workflows, verificar roles, permisos, RLS, índices, claves foráneas, tipos, funciones RPC y recursos por entorno directamente en Supabase/PostgreSQL.
- La migración cubre los objetos definidos en su sección MVP; no es una réplica completa de la base desplegada ni una migración de datos existentes.

## Datos necesarios para completar el esquema real

Para convertir este inventario en documentación del esquema desplegado, agregar una salida anonimizada del catálogo que incluya nombres de esquema/tabla, columnas y tipos, claves y restricciones, índices, políticas RLS y firmas de `match_documents` y `match_documents_test`. No compartir secretos, cadenas de conexión ni filas de datos.
