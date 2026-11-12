# PRÁCTICA 4: Navegación global con algoritmo GPP
## 1. Objetivos de la práctica

Programar un coche autónomo que ejerza de taxi en una ciudad. La planficicación de la ruta de navegación debe basarse en el algoritmo de navegación global *GPP*(Gradient Path Planing). 

Además de planificar la ruta, el coche debe ser capaz de navegar al objetivo de manera autónoma. Una vez llegue al objetivo, se mantendrá quieto hasta que el usuario seleccione otro punto del mapa al que navegar.

## 2. Desarrollo de la práctica
### 2.1 Planteamiento general 

He dividido esta práctica en dos grandes conceptos para poder visualizar de manera más clara los problemas propios de cada uno. Por un lado, la parte de planificación de la ruta, y por otro la parte de navegación.

### 2.2 Planificación de la ruta

Tal como pide la práctica, el algoritmo de navegación que se debe usar es Gradient Path Planning. Este algortimo de navegación global haya la ruta más corta al destino en base a un campo de costes sobre el mapa. Cada celda recibe un valor y GPP sigue la bajada de costes.

Mi aproximación, dado que hay que tener en cuenta que los costes asociados a cada celda varían en función del movimiento (recto, diagonal) y de su proximidad con los obstáculos, ha sido implementar el GPP mediante Dijkstra, que permite ordenar la cola de prioridad de la frontera en función de los distintos costes de cada celda.

Realizando las primeras pruebas de este algoritmo me di cuenta de un problema importante, la asignación del coste extra por proximidad a obstáculos. Expandir, de cada nodo, un radio de nodos para comprobar la proximidad y asignar así la penalización supone un coste computacional elevado y aumenta considerablemente el tiempo que tarda el algoritmo en encontrar la ruta óptima. Para resolver este problema, al principio de la ejecución se crea un mapa de proximidad, con esta inversión de tiempo al inicio en crear el mapa, se consigue reducir considerablemente el tiempo de búsqueda. 

He realizado cuatro mediciones a distintos puntos del mapa, estando el coche siempre en pixel (200, 199), para contrastar si valía realmente la pena. Todas las coordenadas de la tabla están expresadas sobre la cuadrćula del mapa.

- Tiempo 1: Sin mapa de proximidad
- Tiempo 2: Con mapa de proximidad

| Destino | Tiempo 1 (s)| Tiempo 2 (s)|
|:---:|:---:|:---:|
| (107,328) | 30.29 | 4.77 |
| (376, 384) | 30.99 | 4.88 |
| (14, 383) | 33.22 | 5.26 |
| (378, 25) | 32.22 | 5.25 |

