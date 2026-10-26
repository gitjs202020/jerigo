# Instrucciones para agentes

## Contexto del repositorio

Este repositorio organiza exports de workflows de n8n y documentación del asistente JeriGo. Antes de cambiar un workflow, consulta `README.md` y `workflows/README.md`; para la copia de pruebas de ingesta, consulta también `jerigo-test/README.md` y las instrucciones locales en `jerigo-test/workflows/rag-ingestion/AGENTS.md`.

## Límites y cuidados

- Mantén separados los workflows de `workflows/` (fuente principal) y las copias de prueba de `jerigo-test/`. No promociones cambios de prueba a producción ni alteres producción para experimentar.
- Los archivos JSON son exports de n8n. Haz cambios puntuales y conserva IDs, conexiones, parámetros y formato ajenos al cambio; evita reserializar el export completo.
- No agregues credenciales, tokens, claves, datos personales ni documentos reales. Usa placeholders y fixtures anonimizados.
- No actives ni ejecutes workflows contra servicios externos o datos reales sin una solicitud explícita.
- Si un cambio altera comportamiento, dependencias o requisitos de importación, actualiza la documentación de la carpeta correspondiente y distingue claramente lo probado de lo pendiente.
- Valida que cada JSON modificado siga siendo JSON válido y revisa que sus referencias de nodos y conexiones sean coherentes. No afirmes que una integración externa funciona si no se verificó en un entorno autorizado.
