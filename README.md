# LUMEN-WISE

Repositorio dedicado a las arquitecturas grandes de la familia LUMEN-WISE y sus controles experimentales.

## Modelos grandes

| Modelo | Arquitectura | Parámetros verificados |
|---|---|---:|
| LUMEN-WISE x5 | Ronin Lumen v3.3 con neuroplasticidad | 11.231.405 |
| Transformer x5 | Decoder Transformer 4 capas, d=488, 8 heads, FF=1569, 51 residual groups, embeddings tied | 12.022.016 |

Los conteos corresponden a los checkpoints reales de paso 1.000. No deben normalizarse ni presentarse como si ambos modelos tuvieran el mismo número de parámetros.

## Reglas experimentales

- CPU-first, dos jobs independientes y paralelos, cada uno con 4 threads intra-op y 2 inter-op.
- Conservar arquitectura, órganos, plasticidad, memoria, homeostasis y estados del optimizador.
- Reanudar desde checkpoints exactos; no reinicializar pesos ni AdamW para alargar una ejecución.
- Corpus congelado, manifiesto, shards y tokenizer verificados por SHA-256.
- Logs de entrenamiento cada 100 pasos; escaneo completo de parámetros y estado del optimizador cada 500; checkpoints nominales cada 1.000.
- LUMEN añade checkpoint de recuperación cada 250 pasos.
- Separar fallos de preflight, código, entrenamiento, evaluación y runner. Un fallo de verificación posterior no invalida por sí solo un checkpoint que ya se haya guardado y validado de forma independiente.

## Continuación 1k → 11k

La continuación oficial parte de los artefactos del run `45641829810`:

- LUMEN: paso 1.000, 11.231.405 parámetros, AdamW/scheduler/RNG preservados.
- Transformer: paso 1.000, 12.022.016 parámetros, AdamW/scheduler/RNG preservados.
- Corpus V4 público y congelado, 2.293.883 filas, tokenizer y datos identificados por SHA-256.
- Objetivo: 10.000 actualizaciones adicionales, pasos globales 1.001–11.000.

**Corrección importante del trainer LUMEN:** el archivo de arquitectura del padre contiene `total_steps=1000`. El trainer ahora trata `--model-config` como fuente de arquitectura y valores por defecto, pero respeta los argumentos explícitos de CLI (`--total-steps`, `--lr`, `--batch`, `--warmup`, `--seed`). Así, el horizonte 11.000 solicitado no vuelve a ser sustituido silenciosamente por 1.000.

La especificación operativa, las huellas de corpus/checkpoints y el historial de incidencias están en [docs/large-model-training.md](docs/large-model-training.md).

## Evaluación

La loss de entrenamiento no sustituye al holdout. Se informarán por separado:
- loss y estabilidad numérica;
- exactitud/NLL en elección múltiple;
- respuestas abiertas, respuestas vacías y repetición;
- comparación emparejada entre LUMEN y Transformer.

