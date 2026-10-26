# Instrucciones para agentes: ingesta RAG de prueba

Esta carpeta contiene una copia de testing, no los workflows principales. Lee `jerigo-test/README.md` antes de cambiar los exports y conserva esa separación.

## Arquitectura de la prueba

- El coordinador procesa archivos de una carpeta de Google Drive de prueba, omite documentos cuyo hash no cambió y enruta por tipo a tres subflujos: FAQ, coseguros y documentos generales. Aunque el workflow original se llama “Ingesta SharePoint”, el trigger de estos exports usa Google Drive.
- Los nombres de archivo deben comenzar con `FAQ__`, `COSEGURO__` o `GENERAL__`. Mantén el rechazo explícito de tipos desconocidos; no añadas una ruta por defecto que indexe contenido sin clasificar.
- Cada procesador conserva su extracción y fragmentación específicas. Los tres escriben embeddings en el vector store Supabase de prueba.
- Los exports deben permanecer inactivos. Los IDs de subflujos y las credenciales se configuran en la instancia de n8n de testing, no se inventan ni se vinculan a producción desde el repositorio.

## Limitaciones y validación

- El control por hash no elimina del vector store los fragmentos de una versión anterior cuando cambia un archivo. La documentación indica limpiar los vectores de ese documento antes de probar una reingesta; no describas este flujo como idempotente para actualizaciones ni cambies ese comportamiento sin diseñar y validar la limpieza.
- La configuración de pruebas usa recursos separados: `document_control_test`, `document_embeddings_test` y `match_documents_test`. No sustituyas esos recursos por los de producción.
- Si modificas el enrutamiento, el procesamiento, la indexación o esta limitación, actualiza `jerigo-test/README.md` en el mismo cambio.
- Después de editar, comprueba que el JSON sea válido y que las conexiones apunten a nodos existentes. No ejecutes ni actives los workflows contra Drive, Supabase, modelos o credenciales reales sin autorización explícita.
