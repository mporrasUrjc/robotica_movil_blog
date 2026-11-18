# PRÁCTICA 4: Navegación global con algoritmo GPP

# Índice

1. [Objetivos de la práctica](#1-objetivos-de-la-práctica)
2. [Desarrollo de la práctica](#2-desarrollo-de-la-práctica)
   - [2.1. Planteamiento general](#21-planteamiento-general)
   - [2.2. Planificación de la ruta](#22-planificación-de-la-ruta)
   - [2.3. Navegación](#23-navegación)
3. [Vídeo de ejecución](#3-vídeo-ejecución)

## 1. Objetivos de la práctica

Programar un coche autónomo que ejerza de taxi en una ciudad. La planficicación de la ruta de navegación debe basarse en el algoritmo de navegación global *GPP*(Gradient Path Planing). 

Además de planificar la ruta, el coche debe ser capaz de navegar al objetivo de manera autónoma. Una vez llegue al objetivo, se mantendrá quieto hasta que el usuario seleccione otro punto del mapa al que navegar.

## 2. Desarrollo de la práctica
### 2.1 Planteamiento general 

He dividido esta práctica en dos grandes conceptos para poder visualizar de manera más clara los problemas propios de cada uno. Por un lado, la parte de planificación de la ruta, y por otro la parte de navegación.

### 2.2 Planificación de la ruta

Tal como pide la práctica, el algoritmo de navegación que se debe usar es Gradient Path Planning. Este algortimo de navegación global haya la ruta más corta al destino en base a un campo de costes sobre el mapa. Cada celda recibe un valor y GPP sigue la bajada de costes.

Mi aproximación, dado que hay que tener en cuenta que los costes asociados a cada celda varían en función del movimiento (recto, diagonal) y de su proximidad con los obstáculos, ha sido implementar el GPP mediante Dijkstra, que permite ordenar la cola de prioridad de la frontera en función de los distintos costes de cada celda.

Realizando las primeras pruebas de este algoritmo me di cuenta de un problema importante, la asignación del coste extra por proximidad a obstáculos. Expandir, de cada nodo, un radio de nodos para comprobar la proximidad y asignar así la penalización supone un coste computacional elevado y aumenta considerablemente el tiempo que tarda el algoritmo en encontrar la ruta óptima. Para resolver este problema, al principio de la ejecución, se crea un mapa de proximidad expandiendo todos los obstáculos de mapa. Con esta inversión de tiempo al inicio en crear el mapa, se consigue reducir considerablemente el tiempo de búsqueda. 

He realizado cuatro mediciones a distintos puntos del mapa, estando el coche siempre en pixel (200, 200), para contrastar si valía realmente la pena. Todas las coordenadas de la tabla están expresadas sobre la cuadrćula del mapa.

- `Tiempo 1`: Sin mapa de proximidad
- `Tiempo 2`: Con mapa de proximidad

| Destino | Tiempo 1 (s)| Tiempo 2 (s)|
|:---:|:---:|:---:|
| (107,328) | 30.29 | 4.77 |
| (376, 384) | 30.99 | 4.88 |
| (14, 383) | 33.22 | 5.26 |
| (378, 25) | 32.22 | 5.25 |

Invirtiendo un tiempo inicial aproximado de `30 segundos` en la preparación del mapa, vemos que disminuye el tiempo de búsqueda más de `25 segundos` por ejecución. El mapa acaba siendo el siguiente:
<img width="606" height="430" alt="image" src="https://github.com/user-attachments/assets/8267bcc5-53d4-4eda-834d-fdf4edf81b75" />


El algoritmo de búsqueda, una vez encontrado el camino, va retrocediendo de hijo en padre y guardando en un array los puntos del camino para poder dibujarlo luego. Además, crea un diccionario de coordenadas y costes para poder usarlo luego en la navegación.

## 2.3 Navegación

La parte de la navegación debe ser por gradiente según el enunciado, y así mismo está implementada. El algoritmo fija los waypoints en función de la celda de coste más bajo en un radio fijado. Calcula el error angular y lo multiplica por una constante `Kp` para obtener la velocidad angular y fija una velocidad lineal en base a la angular.
Una vez alcanza el destino, vuelve a un estado en el que se queda esperando hasta recibir un nuevo destino y repetir todo el proceso de nuevo.

# 3. Vídeo ejecución

A continuación dejo el vídeo de la ejecución dando varios destinos al coche:

[Ver vídeo](https://drive.google.com/file/d/1ijtzixbtgXMpkun-qt5lfe314sAiwEYu/view?usp=sharing)
