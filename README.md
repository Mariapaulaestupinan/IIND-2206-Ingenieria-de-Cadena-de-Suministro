<h1>IIND 2206 Ingeniería de Cadena de Suministro</h1>
Este repositorio tiene como propósito aterrizar la teoría vista en clase mediante el desarrollo de ejemplos y casos de estudio basados en problemas reales de logística y optimización.

Se abordarán temas relacionados con ruta más corta y ruteo de vehículos utilizando librerías especializadas de Python para construir, solucionar y analizar distintos escenarios desde un enfoque aplicado.

## Contenido del repositorio

- [Ruta más corta](#ruta-más-corta)
- [Ruteo de vehículos](#ruteo-de-vehiculos)
---

### Ruta más corta

En esta sección aprenderemos a modelar y resolver problemas de ruta más corta utilizando la librería NetworkX. Adicionalmente, implementaremos el algoritmo de Dijkstra desde cero, con el fin de comprender en profundidad la lógica y estructura subyacente de este tipo de algoritmos aplicados sobre redes.

A lo largo de los ejercicios, nos enfrentaremos con problemas en los que no solo se busca encontrar la ruta de menor distancia entre dos nodos, sino también minimizar tiempos de recorrido, costos de transporte o cualquier métrica de eficiencia definida sobre la red. Además, se incorporarán restricciones operativas que afectarán la disponibilidad de algunas vías y generarán sobrecostos al recorrer determinados arcos de la red.

### Contenido
- [NetworkX](#networkx)
- [Funciones principales](#funciones-principales)
#### NetworkX

La librería NetworkX resulta especialmente útil en problemas donde la estructura de la red se encuentra completamente definida desde el inicio y no cambia durante la ejecución del algoritmo. En este tipo de ejercicios, los nodos, arcos y pesos asociados permanecen estáticos, por lo que el proceso consiste principalmente en construir el grafo a partir de la información del problema y posteriormente calcular la ruta más corta.

Bajo este enfoque, las restricciones del problema se incorporan previamente mediante el filtrado o construcción de la red. Es decir, antes de resolver el modelo se eliminan arcos no permitidos, se ajustan costos, tiempos o distancias, y se define la conectividad válida entre nodos. Una vez construida la red, el algoritmo de ruta más corta opera sobre un grafo fijo que no se modifica dinámicamente mientras se recorre la solución.

A continuación, se estudiarán las principales funciones de la librería NetworkX. Además, se presentarán algunos ejercicios orientados a comprender su implementación en problemas de ruta más corta.

#### Funciones principales
##### Importación de la librería
```python
import networkx as nx
```
##### Creación del grafo dirigido
A lo largo del curso se trabajará exclusivamente con grafos dirigidos.
```python
G = nx.DiGraph()
```
##### Añadir nodos al grafo 
###### Agregar un nodo
```python
G = nx.DiGraph()
G.add_node("A")
```
###### Agregar una lista de nodos
```python
G = nx.DiGraph()
nodos =["A", "B", "C", "D"]
G.add_nodes_from(nodos)
```
###### Agregar nodos desde un archivo de excel
Suponga que tiene un archivo de Excel que contiene la información de los nodos de una red. En dicho archivo, existe una columna denominada "Nodos", en la cual cada fila representa un nodo distinto. A partir de esta estructura, los nodos pueden extraerse e incorporarse de manera automática para la construcción del grafo.
```python
import pandas as pd 
G = nx.DiGraph()
df= pd.read_excel("Nodos.xlsx")
G.add_nodes_from(df["Nodos"])
```
##### Eliminar nodos de un grafo 
##### Añadir arcos al grafo
Agrega arcos al grafo especificando su nodo de origen, su nodo de destino y un peso que puede ser 
```python
G = nx.DiGraph()
G.add_edge("A", "B", weight = 5)
```
###### Agregar un arco al grafo 
###### Agregar una lista de arcos 
###### Agregar arcos desde un archivo de excel





