# JeriGo Test

Este directorio es el espacio de trabajo para diseñar y documentar pruebas de workflows en una instancia o proyecto n8n separado. Los exports de producción viven en `../workflows/` y no deben modificarse para probar cambios.

## Organización propuesta

```text
jerigo-test/
  README.md
  workflows/
    rag-ingestion/
      coordinator/
        JeriGo Test - Coordinador.json
      processors/
        JeriGo Test - Procesador FAQ.json
        JeriGo Test - Procesador Coseguros.json
        JeriGo Test - Procesador General.json
  fixtures/
    # Documentos de ejemplo anonimizados para cada formato
```

La ingesta de prueba está separada en cuatro exports. Importa primero los tres procesadores y después el coordinador. En el coordinador, selecciona el workflow correspondiente en cada nodo `Llamar subflujo ...`; los IDs de workflow se asignan al importar en tu instancia y por eso no se pueden fijar desde este repositorio.

Todos los exports quedan inactivos y sin credenciales vinculadas. Antes de ejecutar:

- Configura en el trigger del coordinador una carpeta de Google Drive dedicada a pruebas; el ID está reemplazado por `CONFIGURE_TEST_FOLDER_ID`.
- Vincula credenciales de testing en los cuatro workflows.
- Configura las tablas de control `document_control_test`, `document_embeddings_test` y la función de búsqueda `match_documents_test`.
- No uses credenciales ni recursos de producción.

## Diseño a probar para la ingesta

No conviene forzar los tres formatos de documento a un único algoritmo de fragmentación o extracción. La copia de prueba usa un coordinador y tres subflujos:

1. **Coordinador:** procesa los archivos en un loop, descarga y extrae el texto, calcula su hash y consulta `document_control_test` por ID de Drive. Si el hash no cambió, omite el archivo; si es nuevo o cambió, lo clasifica y llama al subflujo correspondiente. Al terminar el subflujo, inserta o actualiza el registro de control.
2. **Procesador FAQ:** separa por `Pregunta:`, extrae metadatos FAQ e indexa.
3. **Procesador de coseguros:** separa por sección del índice, extrae sus metadatos propios e indexa.
4. **Procesador general:** separa por secciones y fragmentos con solapamiento, extrae metadatos generales e indexa.

Cada subflujo conserva su propio chunking y extracción, pero repite el mismo paso final de embeddings y escritura al vector store de pruebas. El export original tenía las ramas de texto general y coseguros desconectadas de la indexación; en esta copia quedan conectadas a sus procesadores.

**Limitación pendiente para reingestas:** el control por hash evita reprocesar archivos sin cambios, pero la copia aún no elimina del vector store los chunks de una versión anterior cuando un archivo existente cambia. Para probar actualizaciones, limpia manualmente en la base de testing los vectores de ese documento antes de reingestarlo. No uses la copia como una actualización idempotente hasta agregar y validar esa limpieza por `fuente`/ID de archivo.

Para que el enrutamiento sea explícito y verificable, nombra los archivos de prueba con uno de estos prefijos: `FAQ__`, `COSEGURO__` o `GENERAL__`. La clasificación ocurre antes de escribir al vector store; si no reconoce un prefijo, detiene la ejecución en vez de elegir un procesador al azar. El workflow de origen vigila Google Drive, por lo que debes configurar una carpeta de Drive de prueba aunque se llame “Ingesta SharePoint”.

## Normalización asistida por IA

Se puede agregar una etapa de normalización, idealmente antes del chunking y después de guardar el archivo original. Para que sea confiable:

- Define una representación canónica de salida y esquemas separados para documentos generales, FAQ e índices de coseguro.
- Pide a la IA que ordene o normalice únicamente el texto recibido; no que complete información ausente ni que descarte condiciones, excepciones, importes o vigencias.
- Mantén el texto original junto al normalizado y registra tipo de documento, versión de prompt/modelo y fuente.
- Valida el JSON, los campos obligatorios y reglas de negocio. Comprueba especialmente que importes, porcentajes, límites y fechas del resultado estén respaldados por el original; si falla, detén o envía a revisión, no indexes silenciosamente.
- Prueba con fixtures representativos y casos difíciles antes de conectar la salida a la tabla vectorial de producción.

Los cuatro subflujos todavía no incorporan ese paso de normalización asistida por IA. Conviene agregarlo y evaluarlo en testing después de verificar el enrutamiento y la indexación de los tres formatos.

La normalización no debería ser una “limpieza” libre: la meta es una estructura consistente y trazable sin perder significado documental.
