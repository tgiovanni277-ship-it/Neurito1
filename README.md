# Neurito

Neurito es una demostración pequeña de una red neuronal que elige una acción para una mascota según tres necesidades: hambre, energía y aburrimiento. La interfaz está hecha con Pygame; el modelo y sus datos sintéticos están en `main.py`.

## Ejecutar

Con Python 3.12 o superior y `uv`:

```bash
uv run python main.py
```

La interfaz del código está configurada en inglés (`LANGUAGE = "en"`). Para mostrarla en español, cambia ese valor a `"es"`.

Controles: `H/J` suben o bajan el hambre, `E/D` la energía, `A/S` el aburrimiento, `R` sortea las tres necesidades, `N` vuelve al estado tranquilo y `Esc` cierra la aplicación.

## Cómo decide

Cada necesidad se representa con un número entre 0 y 1. El entrenamiento crea 3000 ejemplos aleatorios y asigna como etiqueta la necesidad prioritaria:

| Salida | Puntaje usado por la etiqueta |
| --- | --- |
| Comer (`eat`) | hambre |
| Dormir (`sleep`) | `1 - energía` (cansancio) |
| Jugar (`play`) | aburrimiento |

En términos de entradas `[hambre, energía, aburrimiento]`, la etiqueta es el índice del mayor valor entre `[hambre, 1 - energía, aburrimiento]`. Por eso la acción cambia cuando el puntaje de otra necesidad supera al ganador. Si dos puntajes son iguales o casi iguales, una diferencia pequeña puede cambiar la clase ganadora. La red aproxima esa regla aprendiendo de los ejemplos.

Hay una excepción visual: cuando hambre y aburrimiento son menores que `0.4` y energía supera `0.6`, la interfaz muestra `neutral` aunque el modelo siga produciendo probabilidades para las tres acciones. `neutral` no es una cuarta clase entrenada.

## Experimentos

Para comparar configuraciones se mantuvieron los datos de entrenamiento sintéticos, semilla (`42`) y algoritmo de descenso por gradiente por lotes completos que usa el programa. Para que la comparación sea pareja, se usó un conjunto fijo de prueba de 1000 ejemplos generado aparte con semilla `123`. El código original genera la prueba desde el flujo aleatorio que queda tras inicializar los pesos; fijarla aparte evita que cambie al variar el tamaño de la capa. La pérdida es la entropía cruzada al final sobre los 3000 ejemplos de entrenamiento. Las cifras pueden variar si cambian los datos, las semillas, las etiquetas o la implementación.

### Mantener dos necesidades fijas y variar la tercera

Por ejemplo, fijamos energía en `0.50` (cansancio `0.50`) y aburrimiento en `0.10`, y variamos hambre. En el modelo base de 6 neuronas ocultas, la salida fue:

| Hambre | Prob. comer | Prob. dormir | Acción predicha |
| ---: | ---: | ---: | --- |
| 0.49 | 49.3% | 50.7% | Dormir |
| 0.50 | 66.9% | 33.1% | Comer |
| 0.52 | 89.7% | 10.3% | Comer |

La transición ocurre cerca del empate entre hambre y cansancio (`0.50` cada uno). Debido a que la red solo aproxima la regla y aprende con datos finitos, su punto de cambio no tiene por qué coincidir exactamente con el empate matemático. Con prioridades claramente separadas la acción tiende a ser más estable.

### Prioridades casi iguales y Softmax

Softmax convierte los tres puntajes de salida de la red en valores entre 0 y 1 que suman 1. Cuando dos salidas están cerca, ambas pueden recibir probabilidades altas; al inclinar ligeramente las entradas hacia una prioridad, su probabilidad sube y la de las otras baja. En el ejemplo anterior, pasar hambre de `0.49` a `0.50` invirtió la acción ganadora, y llegar a `0.52` aumentó la probabilidad de comer a `89.7%`. Las probabilidades son la confianza relativa del modelo entre sus tres opciones, no una medición directa de las necesidades ni una garantía de que la acción sea correcta.

### Tamaño de la capa oculta

Se cambió solo el número de neuronas ocultas; se conservaron 1200 épocas y tasa de aprendizaje `0.1`.

| Neuronas ocultas | Pérdida de entrenamiento | Precisión de prueba |
| ---: | ---: | ---: |
| 3 | 0.1618 | 96.3% |
| 6 (base) | 0.1225 | 99.4% |
| 10 | 0.1262 | 99.1% |
| 20 | 0.1220 | 99.0% |

Tres neuronas quedaron cortas frente a la configuración base en esta corrida. Diez y veinte neuronas no mejoraron la precisión de prueba: más capacidad no garantiza mejor generalización, especialmente cuando la regla a aprender es simple.

### Épocas y tasa de aprendizaje

En cada comparación se cambió un parámetro y se dejaron los demás en sus valores base (6 neuronas, 1200 épocas o tasa `0.1`, respectivamente).

| Cambio | Pérdida de entrenamiento | Precisión de prueba | Lectura |
| --- | ---: | ---: | --- |
| 200 épocas | 0.4984 | 91.2% | Entrenamiento insuficiente en esta corrida |
| 1200 épocas | 0.1225 | 99.4% | Mejora clara frente a 200 épocas |
| 3000 épocas | 0.0711 | 99.7% | La pérdida sigue bajando; la precisión casi se estanca |
| Tasa `0.01` | 0.7376 | 80.1% | Avance lento con 1200 épocas |
| Tasa `0.1` | 0.1225 | 99.4% | Referencia |
| Tasa `0.5` | 0.0484 | 99.8% | Mejor resultado de los valores ensayados |
| Tasa `1.0` | 0.0447 | 98.4% | La pérdida baja, pero empeora la precisión de prueba |
| Tasa `2.0` | 0.0533 | 95.9% | Resultado inferior; el paso grande perjudica el ajuste |

En estos valores, 1200 épocas ya dan una precisión alta y pasar a 3000 apenas la eleva. Una tasa demasiado baja no alcanza a ajustar bien el modelo en el mismo número de épocas; tasas altas pueden saltarse ajustes útiles y generalizar peor. Esto no identifica una tasa universalmente óptima: habría que comparar varias semillas y separar datos de validación y prueba para decidir con más confianza.

### ¿Qué haría falta para una cuarta acción?

Habría que definir qué significa esa acción y bajo qué condiciones es la respuesta correcta; recopilar ejemplos con las tres entradas (o agregar una nueva variable si la acción depende de algo que el modelo no observa) y etiquetarlos con cuatro clases, incluyendo suficientes casos representativos de la nueva acción. Después se debe cambiar la política que genera las etiquetas, el vector de salida de 3 a 4 valores, la lista de acciones, las etiquetas y la interfaz. También se necesita un conjunto de prueba con ejemplos de las cuatro clases para medir si la nueva acción se reconoce sin confundir las anteriores. Si dos situaciones idénticas en las tres entradas requieren acciones distintas, hace falta agregar información de entrada para que el modelo pueda distinguirlas.

## Explica tus resultados

### 1. ¿En qué se diferencian pesos y sesgos?

Los pesos (`W1`, `W2`) regulan cuánto influye cada entrada o neurona anterior en la siguiente capa. Los sesgos (`b1`, `b2`) se suman a la combinación ponderada y permiten desplazar el umbral de activación de una neurona. Sin sesgos, las decisiones de la red quedarían más restringidas.

### 2. ¿Por qué el cansancio es `1 - energía`?

La energía aumenta en la dirección opuesta al cansancio: mucha energía significa poco cansancio, y poca energía significa mucho cansancio. Como energía está normalizada entre 0 y 1, restarla de 1 invierte la escala en el mismo rango. Por ejemplo, energía `0.2` produce cansancio `0.8`. Es una simplificación de esta demostración, no un modelo fisiológico.

### 3. ¿Qué hacen ReLU y Softmax?

- **ReLU** (`max(0, x)`) se usa en la capa oculta: deja pasar valores positivos y convierte los negativos en cero. Añade una no linealidad que permite aprender reglas más flexibles que una transformación lineal.
- **Softmax** se usa en la salida: transforma los tres valores de la red en probabilidades relativas que suman 1. La predicción toma la clase con probabilidad más alta (`argmax`).

### 4. ¿Por qué una precisión alta puede ser engañosa?

La precisión solo cuenta cuántos ejemplos coinciden con las etiquetas. Aquí tanto los ejemplos de entrenamiento como los de prueba se generan al azar y comparten la misma política sintética, así que `99.4%` indica que la red reproduce bastante bien esa regla artificial; no demuestra que entienda las necesidades reales de una mascota. Además, una precisión agregada puede ocultar errores en clases poco frecuentes o cerca de empates. Para evaluar un caso real harían falta datos representativos, etiquetas justificadas, métricas por clase y pruebas con datos separados de los usados para ajustar el modelo.


