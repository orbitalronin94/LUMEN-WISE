# LUMEN-WISE

Repositorio dedicado a las arquitecturas grandes de la familia LUMEN-WISE y sus controles experimentales.

## Alcance

- LUMEN-WISE x5: 12.022.016 parámetros.
- Transformer x5 de control: 12.022.016 parámetros.
- Código de modelo, entrenamiento, evaluación, configuraciones y pruebas.
- Workflows específicos de benchmark, entrenamiento y holdout.

La separación respecto de `WISE-RONIN` permite mantener los experimentos grandes aislados de los modelos pequeños.

## Reglas experimentales

- CPU-first; conservar la arquitectura completa.
- No reducir mecanismos ni parámetros para ocultar problemas de rendimiento.
- Entrenamientos secuenciales y checkpoints recuperables.
- Corpus congelado, manifiesto y tokenizer verificados por SHA-256.
- Logs suficientes para distinguir fallos del código, del entrenamiento y del runner.

## Estado de migración

Migración inicial en curso. Antes de iniciar entrenamientos oficiales, el corpus V4 deberá estar validado y disponible como artefacto propio de este repositorio. No se consideran migrados los checkpoints ni se dará por completada la separación hasta verificar código, configuraciones y workflows.
