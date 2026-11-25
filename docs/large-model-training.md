# Entrenamiento de modelos grandes: continuación 1k → 11k

## Objetivo

Continuar los dos checkpoints grandes desde el paso global 1.000 hasta el 11.000, con dos jobs independientes en GitHub Actions. Cada job realiza exactamente 10.000 actualizaciones adicionales y usa el mismo corpus V4 y tokenizer congelados.

## Padres inmutables

Origen: [run 45641829810](https://github.com/orbitalronin94/LUMEN-WISE/actions/runs/45641829810).

| Modelo | Artefacto | Paso | Parámetros | Estado |
|---|---|---:|---:|---|
| LUMEN | `lumen-wise-eager-delta1-fused-4x2-lean-step1000-45641829810` | 1.000 | 11.231.405 | model, AdamW, scheduler y RNG presentes |
| Transformer | `lumen-wise-transformer-x5-4x2-lean-step1000-45641829810` | 1.000 | 12.022.016 | model, AdamW, scheduler y RNG presentes |

Ambos padres comparten:
- `data_sha256=b5296e0976a7c02536dcab08796103c8ebb17da501ce80451410110c45a03692`
- `tokenizer_sha256=d7869c11809a28085d5156638086c4833a1dcbacfda4a60b69ec71e26ed3fefe`
- `tokens_seen=972174`
- batch 8, longitud 128, seed 42 y LR objetivo 3e-5.

## Corpus congelado

Release pública de `WISE-RONIN`:
`corpus-v4.0.0-public-20261125`

- 2.293.883 filas y 24 shards.
- SHA-256 del ZIP: `535ae5a1c067f30a93e1ad20e0a8b7e332df5b14558d1dfac7a3e44193de3e9a`.
- El job de preparación comprueba manifiesto, autorización de entrenamiento, hash del tokenizer y checksum de cada shard antes de publicar el artefacto que consumen ambos entrenamientos.
- No se vuelve a tokenizar el corpus ni se descarga una variante mutable.

## Configuración de ejecución

### Común

- GitHub-hosted Ubuntu 22.04, CPU.
- 4 threads intra-op / 2 inter-op.
- batch 8, max length 128, seed 42.
- LR 3e-5, transición del scheduler desde el checkpoint exacto del paso 1.000 hasta el 11.000.
- 10.000 updates: global steps 1.001–11.000.
- Comprobación completa de parámetros y estado del optimizador cada 500 pasos.
- Checkpoint nominal cada 1.000 pasos.
- Transformer registra métricas cada 100 pasos.

### LUMEN

- Eager; sin `torch.compile`.
- Delta parallel refinements = 1.
- Fused homeostasis scatter activado.
- CPU profile 4/2.
- Órganos y ganancias originales preservados: dolor response 0.2, placer response 0.5, pain budget gain 0.15 y pleasure budget gain 0.15.
- Plasticidad custom y demás configuración del perfil original preservadas.
- Logs cada 100 pasos.
- Checkpoint de recuperación `latest_good.pt` cada 250 pasos.
- Checkpoint nominal cada 1.000 pasos.

### Transformer

- Arquitectura verificada: `decoder_transformer_4l_d488_h8_ff1569_rg51_tied_v2`.
- model_dim 488, 8 heads, FF width 1569, 51 residual groups, 4 capas y embeddings tied.
- CPU 4/2; logs cada 100 pasos desde el trainer.
- Checkpoint nominal cada 1.000 pasos.

## Incidencia y corrección del intento anterior

Run de continuación: [45645614244](https://github.com/orbitalronin94/LUMEN-WISE/actions/runs/45645614244).

La preparación del corpus y la verificación de ambos checkpoints terminaron correctamente. El entrenamiento LUMEN falló antes de la primera actualización, aunque el padre, sus órganos y el perfil de plasticidad habían pasado el preflight. La causa fue una precedencia incorrecta en `train/train.py`:

1. El workflow pasaba `--total-steps 11000`.
2. También cargaba el `--model-config` original, cuyo `total_steps` era 1000.
3. El bloque de carga de configuración sobrescribía incondicionalmente los valores de CLI con los valores del JSON.
4. El validador curricular veía por tanto un horizonte de 1.000 y rechazaba el tramo 1.001–11.000.

**Corrección aplicada:** `train/train.py` detecta qué opciones fueron declaradas explícitamente en CLI. `--model-config` sigue definiendo la arquitectura y los valores por defecto, pero no puede sobrescribir `--total-steps`, `--lr`, `--batch`, `--warmup` ni `--seed` cuando se proporcionan explícitamente. Se mantiene el mismo checkpoint padre y el mismo estado de AdamW; no se modifica ni se migra el modelo.

El Transformer anterior alcanzó el paso 1.300 y mostró logs de entrenamiento, pero su job terminó cancelado antes de producir un nuevo checkpoint nominal. La nueva ejecución vuelve a partir del checkpoint validado de 1.000, evitando depender de un estado intermedio no guardado.

## Validación de salida

Cada job debe:
1. finalizar en el paso global 11.000;
2. validar el número de parámetros y la arquitectura;
3. validar los hashes de datos y tokenizer;
4. confirmar presencia de pesos, estado AdamW, scheduler y RNG;
5. publicar checkpoints y logs como artefactos independientes.

No se declara éxito por el mero hecho de que el proceso de entrenamiento haya arrancado. La conclusión solo se considera válida tras verificar el checkpoint 11.000 y su artefacto.

## Evaluación posterior

La evaluación de calidad es una etapa separada. Se deben informar loss, exactitud/NLL de elección múltiple y generación abierta (vacíos, longitud y repetición). Una reducción de loss no se interpretará por sí sola como mejora conversacional.
