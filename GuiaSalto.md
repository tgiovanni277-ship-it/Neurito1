## Experimentos de Neurito 2.0: salto de obstáculos

Esta sección registra las pruebas de la guía `neurito/GUIA_SALTO.md`, ejecutadas sobre `neurito/salto_modelo.py` y la física de `neurito/salto_juego.py`.

### Método

Se mantuvieron semillas fijas: inicialización de la red `42`, datos de entrenamiento `2026` y datos de prueba `2027`. Cada corrida usó 4,000 ejemplos de entrenamiento, 2,000 de prueba y entrenamiento por lotes completos. La velocidad se generó uniformemente entre 4.5 y 8.5 px/frame; la etiqueta se definió con el tiempo hasta el obstáculo (distancia / velocidad). Se reporta entropía cruzada binaria (BCE) y precisión sobre datos sintéticos de prueba.

Para el juego, se ejecutó `JuegoSalto` sin ventana gráfica y se consultó la red en cada fotograma. Cada configuración recorrió 250 obstáculos: cinco secuencias deterministas de 50 obstáculos, con semillas 100–104. Los choques son una medida de esta simulación acotada, no una garantía para cualquier secuencia.

### 1. Entrenamiento gradual

Se usó la configuración base: 12 neuronas ocultas, tasa `0.8` y ventana de etiqueta de 7–13 fotogramas. Los registros son puntos del mismo entrenamiento, no corridas distintas.

| Épocas | BCE entrenamiento | BCE prueba | Precisión prueba | Choques / 250 |
| ---: | ---: | ---: | ---: | ---: |
| 0 (sin entrenar) | 0.71191 | 0.70952 | 45.25% | 250 |
| 100 | 0.26567 | 0.25279 | 91.90% | 15 |
| 200 | 0.23643 | 0.22311 | 92.70% | 2 |
| 300 | 0.21353 | 0.20061 | 93.00% | 0 |
| 500 | 0.14854 | 0.13819 | 94.20% | 0 |
| 1000 | 0.06073 | 0.05536 | 98.35% | 0 |

Hubo 15 choques en la prueba de 100 épocas, 2 en la de 200 y ninguno desde 300. La precisión siguió creciendo después de que el simulador ya registraba cero choques; ambas métricas describen aspectos distintos.

### 2. Tamaño de la capa oculta

Se compararon 3, 12 y 24 neuronas, manteniendo tasa `0.8` y 1,000 épocas.

| Neuronas ocultas | BCE entrenamiento | BCE prueba | Precisión prueba | Choques / 250 |
| ---: | ---: | ---: | ---: | ---: |
| 3 | 0.20611 | 0.19316 | 94.35% | 0 |
| 12 | 0.06073 | 0.05536 | 98.35% | 0 |
| 24 | 0.05247 | 0.04792 | 98.50% | 0 |

La capa de 24 mejoró la precisión solo 0.15 puntos porcentuales frente a 12 y no redujo los choques observados. Tres neuronas clasificaron peor, aunque también evitaron los 250 obstáculos de estas corridas. Más capacidad mejoró la clasificación sintética, pero no mostró una ventaja adicional en la pista medida.

### 3. Tasa de aprendizaje

Con 12 neuronas y 1,000 épocas, se compararon las tasas `0.01`, `0.8` y `1.2`.

| Tasa | BCE entrenamiento | BCE prueba | Precisión prueba | Choques / 250 |
| ---: | ---: | ---: | ---: | ---: |
| 0.01 | 0.43795 | 0.43225 | 87.15% | 250 |
| 0.8 | 0.06073 | 0.05536 | 98.35% | 0 |
| 1.2 | 0.04106 | 0.03734 | 98.55% | 0 |

La tasa `0.01` avanzó poco en 1,000 épocas y el agente chocó en todas las pruebas de pista. `1.2` obtuvo la menor pérdida y la mayor precisión de estos valores, con el mismo resultado de pista que `0.8`. Es el mejor valor entre los tres ensayados, no una tasa universalmente óptima.

### 4. Ventana de salto

Se cambiaron las etiquetas para que la clase positiva cubriera los intervalos indicados; los ejemplos se regeneraron con las mismas semillas. La ventana original del programa es 7–13.

| Ventana TTC (fotogramas) | BCE entrenamiento | BCE prueba | Precisión prueba | Choques / 250 |
| ---: | ---: | ---: | ---: | ---: |
| 4–10 | 0.12168 | 0.12708 | 95.75% | 0 |
| 7–13 (base) | 0.06073 | 0.05536 | 98.35% | 0 |
| 10–16 | 0.04991 | 0.05197 | 98.10% | 21 |

La ventana 10–16 tuvo 98.10% de precisión de prueba y, aun así, chocó en 21 de 250 obstáculos. La precisión mide si la red reproduce las etiquetas sintéticas; el juego también depende de cuándo despega Neurito, cuánto dura el salto, la velocidad y el solapamiento entre las figuras. Una regla de etiqueta puede conservar una precisión alta y producir una política menos compatible con la física.

### Conclusión

Una precisión alta no basta para afirmar que el agente jugará sin choques. También hay que evaluar el comportamiento en la simulación, con distintas velocidades y semillas. En estas corridas, las ventanas 4–10 y 7–13 no tuvieron choques; la ventana 10–16 mostró que una clasificación precisa puede fallar en la pista. Las cifras describen las semillas y los 250 obstáculos probados; más secuencias podrían cambiar los conteos.
