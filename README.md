# Algoritmo-de-deteccion-de-vehiculos-con-OpenCV
## Introducción
En este proyecto desarrollamos un detector de vehículos usando principalmente la librería ***cv2*** en python, de forma que a partir de un vídeo de una autopista se muestra el mismo vídeo con cajas delimitadoras en cada coche durante su movimiento, además de un id y la velocidad aproximada que tiene cada uno

***Vídeo en el que aplicaremos el algoritmo:*** [Enlace al vídeo trafico.mp4](https://alumnosulpgc-my.sharepoint.com/:v:/g/personal/victor_diaz106_alu_ulpgc_es/IQBHLkIDytv6Tocon6da4C_iAR7Sfe0N25ejB0UouSDJSI8?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=G1vs3J)

Primero generamos una imagen promedio para sacar el fondo de la imagen sin vehículos, una vez tengamos la imagen generada y guardada podemos ejecutar el resto del código

__El algoritmo se divide en:__
- ***Rutas y Parámetros***: En esta sección guardamos el vídeo y la imagen promedio que usaremos para detectar el movimiento, además estableceremos todos los valores constantes que usaremos más tarde

- ***Carga y Preparación***: Obtendremos las variables que dependen del vídeo como el tamaño del mismo o su relación entre píxeles y metros, también definimos la máscara para eliminar el movimiento del temporizador

- ***Estado del Tracker***: Definimos los diccionarios que usaremos para almacenar la información y todas las funciones que usaremos en el bucle principal para obtener la posición, la velocidad y el seguimiento de los coches con sus ids asociados

next_object_id = 0

objects = {}  # id -> posición central del coche

disappeared = {}  # id -> número de frames consecutivos en los que ese id no se detectó

tracks = {}  # id -> historial de posiciones (frame_index, posición)

frame_index = 0  # contador global de frames


- ***Bucle Principal***: Procesa cada frame del vídeo para analizar el movimiento, hace todos los cálculos necesarios para dibujar el contador de coches, dónde se encuentra cada uno, su respectivo id y a la velocidad a la que se desplaza en el mismo vídeo y mostrarlo


## Librerías Usadas
· cv2

· numpy

· matplotlib.pyplot

· collections (para usar deque)


## Funciones
```python
def resolve_coord(v, limit):
    ...
```
Se le pasa una coordenada de la imagen y su límite, devuelve la coordenada de la imagen corregido si fuera necesario</br>
ejemplo:</br>
resolve_coord(10, 100) → 10</br>
resolve_coord(-10, 100) → 90 (100 - 10)

```python
def pixels_per_meter_at(y_center, frame_h=frame_h):
    ...
```
Calcula cuántos píxeles corresponden a un metro real según la posición vertical (y_center) del objeto en el frame y la altura del vídeo (frame_h)

```python
def centroid_from_box(box):
    x, y, w, h = box  # x e y son la esquina superior izquierda, w y h el ancho y alto
    return (int(x + w/2), int(y + h/2))
```
Se le pasa una caja (lista con x, y, w, h) y calcula su centro

```python
def euclidean(a, b):
    return np.hypot(a[0]-b[0], a[1]-b[1]) 
```
Calcula la distancia euclídea entre 2 puntos

```python
def register_object(centroid):
    ...
```
Le pasamos un centro y registra un nuevo vehículo en el diccionario

```python
def deregister_object(obj_id):
    ...
```
Le pasamos el id del vehículo y lo elimina

```python
def update_tracker(detections_centroids):
    ...
```
Esta es la función principal del algoritmo pues le pasamos la lista de centroides detectados en el frame actual y actualiza los diccionarios en consecuencia. La sección Diagramas muestra el flujo completo de esta función

```python
def compute_speed_for_track(track_deque):
    ...
```
Le pasamos el historial de posiciones de un vehículo y calcula su velocidad media utilizando pixels_per_meter_at


## Bucle Principal
El bucle principal procesa cada frame del vídeo en tiempo real y lo modifica para obtener el resultado final. Para hacer esto, primero pasa el frame del vídeo a escala de grises, reduce el ruido, elimina la región del cronómetro para que no la cuente y calcula la diferencia entre el fondo sin coches y el actual para detectar dónde están estos, en último lugar, se binariza la imagen para que el algoritmo detecte fácilmente dónde están los cambios, siendo las partes blancas los movimientos detectados y los negros el fondo de la imagen. Después eliminamos el ruido pequeño, rellenamos los huecos negros dentro de zonas blancas y expandimos las regiones detectadas para que los coches aparezcan como áreas sólidas.</br>
A continuación, según la imagen creada, generamos todos los posibles contornos de coches detectados y una lista donde meteremos estas detecciones. Para cada contorno, calcularemos su área y analizaremos según su perspectiva si es suficientemente grande como para poder considerarse un coche, además de eliminar los contornos que estén muy cerca del cronómetro. Para los coches que cumplan el tamaño adecuado, los pequeños los aceptaremos como detecciones independientes, pero los grandes analizaremos si son dos coches juntos e intentaremos separarlos.</br>
Una vez tenemos cada detección encontrada en el frame, eliminamos las cajas que se superponen y fusionamos las que están muy cerca entre ellas, ahora actualizamos los coches registrados, creamos los ids nuevos y controlamos los coches que han desparecido según las detecciones del frame actual.</br>
Con toda esta información, ya podemos dibujar todas las cajas, calcularemos la velocidad y la mostraremos con el ID sobre la caja, la cual será verde si su velocidad es menor a 120 y roja en caso contrario. Y ya para terminar, dibujaremos la escala píxeles/metros de la imagen, los fotogramas por segundo del vídeo y el número de coches que ha detectado.


## Diagrama
Diagrama del funcionamiento de update_tracker:
![Diagrama update_tracker](diagrama_updatetracker.png)


## Fotos del Funcionamiento
Foto del funcionamiento del algoritmo:
![Foto detección coches](foto_funcionamiento.png)

Foto donde detecta el choque entre vehículos:
![Foto detección accidente](foto_choque.png)

