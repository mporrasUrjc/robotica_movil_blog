# PRÁCTICA 2: Control Visual con PID de un Fórmula 1

## ÍNDICE
1. [Objetivos de la práctica](#objetivos-de-la-práctica)
2. [Solución encontrada](#solución-encontrada)
3. [Problemas encontrados](#problemas-encontrados)
## Objetivos de la práctica
El problema propuesto para esta práctica es el siguiente:

Programa un coche de Fórmula1, equipado con una cámara frontal, para que recorra un circuito de carreras que tiene una línea roja en el medio de la pista. Se programará uno o varios controladores PID para gobernar la velocidad lineal del coche (V) y la velocidad angular (W). El coche debe funcionar en varios circuitos, completarlos en el menor tiempo posible y sin alejarse de la línea roja. Cuantas menos oscilaciones tenga mejor, y conviene que sea robusto si en algún momento pierde la línea.

La aplicación robótica se implementará como un bucle infinito. Cada iteración incluirá las instrucciones para materializar el procesamiento de la imagen y para la toma de decisiones sobre los dos actuadores V y W. 
## Desarrollo de la solución

Primeramente, para analizar las imágenes usamos la biblioteca de OpenCV, junto con una imágen en formato HSV. Usar este formato, en lugar de RGB, nos permite detectar de manera más robusta los colores frente a la variación de la luz y el brillo.

El primer paso fue definir los colores que nos conciernen, en este caso el rojo de la línea que tenemos que seguir. En OpenCV se usan las denominadas máscaras para seleccionar colores de las imágenes. Estas máscaras nos permiten ajustar rangos de H, S y V para conseguir colores concretos. 

<img width="274" height="274" alt="image" src="https://github.com/user-attachments/assets/c500d2d2-15c8-4289-9dba-350f90a85757" />

Como podemos ver en la imágen, hay color rojo a ambos lados 

He probado varias soluciones hasta dar con la que me ha resultado más efectiva. Retroceder ante el choque antes de girar, no retroceder, mezclar giros y movimiento lineal, hacer espirales, movimientos en ese ... Al final he optado por una implementación basada en una FSM con los siguientes estados:
 - ***IDLE:*** Listo para empezar a moverse. Lo he introducido para asegurarme de que el láser ya está mandando información antes de empezar a ejecutar.
 - ***MOVING:*** Movimiento normal del robot. Realiza espirales hasta que choca la primera vez, después, debido a los obstáculos, realiza parábolas.
 - ***BUMP:*** Pone a cero tanto la velocidad lineal como la angular ante un choque.
 - ***SPINNING:*** Gira durante un número aleatorio de ticks, al lado contrario que gira el estado MOVING.
   
La mejor lógica que he encontrado al final es que haga una espiral hasta el primer choque, luego gira al lado contrario un número aleatorio de ticks y vuelve a moverse otra vez haciendo una parábola hacia el lado contrario. Con este bucle, dejando fija la velocidad angular y aumentando periodicamente la lineal (hasta un límite) he conseguido una máxima ejecución de 103.17% de recorrido del mapa.


<img src="src/Max_score.png" width="800">

A continuación dejo un vídeo demostración. Ha sido una ejecución de 19 minutos de duración y ha conseguido un 84% de puntuación.

[Vídeo demo](https://drive.google.com/file/d/1t12WhqmM5z1jBowxHswB1Vp1tjMVC7yG/view?usp=sharing)

## Problemas encontrados

La primera solución que encontré para generar un ángulo de giro aleatorio fue con la función *sleep*, pero al perder la reactividad durante el tiempo que dure el giro no era la más óptima. Al final he optado por contar un número de ticks aleatorios.

Otro problema con el que me he encontrado ha sido el aumento periódico de la velocidad lineal, ya que llegaba a una cifra que hacía que la aspiradora se desplazara de manera completamente lineal cuando yo buscaba una parábola. La solución que he encontrado a este problema ha sido definir una velocidad máxima y una mínima. Una vez el bucle llega a la máxima, resetea el valor al mínimo y comienza a incrementarla de nuevo.
