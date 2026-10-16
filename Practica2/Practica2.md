# PRÁCTICA 2: Control Visual con PID de un Fórmula 1

## ÍNDICE
1. [Objetivos de la práctica](#objetivos-de-la-práctica)
2. [Desarrollo de la práctica](#desarrollo-de-la-solución)
   
## Objetivos de la práctica

El problema propuesto para esta práctica es el siguiente:

Programa un coche de Fórmula1, equipado con una cámara frontal, para que recorra un circuito de carreras que tiene una línea roja en el medio de la pista. Se programará uno o varios controladores PID para gobernar la velocidad lineal del coche (V) y la velocidad angular (W). El coche debe funcionar en varios circuitos, completarlos en el menor tiempo posible y sin alejarse de la línea roja. Cuantas menos oscilaciones tenga mejor, y conviene que sea robusto si en algún momento pierde la línea.

La aplicación robótica se implementará como un bucle infinito. Cada iteración incluirá las instrucciones para materializar el procesamiento de la imagen y para la toma de decisiones sobre los dos actuadores V y W. 


## Desarrollo de la solución

Primeramente, para analizar las imágenes usamos la biblioteca de OpenCV, junto con una imágen en formato HSV. Usar este formato, en lugar de RGB, nos permite detectar de manera más robusta los colores frente a la variación de la luz y el brillo.

El primer paso fue definir los colores que nos conciernen, en este caso el rojo de la línea que tenemos que seguir. En OpenCV se usan las denominadas máscaras para seleccionar colores de las imágenes. Estas máscaras nos permiten ajustar rangos de H (Hue), S (Saturation) y V (Value) para conseguir colores concretos. 


<img width="274" height="274" alt="image" src="https://github.com/user-attachments/assets/c500d2d2-15c8-4289-9dba-350f90a85757" />


Como podemos ver en la imágen, hay color rojo a ambos lados del rango del HSV. La solución que he elegido para la detección es definir dos máscaras de rojo y combinarlas para cubrir el espectro completo. A partir de ahí, la idea es sacar el centro de la línea e intentar alinear el morro del coche con él, pero con un detalle, que la cámara está desviada con respecto del morro. Por lo tanto, lo primero era sacar el centro real del coche. 

Para esto colocamos el coche perfectamente centrado sobre la línea y con cv2 imprimimos el valor del centroide. Este valor lo usaré durante todo el programa como constante de posición x central. Una vez ya hayado el centro que hay que seguir, he elegido dibujar el centroide de color verde, el centro real de color azul y una línea que los une, así se pueden ver con más facilidad las oscilaciones del coche.

<img width="924" height="439" alt="image" src="https://github.com/user-attachments/assets/989691a9-3e26-43a9-8312-c69cf93da561" />

Para el control del movimiento, he ideado un controlador de la familia de los PID (un PD). Este controlador afecta únicamente la velocidad angular. Calcula el error en base a la diferencia entre la posición de referencia del coche y el centroide que devuelve cv2. La parte derivativa se calcula simplemente con la diferencia entre el error actual y el error previo. La fórmula usada para calcular la velocidad angular ha sido ***w = (KP x error) + (KD x derivativa)***, con el valor del resultado redondeado a 3 decimales. A partir de aquí, a base de prueba y error, he ajustado las constantes de las partes proporcional y derivativa del PD. 

En cuanto a la parte de la velocidad lineal, he probado a mantenerla dentro de un rango regulado por la desviación con respecto del centroide, pero no me ha dado buenos resultados. Al final he optado por mantenerla a una velocidad constante y me ha dado unos resultados bastante óptimos.

Ante la posibilidad de que el coche pierda la línea, he probado a implementar una máquina de estados sencilla con un estado de recuperación que buscaba la línea. Sin embargo, al notar que con la máquina de estados el coche tardaba más en reaccionar y era más propenso a oscilar decidí no seguir con esa implementación. Igualmente, si el coche pierde el centroide, se coge como referencia el último error calculado y se le hace girar en la dirección contraria hasta recuperar la línea.

A medida que he ido probando velocidades y distintos valores para las constantes, he tenido que decidir entre hacer que el coche complete la vuelta más rápido y perder estabilidad frente a las curvas y las oscilaciones, o que lo complete a una velocidad más controlada y reducir las oscilaciones. He encontrado cierto equilibrio, pero dado el enfoque de la práctica he elegido la segunda opción, reducir un poco la velocidad. De todas formas, dejo también un vídeo del mismo PID a una velocidad superior a la que he acabado usando en el código para que se aprecie que lo soporta sin oscilar de manera exagerada.

[Circuito simple lento](https://drive.google.com/file/d/18jxK0jh_yv9dKil-IVgbkEVpzUwlq4VG/view?usp=sharing)

[Circuito simple rápido](https://drive.google.com/file/d/1Rw6sQ4zeJ8VyP5RnhI-bCLReJi8wwffg/view?usp=sharing)

[Nürburgring](https://drive.google.com/file/d/10vl3NPsPPYNYht3BtATqZT0iPtNgsXGv/view?usp=sharing)


En la primera curva del circuito de Nürburgring, al ser tan cerrada, el coche pierde de vista momentáneamente la línea. He probado con gran variedad de velocidades y valores, pero es demasiado cerrada para que corrija el coche a una velocidad que no considere demasiado lenta para el resto del circuito. He decidido dejarlo así porque considero que también me sirve también como referencia para demostrar la robustez del sistema y el funcionamiento del mecanismo de recuperación.

Al final los tiempos han sido los siguientes.

| Circuito | Tiempo (s)|
|---|---|
| `Simple lento` | 66.17 |
| `Simple rápido` | 47.84 | 
| `Montecarlo` | 222.90 |
| `Montmelo` | 115.36 |  
| `Nürburgring` | 99.69 | 
