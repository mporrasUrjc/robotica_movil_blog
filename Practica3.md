# Práctica 3: Navegación local esquivando obstáculos con algoritmo VFF

## Objetivos de la práctica

Conseguir que el Fórmula 1 recorra el circuíto en el menor tiempo posible, evitando obstáculos y siguiendo los subobjetivos que le proporciona el sistema de GPS. 

## Desarrollo de la práctica

La idea base de esta práctica es implementar una navegación local por medio de VFF. Con los datos del láser hay que construir los vectores de las fuerzas virtuales que afectan al coche, obtener el vector resultante y transformar esa información en velocidad lineal y angular que poder mandar a los actuadores. Al sacar las coordenadas de cada subobjetivo, hay que transformalas a coordenadas relativas al robot, que será durante toda la ejecución del programa punto de referencia del eje de coordenadas. Así mismo, el eje de coordenadas que usaremos será el siguiente:
```
                                                    ^  x
                                                    |
                                                    |
                                                    |
                                                    |
                                        < - - - - - R 
                                        y
```
Este eje es el que usa el láser, así que las coordenadas del obstáculo no hace falta transformarlas. La mejor opción según la documentación para obtener este vector, es asignar a cada rayo un vector de repulsión propio inversamente proporcional a la magnitud de la distancia detectada. Una vez obtenidos todos los vectores, se suman para obtener el resultante. 

Antes de eso, probé simplemente construyendo el vector a partir del rayo que representase la menor distancia con el coche, pero no me dió resultados óptimos. Esto se debe a que en cada iteración el rayo más cercano cambiaba muy facilmente, lo que introducía muchas oscilaciones a la hora de controlar. Aunque la forma definitiva sea mejor, sigue teniendo problemas para esquivar obstáculos que se encuentran alineados con el vector del subobjetivo. Para solucionar este problema, he implementado una función que, en base al espacio libre a los lados del obstáculo, elije si girar el coche hacia a la izquierda o a la derecha.

Con esto aclarado, los pasos que se repiten en cada iteración son:
1. Obtener las coordenadas absolutas del siguiente objetico
2. Obtener las poses del robot y los obstáculos
3. Poner todo en coordenadas relativas al robot
4. Obtener el vector resultante y el ańgulo
5. Transformar todo a velocidad angular y lineal

Además de esto, he añadido un controlador proporcional para la velocidad angular para mejorar los giros y he representado en la pantalla los vectores y cada subbobjetivo. 

<img width="244" height="490" alt="image" src="https://github.com/user-attachments/assets/9c170de9-fcb7-4630-88a0-4333a5f3e748" />

Para saber cuánto tiempo tarda el coche en recorrer todos los checkpoints he añadido un reloj que muestra una traza por terminal del tiempo global. Solo se muestra cada vez que se alcanza un checkpoint para no saturar la salida por terminal y no introducir al programa una latencia innecesaria. En base a las pruebas que he realizado, el tiempo más bajo que he obtenido del primer al último checkpoint ha sido de `129.60 segundos`. A continuación dejo el vídeo demostración del código final:

[Ver vídeo](https://docs.google.com/videos/d/1VQccTLerXHjtfgayNis3-4sP5dYlaWIi_TQXfLO_uJg/play?usp=sharing)

