# Práctica 3: Navegación local esquivando obstáculos con algoritmo VFF

## Índice

- [1. Objetivos de la práctica](#1-objetivos-de-la-práctica)
- [2. Desarrollo de la práctica](#2-desarrollo-de-la-práctica)
  - [2.1. Implementación del algoritmo VFF](#21-implementación-del-algoritmo-vff)
  - [2.2. Cálculo de las fuerzas](#22-cálculo-de-las-fuerzas)
  - [2.3. Resolución de conflictos](#23-resolución-de-conflictos)
  - [2.4. Flujo de ejecución](#24-flujo-de-ejecución)
  - [2.5. Control de movimiento](#25-control-de-movimiento)
  - [2.6. Medición del tiempo de vuelta](#26-medición-del-tiempo-de-vuelta)
- [3. Resultado final](#3-resultado-final)

## 1. Objetivos de la práctica

El objetivo de esta práctica es conseguir que el Fórmula 1 complete el circuíto en el menor tiempo posible, evitando obstáculos y alcanzando los subobjetivos (checkpoints) proporcionados por el sistema de GPS. 

Para ello, se debe implementar un algoritmo de navegación local basado en `VFF` (Virtual Force Field), que permite calcular la dirección del movimiento del vehículo en base a las fuerzas virtuales de atracción y repulsión.

## 2. Desarrollo de la práctica

### 2.1 Implementación del algoritmo VFF

La idea base de esta práctica es implementar una navegación local por medio de VFF. A partir de los datos del láser, y teniendo en cuenta el siguiente checkpoint, se construyen los vectores de las fuerzas virtuales que afectan al coche. 

El vector obtenido mediante el sumatorio de las fuerzas permite determinar la dirección del coche, para después transformar esa información en velocidad lineal y angular que poder mandar a los actuadores. 

Para que los cálculos sean correctos, las coordenadas de los checkpoints deben estar expresadas en el sistema de referencia del coche. De esta manera, el propio coche cumple el papel de origen del sistema de coordenadas. 
El sistema de referencia usado es el siguiente:
```
                                                    ^  x
                                                    |
                                                    |
                                                    |
                                                    |
                                        < - - - - - R 
                                        y
```
Este es el sistema de coordenadas que usa el láser, por lo que no es necesario transformar las coordenadas de los obstáculos detectados. 

### 2.2 Cálculo de las fuerzas

La mejor opción según la documentación para obtener el vector de los obstáculos, es asignar a cada rayo un vector de repulsión propio, inversamente proporcional a la magnitud de la distancia detectada. Una vez calculados todos los vectores individuales, se suman para obtener el vector de repulsión resultante. 

Inicialmente, probé una aproximación menos compleja: construir el vector a partir del rayo que representase la menor distancia con el coche Sin embargo, los resultados obtenidos no fueron óptimos. 

El problema principal era  el rayo más cercano cambiaba muy facilmente entre iteraciones, provocando variaciones bruscas en la dirección del movimiento, lo que introducía, por consecuencia, muchas oscilaciones durante el control. 

Aunque la solución definitiva, basada en la combinación de vectores de repulsión, presenta un comportamiento más estable, sigue teniendo problemas para esquivar obstáculos que se encuentran alineados con el vector del subobjetivo.

### 2.3 Resolución de conflictos

Para solucionar el problema anterior, he implementado una función que analiza el espacio libre a los lados del obstáculo. En base a esta información, la función decide por qué lado resulta más conveniente rebasar el obstáculo. 

Este algoritmo sirve como salvavidas en situaciones en las que el VFF por sí mismo nos es capaz de proporcionar una dirección de movimiento adecuada.

### 2.4 Flujo de ejecución

Los pasos que se repiten en cada iteración son:

1. Obtener las coordenadas absolutas del siguiente checkpoint
2. Obtener las pose del robot y los obstáculos detectados por el láser
3. Transformar las coordenadas del checkpoint al sistema de referencia del robot
4. Calcular los vectores de atracción y repulsión y obtener el vector resultante
5. Calcular el ángulo de movimiento deseado
6. Transformar todo a velocidad angular y lineal
7. Enviar las velocidades a los actuadores

### 2.5 Control de movimiento

He probado a incorporar el control con un `controlador (PD)` para calcular la velocidad angular en función del ángulo deseado. Sin embargo, tras varias pruebas, he llegado a la conclusión de que no es necesario, ya que puede llegar incluso a reducir la reactividad del sistema. Al final el coche presenta oscilaciones, primordialmente en rectas, mientras que en curvas y a la hora de esquivar obstáculos no se observan prácticamente ninguna.

También he añadido una representación gráfica de los vectores usados por el algoritmo y los distintos checkpoints. Esto permite visualizar y depurar el comportamiento del sistema y el comportamiento del VFF.

<img width="244" height="490" alt="image" src="https://github.com/user-attachments/assets/9c170de9-fcb7-4630-88a0-4333a5f3e748" />

### 2.6 Medición del tiempo de vuelta

Para evaluar el rendimiento del algortimo, he incorporado un temporizador que mide el tiempo global transcurrido desde el principio de la ejecución.

El tiempo se muestra por terminal mediante una traza únicamente cuando se alcanza un nuevo `checkpoint`. De esta formar, evita generar una cantidad excesiva de mensajes durante la ejecución, mejora la legibilidad de estos, reduce las operaciones de salida por terminal y minimiza su posible impacto en la latencia del controlador.

Tras realizar diferentes pruebas, el mejor tiempo obtenido desde el principio de la ejecución hasta alcanzar el último `checkpoint` ha sido de `55,01 segundos`. Para realizar una pequeña comparativa, con el controlador PD, mi tiempo máximo fue `129,60 segundos` y en alguna que otra prueba se seguía chocando.

### 3. Resultado final

A continuación, dejo el vídeo de la ejecución del código final.

[Ver vídeo](https://drive.google.com/file/d/1AtpTExZ6WsFxTPyjVyK5uZJhM63VHV0u/view?usp=sharing)

