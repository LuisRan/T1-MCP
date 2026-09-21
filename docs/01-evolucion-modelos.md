# 1. Evolución de los modelos

## Qué es un modelo de lenguaje (LM)

Un **modelo de lenguaje** es un modelo estadístico que asigna probabilidades a secuencias de texto. Dada una secuencia de palabras (o fragmentos de palabras llamados *tokens*), estima cuál es la probabilidad de que venga cada posible token siguiente. Con eso puede completar frases, traducir, resumir o responder preguntas, porque todas esas tareas se pueden plantear como "predecir el texto que sigue".

## Cómo evolucionó hacia un LLM

| Etapa | Idea principal | Limitación |
|---|---|---|
| **Modelos de n-gramas** | Cuentan cuántas veces aparece una palabra después de las *n − 1* anteriores. | Solo ven un contexto muy corto y no generalizan bien a combinaciones que no vieron. |
| **Redes neuronales recurrentes (RNN, LSTM)** | Procesan el texto en orden y guardan un estado interno que resume lo leído. | Difíciles de entrenar en paralelo y con memoria débil en textos largos. |
| **Transformer (2017)** | Sustituye la recurrencia por el mecanismo de *atención*, que permite relacionar cualquier posición del texto con cualquier otra y entrenar en paralelo (Vaswani et al., 2017). | Requiere mucho cómputo, pero escala muy bien. |
| **Preentrenamiento a gran escala** | Se entrena un Transformer con enormes cantidades de texto para predecir el siguiente token y luego se adapta a tareas concretas. | Un modelo preentrenado solo "continúa texto"; no siempre sigue instrucciones. |
| **LLM con ajuste de instrucciones** | Se refina con ejemplos de instrucciones y con retroalimentación humana (Ouyang et al., 2022) para que el modelo siga lo que se le pide. | Sigue sin actuar sobre el mundo: solo produce texto. |

Un **modelo de lenguaje grande (LLM)** es, en esencia, un modelo de lenguaje basado en Transformer con muchísimos parámetros, entrenado con cantidades masivas de datos. El aumento de tamaño, de datos y de cómputo mejora el rendimiento de forma bastante predecible (Kaplan et al., 2020), y con cierta escala aparecen capacidades que los modelos pequeños casi no muestran, como seguir instrucciones complejas o escribir código.

## Modelos con razonamiento explícito

Un **modelo con razonamiento explícito** genera, antes de dar su respuesta final, una secuencia intermedia de pasos (una "cadena de pensamiento") en la que descompone el problema, prueba caminos, se corrige y verifica. Ya se había observado que pedirle al modelo que razonara paso a paso mejoraba su desempeño (Wei et al., 2022).

**Esa capacidad no aparece sola por aumentar el tamaño del modelo.** Viene de dos ingredientes adicionales:

1. **Técnicas de entrenamiento.** Se entrena al modelo, sobre todo con aprendizaje por refuerzo, para que sus cadenas de razonamiento lleguen a respuestas correctas y verificables, por ejemplo en matemáticas o programación. Trabajos como DeepSeek-R1 documentan que este tipo de entrenamiento hace surgir comportamientos como la reflexión y la verificación (DeepSeek-AI, 2025).
2. **Cómputo adicional en el momento de la inferencia.** Al responder, el modelo gasta más pasos y más tokens "pensando". Se ha mostrado que usar de forma óptima el cómputo en inferencia puede rendir más que simplemente usar un modelo mucho más grande (Snell et al., 2024).

En resumen: **tamaño + entrenamiento específico + tiempo de cómputo al responder**. Un modelo grande sin ese entrenamiento no razona de forma explícita solo por ser grande.

## Por qué importa para este trabajo

Ni un LM ni un LLM, ni siquiera uno con razonamiento, actúan sobre el mundo: reciben texto y devuelven texto. Para que puedan leer o modificar archivos hace falta una capa adicional, y ahí es donde entra el Model Context Protocol (ver [`02-aislamiento.md`](02-aislamiento.md)).

## Referencias

DeepSeek-AI. (2025). *DeepSeek-R1: Incentivizing reasoning capability in LLMs via reinforcement learning*. arXiv. https://arxiv.org/abs/2501.12948

Kaplan, J., McCandlish, S., Henighan, T., Brown, T. B., Chess, B., Child, R., Gray, S., Radford, A., Wu, J., & Amodei, D. (2020). *Scaling laws for neural language models*. arXiv. https://arxiv.org/abs/2001.08361

Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., Schulman, J., Hilton, J., Kelton, F., Miller, L., Simens, M., Askell, A., Welinder, P., Christiano, P., Leike, J., & Lowe, R. (2022). *Training language models to follow instructions with human feedback*. arXiv. https://arxiv.org/abs/2203.02155

Snell, C., Lee, J., Xu, K., & Kumar, A. (2024). *Scaling LLM test-time compute optimally can be more effective than scaling model parameters*. arXiv. https://arxiv.org/abs/2408.03314

Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). Attention is all you need. *Advances in Neural Information Processing Systems, 30*. https://arxiv.org/abs/1706.03762

Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., Chi, E., Le, Q., & Zhou, D. (2022). Chain-of-thought prompting elicits reasoning in large language models. *Advances in Neural Information Processing Systems, 35*. https://arxiv.org/abs/2201.11903
