<div align="center">

<h1>IIND 2206 · Ingeniería de Cadena de Suministro</h1>

</div>

## Descripción

Este repositorio tiene como propósito aterrizar la teoría vista en clase mediante el desarrollo de ejemplos y casos de estudio basados en problemas reales de logística y optimización.

Se abordarán temas relacionados con **ruta más corta** y **ruteo de vehículos**, utilizando librerías especializadas de Python para construir, solucionar y analizar distintos escenarios desde un enfoque aplicado.

---

## Contenido del repositorio
- [Ruta más corta](#ruta-más-corta) 
- [Ruteo de vehículos](#ruteo-de-vehículos) 

---

## Ruta más corta

En esta sección aprenderemos a modelar y resolver problemas de ruta más corta utilizando la librería **NetworkX**. Adicionalmente, implementaremos el algoritmo de **Dijkstra** desde cero, con el fin de comprender la lógica y estructura subyacente de este tipo de algoritmos aplicados sobre redes.

A lo largo de los ejercicios, nos enfrentaremos con problemas en los que no solo se busca encontrar la ruta de menor distancia, sino también **minimizar tiempos de recorrido**, **costos de transporte** o cualquier métrica de eficiencia definida sobre la red. Se introducirán restricciones operativas que condicionan la disponibilidad de ciertas vías y generan sobrecostos al momento de transitar por determinados arcos. 

### Contenido

- [NetworkX](#networkx)
- [Funciones principales](#funciones-principales)

---

### NetworkX

La librería NetworkX resulta especialmente útil en problemas donde la estructura de la red se encuentra completamente definida desde el inicio y no cambia durante la ejecución del algoritmo. En este tipo de ejercicios, los nodos, arcos y pesos asociados permanecen estáticos, por lo que el proceso consiste principalmente en construir el grafo a partir de la información del problema y posteriormente calcular la ruta más corta.

Bajo este enfoque, las restricciones del problema se incorporan previamente mediante la construcción de la red. Es decir, antes de resolver el modelo se eliminan los arcos que no cumplen las condiciones operativas y se ajustan los costos, tiempos o distancias de acuerdo a las condiciones del problema. Una vez construida la red, el algoritmo opera sobre un grafo fijo que no se modifica dinámicamente mientras se recorre la solución.

A continuación, se estudiarán las principales funciones de la librería NetworkX. Además, se presentarán algunos ejercicios orientados a comprender su implementación en problemas de ruta más corta.

---

### Funciones principales

#### Importación de la librería

```python
import networkx as nx
```

#### Inicialización del grafo 
A lo largo del curso se trabajará exclusivamente con **grafos dirigidos**.

```python
G = nx.DiGraph()
```
#### Construcción del grafo

<details>
<summary> Agregar nodos al grafo</summary>

---

**- Agregar un solo nodo**

```python
G = nx.DiGraph()
G.add_node("A")
```

**- Agregar una lista de nodos**

```python
G = nx.DiGraph()
nodos = ["A", "B", "C", "D"]
G.add_nodes_from(nodos)
```
**- Agregar nodos desde un archivo Excel**

Suponga que tiene un archivo Excel con una columna llamada `"Nodos"`, donde cada fila representa un nodo distinto.

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Nodos.xlsx")
G.add_nodes_from(df["Nodos"])
```
</details>
<details>
<summary> Agregar arcos al grafo</summary>

---

El parámetro `weight` representa el peso del arco, el cual puede corresponder a una distancia, un tiempo de recorrido o un costo de transporte, según el criterio de optimización definido en el problema.

**- Agregar un solo arco**

```python
G = nx.DiGraph()
G.add_edge("A", "B", weight=5)
```

**- Agregar una lista de arcos**

Cuando cada arco tiene un único atributo de peso:

```python
G = nx.DiGraph()
arcos = [("A", "B", 5), ("B", "C", 3), ("C", "D", 7)]
G.add_weighted_edges_from(arcos)
```

Cuando cada arco tiene múltiples atributos, los atributos de cada arco deben estar organizados en un diccionario, de lo contrario `add_edges_from` no los reconocerá correctamente:

```python
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 5,  "tipo": "restringido", "disponible": True}),
    ("B", "C", {"weight": 3,  "tipo": "normal",      "disponible": True}),
    ("C", "D", {"weight": 7,  "tipo": "restringido", "disponible": False}),
]
G.add_edges_from(arcos)
```

**▸ Agregar arcos desde un archivo Excel**

Suponga que tiene un archivo Excel con columnas "Origen", "Destino" y "Peso", donde cada fila representa un arco de la red y hay un unico atributo:

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"], weight=row["Peso"])
```

Suponga que tiene un archivo Excel con columnas `"Origen"`, `"Destino"`, `"Peso"`, `"Tipo"` y `"Disponible"`, donde cada fila representa un arco de la red y hay varios atributos:

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"],
               weight     = row["Peso"],
               tipo       = row["Tipo"],
               disponible = row["Disponible"])
```

</details>
<details>
<summary> Eliminar arcos al grafo</summary>

---

A diferencia de la construcción del grafo, la eliminación de arcos permite depurar la red en función de restricciones operativas y condiciones del problema.

**- Eliminar los arcos que salen de un nodo**

```python
nodo = "A"
arcos_salida = list(G.out_edges(nodo))
G.remove_edges_from(arcos_salida)
```

**- Eliminar los arcos que llegan a un nodo**

```python
nodo = "A"
arcos_entrada = list(G.in_edges(nodo))
G.remove_edges_from(arcos_entrada)
```

**- Eliminar los arcos con un atributo específico**

Elimina aquellos arcos cuyos atributos no cumplen con las condiciones operativas del problema. Si el grafo fue construido con un único atributo `weight`, se puede filtrar directamente sobre él.

```python
# Arcos con un solo atributo
arcos_a_eliminar = []
for u, v, d in G.edges(data=True): # data=True retorna los atributos del arco como diccionario
    if d["weight"] > 10:
        arcos_a_eliminar.append((u, v))

G.remove_edges_from(arcos_a_eliminar)
```

Cuando los arcos tienen múltiples atributos, es posible combinar condiciones sobre cualquiera de ellos para definir un criterio de eliminación.

```python
# Construcción del grafo con múltiples atributos
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 5,  "tipo": "restringido", "disponible": True}),
    ("B", "C", {"weight": 12, "tipo": "normal",      "disponible": False}),
    ("C", "D", {"weight": 8,  "tipo": "restringido", "disponible": False}),
]
G.add_edges_from(arcos)

# Eliminar arcos que sean de tipo restringido y no estén disponibles
arcos_a_eliminar = []
for u, v, d in G.edges(data=True):
    if d["tipo"] == "restringido":
        if d["disponible"] == False:
            arcos_a_eliminar.append((u, v))

G.remove_edges_from(arcos_a_eliminar)
```

**- Eliminar los arcos desde un archivo Excel**

Suponga que tiene un archivo Excel con las columnas `"Origen"` y `"Destino"`, donde cada fila representa un arco que debe ser eliminado de la red.

```python
import pandas as pd

df = pd.read_excel("Arcos_eliminar.xlsx")

arcos_a_eliminar = []
for _, row in df.iterrows():
    arcos_a_eliminar.append((row["Origen"], row["Destino"]))

G.remove_edges_from(arcos_a_eliminar)
```

</details>
<details>
<summary> Modificar atributos de los arcos</summary>

---

En ciertos problemas es necesario ajustar los atributos de los arcos sin eliminarlos de la red. Esto resulta útil cuando se desea incorporar penalizaciones o sobrecostos asociados a condiciones operativas específicas, como el tipo de vía, el estado de la carretera o restricciones horarias.

**- Modificar el atributo de un arco según un criterio**

Suponga que los arcos tienen los atributos `"Costo"` y `"Tipo_Via"`. Si la vía es de tipo terrestre, se desea aumentar el costo en un 20%:

```python
for u, v, d in G.edges(data=True):  # data=True retorna los atributos del arco como diccionario
    if d["Tipo_Via"] == "terrestre":
        d["Costo"] = d["Costo"] * 1.20
```
</details>

#### Solución y resultados

---

Una vez construida la red, calculamos la ruta más corta entre dos nodos. A partir de la solución obtenida es posible consultar sobre dicha ruta cualquier atributo de interés, como el costo acumulado, la distancia total, el tiempo de recorrido o el número de arcos que cumplen con alguna condición particular.

<details>
<summary>Calcular la ruta más corta </summary>
  
Retorna la secuencia de nodos que conforman el camino óptimo entre el nodo de origen y el nodo de destino, minimizando el atributo especificado en `weight`.

```python
ruta = nx.dijkstra_path(G, source="A", target="D", weight="weight")
print(ruta)  # ["A", "C", "B", "D"]
```
</details>
<details>
<summary> Obtener el valor óptimo de la función objetivo </summary>

Retorna el valor de la función objetivo de la ruta óptima segun el atributo especificado en `weight`. 

```python
valor_funcion_objetivo = nx.dijkstra_path_length(G, source="A", target="D", weight="weight")
print(f"${valor_funcion_objetivo}") # $15
```
</details>
<details>
<summary> Obtener la ruta y el valor de la función objetivo en una sola llamada </summary>

```python
valor_funcion_objetivo, ruta = nx.single_source_dijkstra(G, source="A", target="D", weight="weight")
print(ruta)        # ["A", "B", "C", "D"]
print(f"${valor_funcion_objetivo}") # $15
```
</details>
<details>
<summary> Extraer información adicional de la ruta óptima </summary>

En ocasiones el criterio de optimización no es el único atributo de interés. Una vez obtenida la ruta óptima, es posible calcular el valor acumulado de cualquier otro atributo a lo largo de ella.

Suponga que el problema minimiza el costo pero adicionalmente se desea conocer la distancia total recorrida a lo largo de esa misma ruta. En ese caso se puede calcular de la siguiente manera:

```python
# Calcula simultáneamente el costo mínimo y la ruta óptima
costo_total, ruta = nx.single_source_dijkstra(G, source="A", target="D", weight="costo")

# Acumular la distancia total a lo largo de los arcos de la ruta óptima
distancia_total = 0
for i in range(len(ruta) - 1):
    u = ruta[i]       # Nodo de origen del arco
    v = ruta[i + 1]   # Nodo de destino del arco
    distancia_total += G[u][v]["distancia"]  # Suma la distancia del arco

print(f"Ruta:            {ruta}")
print(f"Costo total:     ${costo_total:,.2f}")
print(f"Distancia total: {distancia_total:,.2f} km")
```
</details>
<details>
<summary> Identificar los arcos que cumplen una condición especifica sobre la ruta óptima </summary>

Una vez obtenida la ruta óptima, es posible recorrer los arcos que la conforman e identificar cuáles de ellos satisfacen alguna condición de interés. Por ejemplo, identificar qué tramos de la ruta corresponden a vías terrestres, a qué zona pertenece cada arco o cuáles superan un número determinado de peajes.

```python
# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="D", weight="costo")

# Identificar los arcos de la ruta óptima que son de tipo terrestre 
arcos_terrestres = []
for i in range(len(ruta) - 1):
    u = ruta[i]      # Nodo de origen del arco
    v = ruta[i + 1]  # Nodo de destino del arco
    if G[u][v]["tipo"] == "terrestre":  # Condición sobre el atributo del arco
        arcos_terrestres.append((u, v))

print(f"Ruta:             {ruta}")           # Ruta:             ["A", "B", "C", "D"]
print(f"Arcos terrestres: {arcos_terrestres}") # Arcos terrestres: [("A", "B"), ("C", "D")]
```
</details>
<details>
<summary> Listar todos los caminos posibles entre un origen y un destino </summary>

Cuando el análisis requiere explorar la totalidad de caminos existentes entre dos nodos y no únicamente el óptimo, es posible obtener una lista exhaustiva de todas las rutas disponibles en la red.

```python
todos_los_caminos = list(nx.all_simple_paths(G, source="A", target="D"))

for camino in todos_los_caminos:
    print(camino)
```

Para cada camino encontrado es posible calcular el valor acumulado de cualquier atributo de interés, como el costo total:

```python
todos_los_caminos = list(nx.all_simple_paths(G, source="A", target="D"))

for camino in todos_los_caminos:
    costo = 0
    for i in range(len(camino) - 1):
        u = camino[i]
        v = camino[i + 1]
        costo += G[u][v]["weight"]
    print(f"Camino: {camino} → Costo: {costo}")
```
</details>


#### Visualización del grafo

---

Una vez construida la red y obtenida la solución, es posible visualizar el grafo junto con la ruta óptima para facilitar la interpretación de los resultados.
<details>
<summary> Visualización a partir de datos ingresados manualmente </summary>

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np

# Construcción del grafo
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 4,  "tipo": "terrestre"}),
    ("A", "C", {"weight": 2,  "tipo": "aéreo"}),
    ("B", "C", {"weight": 1,  "tipo": "terrestre"}),
    ("B", "D", {"weight": 5,  "tipo": "aéreo"}),
    ("C", "D", {"weight": 8,  "tipo": "terrestre"}),
    ("C", "E", {"weight": 3,  "tipo": "aéreo"}),
    ("D", "E", {"weight": 2,  "tipo": "terrestre"}),
]
G.add_edges_from(arcos)

# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="E", weight="weight")

# Identificar los arcos de la ruta óptima
arcos_ruta = [(ruta[i], ruta[i + 1]) for i in range(len(ruta) - 1)]

# Definir posición de los nodos
pos = nx.spring_layout(G)

# Dibujar el grafo completo
nx.draw_networkx_nodes(G, pos, node_color="lightblue", node_size=800)
nx.draw_networkx_labels(G, pos, font_size=12, font_weight="bold")
nx.draw_networkx_edges(G, pos, edge_color="gray", arrows=True, arrowsize=20)

# Resaltar la ruta óptima
nx.draw_networkx_edges(G, pos, edgelist=arcos_ruta, edge_color="red", width=2.5, arrows=True, arrowsize=20)

# Etiqueta superior: distancia en km (desplazada hacia arriba)
etiquetas_arriba = {(u, v): f"{d['weight']} km" for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiquetas_arriba, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="bottom")

# Etiqueta inferior: tipo de vía (desplazada hacia abajo)
etiquetas_abajo = {(u, v): d["tipo"] for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiquetas_abajo, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="top")

plt.title(f"Ruta óptima: {' → '.join(ruta)}")
plt.axis("off")
plt.tight_layout()
plt.show()
```
<div align="center">
  <img src="grafo 1.png" width="600"/>
</div>
</details>
<details>
<summary> Visualizar el grafo y la ruta óptima con datos desde un archivo Excel </summary>
El siguiente archivo contiene la estructura de red utilizada en este ejemplo. Cada fila representa un arco de la red, definido por su nodo de origen, su nodo de destino y el costo asociado al trayecto. <a href="Arcos.xlsx?raw=true" download> Descargar Arcos.xlsx</a>

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# Construcción del grafo desde Excel
G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"], weight=row["Costo"])

# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="K", weight="weight")

# Identificar los arcos de la ruta óptima
arcos_ruta = [(ruta[i], ruta[i + 1]) for i in range(len(ruta) - 1)]

# Definir posición de los nodos
pos = nx.spring_layout(G, seed=42)

# Dibujar el grafo completo
nx.draw_networkx_nodes(G, pos, node_color="lightblue", node_size=800)
nx.draw_networkx_labels(G, pos, font_size=12, font_weight="bold")
nx.draw_networkx_edges(G, pos, edge_color="gray", arrows=True, arrowsize=20)

# Resaltar la ruta óptima
nx.draw_networkx_edges(G, pos, edgelist=arcos_ruta, edge_color="red", width=2.5, arrows=True, arrowsize=20)

# Etiqueta: costo en km (desplazada hacia arriba)
etiqueta = {(u, v): f"{d['weight']} km" for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiqueta, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="bottom")

plt.title(f"Ruta óptima: {' → '.join(ruta)}")
plt.axis("off")
plt.tight_layout()
plt.show()
```
<div align="center">
  <img src="grafo 2.png" width="600"/>
</div>
</details>

#### Ejercicios 

En esta sección se trabajarán ejercicios de ruta más corta que permitirán aplicar de manera integrada las distintas funcionalidades estudiadas previamente en la librería NetworkX.
<details>
<summary> Red de Distribución de Vacunas en Estados Unidos </summary>
Una entidad nacional de salud pública en Estados Unidos enfrenta el reto de coordinar la distribución de vacunas entre distintas ciudades del país, considerando restricciones operativas asociadas al modo de transporte permitido a lo largo de la red y a las condiciones de refrigeración necesarias para garantizar la conservación adecuada de las vacunas durante su distribución.

**Enunciado:** <a href="Red de Distribución de Vacunas.pdf?raw=true" download> Enunciado Red de Distribución de Vacunas en Estados Unidos</a>

**Base de datos:** <a href="Distribución_Vacunas.xlsx?raw=true" download> Base de Datos Red de Distribución de Vacunas en Estados Unidos</a>
</details>
<details>
<summary> Ejercicio Nivel Intermedio </summary>
</details>
<details>
<summary> Ejercicio Nivel Dificil </summary>
</details>




