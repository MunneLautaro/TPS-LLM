# Informe de cierre - TP4

## Objetivo

Se implementó un pipeline RAG para soporte de e-commerce. El sistema carga el manual de políticas, lo fragmenta, genera embeddings, recupera los fragmentos más relevantes y los inyecta en el prompt del modelo para producir una respuesta JSON validada con Pydantic.

## Implementación

- Documento utilizado: `politicas_ecommerce_integral.md`.
- Fragmentación estructural y por tamaño fijo.
- Estrategia seleccionada: 100 tokens con 10% de solapamiento.
- Vector store: FAISS `IndexFlatIP`.
- Embeddings: `paraphrase-multilingual-MiniLM-L12-v2`, normalizados para similitud coseno.
- Recuperación configurable con `k` entre 2 y 4.
- Prompt con contexto dinámico y salida JSON estricta.
- `politica_aplicada` incorpora el fragmento recuperado con mayor solapamiento léxico con la consulta, evitando citar siempre el primer resultado.
- El costo se calcula con el tokenizer `cl100k_base`, en lugar de contar palabras.

## Evaluaciones

Se ejecutaron las 9 combinaciones solicitadas:

- Tamaños: 10, 100 y 1000 tokens.
- Solapamientos: 0%, 10% y 80%.
- La configuración de 100 tokens y 10% produjo 29 fragmentos y 2.386 tokens indexados, ofreciendo el mejor equilibrio entre continuidad y costo.

También se evaluó `k=2`, `k=3` y `k=4`:

- `k=2`: 249,3 tokens promedio; menor costo, pero menor cobertura.
- `k=3`: 358,3 tokens promedio; mejor equilibrio entre cobertura y ruido.
- `k=4`: 477,3 tokens promedio; mayor cobertura, pero menor similitud promedio y más ruido.

Se seleccionó `k=3` para el pipeline final.

Además, se incorporó una evaluación empírica para las 9 configuraciones de chunking:
se mide `Recall@3` sobre consultas con sección y término crítico conocidos, junto
con los tokens totales indexados. También se reporta el porcentaje de ruido del
Top-4 respecto de la sección esperada.

## Casos borde

- CB-1: cafetera defectuosa a los 45 días. Resultado: `ACC_CREAR_TICKET_GARANTIA`.
- CB-2: televisor dañado por fluctuación eléctrica. Resultado: `ACC_RECHAZAR_SOLICITUD`.
- CB-3: nota de crédito solicitada después de 25 días hábiles. Resultado: `ACC_RECHAZAR_SOLICITUD`.

Los tres casos recuperaron evidencia textual de las secciones correspondientes y pasaron las validaciones automáticas.

Se agregó un caso de grounding en el que el usuario afirma una política de
reembolso inexistente. La respuesta se valida contra el contexto oficial y no
puede seleccionar `ACC_REEMBOLSO` basándose únicamente en la afirmación del usuario.
