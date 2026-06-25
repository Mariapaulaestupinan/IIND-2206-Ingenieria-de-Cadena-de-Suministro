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
  - [Funciones principales NetworkX](#funciones-principales-networkx)
  - [Ejercicios NetworkX](#ejercicios-networkx)
- [Dijkstra](#dijkstra)
  - [Pseudocodigo del algoritmo de Dijkstra](#pseudocodigo-del-algoritmo-de-dijkstra)
  - [Ejercicios Dijkstra](#ejercicios-dijkstra)
---

### NetworkX

La librería NetworkX resulta especialmente útil en problemas donde la estructura de la red se encuentra completamente definida desde el inicio y no cambia durante la ejecución del algoritmo. En este tipo de ejercicios, los nodos, arcos y pesos asociados permanecen estáticos, por lo que el proceso consiste principalmente en construir el grafo a partir de la información del problema y posteriormente calcular la ruta más corta.

Bajo este enfoque, las restricciones del problema se incorporan previamente mediante la construcción de la red. Es decir, antes de resolver el modelo se eliminan los arcos que no cumplen las condiciones operativas y se ajustan los costos, tiempos o distancias de acuerdo a las condiciones del problema. Una vez construida la red, el algoritmo opera sobre un grafo fijo que no se modifica dinámicamente mientras se recorre la solución.

A continuación, se estudiarán las principales funciones de la librería NetworkX. Además, se presentarán algunos ejercicios orientados a comprender su implementación en problemas de ruta más corta.

---

### Funciones principales NetworkX

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
El siguiente archivo contiene la estructura de red utilizada en este ejemplo. Cada fila representa un arco de la red, definido por su nodo de origen, su nodo de destino y el costo asociado al trayecto. <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Arcos.xlsx" download>Descargar Arcos.xlsx</a>

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

#### Ejercicios NetworkX

---

En esta sección se trabajarán ejercicios de ruta más corta que permitirán aplicar de manera integrada las distintas funcionalidades estudiadas previamente en la librería NetworkX.
<details>
<summary> Red de Distribución de Vacunas en Estados Unidos </summary>
Una entidad nacional de salud pública en Estados Unidos enfrenta el reto de coordinar la distribución de vacunas entre distintas ciudades del país, considerando restricciones operativas asociadas al modo de transporte permitido a lo largo de la red y a las condiciones de refrigeración necesarias para garantizar la conservación adecuada de las vacunas durante su distribución.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Red%20de%20Distribuci%C3%B3n%20de%20Vacunas.pdf" download>Enunciado Red de Distribución de Vacunas</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Distribuci%C3%B3n_Vacunas.xlsx" download>Base de Datos Red de Distribución de Vacunas</a>

**Solución:** 

Importación de las librerías requeridas:

```python
import pandas as pd
import networkx as nx
```
Parámetros globales del problema:

La variable **Costo_Km** establece la tarifa asociada a cada modo de transporte, considerando un costo de **2.5 $/km para transporte terrestre** y **4.0 $/km para transporte aéreo**. Estos valores serán utilizados posteriormente para calcular el costo de recorrer cada arco de la red de distribución.

Por otro lado, las variables **Origen** y **Destino** definen las ciudades de inicio y llegada entre las que se desea determinar la ruta más corta. 

```python
Costo_Km = {'Terrestre': 2.5, 'Aéreo': 4.0}
Origen  = 'Atlanta'
Destino = 'Las Vegas'
```
Cargar Datos: 

Esta función permite cargar la información de la red de distribución a partir de un archivo en formato Excel. Recibe como parámetro la ruta del archivo, utiliza la función `pd.read_excel()` de la librería **Pandas** para leer los datos y retorna la información almacenada en un **DataFrame**.

```python
def cargar_datos(ruta_excel: str) -> pd.DataFrame:
    df = pd.read_excel(ruta_excel)
    return df
```
Calcular tiempo:

Esta función permite calcular el tiempo total asociado a cada arco de la red de distribución. Recibe como parámetro el **DataFrame** con la información de los arcos y calcula el tiempo de recorrido dividiendo la distancia entre la velocidad. Posteriormente, incorpora el tiempo de revisión al tiempo de recorrido calculado y almacena el tiempo total resultante en una nueva columna denominada **Tiempo_Total**.

```python
def calcular_tiempo(df: pd.DataFrame) -> pd.DataFrame:
    df['Tiempo_Total'] = (df['Distancia'] / df['Velocidad']) + df['Tiempo_Revision']
    return df
```
Calcular costo:

Esta función permite calcular el costo asociado a cada arco de la red de distribución utilizando la fórmula:

**C<sub>ij</sub> = U<sub>m</sub> × d<sub>ij</sub> + 100 × NF<sub>ij</sub> + 200 × T<sub>ij</sub>**

Posteriormente, se calcula el costo total de cada arco considerando la tarifa asociada al modo de transporte y la distancia recorrida, junto con los costos adicionales derivados del nivel de refrigeración requerido y el tiempo total de tránsito. El resultado final se almacena en una nueva columna denominada **Costo**.

```python
def calcular_costo(df: pd.DataFrame) -> pd.DataFrame:
    df['U_m']   = df['Modo_Transporte'].map(Costo_Km)
    df['Costo'] = (df['U_m'] * df['Distancia']
                   + 100 * df['Nivel_Frio']
                   + 200 * df['Tiempo_Total'])
    return df
```
Construir grafo - Literal b:

Esta función permite construir un grafo dirigido a partir de la información contenida en el **DataFrame**, donde cada arco representa una conexión entre una ciudad de origen y una ciudad de destino dentro de la red de distribución.

Para cada registro del DataFrame se agrega un arco dirigido desde el nodo de origen hasta el nodo de destino, almacenando como atributos la distancia, el tiempo total, el modo de transporte y el nivel de refrigeración requerido. Adicionalmente, el atributo **weight** se establece utilizando el costo calculado previamente, ya que este será el valor utilizado por el algoritmo de **Dijkstra** para determinar la ruta de menor costo dentro de la red.

```python
def construir_grafo_b(df: pd.DataFrame) -> nx.DiGraph:
    """Grafo completo. weight = costo (Dijkstra minimiza costo)."""
    G = nx.DiGraph()
    for _, f in df.iterrows():
        G.add_edge(f['Origen'], f['Destino'],
                   weight    = f['Costo'],
                   costo     = f['Costo'],
                   distancia = float(f['Distancia']),
                   tiempo    = f['Tiempo_Total'],
                   modo      = f['Modo_Transporte'],
                   nivel_frio= int(f['Nivel_Frio']))
    return G
```
Construir grafo - Literal c:

Esta función permite construir un grafo dirigido incorporando las restricciones adicionales establecidas para el literal c. Primero, filtra el DataFrame conservando únicamente los arcos con nivel de frío igual a 3. Luego, a los arcos cuyo modo de transporte es terrestre se les suma una penalización de 50 al costo.  
A diferencia del literal b, el atributo **weight** se define utilizando el tiempo total de tránsito, por lo que el algoritmo de **Dijkstra** utilizará este valor como criterio de optimización para encontrar la ruta de menor tiempo. 

```python
def construir_grafo_c(df: pd.DataFrame,
                      penalizacion_terrestre: float = 50.0) -> nx.DiGraph:
    """
    Literal c:
      - Arcos con Nivel_Frio != 3 son ELIMINADOS del grafo.
      - Rutas terrestres llevan +50 al costo.
      - weight = tiempo (Dijkstra minimiza tiempo).
    """
    df_c = df[df['Nivel_Frio'] == 3].copy()
    df_c['Costo_c'] = df_c['Costo'].where(
        df_c['Modo_Transporte'] != 'Terrestre',
        df_c['Costo'] + penalizacion_terrestre
    )
    G = nx.DiGraph()
    for _, f in df_c.iterrows():
        G.add_edge(f['Origen'], f['Destino'],
                   weight    = f['Tiempo_Total'],
                   costo     = f['Costo_c'],
                   distancia = float(f['Distancia']),
                   tiempo    = f['Tiempo_Total'],
                   modo      = f['Modo_Transporte'],
                   nivel_frio= int(f['Nivel_Frio']))
    return G
```
Calcular ruta:

Esta función permite encontrar la ruta óptima entre una ciudad de origen y una ciudad de destino utilizando el algoritmo de **Dijkstra**. El criterio de optimización depende del atributo **weight** asignado previamente a cada arco del grafo, el cual puede representar el costo o el tiempo total según el caso.

Si no existe una conexión posible entre los nodos seleccionados o alguno de ellos no pertenece al grafo, la función retorna un mensaje indicando que no fue posible encontrar una ruta. En caso contrario, recorre los arcos que conforman la ruta encontrada y calcula el costo, distancia y tiempo total. Finalmente, retorna la secuencia de nodos de la ruta óptima junto con las métricas calculadas.

```python
def calcular_ruta(G: nx.DiGraph, origen: str, destino: str) -> dict:
    try:
        ruta = nx.shortest_path(G, origen, destino, weight='weight')
    except (nx.NetworkXNoPath, nx.NodeNotFound):
        print(f" No se encontró la ruta más corta entre {origen} y {destino}.")
        return {'encontrado': False, 'ruta': [],
                'costo': 0, 'distancia': 0, 'tiempo': 0}
 
    costo = distancia = tiempo = 0.0
    for u, v in zip(ruta, ruta[1:]):
        a = G[u][v]
        costo     += a['costo']
        distancia += a['distancia']
        tiempo    += a['tiempo']
 
    return {'encontrado': True, 'ruta': ruta,
            'costo': costo, 'distancia': distancia, 'tiempo': tiempo}
```
Imprimir resultado:

Esta función permite presentar los resultados obtenidos de la ruta óptima. Recibe como parámetros la información de la ruta encontrada y el título correspondiente al escenario que se esta solucionando.

En caso de que no exista una ruta posible bajo las restricciones establecidas, la función muestra un mensaje de error. De lo contrario, imprime la secuencia de ciudades que conforman el recorrido, la cantidad de arcos utilizados, el costo total, la distancia total recorrida y el tiempo total, expresado tanto en horas como en minutos.

```python
def imprimir_resultado(res: dict, titulo: str):
    sep = "─" * 65
    print(f"\n{sep}")
    print(f"  {titulo}")
    print(f"  {Origen}  →  {Destino}")
    print(sep)
    if not res['encontrado']:
        print(" Sin ruta posible con las restricciones dadas.")
        print(sep)
        return
    ruta_c = '  →  '.join(res['ruta'])
    print(f"  Ruta (ciudades) : {ruta_c}")
    print(f"  Arcos           : {len(res['ruta']) - 1}")
    print(f"  Costo total     : ${res['costo']:,.2f}")
    print(f"  Distancia total : {res['distancia']:,.1f} km")
    print(f"  Tiempo total    : {res['tiempo']:.2f} h  ({res['tiempo']*60:.0f} min)")
    print(sep)
```
Ejecución: 

En este bloque se realiza la ejecución completa del modelo desarrollado. Inicialmente, se define la ruta del archivo Excel que contiene la información de la red de distribución y se cargan los datos mediante la función `cargar_datos()`.

Posteriormente, se calculan los tiempos y costos asociados a cada arco utilizando las funciones `calcular_tiempo()` y `calcular_costo()`. Con esta información, se construye el grafo correspondiente al **literal b**, donde se busca la ruta de menor costo utilizando el algoritmo de Dijkstra, y se muestran los resultados obtenidos.

Finalmente, se repite el mismo proceso para el **literal c**, construyendo un grafo con la restriccion del nivel de frío y la penalización para los arcos cuyo modo de transporte es terrestre. En este caso, el criterio de optimización cambia, buscando la ruta de menor tiempo.

```python
ruta = 'Distribución_Vacunas.xlsx'
df   = cargar_datos(ruta)
 
df = calcular_tiempo(df)
df = calcular_costo(df)
 
# Literal b — Menor costo
G_b   = construir_grafo_b(df)
res_b = calcular_ruta(G_b, Origen, Destino)
imprimir_resultado(res_b, "Literal b — Menor costo")
 
# Literal c — Menor tiempo
G_c   = construir_grafo_c(df, penalizacion_terrestre=50.0)
res_c = calcular_ruta(G_c, Origen, Destino)
imprimir_resultado(res_c, "Literal c — Menor tiempo | NF=3 | +50 terrestre")
```
**Resultado:** 

Ruta de Menor costo

| Métrica | Resultado |
|--------|----------|
| Ruta | Atlanta → Houston → Phoenix → Las Vegas |
| Arcos | 3 |
| Costo total | $16,791.55 |
| Distancia total | 3,434.0 km |
| Tiempo total | 25.33 h (1520 min) |

Ruta de Menor tiempo

| Métrica | Resultado |
|--------|----------|
| Ruta | Atlanta → Seattle → Dallas → Washington DC → Las Vegas |
| Arcos | 4 |
| Costo total | $52,342.65 |
| Distancia total | 11,722.0 km |
| Tiempo total | 21.27 h (1276 min) |

</details>
<details>
<summary> Red Vial en Bogotá </summary>
Una empresa de mensajería urbana en Bogotá necesita calcular la ruta de menor costo para un mensajero en moto que parte desde la intersección 1 y debe llegar a la intersección 12. El costo de cada tramo no depende únicamente de la distancia recorrida, sino también de la velocidad del tráfico según la hora del día, la presencia de peajes en la vía y la cantidad de semáforos. 
<br><br>

**Nota:** Este ejercicio idealmente debería resolverse mediante una implementación propia del algoritmo de Dijkstra, debido a que el costo real de una ruta depende del estado dinámico de la red durante el recorrido. En particular, para calcular correctamente el costo de una alternativa sería necesario conocer la hora estimada de llegada a cada nodo, ya que el factor de congestión de una vía puede cambiar dependiendo del momento en que sea utilizada. Adicionalmente, la disponibilidad de las vías debe evaluarse dinámicamente, considerando posibles bloqueos o restricciones que puedan activarse durante el recorrido.

Para utilizar la librería `NetworkX` en este ejercicio se realizó una simplificación, asumiendo que el factor de congestión permanece constante durante todo el trayecto y que las restricciones de las vías son conocidas previamente antes de iniciar el recorrido. Sin embargo, esta aproximación puede generar diferencias frente a una solución completamente dinámica.

En la sección de **[Dijkstra](#dijkstra)**, se implementará nuevamente este problema utilizando un grafo dinámico construido desde cero, con el objetivo de comparar ambas aproximaciones y analizar la diferencia en la calidad de la solución obtenida en términos de costo.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20vial%20en%20Bogot%C3%A1.pdf" download>Enunciado Red Vial en Bogotá</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/base_datos_red_vial_bogota.xlsx" download>Base de Datos Red vial en Bogotá</a>

**Solución:** 

Importación de las librerías requeridas:

```python
import networkx as nx
import pandas as pd
import re
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
```
Importación y carga de datos base:

Carga la información inicial de la red vial desde el archivo Excel. Primero, se leen de manera independiente las diferentes hojas del archivo, donde cada una contiene información específica de la red, como distancias, velocidades, costos de peaje, cantidad de semáforos y restricciones de las vías.

Posteriormente, se construye un único **DataFrame** consolidado tomando como base la información de distancias y agregando las variables complementarias provenientes de las demás hojas. De esta manera, se integra toda la información necesaria en una sola estructura de datos, facilitando su posterior procesamiento y análisis para la construcción de la red vial.

```python
Excel_Path = 'base_datos_red_vial_bogota.xlsx'

df_dist = pd.read_excel(Excel_Path, sheet_name='Distancias')
df_vel  = pd.read_excel(Excel_Path, sheet_name='Velocidad')
df_pea  = pd.read_excel(Excel_Path, sheet_name='Peajes')
df_sem  = pd.read_excel(Excel_Path, sheet_name='Número de semaforos')
df_res  = pd.read_excel(Excel_Path, sheet_name='Restricciones')

df = df_dist.copy()
df['Velocidad base (Km/h)'] = df_vel['Velocidad base (Km/h)']
df['Costo por peaje ($)']   = df_pea['Costo por peaje ($)']
df['Semaforos']             = df_sem['Semaforos']
df['Restriccion']           = df_res['Restriccion']
```
Parsear restricciones:

Esta función permite transformar la información de la columna **Restriccion** de cada vía en una estructura más fácil de procesar durante el análisis de la red vial. Recibe como entrada el texto asociado a la restricción de una vía y lo convierte en una lista de tuplas con el formato **(tipo de restricción, hora inicial, hora final)**.

Si la vía tiene la restricción **Libre**, se asigna una disponibilidad completa durante las 24 horas del día. En el caso de que la restricción corresponda a **Obras**, se genera una tupla cuya hora de inicio y hora de fin es la misma indicando que la vía permanece cerrada de manera permanente. Para las restricciones de **Pico y placa**, se extraen los rangos horarios definidos en el texto y se genera una tupla individual para cada intervalo de tiempo durante el cual aplica la restricción.

```python
def _parsear_restriccion(texto):
    t = str(texto).strip()
    if t == 'Libre':
        return [('libre', 0, 24)]
    if 'Obras' in t:
        return [('obras', 0, 0)]
    franjas = re.findall(r'(\d+)-(\d+)', t)
    return [('pico_placa', int(a), int(b)) for a, b in franjas]
```
Datos base y constantes:

Este bloque permite transformar el **DataFrame** consolidado en una lista de tuplas denominada **vias**, donde cada elemento representa la información completa de una vía de la red vial. Cada tupla contiene los nodos de origen y destino, la distancia, la velocidad base, el costo del peaje, la cantidad de semáforos, el tipo de restricción, la franja horaria asociada y el nombre de la vía.

Para construir esta estructura, se recorren las filas del DataFrame y se aplica la función `_parsear_restriccion()`, con el objetivo de descomponer las restricciones de cada vía en las franjas horarias correspondientes y almacenarlos de manera organizada.

Posteriormente, se establecen las condiciones asociadas a los nodos de la red. El diccionario **nodos_evento** contiene los nodos que presentan bloqueos temporales debido a eventos, junto con sus respectivas franjas horarias de afectación. Por otro lado, **nodos_horario** define los nodos que únicamente permiten el paso durante determinados intervalos de tiempo.

Finalmente, se declaran las constantes utilizadas en el modelo. Estas incluyen el costo asociado al tiempo de recorrido (**Costo_Min**), el tiempo que se demora cada semáforo (**Demora_Sem**) y la lista **factores_congestion**, que contiene los factores de ajuste de velocidad según la franja horaria, permitiendo representar los cambios en las condiciones de tráfico.

```python
vias = []
for _, row in df.iterrows():
    o      = int(row['Nodo de origen'])
    d      = int(row['Nodo de destino'])
    dist   = float(row['Distancia (Km)'])
    vel    = float(row['Velocidad base (Km/h)'])
    pea    = float(row['Costo por peaje ($)'])
    sem    = int(row['Semaforos'])
    nombre = str(row['Via'])
    for tipo, h_ini, h_fin in _parsear_restriccion(row['Restriccion']):
        vias.append((o, d, dist, vel, pea, sem, tipo, h_ini, h_fin, nombre))

# ── Condiciones sobre nodos ──────────────────────────────────────
nodos_evento = {
    7: (8,  12),   
    4: (10, 14),  
}

nodos_horario = {
    9: (6,  16),
    8: (7,  20),
}

# ── Constantes ───────────────────────────────────────────────────
Costo_Min = 800   
Demora_Sem = 1.5    

factores_congestion = [
    (6,  7,  0.90),
    (7,  9,  0.50),
    (9,  16, 1.00),
    (16, 19, 0.50),
    (19, 24, 0.85),
]
```
Factor de congestión: 

Esta función permite determinar el factor de congestión asociado a una hora específica del día. Recibe como parámetros la hora y la lista de franjas horarias con sus respectivos factores de ajuste, y recorre cada intervalo para identificar en cuál de ellos se encuentra la hora ingresada.

En caso de que la hora no se encuentre dentro de ninguna franja establecida, retorna un valor de **1.0**, indicando que no se aplica ningún ajuste sobre la velocidad.

```python
def get_factor(hora, factores):
    """
    Retorna el factor de velocidad para la hora dada.

    Parámetros
    ----------
    hora    : int   — hora del día (0-23)
    factores: list  — [(h_ini, h_fin, factor), ...]

    Retorna
    -------
    float — factor de congestión
    """
    for h_ini, h_fin, factor in factores:
        if h_ini <= hora < h_fin:
            return factor
    return 1.0   
```
Cálcular costo de una vía:

Esta función permite calcular el costo total asociado a recorrer una vía en una hora específica del día. Inicialmente, obtiene el factor de congestión mediante la función `get_factor()` y lo utiliza para ajustar la velocidad base de la vía, obteniendo así la velocidad real de circulación.

Posteriormente, con la velocidad ajustada se calcula el costo asociado al tiempo de recorrido según la distancia de la vía. A este valor se le suma el costo fijo del peaje y el costo generado por la demora asociada al número de semáforos presentes en el tramo. Finalmente, la función retorna el costo total de transitar la vía.

```python
def calcular_costo(dist, vel_base, peaje, semaforos, hora):
    """
    Retorna el costo total de transitar la vía a la hora dada.

    Parámetros
    ----------
    dist      : float — distancia en km
    vel_base  : float — velocidad base en km/h
    peaje     : float — costo fijo del peaje en pesos
    semaforos : int   — número de semáforos en el tramo
    hora      : int   — hora del día (0-23)

    Retorna
    -------
    float — costo total en pesos
    """
    factor       = get_factor(hora, factores_congestion)
    velocidad    = vel_base * factor
    costo_tiempo = (dist / velocidad) * 60 * Costo_Min
    costo_sem    = semaforos * Demora_Sem * Costo_Min
    return costo_tiempo + peaje + costo_sem
```
Filtrar red vial:

Esta función permite determinar cuáles vías de la red se encuentran disponibles para circular en una hora específica, considerando todas las restricciones definidas en el problema. El proceso se realiza en dos etapas: identificación de vías bloqueadas y construcción de la red disponible.

En la primera etapa, se identifican los nodos que presentan restricciones temporales. Para ello, se recorre el diccionario `nodos_evento` y se verifica si la hora actual se encuentra dentro de la franja de bloqueo de cada nodo. De manera similar, se recorre el diccionario `nodos_horario` para identificar los nodos cuya hora actual no se encuentra dentro del horario permitido de circulación.

Posteriormente, se recorren todas las vías y se identifican aquellas que deben ser bloqueadas debido a alguna restricción. Una vía se considera no disponible si cumple al menos una de las siguientes condiciones: que sea una vía cerrada, que tenga pico y placa activo, que alguno de sus nodos esté bloqueado por un evento o se encuentre fuera de su horario permitido de circulación. Cada vía eliminada se almacena en el diccionario **eliminadas**, clasificándola según la causa que generó su bloqueo.

En la segunda etapa, se construye la lista de vías disponibles conservando únicamente aquellas que no fueron marcadas como bloqueadas. Para evitar duplicados que puedan aparecer cuando una vía tiene múltiples franjas de pico y placa, se lleva un registro de las aristas ya agregadas y se omite cualquier repetición.

Finalmente, la función retorna tres elementos: la lista de vías habilitadas para circular en la hora evaluada, el diccionario de vías eliminadas clasificadas por motivo de bloqueo y el conjunto de nodos que no se encuentran disponibles.

```python
def filtrar_red(vias, nodos_evento, nodos_horario, hora):
    """
    Parámetros
    ----------
    vias          : list — lista de tuplas con la información de cada vía
    nodos_evento  : dict — nodos bloqueados por evento con su franja de bloqueo
    nodos_horario : dict — nodos con horario de paso restringido
    hora          : int  — hora del día (0-23)

    Retorna
    -------
    disponibles : list — vías disponibles a la hora dada
    eliminadas  : dict — vías eliminadas clasificadas por motivo de bloqueo
    bloqueados  : set  — conjunto de nodos inaccesibles a la hora dada
    """
    eliminadas = {"obras": set(), "pico_placa": set(), "evento": set(), "horario": set()}

    # Nodos bloqueados por evento a esta hora
    nodos_bloq = {
        nodo for nodo, (hi, hf) in nodos_evento.items()
        if hi <= hora < hf
    }

    # Nodos fuera de su horario de paso permitido
    nodos_fuera = {
        nodo for nodo, (hi, hf) in nodos_horario.items()
        if not (hi <= hora < hf)
    }

    # ── PASO 1: marcar todas las aristas que deben bloquearse ─────────────────
    aristas_bloqueadas = set()

    for via in vias:
        o, d, dist, vel, peaje, sem, tipo, h_ini, h_fin, nombre = via

        if tipo == "obras":
            aristas_bloqueadas.add((o, d))
            eliminadas["obras"].add(nombre)

        elif tipo == "pico_placa" and h_ini <= hora < h_fin:
            aristas_bloqueadas.add((o, d))
            eliminadas["pico_placa"].add(nombre)

        if o in nodos_bloq or d in nodos_bloq:
            aristas_bloqueadas.add((o, d))
            eliminadas["evento"].add(nombre)

        if o in nodos_fuera or d in nodos_fuera:
            aristas_bloqueadas.add((o, d))
            eliminadas["horario"].add(nombre)

    # ── PASO 2: construir lista de vías disponibles ────────────────────────────
    disponibles = []
    vistas = set()

    for via in vias:
        o, d, dist, vel, peaje, sem, tipo, h_ini, h_fin, nombre = via

        clave = (o, d)

        if clave in aristas_bloqueadas:
            continue

        if clave in vistas:
            continue

        disponibles.append(via)
        vistas.add(clave)

    eliminadas = {k: sorted(v) for k, v in eliminadas.items()}

    return disponibles, eliminadas, nodos_bloq | nodos_fuera
```
Construcción del grafo:

Esta función construye un grafo dirigido a partir de las vías disponibles en un momento específico del tiempo. Cada nodo del grafo representa una intersección y cada arista representa una vía entre dos puntos de la red, incluyendo su costo asociado.

Para cada vía disponible, se calcula el costo de tránsito utilizando la función `calcular_costo()`, el cual depende de la distancia, la velocidad base, la presencia de peajes, el número de semáforos y la hora del día. Este valor se asigna como atributo **weight** de la arista, junto con las demás características de la vía.

```python
def construir_grafo(vias_disponibles, hora):
    """
    Construye un DiGraph con las vías disponibles y sus costos.

    Parámetros
    ----------
    vias_disponibles : list — vías filtradas
    hora             : int  — hora del día (0-23)

    Retorna
    -------
    nx.DiGraph con atributo 'weight' en cada arista
    """
    G = nx.DiGraph()
    for via in vias_disponibles:
        o, d, dist, vel, peaje, sem, *_ = via
        peso = calcular_costo(dist, vel, peaje, sem, hora)

        if G.has_edge(o, d):
            if peso < G[o][d]['weight']:
                G[o][d]['weight'] = peso
        else:
            G.add_edge(o, d, weight=peso,
                       dist=dist, vel=vel, peaje=peaje, sem=sem)

    return G
```
Análisis por horario:

Este bloque ejecuta el análisis completo de la red vial para los horarios definidos (6:00, 8:00, 11:00 y 18:00). Para cada hora, primero se filtra la red, identificando las vías que deben ser eliminadas según las restricciones activas en ese momento.

A continuación, se construye el grafo con las vías disponibles utilizando la función `construir_grafo`, y posteriormente se aplica el algoritmo de **Dijkstra** para encontrar la ruta de menor costo entre el nodo 1 y el nodo 12.

Si existe una ruta posible, se imprime la secuencia de nodos que la conforman junto con el costo total del recorrido en pesos. En caso contrario, se indica que no existe un camino disponible entre los nodos para esa hora específica.

```python
resultados = {}  

for hora in [6, 8, 11, 18]:
    print(f"\n{'='*58}")
    print(f"  HORA: {hora}:00")
    print(f"{'='*58}")

    disponibles, eliminadas, bloqueados = filtrar_red(
        vias, nodos_evento, nodos_horario, hora
    )

    print('Vias eliminadas:')
    hay_eliminadas = False
    for motivo, nombres in eliminadas.items():
        if nombres:
            hay_eliminadas = True
            print(f'  {motivo:12s}: {", ".join(nombres)}')
    if not hay_eliminadas:
        print('  (ninguna)')

    vias_por_nodo = sorted(set(eliminadas.get('evento', []) + eliminadas.get('horario', [])))

    print(f'Vias disponibles         : {len(disponibles)}')

    G = construir_grafo(disponibles, hora)

    try:
        ruta  = nx.dijkstra_path(G, 1, 12, weight='weight')
        costo = nx.dijkstra_path_length(G, 1, 12, weight='weight')
        print(f'Ruta optima  : {" -> ".join(map(str, ruta))}')
        print(f'Costo total  : ${costo:,.0f} pesos')
    except nx.NetworkXNoPath:
        print('No existe camino de 1 a 12 a esta hora.')
        resultados[hora] = None
```
**Resultado:**

Hora 6:00

| Métrica | Resultado |
|--------|----------|
| Vías eliminadas (Obra) | V16, V3, V8 |
| Vías eliminadas (Paso restringido) | V14, V6 |
| Vías disponibles | 14 |
| Ruta óptima | 1 → 3 → 4 → 5 → 6 → 12 |
| Costo total | $22,584 |

Hora 8:00

| Métrica | Resultado |
|--------|----------|
| Vías eliminadas (Obra) | V16, V3, V8 |
| Vías eliminadas (Pico y placa) | V1, V12, V17, V4, V6 |
| Vías eliminadas (Evento) | V11, V13, V8 |
| Vías disponibles | 9 |
| Ruta óptima | No existe ruta |
| Costo total | — |

Hora 11:00

| Métrica | Resultado |
|--------|----------|
| Vías eliminadas (Obra) | V16, V3, V8 |
| Vías eliminadas (Evento) | V11, V13, V3, V5, V7, V8 |
| Vías disponibles | 12 |
| Ruta óptima | 1 → 3 → 8 → 11 → 12 |
| Costo total | $17,371 |

Hora 18:00

| Métrica | Resultado |
|--------|----------|
| Vías eliminadas (Obra) | V16, V3, V8 |
| Vías eliminadas (Pico y placa) | V1, V10, V12, V17, V4 |
| Vías eliminadas (Paso restringido) | V10, V15, V16, V4 |
| Vías disponibles | 10 |
| Ruta óptima | 1 → 3 → 8 → 11 → 12 |
| Costo total | $25,143 |
</details>

<details>
<summary> Red de distribución energética </summary>
Hydrogen Logistics North America (HLNA) es una compañía de logística energética que coordina la distribución de hidrógeno verde y combustibles criogénicos (oxígeno e hidrógeno líquidos) entre centros de producción, plantas industriales y puertos de exportación distribuidos a lo largo de Estados Unidos y Canadá. Recientemente la empresa firmó un contrato para entregar un cargamento de 35 toneladas de hidrógeno verde desde su planta principal en Vancouver hacia el puerto de Chicago, donde será embarcado hacia Europa. Estos productos son altamente sensibles: requieren mantenimiento estricto de presión y temperatura, son inflamables y deben moverse por corredores logísticos certificados.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20red%20de%20distribuci%C3%B3n%20energ%C3%A9tica%20Estados%20Unidos%20.pdf" download>Enunciado red de distribución energética</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Datos%20red%20de%20distribuci%C3%B3n%20energ%C3%A9tica.xlsx" download>Datos red de distribución energética</a>

**Solución:** 

Importación de las librerías requeridas:

```python
import pandas as pd
import networkx as nx
```
Parámetros del modelo:

En primer lugar, definimos la ruta del archivo, junto con las ciudades de origen y destino, y el volumen de carga a transportar en toneladas.

Posteriormente, se definen los parámetros utilizados en la formulación del costo. El diccionario **Um** define el costo asociado por kilómetro para cada modo de transporte, **alfa** contiene los factores de ajuste asociados al tipo de vía y  **gamma** almacena los recargos porcentuales asociados a cada modo de transporte, los cuales serán aplicados en el último literal del ejercicio para ajustar el costo final del recorrido.

Adicionalmente, se define el costo del operador (**co**), el cual permite incorporar el costo asociado al tiempo de operación dentro del cálculo del costo total de transporte.

```python
Excel_Path = "Datos red de distribución energética.xlsx"

Origen = "Vancouver"
Destino = "Chicago"
Carga = 35  # toneladas

# Costo por kilómetro según modo de transporte (USD/Km)
Um = {
    "Terrestre": 4.20,
    "Ferroviario": 2.80,
    "Fluvial": 1.90
}

# Factor de ajuste por tipo de vía
alfa = {
    "Interestatal": 1.00,
    "Federal": 1.10,
    "Secundaria": 1.25,
    "Industrial": 0.95
}

# Costo por hora del operador (USD/h)
co = 38.0

# Recargo porcentual por modo de transporte
gamma = {
    "Terrestre": 0.20,
    "Ferroviario": 0.10,
    "Fluvial": 0.30
}
```
Carga de datos:

Este bloque permite cargar la información de la red logística desde el archivo Excel. Para ello, se lee la hoja **Rutas_Logistica** y se almacena su contenido en un **DataFrame**, donde cada fila representa una ruta disponible con sus respectivas características. 

```python
df = pd.read_excel(Excel_Path, sheet_name="Rutas_Logistica")
```
Construcción del grafo:

Este bloque permite construir el grafo dirigido de la red logística a partir de la información contenida en el **DataFrame**. Para cada registro de la tabla se extraen las características de la ruta, como nodo de origen, nodo de destino, distancia, velocidad promedio, tiempo de aduana, modo de transporte, capacidad máxima, tipo de vía, nivel de riesgo, emisiones, peaje e índice de confiabilidad.

Antes de agregar cada ruta al grafo, se validan las restricciones operativas establecidas en el problema. En primer lugar, se verifica que la capacidad máxima del arco sea suficiente para transportar la carga definida. Posteriormente, se descartan aquellas rutas que tengan una zona de riesgo alta y sean una vía secundaria, y finalmente se exige que el índice de confiabilidad de la ruta sea mayor o igual a 0.70. Si alguna condición no se cumple, la ruta es descartada mediante y no se incorpora al grafo.

Para las rutas que cumplen con todas las restricciones operativas, se calcula inicialmente el tiempo total de tránsito del arco, considerando tanto el tiempo requerido para recorrer la distancia de la ruta como el tiempo adicional asociado al proceso de aduana. 

Posteriormente, se determina el costo total de la ruta aplicando la fórmula establecida en el enunciado. Este cálculo integra el costo por kilómetro ajustado según el modo de transporte y el tipo de vía, el costo del operador asociado al tiempo total de operación, los costos fijos de peaje y el costo adicional relacionado con las emisiones de CO₂ generadas durante el recorrido.

Finalmente, cada ruta válida se agrega al grafo como una arista dirigida, almacenando como atributos la distancia, el tiempo total, el costo calculado y el modo de transporte. 

```python
G = nx.DiGraph()

for _, row in df.iterrows():

    origen = row["Origen"]
    destino = row["Destino"]
    dist = float(row["Distancia_Km"])
    vel = float(row["Velocidad_Promedio_KmH"])
    t_aduana = float(row["Tiempo_Aduana_H"])
    modo = row["Modo_Transporte"]
    cap = float(row["Capacidad_Maxima_Ton"])
    via = row["Tipo_Via"]
    riesgo = row["Zona_Riesgo"]
    emision = float(row["Emision_CO2_Kg/Km"])
    peaje = float(row["Peaje_USD"])
    confiab = float(row["Indice_Confiabilidad"])

    # ------ Restricciones operativas ------
    # 1) Capacidad del arco >= volumen de la carga
    if cap < Carga:
        continue

    # 2) No usar arcos que sean Riesgo Alto y vía Secundaria
    if riesgo == "Alta" and via == "Secundaria":
        continue

    # 3) Confiabilidad >= 0.70
    if confiab < 0.70:
        continue

    # ------ Tiempo total del arco ------
    # tij = tiempo de viaje + tiempo de aduana
    tij = (dist / vel) + t_aduana

    # ------ Costo del arco según fórmula ------
    # cij = Um * alfa_via * dij + co * tij + Peajeij + 0.12 * eij
    cij = Um[modo] * alfa[via] * dist + co * tij + peaje + 0.12 * emision

    # ------ Agregar arco al grafo ------
    G.add_edge(
        origen,
        destino,
        distancia=dist,
        tiempo=tij,
        costo=cij,
        modo=modo
    )
```
Ruta de menor costo:

Utiliza el algoritmo de **Dijkstra** sobre el grafo construido para encontrar la ruta óptima de menor costo entre la ciudad de origen (**Vancouver**) y la ciudad de destino (**Chicago**). 

Una vez obtenida la secuencia de nodos que conforman la ruta óptima, se recorren los arcos correspondientes dentro del grafo para calcular el costo total, la distancia total y el tiempo total de tránsito.

Finalmente, se imprime la ruta encontrada, junto con los valores calculados de costo, distancia y tiempo.

```python
ruta_costo = nx.shortest_path(G, source=Origen, target=Destino, weight="costo")

# Acumular métricas a lo largo de la ruta
costo_total_2 = 0.0
distancia_total_2 = 0.0
tiempo_total_2 = 0.0

for k in range(len(ruta_costo) - 1):
    u = ruta_costo[k]
    v = ruta_costo[k + 1]
    arco = G[u][v]

    costo_total_2 += arco["costo"]
    distancia_total_2 += arco["distancia"]
    tiempo_total_2 += arco["tiempo"]

print("=" * 70)
print("PUNTO 2 - RUTA DE MENOR COSTO (Vancouver -> Chicago)")
print("=" * 70)

print("Ruta:")
print(" -> ".join(ruta_costo))

print()

print(f"Costo total:      {costo_total_2:,.2f} USD")
print(f"Distancia total:  {distancia_total_2:,.2f} Km")
print(f"Tiempo total:     {tiempo_total_2:,.2f} h")

print()
```
Construcción del grafo con recargo: 

Construye un nuevo grafo dirigido incorporando un ajuste adicional sobre los costos de transporte. La estructura del grafo mantiene las mismas restricciones operativas definidas previamente: capacidad suficiente para transportar la carga, rutas que no presenten un nivel de riesgo alto y cuyo tipo de vía no corresponda a una vía secundaria, y un índice de confiabilidad mínimo requerido.

Para cada ruta que cumple las restricciones, se calcula inicialmente el costo base del arco utilizando la misma formula del enunciado, considerando el modo de transporte, el tipo de vía, la distancia recorrida, el tiempo de operación, los peajes y el componente asociado a emisiones de CO₂.

Posteriormente, se aplica un recargo porcentual según el modo de transporte mediante el factor **gamma**, ajustando el costo del arco.

Este incremento representa el sobrecosto asociado al transporte urgente del medicamento biotecnológico. 

```python
G_t = nx.DiGraph()

for _, row in df.iterrows():

    origen = row["Origen"]
    destino = row["Destino"]
    dist = float(row["Distancia_Km"])
    vel = float(row["Velocidad_Promedio_KmH"])
    t_aduana = float(row["Tiempo_Aduana_H"])
    modo = row["Modo_Transporte"]
    cap = float(row["Capacidad_Maxima_Ton"])
    via = row["Tipo_Via"]
    riesgo = row["Zona_Riesgo"]
    emision = float(row["Emision_CO2_Kg/Km"])
    peaje = float(row["Peaje_USD"])
    confiab = float(row["Indice_Confiabilidad"])

    # 1) Capacidad del arco >= volumen de la carga
    if cap < Carga:
        continue

    # 2) No usar arcos que sean Riesgo Alto y vía Secundaria
    if riesgo == "Alta" and via == "Secundaria":
        continue

    # 3) Confiabilidad >= 0.70
    if confiab < 0.70:
        continue

    tij = (dist / vel) + t_aduana

    cij = Um[modo] * alfa[via] * dist + co * tij + peaje + 0.12 * emision

    # Costo con recargo según modo de transporte
    cij_recargo = cij * (1 + gamma[modo])

    G_t.add_edge(
        origen,
        destino,
        distancia=dist,
        tiempo=tij,
        costo=cij_recargo,
        modo=modo
    )
```
Ruta de menor tiempo:

Utiliza el algoritmo de **Dijkstra** sobre el grafo construido para encontrar la ruta óptima de menor tiempo entre la ciudad de origen (**Vancouver**) y la ciudad de destino (**Chicago**). 

Una vez obtenida la secuencia de nodos que conforman la ruta óptima, se recorren los arcos correspondientes dentro del grafo para calcular el costo total, la distancia total y el tiempo total de tránsito.

Finalmente, se imprime la ruta encontrada, junto con los valores calculados de costo, distancia y tiempo.

```python
# Ruta de menor tiempo
ruta_tiempo = nx.shortest_path(G_t, source=Origen, target=Destino, weight="tiempo")

costo_total_3 = 0.0
distancia_total_3 = 0.0
tiempo_total_3 = 0.0

for k in range(len(ruta_tiempo) - 1):
    u = ruta_tiempo[k]
    v = ruta_tiempo[k + 1]

    arco = G_t[u][v]

    costo_total_3 += arco["costo"]
    distancia_total_3 += arco["distancia"]
    tiempo_total_3 += arco["tiempo"]

print("=" * 70)
print("PUNTO 3 - RUTA DE MENOR TIEMPO (Vancouver -> Chicago) con recargo")
print("=" * 70)

print("Ruta:")
print(" -> ".join(ruta_tiempo))

print()

print(f"Tiempo total:     {tiempo_total_3:,.2f} h")
print(f"Costo total:      {costo_total_3:,.2f} USD")
print(f"Distancia total:  {distancia_total_3:,.2f} Km")

print("=" * 70)
```
**Resultado:**
Ruta de menor costo (Vancouver → Chicago)

| Métrica | Resultado |
|--------|------|
| Ruta | Vancouver → Toronto → Chicago |
| Costo total | 18,518.88 USD |
| Distancia total | 5,152.00 Km |
| Tiempo total | 67.26 h |

Ruta de menor tiempo (Vancouver → Chicago)

| Métrica | Resultado |
|--------|------|
| Ruta | Vancouver → Seattle → Salt Lake City → Denver → Kansas City → Chicago |
| Costo total | 22,627.49 USD |
| Distancia total | 4,278.40 Km |
| Tiempo total | 66.42 h |
</details>
  
### Dijkstra

---

Las librerías como NetworkX proporcionan implementaciones eficientes del algoritmo de Dijkstra, las cuales son especialmente útiles cuando se trabaja con grafos estáticos y completamente definidos. No obstante, existen situaciones en las que estas herramientas resultan insuficientes, particularmente cuando el grafo varía dinámicamente durante la ejecución del algoritmo.

Este tipo de escenarios ocurre, por ejemplo, cuando la disponibilidad de una vía depende de horarios específicos, lo que implica que la estructura del grafo cambia según el momento en que el nodo es alcanzado. De igual forma, puede suceder que el costo de un arco no sea fijo sino que varíe según el modo de transporte utilizado en el tramo anterior, como ocurre cuando se aplica un recargo adicional al realizar un transbordo entre modo ferroviario y terrestre, haciendo que el peso del arco dependa de cómo se llegó al nodo y no únicamente de sus atributos propios. Una situación similar se presenta cuando cada nodo intermedio de la red tiene un límite máximo de riesgo permitido, de modo que la viabilidad de continuar por un arco solo puede verificarse conociendo el camino exacto tomado hasta ese momento. En todos estos casos la lógica de exploración debe adaptarse en cada paso del algoritmo, lo que hace necesario contar con una implementación propia.

En todos estos contextos, la exploración del grafo debe adaptarse dinámicamente en cada iteración del algoritmo, lo que requiere una implementación personalizada en lugar de depender de soluciones predefinidas.

A continuación, se presenta la implementación paso a paso del algoritmo de Dijkstra para ejercicios con grafos dinamicos.

#### Pseudocodigo del algoritmo de Dijkstra

---

El algoritmo inicia con la configuración de las estructuras necesarias para realizar la búsqueda de la ruta más corta. A cada nodo del grafo se le asigna una distancia inicial infinita, representando que aún no existe un camino conocido hacia ellos. La única excepción es el nodo de origen, cuya distancia se establece en cero debido a que es el punto de partida.

Adicionalmente, se crea un registro de predecesores para cada nodo, el cual permitirá reconstruir la ruta óptima una vez finalice el algoritmo. También se define un conjunto de nodos visitados, inicialmente vacío, encargado de almacenar aquellos nodos cuyo costo mínimo ya fue determinado.

Finalmente, se inicializa una cola de prioridad, que constituye la estructura principal del algoritmo, ya que permite seleccionar en cada iteración el nodo con menor distancia acumulada y explorar sus vecinos de manera eficiente. Esta cola comienza únicamente con el nodo origen y una distancia asociada de cero.

```text
1: for each node n ∈ V do
2:     distance[n] ← ∞
3:     predecessor[n] ← None
4: end for

5: distance[origin] ← 0
6: visited ← ∅
7: priority_queue ← {(0, origin)}
```
En cada iteración, se extrae de la cola de prioridad el nodo con menor distancia acumulada. Si dicho nodo ya fue procesado anteriormente, se descarta y se continúa con la siguiente iteración. En caso contrario, se marca como visitado y se procede a explorar sus vecinos. Este procedimiento garantiza que cada nodo sea evaluado una única vez y siguiendo el orden de menor costo acumulado.

```text
8: while priority_queue is not empty do
9:     (current_dist, u) ← extract-min(priority_queue)

10:    if u ∈ visited then
11:        continue
12:    end if

13:    visited ← visited ∪ {u}
```

Para cada vecino del nodo actual se calcula la distancia tentativa como la suma de la distancia acumulada hasta el nodo actual y el peso del arco que conecta ambos nodos. Si esta nueva distancia es menor que la distancia registrada para ese vecino, se actualiza el valor mínimo encontrado, se registra el nodo actual como su predecesor y se incorpora nuevamente a la cola de prioridad con la nueva distancia. De esta forma el algoritmo siempre explora primero los caminos más prometedores.

```text
14:    for each neighbor v of u do
15:        tentative_dist ← distance[u] + weight(u, v)

16:        if tentative_dist < distance[v] then
17:            distance[v] ← tentative_dist
18:            predecessor[v] ← u
19:            insert (tentative_dist, v) into priority_queue
20:        end if
21:    end for
22: end while
```

Una vez finaliza la ejecución del algoritmo, la ruta óptima se reconstruye utilizando el registro de predecesores almacenado durante el proceso de búsqueda. Para ello, se inicia desde el nodo destino y se retrocede siguiendo la cadena de predecesores hasta llegar al nodo origen. Debido a que este recorrido se realiza en sentido inverso, la secuencia obtenida se organiza nuevamente para mostrar la ruta en el orden correcto desde el origen hasta el destino.

```text
23: path ← empty list
24: current_node ← destination

25: while current_node is not None do
26:     insert current_node at beginning of path
27:     current_node ← predecessor[current_node]
28: end while

29: return path, distance[destination]
```
#### Ejercicios Dijkstra

---


En esta sección se desarrollan ejercicios de ruta más corta que requieren implementar el algoritmo de Dijkstra de forma manual, adaptando su lógica de exploración a restricciones que no pueden resolverse con librerías estándar como NetworkX. Cada ejercicio introduce una modificación diferente al estado del algoritmo, lo que permite comprender cómo extender Dijkstra más allá de su formulación clásica para abordar problemas reales con condiciones dinámicas sobre los arcos y los nodos.

<details>
<summary> Red de Distribución RiskShield </summary>
RiskShield Logistics es una empresa especializada en el transporte de sustancias químicas peligrosas que debe entregar un cargamento de peróxido de hidrógeno industrial desde Seattle hasta Houston. La empresa opera una red de rutas terrestres y ferroviarias a lo largo de Estados Unidos, y debe encontrar la ruta de menor costo garantizando que la exposición al riesgo operativo a lo largo del recorrido no supere los límites establecidos para cada ciudad intermedia.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20de%20Distribuci%C3%B3n%20RiskShield.pdf" download>Enunciado Red de Distribución RiskShield</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/red_RiskShield.xlsx" download>Red RiskShield</a>

**Solución:** 
Importación de librerías requeridas:

```python
import pandas as pd
import heapq
```
Parámetros del modelo:

Define los parametros necesarios para el ejercicio. Se establece la ruta del archivo Excel, las ciudades de origen y destino del recorrido. Luego se define el diccionario `Um` con el costo por kilómetro según el modo de transporte, y la constante `Co` con el costo por hora del operador en dólares, ambos necesarios para aplicar la fórmula de costo de cada arco.

```python
Excel_Path = "red_RiskShield.xlsx"
Origen     = "Seattle"
Destino    = "Houston"

Um = {"Terrestre": 4.20, "Ferroviario": 2.80, "Fluvial": 1.70}
Co = 38.0
```
Carga de datos:

Se leen las dos hojas del archivo Excel: `Red_Rutas` con la información de cada arco de la red y `Nodos` con los límites de riesgo de cada ciudad. A partir de la hoja de nodos se construye el diccionario `riesgo_max`, que asocia cada ciudad con su límite máximo de riesgo acumulado permitido.

```python
df_rutas = pd.read_excel(Excel_Path, sheet_name="Red_Rutas")
df_nodos = pd.read_excel(Excel_Path, sheet_name="Nodos")

riesgo_max = dict(zip(df_nodos["Nodo"], df_nodos["Riesgo Maximo"]))
```
Construcción del grafo:

Construye el grafo como un diccionario de adyacencia donde cada ciudad de origen tiene asociada una lista de tuplas con la información de sus arcos salientes. Por cada fila del DataFrame se extraen los atributos del arco, se calcula el tiempo de recorrido dividiendo la distancia entre la velocidad promedio, y se aplica la fórmula de costo. Cada arco queda almacenado como una tupla con el destino, la distancia, el tiempo de recorrido, el costo total y el índice de riesgo. Finalmente, se construye el conjunto `nodos` con todas las ciudades presentes en la red, tanto como origen como destino.

```python
grafo = {}

for _, row in df_rutas.iterrows():
    o      = row["Origen"]
    d      = row["Destino"]
    dist   = float(row["Distancia (Km)"])
    vel    = float(row["Velocidad (Km/h)"])
    modo   = row["Modo Transporte"]
    peaje  = float(row["Peaje (USD)"])
    riesgo = float(row["Indice Riesgo"])

    tij = dist / vel
    cij = Um[modo] * dist + Co * tij + peaje

    if o not in grafo:
        grafo[o] = []

    grafo[o].append((d, dist, tij, cij, riesgo))

nodos = set(df_rutas["Origen"]) | set(df_rutas["Destino"])
```
Inicialización de estructuras de datos:

Inicializa las estructuras de datos necesarias para el algoritmo. El diccionario `costo` asigna un valor infinito a cada nodo, representando que al inicio del algoritmo no se ha encontrado ningún camino hacia ellos y cualquier ruta que se descubra durante la exploración será necesariamente menor. El diccionario `predecesor` se inicializa en `None` para todos los nodos, y será el que permita reconstruir la ruta óptima al finalizar el algoritmo. Adicionalmente, dado que este ejercicio extiende el algoritmo clásico de Dijkstra, se inicializan también los diccionarios `riesgo_ac`, `distancia` y `tiempo`, que almacenarán el riesgo acumulado, la distancia recorrida y el tiempo de tránsito correspondientes al mejor camino encontrado hacia cada nodo.

```python
costo       = {n: float("inf") for n in nodos}
predecesor  = {n: None for n in nodos}
riesgo_ac   = {n: float("inf") for n in nodos}
distancia   = {n: 0.0 for n in nodos}
tiempo      = {n: 0.0 for n in nodos}
```
Inicialización del origen:

Inicializa el punto de partida del algoritmo. El costo y el riesgo acumulado del nodo origen se establecen en cero, ya que no se ha recorrido ningún arco todavía. Se inicializa el conjunto `visitados` vacío y se inserta el nodo origen en la cola de prioridad como único elemento inicial, con costo, riesgo, distancia y tiempo en cero, marcando así el punto de partida desde el cual comenzará la exploración.

```python
costo[Origen]     = 0.0
riesgo_ac[Origen] = 0.0

visitados = set()

cola_prioridad = [
    (0.0, Origen, 0.0, 0.0, 0.0)  # (costo, nodo, riesgo_ac, dist_ac, tiempo_ac)
]
```
Ciclo principal del algoritmo:

El ciclo se ejecuta mientras la cola de prioridad no esté vacía. En cada iteración se extrae el nodo con menor costo acumulado usando `heapq.heappop`, obteniendo junto con él el riesgo acumulado, la distancia recorrida y el tiempo transcurrido hasta llegar a ese nodo. Si el nodo ya fue visitado se descarta y se continúa con la siguiente iteración; de lo contrario se marca como visitado y se procede a explorar sus vecinos.

Para cada vecino del nodo actual, se calcula el nuevo riesgo acumulado sumando el riesgo del arco actual al riesgo acumulado hasta el nodo actual. Antes de continuar se verifica la restricción operativa: si el nuevo riesgo supera el límite permitido para el nodo vecino, el arco se descarta. En caso contrario se calcula el costo tentativo sumando el costo acumulado hasta el nodo actual más el costo del arco. Si este costo tentativo es menor que el mejor costo registrado para ese vecino, se actualizan el costo, el predecesor, el riesgo acumulado, la distancia y el tiempo, y el vecino se inserta en la cola de prioridad con los nuevos valores para ser explorado en iteraciones posteriores.

```python
while cola_prioridad:

    # Extraer nodo de menor costo
    costo_actual, u, riesgo_actual, dist_actual, tiempo_actual = heapq.heappop(cola_prioridad)

    if u in visitados:
        continue

    # Marcar nodo como visitado
    visitados.add(u)

    # Explorar vecinos
    for v, dist_arco, tij, cij, riesgo_arco in grafo.get(u, []):

        nuevo_riesgo = riesgo_actual + riesgo_arco

        # Verificar límite de riesgo del nodo destino
        limite = riesgo_max.get(v, 1.0)
        if nuevo_riesgo > limite:
            continue

        # Calcular costo tentativo
        costo_tentativo = costo_actual + cij

        # Actualizar información del vecino si se encontró un camino de menor costo
        if costo_tentativo < costo[v]:
            costo[v]      = costo_tentativo
            predecesor[v] = u
            riesgo_ac[v]  = nuevo_riesgo
            distancia[v]  = dist_actual + dist_arco
            tiempo[v]     = tiempo_actual + tij

            # Insertar el vecino en la cola de prioridad
            heapq.heappush(
                cola_prioridad,
                (costo_tentativo, v, nuevo_riesgo,
                 dist_actual + dist_arco, tiempo_actual + tij)
            )
```
Reconstrucción de la ruta:

Una vez finalizado el ciclo principal, se reconstruye la ruta óptima utilizando el diccionario `predecesor`. Partiendo desde el nodo destino, se recorre la cadena de predecesores hacia atrás insertando cada nodo al inicio de la lista `ruta`, de modo que al finalizar el recorrido la secuencia quede ordenada de origen a destino. El ciclo termina cuando `nodo_actual` es `None`, lo que indica que se ha llegado al nodo origen, cuyo predecesor fue inicializado en `None`.

```python
ruta = []
nodo_actual = Destino

while nodo_actual is not None:

    ruta.insert(0, nodo_actual)              
    nodo_actual = predecesor[nodo_actual]    
```
Resultados del algoritmo

Imprime los resultados del algoritmo. Si el costo del nodo destino sigue siendo infinito, significa que no se encontró ninguna ruta válida que respete los límites de riesgo acumulado en todos los nodos intermedios. En caso contrario, se imprime la secuencia ordenada de ciudades que conforman la ruta óptima, el número de arcos recorridos, el costo total en dólares, la distancia total en kilómetros, el tiempo estimado en horas y el riesgo acumulado final de la ruta.

```python
print("=" * 65)
print("  RISKSHIELD LOGISTICS — Ruta de menor costo con riesgo")
print("=" * 65)

if costo[Destino] == float("inf"):
    print("  No existe ruta válida entre los nodos dados.")
else:
    print(f"\n  Ruta              : {' → '.join(ruta)}")
    print(f"  Arcos             : {len(ruta) - 1}")
    print(f"  Costo total       : ${costo[Destino]:,.2f} USD")
    print(f"  Distancia total   : {distancia[Destino]:,.1f} km")
    print(f"  Tiempo total      : {tiempo[Destino]:.2f} h")
    print(f"  Riesgo acumulado  : {riesgo_ac[Destino]:.4f}")
    print("=" * 65)
```
**Resultado obtenido:**

| Parámetro | Resultado |
|-----------|-----------|
| Ruta óptima | Seattle → Portland → Boise → Salt Lake → Denver → Oklahoma City → Dallas → Houston |
| Número de arcos recorridos | 7 |
| Costo total | $17,215.59 USD |
| Distancia total | 3,830.0 km |
| Tiempo total estimado| 45.12 h |
| Riesgo acumulado final | 1.3600 |

</details>
<details>
<summary> Red de Distribución FreshChain </summary>
FreshChain Logistics es una empresa especializada en la distribución de productos farmacéuticos refrigerados que debe transportar un cargamento de medicamentos desde Monterrey hasta Cartagena a través de una red multimodal que combina tramos terrestres, ferroviarios y fluviales. Dado que los productos requieren condiciones controladas de temperatura, cada vez que se realiza un cambio de modo de transporte se incurre en un recargo adicional por concepto de manipulación y re-acondicionamiento de la carga en los puntos de transferencia, haciendo que el costo efectivo de un arco no sea un valor fijo sino que dependa del modo de transporte utilizado en el tramo inmediatamente anterior.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20de%20Distribuci%C3%B3n%20FreshChain.pdf" download>Enunciado Red de Distribución FreshChain</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/red_FreshChain.xlsx" download>Red FreshChain</a>

**Solución:** 

Importación de librerías:

```python
import pandas as pd
import heapq
```
Parámetros del modelo:

Define los parámetros necesarios para el modelo. Se establece la ruta del archivo Excel, las ciudades de origen y destino del recorrido. Luego se define el diccionario `Um` con el costo por kilómetro según el modo de transporte, la constante `Co` con el costo por hora del operador en dólares, y la constante Recargo_Transbordo que representa el costo adicional de en el que se incurre cuando se realiza un transbordo entre modos de transporte distintos, cubriendo los gastos de manipulación y re-acondicionamiento de la carga en el punto de transferencia.

```python
Excel_Path         = "red_FreshChain.xlsx"
Origen             = "Monterrey"
Destino            = "Cartagena"

Um                 = {"Terrestre": 3.80, "Ferroviario": 2.50, "Fluvial": 1.70}
Co                 = 38.0
Recargo_Transbordo = 120.0
```
Carga de datos:

Se lee la hoja `Red_Multimodal` del archivo Excel y se almacena en un DataFrame que contiene toda la información de la red de rutas disponibles, incluyendo el origen, destino, distancia, velocidad, modo de transporte y peaje de cada arco.

```python
df = pd.read_excel(Excel_Path, sheet_name="Red_Multimodal")
```

Construcción del grafo:

Construye el grafo como un diccionario de adyacencia donde cada ciudad de origen tiene asociada una lista de tuplas con la información de sus arcos salientes. Por cada fila del DataFrame se extraen los atributos del arco, se calcula el tiempo de recorrido dividiendo la distancia entre la velocidad promedio, y se aplica la fórmula de costo base sin incluir aún el recargo por transbordo, ya que este solo puede determinarse durante la exploración del algoritmo al conocer el modo del arco anterior. Cada arco queda almacenado como una tupla con el destino, la distancia, el tiempo, el costo base y el modo de transporte. Finalmente, se construye el conjunto `nodos` con todas las ciudades presentes en la red.

```python
grafo = {}

for _, row in df.iterrows():
    o    = row["Origen"]
    d    = row["Destino"]
    dist = float(row["Distancia (Km)"])
    vel  = float(row["Velocidad (Km/h)"])
    modo = row["Modo_Transporte"]
    pea  = float(row["Peaje (USD)"])

    tij = dist / vel
    cij = Um[modo] * dist + Co * tij + pea

    if o not in grafo:
        grafo[o] = []

    grafo[o].append((d, dist, tij, cij, modo))

nodos = set(df["Origen"]) | set(df["Destino"])
```

Inicialización de las estructuras de datos:

En el algoritmo de Dijkstra cada nodo tiene asociado un único costo porque el peso de los arcos es fijo e independiente del camino recorrido. En este ejercicio esto no es suficiente, ya que el costo de llegar a un nodo puede variar según el modo de transporte del tramo anterior: si hubo un transbordo se aplica un recargo adicional, y si no lo hubo se mantiene el costo base.

Por esta razón, el concepto de estado del algoritmo se extiende más allá del nodo. En el Dijkstra clásico el estado corresponde únicamente a la ciudad en la que se encuentra el algoritmo. En este caso, el estado está definido por la combinación `(nodo, modo_anterior)`, es decir, no basta con conocer la ciudad actual, sino también el modo de transporte utilizado para llegar a ella, ya que esta información determina si el siguiente arco tendrá o no un recargo por transbordo.

Esto implica que una misma ciudad puede representar estados diferentes. Por ejemplo, llegar a una ciudad en tren es un estado distinto a llegar a la misma ciudad en camión, debido a que el costo de continuar la ruta puede cambiar dependiendo del modo de transporte anterior.

Esta extensión del estado implica que no es posible inicializar el diccionario `costo` con valor infinito para todos los estados desde el principio, como ocurre en el algoritmo clásico, porque los estados posibles `(nodo, modo)` no se conocen previamente. Estos estados se generan dinámicamente a medida que el algoritmo explora la red y descubre nuevas combinaciones de nodo y modo de transporte.

El estado del nodo origen se inicializa con costo cero y modo de transporte anterior `None`, ya que al partir no se ha recorrido ningún arco previo y por tanto no existe un modo de transporte anterior. Se inicializa el conjunto `visitados` vacío y se inserta el estado inicial en la `cola_prioridad` con costo, distancia y tiempo en cero.

Finalmente, se inicializa el diccionario auxiliar `registro_ruta`, que no forma parte del pseudocódigo pero es necesario en este ejercicio para almacenar la secuencia de ciudades asociada a cada estado `(nodo, modo)`. Debido a que los estados son compuestos, reconstruir la ruta siguiendo únicamente los predecesores requeriría conocer en cada paso no solo el nodo anterior sino también el modo con el que se llegó a él. El diccionario `registro_ruta` evita esta complejidad almacenando directamente la secuencia completa de ciudades cada vez que se actualiza un estado, permitiendo recuperar la ruta óptima al finalizar el algoritmo.

```python
# ── Inicialización de costos y predecesores ──────────
# El estado de cada nodo incluye el modo de transporte con el que se llegó,
# por lo que costo y predecesor se indexan por (nodo, modo_anterior)

costo = {}
predecesor = {}

# ── Inicialización del origen ─────────────────────────
# El origen no tiene modo anterior (None) y su costo es 0

costo[(Origen, None)] = 0.0
predecesor[(Origen, None)] = None

visitados = set()  

cola_prioridad = [
    (0.0, Origen, None, 0.0, 0.0)
]  
# (costo, nodo, modo_prev, dist_ac, tiempo_ac)

# Registro auxiliar para reconstruir la ruta completa
registro_ruta = {
    (Origen, None): [Origen]
}
```

Ciclo principal del algoritmo:

El ciclo se ejecuta mientras la cola de prioridad no esté vacía. En cada iteración se extrae el estado de menor costo, obteniendo la ciudad actual, el modo de transporte con el que se llegó, el costo acumulado, la distancia recorrida y el tiempo transcurrido hasta ese punto.

Si el estado `(nodo, modo_anterior)` ya fue visitado, se descarta y se continúa con la siguiente iteración. De lo contrario, el estado se marca como visitado y se procede a explorar sus vecinos.

Para cada vecino del nodo actual, se verifica si el modo de transporte del arco actual es diferente al modo con el que se llegó al nodo actual. Si existe una diferencia entre modos, se aplica el recargo por transbordo; en caso contrario, el recargo es cero. Con esto se calcula el costo tentativo sumando el costo acumulado hasta el nodo actual, el costo base del arco y el recargo correspondiente.

A diferencia del algoritmo clásico, donde se compara contra el costo del nodo vecino, en este caso la comparación se realiza contra el costo del estado `(vecino, modo)`, debido a que un mismo nodo puede tener costos diferentes dependiendo del modo de transporte utilizado para llegar a él.

Si el costo tentativo mejora el mejor costo registrado para ese estado, se actualizan el costo, el predecesor y la secuencia de ciudades almacenada en `registro_ruta`. Finalmente, el nuevo estado se inserta en la cola de prioridad con sus valores actualizados para continuar la exploración en iteraciones posteriores.

```python
while cola_prioridad:

    # Extraer estado de menor costo
    costo_actual, u, modo_prev, dist_actual, tiempo_actual = heapq.heappop(cola_prioridad)

    # Si el estado (nodo, modo_anterior) ya fue visitado, descartar
    estado = (u, modo_prev)

    if estado in visitados:
        continue

    # Marcar estado como visitado
    visitados.add(estado)

    # Explorar vecinos
    for v, dist_arco, tij, cij_base, modo in grafo.get(u, []):

        # Recargo si existe transbordo entre modos diferentes
        recargo = Recargo_Transbordo if (
            modo_prev is not None and modo != modo_prev
        ) else 0.0

        # Calcular costo tentativo incluyendo recargo
        costo_tentativo = costo_actual + cij_base + recargo

        estado_vecino = (v, modo)

        # Actualizar si se encontró un camino de menor costo
        if estado_vecino not in costo or costo_tentativo < costo[estado_vecino]:

            costo[estado_vecino] = costo_tentativo       
            predecesor[estado_vecino] = estado            

            registro_ruta[estado_vecino] = (
                registro_ruta[estado] + [v]
            )

            # Insertar estado actualizado en la cola de prioridad
            heapq.heappush(
                cola_prioridad,
                (
                    costo_tentativo,
                    v,
                    modo,
                    dist_actual + dist_arco,
                    tiempo_actual + tij
                )
            )
```
Selección del estado de destino óptimo:

Como el destino puede haberse alcanzado a través de distintos modos de transporte, pueden existir múltiples estados `(Destino, modo)` dentro del diccionario `costo`, cada uno con un costo acumulado diferente dependiendo de la ruta recorrida y los transbordos realizados.

Este bloque selecciona entre todos los estados asociados al nodo destino aquel que tenga el menor costo acumulado. Dicho estado corresponde a la mejor alternativa encontrada por el algoritmo y representa la ruta óptima considerando los costos base y los recargos por transbordo.

```python
# Buscar el estado del destino con menor costo entre todos los modos de transporte
estado_destino = min(
    (e for e in costo if e[0] == Destino),
    key=lambda e: costo[e],
    default=None
)
```
Resultado algoritmo: 

Imprime los resultados obtenidos por el algoritmo. Si el destino no aparece en ninguno de los estados registrados en el diccionario `costo`, significa que el algoritmo no encontró ningún camino posible.

En caso contrario, dado que el nodo de destino pudo haberse alcanzado por distintos modos de transporte, se selecciona el que resultó en menor costo y se recupera la ruta optima desde el diccionadio `registro_ruta`. Posteriormente, se calcula la distancia total y el tiempo total recorriendo los arcos correspondientes a la ruta encontrada.

Además, se identifican los transbordos recorriendo la ruta arco por arco y comparando el modo de transporte de cada tramo con el modo de transporte utilizado en el tramo anterior. Cada vez que se detecta un cambio en el modo de transporte, se registra el transbordo indicando las ciudades involucradas y los modos de transporte entre los cuales ocurrió el cambio.

Finalmente, se imprime la secuencia de ciudades, el número de arcos recorridos, el costo total, la distancia acumulada, el tiempo estimado y la lista de transbordos detectados con su recargo asociado. Si la ruta no presenta cambios de modo de transporte, se muestra un mensaje indicando que no hubo transbordos.

```python
print("=" * 65)
print("  FRESHCHAIN LOGISTICS — Ruta de menor costo con transbordo")
print("=" * 65)

if estado_destino is None:
    print("  No existe ruta válida entre los nodos dados.")

else:
    ruta = registro_ruta[estado_destino]
    costo_total = costo[estado_destino]

    distancia_total = sum(
        next(
            dist for vec, dist, tij, cij, modo in grafo.get(ruta[i], [])
            if vec == ruta[i+1]
        )
        for i in range(len(ruta) - 1)
    )

    tiempo_total = sum(
        next(
            tij for vec, dist, tij, cij, modo in grafo.get(ruta[i], [])
            if vec == ruta[i+1]
        )
        for i in range(len(ruta) - 1)
    )

    # Detectar transbordos
    modos_ruta = []
    transbordos = []

    for i in range(len(ruta) - 1):
        u, v = ruta[i], ruta[i+1]

        for vecino, dist, tij, cij_base, modo in grafo.get(u, []):

            if vecino == v:

                if modos_ruta and modo != modos_ruta[-1]:
                    transbordos.append(
                        f"{u} → {v} ({modos_ruta[-1]} → {modo})"
                    )

                modos_ruta.append(modo)
                break

    print(f"\n  Ruta     : {' → '.join(ruta)}")
    print(f"  Arcos    : {len(ruta) - 1}")
    print(f"  Costo    : ${costo_total:,.2f} USD")
    print(f"  Distancia: {distancia_total:,.1f} km")
    print(f"  Tiempo   : {tiempo_total:.2f} h")

    if transbordos:
        print(f"\n  Transbordos detectados:")

        for t in transbordos:
            print(f" {t}  (+${Recargo_Transbordo:.0f} USD)")

    else:
        print("\n  Sin transbordos en la ruta.")

    print("=" * 65)
```
**Resultado obtenido:**

| Parámetro | Resultado |
|-----------|-----------|
| Ruta óptima| Monterrey → San Luis → Mexico City → Oaxaca → Tapachula → Cartagena |
| Número de arcos recorridos| 5 |
| Costo total| $7,454.58 USD |
| Distancia total| 2,620.0 km |
| Tiempo total estimado| 35.80 h |
| Transbordos detectados| Tapachula → Cartagena (Ferroviario → Fluvial) |
| Recargo por transbordo| +$120 USD |
</details>

<details>
<summary> Red vial Bogotá Dinamica </summary>
Una empresa de mensajería urbana en Bogotá debe encontrar la ruta de menor costo para un mensajero en moto que parte desde la intersección 1 y debe llegar a la intersección 12. El desafío de este ejercicio va más allá de encontrar el camino más corto en una red estática, ya que la disponibilidad de cada vía y su costo dependen de la hora en que el mensajero llega a cada nodo. Esto significa que una vía que está disponible al inicio del recorrido puede estar bloqueada más adelante si el mensajero llega a ella durante un horario restringido, y el costo de transitarla varía según el nivel de congestión en ese momento. Por esta razón, la validez y el costo de cada arco solo pueden determinarse en el momento exacto en que el mensajero llega al nodo desde el que inicia ese tramo.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20vial%20Bogot%C3%A1%20Dijkstra.pdf" download>Enunciado Red vial Bogotá Dijkstra</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/base_datos_red_vial_bogota.xlsx" download>Base de datos Red vial Bogotá</a>

**Solución:**

Importación de librerías:
```python
import pandas as pd
import heapq
import re
```
Parámetros del modelo:

Define los parámetros necesarios para el ejercicio. Se establece la ruta del archivo Excel, el nodo de origen y el nodo de destino. Luego se definen el costo por minuto en pesos y la demora fija por semáforo en minutos, ambos utilizados en la fórmula de costo de cada vía.

La lista `factores_congestion` almacena las franjas horarias junto con su respectivo factor de ajuste de velocidad, el cual se consulta cada vez que se calcula el costo de transitar una vía en una hora determinada.

El diccionario `nodos_evento` registra los nodos afectados por eventos con sus respectivas franjas de bloqueo: el nodo 7 por la marcha en la NQS entre las 8:00 y las 12:00, y el nodo 4 por el cierre de Corferias entre las 10:00 y las 14:00.

Finalmente, el diccionario `nodos_horario` indica los nodos con restricciones de circulación según la hora: el nodo 9 permite paso únicamente entre las 6:00 y las 16:00, mientras que el nodo 8 permite paso entre las 7:00 y las 20:00.

```python
Excel_Path = "base_datos_red_vial_bogota.xlsx"
Origen     = 1
Destino    = 12

# ── Constantes ────────────────────────────────────────────────────
Costo_Min  = 800    # pesos por minuto
Demora_Sem = 1.5    # minutos por semáforo

factores_congestion = [
    (6,  7,  0.90),
    (7,  9,  0.50),
    (9,  16, 1.00),
    (16, 19, 0.50),
    (19, 24, 0.85),
]

nodos_evento = {
    7: (8,  12),   # Marcha NQS
    4: (10, 14),   # Cierre Corferias
}

nodos_horario = {
    9: (6,  16),
    8: (7,  20),
}
```
Carga de datos:

Carga la información de la red vial desde el archivo Excel, leyendo cada hoja por separado en un DataFrame individual. Luego construye un único DataFrame consolidado tomando como base la hoja de distancias y agregando como columnas adicionales la velocidad base, el costo del peaje, el número de semáforos y la restricción de cada vía, de modo que toda la información necesaria quede unificada en una sola tabla para su posterior procesamiento.

```python
df_dist = pd.read_excel(Excel_Path, sheet_name='Distancias')
df_vel  = pd.read_excel(Excel_Path, sheet_name='Velocidad')
df_pea  = pd.read_excel(Excel_Path, sheet_name='Peajes')
df_sem  = pd.read_excel(Excel_Path, sheet_name='Número de semaforos')
df_res  = pd.read_excel(Excel_Path, sheet_name='Restricciones')

df = df_dist.copy()

df['Velocidad base (Km/h)'] = df_vel['Velocidad base (Km/h)']
df['Costo por peaje ($)']   = df_pea['Costo por peaje ($)']
df['Semaforos']             = df_sem['Semaforos']
df['Restriccion']           = df_res['Restriccion']
```
Parsear restricciones:

Esta función recibe el texto de la columna `Restriccion` de cada vía y lo transforma en una lista de tuplas que representan el tipo de restricción y su respectiva franja horaria.

Si el texto corresponde a `Libre`, retorna una tupla indicando que la vía está disponible durante las 24 horas del día. Si contiene `Obras`, retorna la tupla `(obras, 0, 0)`, indicando que la vía permanece cerrada de forma permanente y no tiene una franja horaria asociada.

En el caso de restricciones por `Pico y placa`, la función extrae todas franjas horarias presentes en el texto, como `7-9` o `17-19`, y genera una tupla por cada intervalo en el que la vía presenta restricción de circulación.

```python
def _parsear_restriccion(texto):
    """Convierte el texto de restricción en lista de tuplas (tipo, h_ini, h_fin)."""

    t = str(texto).strip()

    if t == 'Libre':
        return [('libre', 0, 24)]

    if 'Obras' in t:
        return [('obras', 0, 0)]

    franjas = re.findall(r'(\d+)-(\d+)', t)

    return [
        ('pico_placa', int(a), int(b))
        for a, b in franjas
    ]
```

Construcción de la lista de vías:

Transforma el DataFrame consolidado en una lista de tuplas llamada `vias`, donde cada tupla contiene toda la información de una vía: nodo de origen, nodo de destino, distancia, velocidad base, costo del peaje, número de semáforos, tipo de restricción, franja horaria de la restricción y nombre de la vía.

Para construir esta estructura, se recorre cada fila del DataFrame y se aplica la función `_parsear_restriccion`, la cual descompone la restricción de cada vía en sus respectivas franjas horarias. De esta manera, una vía con restricciones de tipo pico y placa en múltiples intervalos se representa como varias tuplas independientes dentro de la lista `vias`.

```python
vias = []

for _, row in df.iterrows():

    o      = int(row['Nodo de origen'])
    d      = int(row['Nodo de destino'])
    dist   = float(row['Distancia (Km)'])
    vel    = float(row['Velocidad base (Km/h)'])
    peaje  = float(row['Costo por peaje ($)'])
    sem    = int(row['Semaforos'])
    nombre = str(row['Via'])

    for tipo, h_ini, h_fin in _parsear_restriccion(row['Restriccion']):

        vias.append(
            (o, d, dist, vel, peaje, sem, tipo, h_ini, h_fin, nombre)
        )
```

Factor de congestión:

Esta función recorre la lista de franjas horarias y retorna el factor de congestión correspondiente a la hora recibida, identificando en qué intervalo se encuentra. Este factor se utiliza para ajustar la velocidad base de cada vía y así calcular la velocidad real de circulación en un momento específico.

Si la hora no pertenece a ninguna de las franjas definidas, la función retorna `1.0` como valor por defecto, lo que equivale a no aplicar ningún ajuste sobre la velocidad base.

```python

def get_factor(hora):
    """
    Retorna el factor de congestión correspondiente a la hora recibida
    según la tabla definida en el enunciado.

    Parámetros
    ----------
    hora : float
        Hora del día (puede ser decimal, ej: 8.5 = 8:30)

    Retorna
    -------
    float
        Factor de congestión
    """

    for h_ini, h_fin, factor in factores_congestion:
        if h_ini <= hora < h_fin:
            return factor

    return 1.0
```
Cálculo del costo de una vía: 

Esta función calcula el costo total en pesos de transitar una vía a una hora determinada.

Primero, obtiene el factor de congestión mediante la función `get_factor` y lo multiplica por la velocidad base para determinar la velocidad real de circulación en ese instante. Con esta velocidad ajustada, se calcula el costo asociado a la distancia recorrida, dividiendo la distancia entre la velocidad real, convirtiendo el resultado a minutos y multiplicándolo por el costo por minuto.

Posteriormente, se calcula el costo asociado a los semáforos, multiplicando el número de semáforos por la demora fija de cada uno y por el costo por minuto. Finalmente, se suma el costo por distancia, el costo de peaje y el costo de semáforos para obtener el costo total de la vía.

```python
def calcular_costo(dist, vel_base, peaje, semaforos, hora):
    """
    Retorna el costo total en pesos de transitar una vía a la hora dada,
    aplicando la fórmula definida en el enunciado.

    Parámetros
    ----------
    dist      : float — distancia en km
    vel_base  : float — velocidad base en km/h
    peaje     : float — costo fijo del peaje en pesos
    semaforos : int   — número de semáforos en el tramo
    hora      : float — hora estimada de llegada al nodo origen del arco

    Retorna
    -------
    float — costo total en pesos
    """

    factor     = get_factor(hora)
    velocidad  = vel_base * factor

    costo_dist = (dist / velocidad) * 60 * Costo_Min
    costo_sem  = semaforos * Demora_Sem * Costo_Min

    return costo_dist + peaje + costo_sem
```

Arcos disponibles desde un nodo: 

Esta función recibe un nodo y la hora estimada de llegada a ese nodo, y retorna únicamente los arcos salientes que están disponibles en ese momento.

Para ello recorre la lista completa de vías y filtra primero aquellas que parten del nodo recibido. Posteriormente, aplica las restricciones definidas en el enunciado:

- Arcos cerrados por obras  
- Arcos con pico y placa activo a la hora dada  
- Arcos cuyo nodo de destino se encuentra bloqueado por un evento en curso  
- Arcos cuyo nodo de destino está fuera de su horario de circulación permitido  

Para los arcos que cumplen todas las condiciones, se calcula la velocidad real ajustando la velocidad base con el factor de congestión correspondiente a la hora actual. Con esta velocidad se obtiene el tiempo de recorrido del arco y su costo total en pesos mediante la función `calcular_costo`.

Finalmente, se retorna la lista de arcos disponibles con toda su información, lista para ser utilizada por el algoritmo de Dijkstra.

```python
def arcos_disponibles(nodo, hora):
    """
    Dado un nodo y la hora estimada de llegada a ese nodo, retorna
    únicamente los arcos salientes disponibles en ese momento.

    Se excluyen:
      - Arcos cerrados por obras
      - Arcos con pico y placa activo a esa hora
      - Arcos cuyo nodo de destino esté bloqueado por un evento
      - Arcos cuyo nodo de destino esté fuera de su horario de circulación

    Parámetros
    ----------
    nodo : int
        Nodo de origen desde el que se exploran los arcos
    hora : float
        Hora estimada de llegada al nodo

    Retorna
    -------
    list
        Lista de tuplas:
        (destino, dist, velocidad, peaje, sem, tiempo_arco, costo_arco)
    """

    disponibles = []

    for o, d, dist, vel, peaje, sem, tipo, h_ini, h_fin, nombre in vias:

        if o != nodo:
            continue

        # Arco cerrado por obras
        if tipo == 'obras':
            continue

        # Arco con pico y placa activo
        if tipo == 'pico_placa' and h_ini <= hora < h_fin:
            continue

        # Nodo destino bloqueado por evento
        if d in nodos_evento:
            hi, hf = nodos_evento[d]
            if hi <= hora < hf:
                continue

        # Nodo destino fuera de horario permitido
        if d in nodos_horario:
            hi, hf = nodos_horario[d]
            if not (hi <= hora < hf):
                continue

        factor = get_factor(hora)
        velocidad = vel * factor

        tiempo_arco = dist / velocidad  # horas
        costo_arco = calcular_costo(dist, vel, peaje, sem, hora)

        disponibles.append(
            (d, dist, velocidad, peaje, sem, tiempo_arco, costo_arco)
        )

    return disponibles
```

Implementación del algoritmo de Dijkstra:

La función implementa una versión extendida del algoritmo de Dijkstra adaptada a una red vial con restricciones dinámicas dependientes del tiempo.

El algoritmo comienza construyendo el conjunto de nodos de la red a partir de la lista de vías. Posteriormente, se inicializan las estructuras de datos principales: `costo`, `predecesor` y `hora_nodo`. El diccionario `costo` asigna inicialmente infinito a cada nodo, indicando que aún no se ha encontrado una ruta hacia ellos. El diccionario `predecesor` se inicializa en `None` para permitir la reconstrucción de la ruta óptima al finalizar el algoritmo. Por su parte, `hora_nodo` almacena la hora estimada de llegada a cada nodo, inicializada también en infinito.

El nodo origen se inicializa con costo cero y con la hora de inicio del recorrido. Se crea el conjunto `visitados` vacío y se inserta el nodo origen en la cola de prioridad junto con su costo y hora de llegada.

El ciclo principal se ejecuta mientras la cola de prioridad no esté vacía. En cada iteración se extrae el nodo con menor costo acumulado junto con su hora de llegada. Si el nodo ya fue visitado, se descarta; de lo contrario, se marca como visitado.

Luego, se obtienen los arcos disponibles desde el nodo actual utilizando la función `arcos_disponibles(u, hora_actual)`, la cual filtra únicamente las vías habilitadas según restricciones de tiempo, eventos y condiciones de circulación.

Para cada vecino alcanzable, se calcula el costo tentativo sumando el costo acumulado hasta el nodo actual y el costo del arco. También se calcula la hora estimada de llegada al vecino sumando el tiempo de recorrido del arco a la hora actual.

Si el costo tentativo mejora el mejor costo registrado para ese nodo, se actualizan el costo, el predecesor y la hora de llegada. Finalmente, el vecino se inserta en la cola de prioridad para continuar la exploración en iteraciones posteriores.

```python
def dijkstra(hora_inicio):
    """
    Implementa el algoritmo de Dijkstra de forma manual.
    En cada paso, al explorar los vecinos de un nodo, usa arcos_disponibles()
    para determinar qué arcos están disponibles según la hora estimada
    de llegada a ese nodo.

    Parámetros
    ----------
    hora_inicio : float
        Hora de inicio del recorrido

    Retorna
    -------
    costo      : dict
        Costo mínimo acumulado a cada nodo
    predecesor : dict
        Nodo predecesor en la ruta óptima
    hora_nodo  : dict
        Hora estimada de llegada a cada nodo
    """

    nodos = set(o for o, d, *_ in vias) | set(d for o, d, *_ in vias)

    # Inicialización estructuras de datos
    costo      = {n: float('inf') for n in nodos}
    predecesor = {n: None for n in nodos}
    hora_nodo  = {n: float('inf') for n in nodos}

    # Inicialización del origen
    costo[Origen]     = 0.0
    hora_nodo[Origen] = hora_inicio

    visitados = set()
    cola_prioridad = [(0.0, Origen, hora_inicio)]  # (costo, nodo, hora_llegada)

    # Ciclo principal
    while cola_prioridad:

        # Extraer nodo de menor costo
        costo_actual, u, hora_actual = heapq.heappop(cola_prioridad)

        # Si ya fue visitado, descartar
        if u in visitados:
            continue

        # Marcar como visitado
        visitados.add(u)

        # Explorar vecinos disponibles según la hora actual
        for v, dist, vel, peaje, sem, tiempo_arco, costo_arco in arcos_disponibles(u, hora_actual):

            # Calcular costo tentativo
            costo_tentativo = costo_actual + costo_arco
            hora_llegada_v  = hora_actual + tiempo_arco

            # Actualización si se encuentra mejor camino
            if costo_tentativo < costo[v]:
                costo[v]      = costo_tentativo     # Línea 17
                predecesor[v] = u                   # Línea 18
                hora_nodo[v]  = hora_llegada_v

                # Insertar en la cola de prioridad
                heapq.heappush(
                    cola_prioridad,
                    (costo_tentativo, v, hora_llegada_v)
                )

    return costo, predecesor, hora_nodo
```

Resultado algortimo: 

Ejecuta el algoritmo de Dijkstra para cada uno de los horarios definidos. Para cada hora del día, se invoca la función `dijkstra()` con la hora de inicio correspondiente y se verifica si el costo del nodo destino es infinito, lo que indicaría que no existe una ruta válida en ese momento debido a las restricciones activas en la red.

En caso de existir una ruta válida, se procede a reconstruir la secuencia óptima de nodos. Este proceso se realiza siguiendo la cadena de predecesores desde el nodo destino hacia atrás hasta llegar al nodo origen. Cada nodo se inserta al inicio de la lista para asegurar que la ruta final quede ordenada desde el origen hasta el destino.

Finalmente, se imprime la ruta óptima como una secuencia ordenada de intersecciones junto con el costo total en pesos.

```python
for hora in [6, 8, 11, 18]:

    print(f"\n{'='*60}")
    print(f"  HORA: {hora}:00")
    print(f"{'='*60}")

    costo, predecesor, hora_nodo = dijkstra(hora_inicio=hora)

    if costo[Destino] == float('inf'):
        print("  No existe ruta válida a esta hora.")
        continue

    # Reconstrucción de la ruta
    ruta = []
    nodo_actual = Destino

    while nodo_actual is not None:
        ruta.insert(0, nodo_actual)
        nodo_actual = predecesor[nodo_actual]

    print(f"  Ruta óptima : {' → '.join(map(str, ruta))}")
    print(f"  Costo total : ${costo[Destino]:,.0f} pesos")
```
**Resultado obtenido:**

| Hora | Ruta óptima | Costo total |
|------|-------------|--------------|
| 06:00 | 1 → 3 → 4 → 5 → 6 → 12 | $22,584 pesos |
| 08:00 | 1 → 3 → 4 → 5 → 6 → 12 | $32,011 pesos |
| 11:00 | 1 → 3 → 8 → 11 → 12 | $17,371 pesos |
| 18:00 | 1 → 3 → 8 → 11 → 12 | $25,143 pesos |

</details>

## Ruteo de vehículos

--

En esta sección se abordarán problemas de ruteo de vehículos utilizando la librería **PyVRP**.

A diferencia de los problemas de ruta más corta, donde el objetivo es encontrar el camino óptimo entre dos nodos de una red, los problemas de ruteo de vehículos buscan determinar el conjunto de rutas que debe seguir una flota de vehículos para atender a un grupo de clientes.

El objetivo principal es minimizar una función de costo, que puede estar asociada a la distancia recorrida, el tiempo de operación, el costo total de transporte o una combinación de estos criterios. Para ello, se consideran restricciones operativas, como la capacidad máxima de los vehículos, las ventanas de tiempo de los clientes, la cantidad de vehículos disponibles y la compatibilidad de los vehículos con los clientes.

### Contenido

- [PyVRP](#pyvrp)
  - [Funciones principales PyVRP](#funciones-principales-pyvrp)
  - [Ejercicios PyVRP](#ejercicios-pyvrp)

### PyVRP

**PyVRP** es una librería de optimización de código abierto especializada en la resolución de problemas de ruteo de vehículos (*Vehicle Routing Problems, VRP*).

Internamente implementa el algoritmo de **búsqueda local iterada (ILS, por sus siglas en inglés)**, el cual parte de una solución inicial y la mejora de manera progresiva mediante un proceso iterativo. En cada iteración combina dos mecanismos: las perturbaciones, que modifican la solución actual para explorar nuevas regiones del espacio de búsqueda y evitar quedar atrapado en óptimos locales; y la busqueda local, que refinan las soluciones generadas para lograr mejoras adicionales. 

Entre las principales funcionalidades de `PyVRP` se encuentran:

- Capacidad de los vehículos.
- Ventanas de tiempo para atención de clientes.
- Flotas homogéneas y heterogéneas.
- Múltiples depósitos.
- Visitas opcionales a clientes.
- Compatibilidad de vehículos
- Distancia máxima de recorrido

Estas características permiten modelar problemas reales de distribución y logística, donde las decisiones de ruteo deben considerar múltiples restricciones operativas y criterios de eficiencia.

#### Funciones principales PyVRP
 
En esta sección se presentan las funciones y clases principales de PyVRP necesarias para modelar y resolver problemas de ruteo de vehículos. Se cubrirá la instalación de la librería, la importación de los módulos necesarios, la creación del modelo y la definición de todos los atributos.
 
---

<details>
<summary> Instalación </summary>
 
PyVRP puede instalarse directamente desde el administrador de paquetes de Python. Basta con ejecutar el siguiente comando en la terminal:
 
```bash
pip install pyvrp
```
</details>

<details>
<summary> Importación de librerías </summary>
 
Una vez instalada la librería, se importan los módulos necesarios. `Model` es la clase principal con la que se construye el problema.
 
```python
from pyvrp import Model
```
</details>

<details>
<summary> Creación del modelo </summary>
 
`Model` es la clase central de PyVRP. Es el objeto sobre el que se construye el problema completo: se le agregan clientes, depósitos, vehículos y arcos. Todo el problema queda contenido en una única instancia de esta clase antes de ser enviado al solver.
 
```python
m = Model()
```
</details> 

<details>
<summary> Clientes </summary>
 
Un cliente representa cada punto de la red que debe ser visitado por un vehículo. Se agrega al modelo usando el método `add_client()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `x` | float | Si | Coordenada x de la ubicación del cliente |
| `y` | float | Si | Coordenada y de la ubicación del cliente |
| `delivery` | int | No | Unidades que el cliente solicita del depósito. Por defecto 0 |
| `pickup` | int | No | Unidades que el cliente devuelve al depósito. Por defecto 0 |
| `service_duration` | int | No | Tiempo que tarda el vehículo en atender al cliente antes de continuar con el recorrido. Por defecto 0 |
| `tw_early` | int | No | Inicio de la ventana de atención del cliente. Por defecto 0 |
| `tw_late` | int | No |Tiempo máximo permitido para iniciar el servicio al cliente. Sin restricción si no se especifica |
| `release_time` | int | No | Momento más temprano en el que el vehículo puede partir del depósito hacia este cliente. Por defecto 0 |
| `prize` | int | No | Ingreso que se obtiene al visitar este cliente. Relevante cuando la visita es opcional. Por defecto 0 |
| `required` | bool | No | Indica si el cliente debe ser visitado obligatoriamente. Por defecto `True` |
| `name` | str | No | Nombre descriptivo del cliente. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`service_duration`, `tw_early`, `tw_late`, `release_time`) deben expresarse en **minutos** o **segundos**.

<details>
<summary> Ejemplo 1 — Crear un cliente con coordenadas x, y </summary>
   
La forma más directa de agregar un cliente es proporcionando sus coordenadas geográficas. A continuación se crea un cliente ubicado en las coordenadas (-74.0721, 4.7110), con una demanda de 15 unidades y una ventana de tiempo entre las 8:00 y las 12:00. Dado que PyVRP requiere que los tiempos sean enteros, la ventana de tiempo se convierte a minutos: 8:00 corresponde al minuto 480 y las 12:00 al minuto 720 contados desde la medianoche. El tiempo de servicio es de 10 minutos.
 
```python
m = Model()
 
# Ventana de tiempo en horas
tw_early_h = 8    # 8:00
tw_late_h  = 12   # 12:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 720
 
m.add_client(
    x                = -74.0721,
    y                = 4.7110,
    delivery         = 15,
    service_duration = 10,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    name             = "Cliente_1"
)
```
</details>

<details>
<summary> Ejemplo 2 — Crear un cliente sin coordenadas x, y </summary>

Cuando la red se representa mediante una matriz de distancias en lugar de coordenadas en el plano, el cliente no tiene una posición geográfica definida. En ese caso, en lugar de `x` e `y`, se usa el parámetro `location`, que corresponde al índice de la fila y columna del cliente dentro de la matriz de distancias. A continuación se crea un cliente que corresponde al índice 1 de la matriz, con una demanda de 20 unidades, una ventana de tiempo entre las 9:00 y las 14:00, y un tiempo de servicio de 15 minutos.
 
```python
m = Model()
 
# Ventana de tiempo en horas
tw_early_h = 9    # 9:00
tw_late_h  = 14   # 14:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 540
tw_late_min  = tw_late_h  * 60   # 840
 
m.add_client(
    location         = 1,
    delivery         = 20,
    service_duration = 15,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    name             = "Cliente_1"
)
```
 
> **Nota:** La matriz de distancias debe ser cuadrada y tanto las filas como las columnas deben seguir el mismo orden. El índice 0 debe corresponder al depósito y los índices siguientes a cada cliente en el mismo orden en que fueron agregados al modelo. Si las filas y columnas no siguen el mismo orden, o si el orden no coincide con el de los clientes en el modelo, las distancias calculadas serán incorrectas.
</details>

<details>
<summary> Ejemplo 3 — Crear un cliente sin coordenadas usando x=0, y=0 </summary>

Una alternativa más sencilla cuando se trabaja con matriz de distancias es asignar `x=0` e `y=0` a todos los clientes como valores artificiales. De esta forma se cumple con la firma del método sin necesidad de coordenadas reales, y las distancias entre nodos quedan definidas exclusivamente por la matriz que se provee al modelo. Esta opción es útil cuando las distancias reales ya están precalculadas y no se quiere depender de coordenadas geográficas.
 
```python
m = Model()
 
# Ventana de tiempo en horas
tw_early_h = 9    # 9:00
tw_late_h  = 14   # 14:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 540
tw_late_min  = tw_late_h  * 60   # 840
 
m.add_client(
    x                = 0,
    y                = 0,
    delivery         = 20,
    service_duration = 15,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    name             = "Cliente_1"
)
```
 
> **Nota:** Al usar coordenadas artificiales `x=0, y=0`, es obligatorio proveer la matriz de distancias al modelo. Si no se hace, se calcularán las distancias desde el origen, lo que producirá resultados incorrectos.

</details>

<details>
<summary> Ejemplo 4 — Cargar clientes desde un archivo Excel </summary>
  
Cuando el número de clientes es grande, la forma más práctica es cargarlos desde un archivo Excel. Imaginemos que tenemos un archivo llamado `clientes_pyvrp.xlsx` con la siguiente estructura:

Donde cada fila representa un cliente con sus coordenadas, demanda en unidades, ventana de tiempo de atención en minutos y tiempo de servicio en minutos. A continuación se muestra cómo leer este archivo y agregar todos los clientes al modelo de forma automática.
 
```python
m = Model()
 
df = pd.read_excel("clientes_pyvrp.xlsx", sheet_name="Clientes")
 
for _, row in df.iterrows():
    m.add_client(
        x                = int(row["x"]),
        y                = int(row["y"]),
        delivery         = int(row["demanda"]),
        service_duration = int(row["service_duration"]),
        tw_early         = int(row["tw_early"]),
        tw_late          = int(row["tw_late"]),
        name             = str(row["nombre"])
    )

print(f"Clientes cargados: {len(m.locations)}")
print("\nPrimeros 5 clientes:")
for client in list(m.locations)[:5]:
    print(f"  {client.name} — demanda: {client.delivery} | tw: [{client.tw_early}, {client.tw_late}] | servicio: {client.service_duration} min")
```
**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/clientes_pyvrp.xlsx" download> Clientes Pyvrp</a>

**Solución:**

Se han cargado un total de **10 clientes** en la instancia del problema.

Primeros 5 clientes: 

| Cliente   | Demanda | Ventana de tiempo (min) | Tiempo de servicio |
|-----------|---------|--------------------------|---------------------|
| Cliente_1 | 15      | [0, 240]                 | 10 min              |
| Cliente_2 | 20      | [60, 300]                | 15 min              |
| Cliente_3 | 10      | [30, 180]                | 8 min               |
| Cliente_4 | 25      | [90, 360]                | 12 min              |
| Cliente_5 | 18      | [120, 420]               | 10 min              |
</details>
</details>

<details>
<summary> Deposito </summary>

El depósito es el punto desde el cual parten y al cual regresan los vehículos. Se agrega al modelo usando el método `add_depot()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `x` | float | Si | Coordenada x de la ubicación del depósito |
| `y` | float | Si | Coordenada y de la ubicación del depósito |
| `tw_early` | int | No | Inicio de operación del depósito. Por defecto 0 |
| `tw_late` | int | No | Fin de operación del depósito. Sin restricción si no se especifica |
| `service_duration` | int | No | Tiempo que toma cargar un vehículo al inicio de cada ruta. Por defecto 0 |
| `name` | str | No | Nombre descriptivo del depósito. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`tw_early`, `tw_late`, `service_duration`) deben expresarse en **minutos** o **segundos**
> **Nota:** Los ejercicios que se realizan dentro del curso no contempla escenarios multi depositos

<details>
<summary> Ejemplo 1 — Crear un depósito con coordenadas x, y </summary>

La forma más directa de agregar un depósito es proporcionando sus coordenadas geográficas. A continuación se crea un depósito ubicado en las coordenadas (4.7110, -74.0721), con horario de apertura a las 8:00 y cierre a las 18:00, y un tiempo de carga de 15 minutos.
 
```python
m = Model()
 
# Horario del depósito en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
m.add_depot(
    x                = 4.7110,
    y                = -74.0721,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    service_duration = 15,
    name             = "Deposito_Central"
)
```
</details>

<details>
<summary> Ejemplo 2 — Crear un depósito sin coordenadas x, y </summary>
 
Cuando se trabaja con una matriz de distancias, el depósito se identifica por su índice dentro de dicha matriz mediante el parámetro `location`. Siguiendo la convención de PyVRP, el depósito debe corresponder al índice 0 de la matriz.
 
```python
m = Model()
 
# Horario del depósito en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
m.add_depot(
    location         = 0,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    service_duration = 15,
    name             = "Deposito_Central"
)
```
</details>

<details>
<summary> Ejemplo 3 — Crear un depósito usando x=0, y=0 </summary>
  
Al igual que con los clientes, cuando se dispone de una matriz de distancias precalculada y no se quiere depender de coordenadas geográficas, se pueden usar `x=0` e `y=0` como valores artificiales. En ese caso es obligatorio proveer la matriz de distancias al modelo.
 
```python
m = Model()
 
# Horario del depósito en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
m.add_depot(
    x                = 0,
    y                = 0,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    service_duration = 15,
    name             = "Deposito_Central"
)
```
 
> **Nota:** Al usar coordenadas artificiales `x=0, y=0`, es obligatorio proveer la matriz de distancias al modelo. Si no se hace, PyVRP calculará distancias euclidianas desde el origen, lo que producirá resultados incorrectos.

</details>
</details>

<details>
<summary> Tipos de vehículos </summary>

Un tipo de vehículo agrupa las características operativas que comparten los vehículos de una misma categoría dentro de la flota, como su capacidad, horario de operación, costos asociados y perfil de enrutamiento. Se agrega al modelo usando el método `add_vehicle_type()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `num_available` | int | No | Número de vehículos disponibles de este tipo. Por defecto 1 |
| `capacity` | int | No | Capacidad máxima en unidades que puede transportar el vehículo. Por defecto sin restricción |
| `start_depot` | int | No | Índice del depósito desde el que parten los vehículos de este tipo. Por defecto 0 |
| `end_depot` | int | No | Índice del depósito al que regresan los vehículos de este tipo. Por defecto 0 |
| `fixed_cost` | int | No | Costo fijo por utilizar un vehículo de este tipo, independiente de la ruta. Por defecto 0 |
| `tw_early` | int | No | Inicio de operación del vehículo. Por defecto 0 |
| `tw_late` | int | No | Tiempo de finalización del turno del vehículo. Sin restricción si no se especifica |
| `shift_duration` | int | No | Duración máxima de la ruta. Puede extenderse con horas extra. Sin restricción si no se especifica |
| `max_distance` | int | No | Distancia máxima que puede recorrer el vehículo en una ruta. Sin restricción si no se especifica |
| `unit_distance_cost` | int | No | Costo por unidad de distancia recorrida. Por defecto 1 |
| `unit_duration_cost` | int | No | Costo por unidad de tiempo de duración de la ruta. Por defecto 0 |
| `max_overtime` | int | No | Tiempo adicional permitido después de superar la duración máxima establecida. Por defecto 0 |
| `unit_overtime_cost` | int | No | Costo por unidad de tiempo extra. Por defecto 0 |
| `profile` | Profile | No | Perfil de enrutamiento del vehículo, que define los nodos a atender y la matriz de distancias. Por defecto 0 |
| `name` | str | No | Nombre descriptivo del tipo de vehículo. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`tw_early`, `tw_late`, `shift_duration`, `max_overtime`) deben expresarse en **minutos** o **segundos**. El depósito debe haberse agregado al modelo antes de definir los tipos de vehículos.

<details>
<summary> Ejemplo 1 — Flota homogénea </summary>
 
Una flota homogénea está compuesta por vehículos del mismo tipo con las mismas características operativas. Basta con definir un único tipo de vehículo e indicar cuántas unidades están disponibles. A continuación se define una flota de 3 camiones estándar con una capacidad de 50 unidades, un costo fijo de 500 USD por vehículo, turno entre las 8:00 y las 18:00 con una duración de 8 horas, distancia máxima de 200 km, costo de 2 USD/Km y 1 USD/min. Adicionalmente, los vehículos disponen de hasta 30 minutos extra para completar la ruta y regresar al depósito, con un costo adicional de 3 USD por minuto.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Turno en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
shift_h    = 8    # 8 horas de turno

# Crear perfil de enrutamiento
perfil = m.add_profile()

# Conversión a minutos
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
shift_min    = shift_h    * 60   # 480
 
m.add_vehicle_type(
    num_available      = 3,
    capacity           = 50,
    fixed_cost         = 500,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = shift_min,
    max_distance       = 200,
    unit_distance_cost = 2,
    unit_duration_cost = 1,
    max_overtime       = 30,
    unit_overtime_cost = 3,
    profile            = perfil,
    name               = "Camion_Estandar"
)
```
</details>
<details>
<summary> Ejemplo 2 — Flota heterogénea </summary>
 
Una flota heterogénea combina vehículos de distintos tipos con diferentes capacidades, costos y restricciones operativas. Cada tipo se agrega por separado al modelo. A continuación se definen dos tipos de vehículos: un furgón y un camión grande.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Turno en horas — compartido por ambos tipos
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00

# Crear perfiles de enrutamiento por tipo de vehículo
perfil_furgon = m.add_profile(name="Mediano")
perfil_camion = m.add_profile(name="Pesado")
 
# Conversión a minutos
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
# Tipo 1 — Furgón
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 30,
    fixed_cost         = 300,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = 8 * 60,    # 480 min
    max_distance       = 150,
    unit_distance_cost = 3,
    unit_duration_cost = 1,
    max_overtime       = 20,
    unit_overtime_cost = 4,
    profile            = perfil_furgon,
    name               = "Furgon"
)
 
# Tipo 2 — Camión grande
m.add_vehicle_type(
    num_available      = 1,
    capacity           = 80,
    fixed_cost         = 800,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = 10 * 60,   # 600 min
    max_distance       = 300,
    unit_distance_cost = 2,
    unit_duration_cost = 1,
    max_overtime       = 60,
    unit_overtime_cost = 3,
    profile            = perfil_camion,
    name               = "Camion_Grande"
)
```
</details>

<details>
<summary> Ejemplo 3 — Cargar tipos de vehículos desde un archivo Excel </summary>
 
Cuando la flota tiene varios tipos de vehículos, la forma más práctica es cargarlos desde un archivo Excel. Imaginemos que tenemos un archivo llamado `vehiculos_pyvrp.xlsx` donde los vehículos están distribuidos en tres perfiles: **0** para vehículos livianos, **1** para furgones de tamaño mediano y **2** para camiones pesados.
 
> **Nota:** Los perfiles deben crearse antes de cargar los vehículos. El índice de la columna `profile` en el Excel se usa para seleccionar el objeto perfil correspondiente desde un diccionario.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Crear los tres perfiles de enrutamiento
perfiles = {
    0: m.add_profile(name="Liviano"),
    1: m.add_profile(name="Mediano"),
    2: m.add_profile(name="Pesado")
}
 
df = pd.read_excel("vehiculos_pyvrp.xlsx", sheet_name="Vehiculos")
 
for _, row in df.iterrows():
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        fixed_cost         = int(row["fixed_cost"]),
        tw_early           = int(row["tw_early"]),
        tw_late            = int(row["tw_late"]),
        shift_duration     = int(row["shift_duration"]),
        max_distance       = int(row["max_distance"]),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        max_overtime       = int(row["max_overtime"]),
        unit_overtime_cost = int(row["unit_overtime_cost"]),
        profile            = perfiles[int(row["profile"])],
        name               = str(row["nombre"])
    )
 
print(f"Tipos de vehículos cargados: {len(m.vehicle_types)}")
for vt in m.vehicle_types:
    print(f"  {vt.name} — capacidad: {vt.capacity} | disponibles: {vt.num_available} | perfil: {vt.profile}")
```

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/vehiculos_pyvrp.xlsx" download> Vehiculos Pyvrp</a>

**Solución:**

Tipos de vehículos cargados: 5

| Tipo de vehículo | Capacidad | Vehículos disponibles | Perfil |
|------------------|-----------|------------------------|--------|
| Moto | 15 | 4 | 0 |
| Furgón pequeño | 30 | 3 | 1 |
| Furgón grande | 50 | 2 | 1 |
| Camión | 80 | 2 | 2 |
| Camión pesado | 120 | 1 | 2 |

</details>
</details>

<details>
<summary> Arcos </summary>

Un arco conecta dos ubicaciones de la red e indica la distancia y duración del recorrido entre ellas. Se agrega al modelo usando el método `add_edge()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `frm` | Client \| Depot | Si | Ubicación de origen del arco |
| `to` | Client \| Depot | Si | Ubicación de destino del arco |
| `distance` | int | Si | Distancia del recorrido |
| `duration` | int | No | Duración del recorrido. Por defecto 0 |
| `profile` | Profile | No | Perfil de enrutamiento al que pertenece el arco |
 
> **Importante:** Los arcos deben agregarse para **todos los pares ordenados** de ubicaciones del modelo, incluyendo los arcos de una ubicación a sí misma con `distance=0` y `duration=0`. Si se omite algún par, el modelo puede quedar infactible.
 
Aproximación de enteros
 
PyVRP requiere que `distance` y `duration` sean enteros. Dependiendo del origen de los datos, esto puede requerir distintos tratamientos:
 
- **Coordenadas geográficas (latitud y longitud):** la distancia euclidiana no es válida con coordenadas geográficas porque un grado de latitud y un grado de longitud no miden lo mismo en kilómetros, y esa diferencia varía según la ubicación en el planeta. La forma correcta es usar la fórmula de **Haversine**, que calcula la distancia real en kilómetros sobre la superficie de la Tierra. Como el resultado es decimal, es obligatorio escalar las distancias y duraciones multiplicándolas por un factor suficientemente grande antes de convertirlas a entero.

- **Matriz de distancias o duraciones con decimales:** si los valores de la matriz son decimales, aplicar `int()` directamente introduce un error. Si los valores son suficientemente grandes el error es despreciable. Si son pequeños o requieren precisión, se deben escalar también.
  
En todos los casos donde se escale, es necesario escalar de forma consistente todos los tiempos y costos del modelo (`tw_early`, `tw_late`, `service_duration`, `shift_duration`, `unit_distance_cost`, `unit_duration_cost`, `fixed_cost`) por el mismo factor, y desescalar los resultados al reportar.

> **Nota:** si se escala la distancia, no se debe escalar `unit_distance_cost`, y si se escala la duración, no se debe escalar `unit_duration_cost`. Escalar ambos factores del mismo producto haría que ese componente del costo quede multiplicado por `ESCALA²` mientras que el `fixed_cost` y el `prize` solo por `ESCALA`, rompiendo el balance del objetivo. Adicionalmente, si el modelo tiene restricción de autonomía `(max_distance)`, esta también debe escalarse por el mismo factor que las distancias para que PyVRP pueda compararlas correctamente.

<details>
<summary> Ejemplo 1 — Distancia euclidiana calculada desde coordenadas </summary>
 
Los clientes tienen coordenadas geográficas `x` e `y` (latitud y longitud) almacenadas en un archivo Excel. La distancia entre cada par de ubicaciones se calcula con la fórmula de **Haversine**, que mide la distancia real en kilómetros sobre la superficie de la Tierra teniendo en cuenta su curvatura. La duración se obtiene dividiendo esa distancia entre la velocidad promedio, expresada en minutos.
 
```python
import math
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime

ESCALA    = 10000   # factor de escala 
VELOCIDAD = 40      # km/h

def haversine(lat1, lon1, lat2, lon2):
    """
    Calcula la distancia en kilómetros entre dos puntos
    dados en grados de latitud y longitud.
    """
    R    = 6371  # radio de la Tierra en km
    phi1 = math.radians(lat1)
    phi2 = math.radians(lat2)
    dphi = math.radians(lat2 - lat1)
    dlam = math.radians(lon2 - lon1)
    a    = math.sin(dphi/2)**2 + math.cos(phi1) * math.cos(phi2) * math.sin(dlam/2)**2
    return 2 * R * math.asin(math.sqrt(a))

m = Model()
perfil = m.add_profile()

df = pd.read_excel("arcos_clientes_coordenadas.xlsx", sheet_name="Clientes")

# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(
    x                = fila_dep["x"],
    y                = fila_dep["y"],
    tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
    tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
    service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
    name             = fila_dep["nombre"]
)

# ── Clientes ──────────────────────────────────────────────────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = row["x"],
        y                = row["y"],
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]         * ESCALA),
        tw_late          = int(row["tw_late (min)"]           * ESCALA),
        service_duration = int(row["service_duration (min)"] * ESCALA),
        name             = str(row["nombre"])
    )

# ── Tipo de vehículo ──────────────────────────────────────────────
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = int(480 * ESCALA),
    tw_late            = int(1080 * ESCALA),
    shift_duration     = int(50  * ESCALA),
    unit_distance_cost = 3,
    unit_duration_cost = 2,
    profile            = perfil,
    name               = "Vehiculo"
)

# ── Arcos: distancia escalada y duración ──
for frm in m.locations:
    for to in m.locations:
        dist_real = haversine(frm.x, frm.y, to.x, to.y)   # km reales
        dur_min   = (dist_real / VELOCIDAD) * 60
        m.add_edge(frm, to,
                   distance = int(dist_real * ESCALA),
                   duration = int(dur_min   * ESCALA),
                   profile  = perfil)

# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best

# ── Resultados desescalados ───────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.4f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")

for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta        : {' → '.join(nombres)}")
    print(f"  Distancia : {route.distance() / ESCALA:.4f} km")
    print(f"  Duración  : {route.duration() / ESCALA:.2f} min")
```

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_clientes_coordenadas.xlsx" download> Clientes Coordenadas</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 9.8335 km |
| Duración total | 91.75 min |
| Costo por distancia | 29.50 |
| Costo por duración | 183.50 |
| Costo total | 213.00 |

Rutas asignadas

| Ruta | Distancia | Duración |
|------|-----------|----------|
| Cliente_2 → Cliente_4 | 6.4717 km | 46.71 min |
| Cliente_3 → Cliente_1 | 3.3618 km | 45.04 min |

</details>

<details>
<summary> Ejemplo 2 — Matriz de distancias </summary>
 
Las distancias entre ubicaciones se obtienen a partir de una matriz almacenada en el archivo Excel, mientras que la duración de cada arco se calcula dividiendo la distancia recorrida por la velocidad promedio definida para el vehículo.

Los clientes se incorporan al modelo respetando el mismo orden en el que aparecen registrados en la hoja de clientes. Posteriormente, durante la construcción de los arcos, se recorren las ubicaciones generadas dentro del modelo y se utiliza el nombre asociado a cada una para consultar directamente la distancia correspondiente en la matriz, utilizando la ubicación de origen como fila y la de destino como columna.

Este procedimiento garantiza que cada arco utilice la distancia correcta entre las ubicaciones correspondientes, evitando depender del orden interno asignado por `PyVRP` y asegurando la consistencia entre los datos de entrada y la representación del problema dentro del modelo.

El archivo `arcos_distancias_matriz.xlsx` contiene dos hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo y tiempo de servicio.
- **Distancias:** matriz cuadrada con nombres en filas y columnas, en el mismo orden en que los clientes fueron agregados al modelo.
 
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
m = Model()
perfil = m.add_profile()
 
df = pd.read_excel("arcos_distancia_matriz.xlsx", sheet_name="Clientes")
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(x=0,
                    y=0, 
                    tw_early = int(fila_dep["tw_early (min)"]),
                    tw_late  = int(fila_dep["tw_late (min)"]),
                    service_duration = int(fila_dep["service_duration (min)"]),
                    name=fila_dep["nombre"])
 
# ── Clientes con location asignado por orden de aparición ─────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]),
        tw_late          = int(row["tw_late (min)"]),
        service_duration = int(row["service_duration (min)"]),
        name             = str(row["nombre"])
    )
 
# ── Tipo de vehículo ──────────────────────────────────────────────
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = 480,
    tw_late            = 1080,
    shift_duration     = 100,
    unit_distance_cost = 3,
    unit_duration_cost = 2,
    profile            = perfil,
    name               = "Vehiculo"
)
 
# ── Arcos desde matriz de distancias, duración desde velocidad ────
df_dist   = pd.read_excel("arcos_distancia_matriz.xlsx", sheet_name="Distancias Km", index_col=0)
VELOCIDAD = 40 # Km/ h
 
# Los nodos se recorren en el mismo orden en que fueron agregados al modelo
locs  = list(m.locations)
nodos = [loc.name for loc in locs]
 
for frm in locs:
    for to in locs:
        dist = df_dist.loc[frm.name, to.name]
        dur  = (dist / VELOCIDAD) * 60
        m.add_edge(frm, to,
                   distance = int(dist),
                   duration = int(dur),
                   profile  = perfil)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()} km")
print(f"Duración total  : {sol.duration()} min")
print(f"Costo distancia : {sol.distance_cost()}")
print(f"Costo duración  : {sol.duration_cost()}")
print(f"Costo total     : {sol.distance_cost() + sol.duration_cost()}")
 
for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta        : {' → '.join(nombres)}")
    print(f"  Distancia : {route.distance()} km")
    print(f"  Duración  : {route.duration()} min")
```
 
> **Nota:** Los nombres de las ubicaciones en el modelo deben coincidir exactamente con los nombres de las filas y columnas de la matriz, ya que los arcos se buscan por nombre con `df_dist.loc[frm.name, to.name]`. Cualquier diferencia de mayúsculas, espacios o caracteres hará que la búsqueda falle.

> **Nota:** Este código funciona tanto para matrices simétricas como asimétricas, ya que recorre todos los pares ordenados `(frm, to)` y agrega un arco independiente para cada dirección.

> **Nota:** En este ejemplo no se escala porque las distancias de la matriz ya son enteros en kilómetros, por lo que `int(dist)` no produce pérdida de información. La duración calculada como `(dist / VELOCIDAD) * 60` puede resultar decimal en algunos casos, pero el error de truncamiento con `int()` es de menos de un minuto, lo cual es despreciable para este tipo de problema. Si se requiere mayor precisión, se debe escalar toda la información del modelo por un factor común.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_distancia_matriz.xlsx" download> Clientes Matriz Distancia</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 69 km |
| Duración total | 180 min |
| Costo por distancia | 207 |
| Costo por duración | 360 |
| Costo total | 567 |

Rutas asignadas

| Ruta | Distancia | Duración |
|------|-----------|----------|
| Cliente_4 → Cliente_3 | 30 km | 85 min |
| Cliente_1 → Cliente_2 | 39 km | 95 min |
</details>

<details>
<summary> Ejemplo 3 — Matriz de duraciones </summary>
 
Cuando no se dispone de distancias sino únicamente de tiempos de recorrido entre ubicaciones, se puede usar solo la matriz de duraciones y definir `distance=0` en todos los arcos. En este caso el costo del vehículo se define únicamente por unidad de duración. El archivo `arcos_matriz_duracion.xlsx` contiene dos hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo y tiempo de servicio.
- **Duraciones:** matriz cuadrada de tiempos de recorrido en minutos con nombres en filas y columnas.
 
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
m = Model()
perfil = m.add_profile()
 
df = pd.read_excel("arcos_matriz_duracion.xlsx", sheet_name="Clientes")
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(x=0, 
                    y=0, 
                    tw_early = int(fila_dep["tw_early (min)"]),
                    tw_late  = int(fila_dep["tw_late (min)"]),
                    service_duration = int(fila_dep["service_duration (min)"]),
                    name=fila_dep["nombre"])
 
# ── Clientes con location asignado por orden de aparición ─────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]),
        tw_late          = int(row["tw_late (min)"]),
        service_duration = int(row["service_duration (min)"]),
        name             = str(row["nombre"])
    )
 
# ── Tipo de vehículo ──────────────────────────────────────────────
# Sin unit_distance_cost porque no hay distancias
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = 480,
    tw_late            = 1080,
    shift_duration     = 120,
    unit_distance_cost = 0,
    unit_duration_cost = 2,
    profile            = perfil,
    name               = "Vehiculo"
)
 
# ── Arcos desde matriz de duraciones, distance=0 ──────────────────
df_dur = pd.read_excel("arcos_matriz_duracion.xlsx", sheet_name="Duraciones (min)", index_col=0)
locs   = list(m.locations)
 
for frm in locs:
    for to in locs:
        dur = df_dur.loc[frm.name, to.name]
        m.add_edge(frm, to,
                   distance = 0,
                   duration = int(dur),
                   profile  = perfil)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible       : {sol.is_feasible()}")
print(f"Duración total : {sol.duration()} min")
print(f"Costo duración : {sol.duration_cost()}")
 
for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta       : {' → '.join(nombres)}")
    print(f"  Duración : {route.duration()} min")
    print(f"  Costo    : {route.duration_cost()}")
```
> **Nota:** Este código funciona tanto para matrices simétricas como asimétricas, ya que recorre todos los pares ordenados `(frm, to)` y agrega un arco independiente para cada dirección.

> **Nota:** En este ejemplo no es necesario escalar ya que las duraciones de la matriz son enteras en minutos, por lo que `int(dur)` no produce pérdida de información.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_matriz_duracion.xlsx" download> Clientes Matriz Duración</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Duración total | 200 min |
| Costo por duración | 400 |

Rutas asignadas

| Ruta | Duración | Costo |
|------|----------|-------|
| Cliente_3 → Cliente_4 | 93 min | 186 |
| Cliente_2 → Cliente_1 | 107 min | 214 |

</details>
</details>
<details>
<summary> Perfiles de enrutamiento </summary>
 
Un perfil de enrutamiento define un conjunto de arcos con sus distancias y duraciones, formando una matriz completa de conectividad entre ubicaciones. Cada tipo de vehículo tiene un perfil asignado, y ese perfil determina exactamente qué arcos puede recorrer durante su ruta.
 
Esto permite modelar dos tipos de restricciones operativas que no pueden expresarse directamente con los atributos estándar de PyVRP:
 
- **Restricciones sobre clientes:** ciertos vehículos solo pueden atender a un subconjunto de clientes. Por ejemplo, furgones refrigerados para clientes que requieren cadena de frío, y motos para clientes de paquetería ligera. Esto se modela asignando una distancia muy grande (`INF`) a los arcos que conectan con los clientes que no se pueden atender, haciendo que nunca se incluyan en ninguna ruta.
- **Restricciones sobre caminos:** ciertos vehículos solo pueden circular por ciertas vías. Por ejemplo, los camiones de carga pesada solo pueden circular por vías principales debido a restricciones de peso y dimensiones, mientras que los furgones, al ser vehículos más compactos, tienen acceso tanto a vías principales como a vías secundarias, lo que les permite alcanzar zonas donde los camiones no pueden ingresar. Esto se modela con matrices de distancias distintas para cada perfil, donde los arcos no permitidos tienen distancia `INF`.
En ambos casos, `INF` debe ser un número entero suficientemente grande para que los arcos nunca se elijan, pero que no cause desbordamiento numérico. Un valor como `100000` funciona bien en la mayoría de los casos.

<details>
<summary> Ejemplo 1 — Vehículos con clientes restringidos </summary>
 
FreshRoute opera una flota mixta de **furgones** y **motos** para distribución urbana. Los furgones tienen mayor capacidad y atienden clientes con pedidos grandes, mientras que las motos son más ágiles y atienden clientes con pedidos pequeños. Por política operativa, cada tipo de vehículo solo puede atender a su grupo de clientes asignado.
 
El archivo `perfiles_clientes_restringidos.xlsx` contiene tres hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo (min), tiempo de servicio (min) y `tipo_atencion` que indica si el cliente es atendido por `furgon` o `moto`.
- **Vehiculos:** nombre, cantidad disponible, capacidad, velocidad (Km/h), ventanas de tiempo (min), duración del turno (min) y costos unitarios.
- **Distancias:** matriz de distancias en km entre todas las ubicaciones.

```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
ESCALA = 100      # factor de escala
INF    = 10**4   # distancia prohibida
 
m = Model()
perfil_furgon = m.add_profile(name="Furgon")
perfil_moto   = m.add_profile(name="Moto")
 
df_cli  = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Clientes")
df_veh  = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Vehiculos")
df_dist = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Distancias", index_col=0)
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df_cli[df_cli["nombre"] == "Deposito"].iloc[0]
m.add_depot(x=0, y=0,
            tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
            tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
            service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
            name             = fila_dep["nombre"])
 
# ── Clientes ──────────────────────────────────────────────────────
# Se guarda el tipo de atención de cada cliente para usarlo al construir los arcos
tipo_atencion = {}
for _, row in df_cli[df_cli["nombre"] != "Deposito"].iterrows():
    m.add_client(x=0, y=0,
                 delivery         = int(row["delivery"]),
                 tw_early         = int(row["tw_early (min)"]         * ESCALA),
                 tw_late          = int(row["tw_late (min)"]           * ESCALA),
                 service_duration = int(row["service_duration (min)"] * ESCALA),
                 name             = str(row["nombre"]))
    tipo_atencion[row["nombre"]] = row["tipo_atencion"]
 
# ── Tipos de vehículo ─────────────────────────────────────────────
perfiles_veh = {"Furgon": perfil_furgon, "Moto": perfil_moto}
vel_por_tipo  = {}
 
for _, row in df_veh.iterrows():
    nombre = row["nombre"]
    vel_por_tipo[nombre] = float(row["velocidad (Km/h)"])
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        tw_early           = int(row["tw_early (min)"]       * ESCALA),
        tw_late            = int(row["tw_late (min)"]         * ESCALA),
        shift_duration     = int(row["shift_duration (min)"] * ESCALA),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        profile            = perfiles_veh[nombre],
        name               = nombre
    )
 
# ── Arcos ─────────────────────────────────────────────────────────
# Para cada perfil, los arcos cuyo nodo de salida o llegada no sea atendido por ese vehículo reciben distancia INF
locs = list(m.locations)
 
# Self loops: distancia y duración 0 para ambos perfiles
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_furgon)
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_moto)
 
# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
 
        dist_km = df_dist.loc[frm.name, to.name]
 
        # Perfil furgon: INF si el arco involucra un cliente de moto
        if tipo_atencion.get(to.name) == "moto" or tipo_atencion.get(frm.name) == "moto":
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_furgon)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Furgon"]) * 60 * ESCALA),
                       profile  = perfil_furgon)
 
        # Perfil moto: INF si el arco involucra un cliente de furgon
        if tipo_atencion.get(to.name) == "furgon" or tipo_atencion.get(frm.name) == "furgon":
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_moto)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Moto"]) * 60 * ESCALA),
                       profile  = perfil_moto)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.2f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\n  {vt.name}: {' → '.join(nombres)}")
    print(f"    Distancia : {route.distance() / ESCALA:.2f} km")
    print(f"    Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Escalado:** las distancias de la matriz son decimales, por lo que se escala por `ESCALA = 100` para preservar la exactitud tanto en la distancia como en la duración calculada.

> **INF:** los arcos prohibidos reciben `distance = INF = 10**4`. Los arcos cuyo nodo de salida y nodo de llegada es igual siempre tienen `distance = 0` y `duration = 0` independientemente del perfil, ya que PyVRP lo exige explícitamente.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/perfiles_clientes_restringidos.xlsx" download> Perfiles Clientes Restringidos</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 102.79 km |
| Duración total | 257.09 min |
| Costo por distancia | 247.87 |
| Costo por duración | 381.58 |
| Costo total 629.45 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración |
|----------|------|-----------|----------|
| Furgón | Cliente_1 → Cliente_2 → Cliente_3 | 42.29 km | 124.49 min |
| Moto | Cliente_4 → Cliente_6 | 38.90 km | 79.68 min |
| Moto | Cliente_5 | 21.60 km | 52.92 min |
 
</details>
<details>
<summary> Ejemplo 2 — Vehículos con caminos restringidos </summary>
 
LogiRed opera una red de distribución que utiliza **camiones** y **furgones**. Los camiones, por restricciones de peso, solo pueden circular por vías principales, mientras que los furgones, al ser vehículos más compactos, tienen acceso únicamente a vías secundarias, lo que les permite alcanzar zonas donde los camiones no pueden ingresar. La red tiene una única matriz de distancias, pero cada arco tiene asociado un tipo de vía que determina qué vehículos pueden usarlo.
 
El archivo `perfiles_caminos_restringidos.xlsx` contiene cuatro hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo (min) y tiempo de servicio (min).
- **Vehiculos:** nombre, cantidad, capacidad, velocidad (Km/h), ventanas de tiempo (min), duración del turno (min), costos unitarios y `vias_permitidas` que indica el tipo de vía que puede usar ese vehículo (`primaria` o `secundaria`).
- **Distancias:** matriz de distancias en km entre todas las ubicaciones.
- **Vias:** matriz con el tipo de vía de cada arco (`primaria` o `secundaria`).
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
ESCALA = 100       # factor de escala
INF    = 10**4    # distancia prohibida
 
m = Model()
perfil_camion = m.add_profile(name="Camion")
perfil_furgon = m.add_profile(name="Furgon")
 
df_cli  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Clientes")
df_veh  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Vehiculos")
df_dist = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Distancias", index_col=0)
df_via  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Vias",       index_col=0)
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df_cli[df_cli["nombre"] == "Deposito"].iloc[0]
m.add_depot(x=0, y=0,
            tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
            tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
            service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
            name             = fila_dep["nombre"])
 
# ── Clientes ──────────────────────────────────────────────────────
for _, row in df_cli[df_cli["nombre"] != "Deposito"].iterrows():
    m.add_client(x=0, y=0,
                 delivery         = int(row["delivery"]),
                 tw_early         = int(row["tw_early (min)"]         * ESCALA),
                 tw_late          = int(row["tw_late (min)"]           * ESCALA),
                 service_duration = int(row["service_duration (min)"] * ESCALA),
                 name             = str(row["nombre"]))
 
# ── Tipos de vehículo ─────────────────────────────────────────────
perfiles_map  = {"Camion": perfil_camion, "Furgon": perfil_furgon}
vel_por_tipo  = {}
vias_por_tipo = {}
 
for _, row in df_veh.iterrows():
    nombre = row["nombre"]
    vel_por_tipo[nombre]  = float(row["velocidad (Km/h)"])
    vias_por_tipo[nombre] = row["vias_permitidas"]
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        tw_early           = int(row["tw_early (min)"]       * ESCALA),
        tw_late            = int(row["tw_late (min)"]         * ESCALA),
        shift_duration     = int(row["shift_duration (min)"] * ESCALA),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        profile            = perfiles_map[nombre],
        name               = nombre
    )
 
# ── Arcos ─────────────────────────────────────────────────────────
# Se consulta la hoja Vias para saber si cada arco está permitido
# para cada tipo de vehículo según su vias_permitidas
locs = list(m.locations)
 
# Self loops: distancia y duración 0 para ambos perfiles
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_camion)
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_furgon)
 
# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
 
        dist_km  = df_dist.loc[frm.name, to.name]
        tipo_via = df_via.loc[frm.name, to.name]
 
        # Camion: INF si el tipo de vía no coincide con su vía permitida
        if tipo_via != vias_por_tipo["Camion"]:
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_camion)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Camion"]) * 60 * ESCALA),
                       profile  = perfil_camion)
 
        # Furgon: INF si el tipo de vía no coincide con su vía permitida
        if tipo_via != vias_por_tipo["Furgon"]:
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_furgon)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Furgon"]) * 60 * ESCALA),
                       profile  = perfil_furgon)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.2f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\n  {vt.name}: {' → '.join(nombres)}")
    print(f"    Distancia : {route.distance() / ESCALA:.2f} km")
    print(f"    Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Escalado:** las distancias de la matriz son decimales, por lo que se escala por `ESCALA = 100` para preservar la exactitud tanto en la distancia como en la duración calculada. Todos los tiempos del modelo se escalan por el mismo factor para mantener consistencia interna.

> **Vías restringidas:** en lugar de tener matrices separadas por tipo de vehículo, se usa una única matriz de distancias y una hoja adicional que indica el tipo de vía de cada arco. Al construir los arcos, se consulta esa hoja y se asigna `INF` cuando el tipo de vía no coincide con el permitido por el vehículo, impidiendo que se utilicen esos arcos en la solución.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/perfiles_caminos_restringidos.xlsx" download> Perfiles Caminos Restringidos</a>

**Solución:**
Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 302.79 km |
| Duración total | 234.05 min |
| Costo por distancia | 908.37 |
| Costo por duración | 234.05 |
| Costo total | 1142.42 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración |
|----------|------|-----------|----------|
| Furgón | Cliente_6 → Cliente_2 → Cliente_5 | 164.90 km | 133.52 min |
| Furgón | Cliente_3 → Cliente_1 → Cliente_4 | 137.89 km | 100.53 min |

</details>
</details>
<details>
<summary> Solución </summary>
 
Una vez construido el modelo con clientes, depósito, tipos de vehículo y arcos, se resuelve llamando al método `m.solve()`. Este método recibe obligatoriamente un criterio de parada y opcionalmente una semilla para el generador aleatorio.
 
```python
from pyvrp.stop import MaxRuntime, MaxIterations
 
result = m.solve(stop=MaxRuntime(30), seed=42)
```
 
<details>
<summary> Criterios de parada </summary>
 
PyVRP implementa un algoritmo de búsqueda iterativa que mejora la solución progresivamente. Es necesario indicarle cuándo debe detenerse mediante uno de los siguientes criterios:
 
**`MaxRuntime(segundos)`** — detiene el algoritmo cuando se alcanza el tiempo máximo de ejecución en segundos. Es el criterio más común en la práctica porque permite controlar cuánto tiempo se dedica a la exploración del espacio de busqueda.
 
```python
result = m.solve(stop=MaxRuntime(60), seed=42)   # máximo 60 segundos
```
 
**`MaxIterations(iteraciones)`** — detiene el algoritmo cuando se alcanza el número máximo de iteraciones. Útil cuando se quiere garantizar exactamente la misma cantidad de trabajo computacional entre ejecuciones.
 
```python
result = m.solve(stop=MaxIterations(10000), seed=42)   # máximo 10000 iteraciones
```
</details>
<details>
<summary> Semilla </summary>
 
El parámetro `seed` controla el generador de números aleatorios del algoritmo. Dado que PyVRP es un algoritmo heurístico con componentes aleatorios, dos ejecuciones con la misma semilla producen exactamente el mismo resultado, lo que permite reproducir soluciones. Cambiar la semilla puede dar soluciones distintas de diferente calidad.
 
```python
result = m.solve(stop=MaxRuntime(30), seed=42)    # reproducible
result = m.solve(stop=MaxRuntime(30), seed=123)   # puede dar resultado diferente
```
</details>
<details>
<summary> Resultados globales </summary>
 
La solución se accede a través de `result.best`, que contiene la mejor solución encontrada durante la ejecución.
 
```python
sol = result.best
```
 
Los principales atributos globales disponibles son:
 
```python
print(f"Factible            : {sol.is_feasible()}")
print(f"Distancia total     : {sol.distance()}")
print(f"Duración total      : {sol.duration()}")
print(f"Costo de distancia  : {sol.distance_cost()}")
print(f"Costo de duración   : {sol.duration_cost()}")
print(f"Costo fijo total    : {sol.fixed_vehicle_cost()}")
print(f"Costo total         : {sol.distance_cost() + sol.duration_cost() + sol.fixed_vehicle_cost()}")
```
 
Ejemplo de salida:
 
```
Factible            : True
Distancia total     : 8450
Duración total      : 24320
Costo de distancia  : 25350
Costo de duración   : 48640
Costo fijo total    : 1000
Costo total         : 74990
```
 
> **Nota:** Si se usó escalado, todos estos valores deben desescalarse dividiendo por el factor correspondiente antes de interpretarlos.

</details>
<details>
<summary> Resultados por ruta </summary>
 
Para acceder a los detalles de cada ruta se itera sobre `sol.routes()`. Cada ruta registra su secuencia de nodos, distancia, duración, costos y los tiempos de inicio y fin del servicio en cada nodo mediante `route.schedule()`.
 
```python
def min_a_hora(minutos):
    """Convierte minutos desde medianoche a formato HH:MM."""
    h = int(minutos) // 60
    m = int(minutos) % 60
    return f"{h:02d}:{m:02d}"
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
 
    print(f"Vehículo  : {vt.name}")
    print(f"Ruta      : {' → '.join(nombres)}")
    print(f"Distancia : {route.distance()}")
    print(f"Duración  : {route.duration()} min")
    print(f"Costo dist: {route.distance_cost()}")
    print(f"Costo dur : {route.duration_cost()}")
    print(f"Inicio    : {min_a_hora(route.start_time())}")
    print(f"Fin       : {min_a_hora(route.end_time())}")
```
 
Ejemplo de salida:
 
```
Vehículo  : Furgon
Ruta      : Cliente_2 → Cliente_5 → Cliente_3
Distancia : 3120
Duración  : 8740 min
Costo dist: 9360
Costo dur : 17480
Inicio    : 08:00
Fin       : 14:33
```
 
> **Nota:** Los tiempos de `start_time()`, `end_time()`, `start_service` y `end_service` están en las mismas unidades con las que se definieron las ventanas de tiempo en el modelo. Si se usó escalado, deben desescalarse antes de convertirlos a formato `HH:MM`. La función `min_a_hora()` asume que los valores ya están en minutos desde medianoche.

</details>
</details>

#### Ejercicios PyVRP
<details>
<summary> Red de distribución FrescoDistribución S.A.S </summary>
  
FrescoDistribución S.A.S. es una empresa de distribución de productos refrigerados que opera desde un único depósito y atiende a ocho clientes ubicados en distintas zonas de la ciudad. El problema consiste en diseñar las rutas de distribución que maximicen la utilidad neta de la operación, entendida como los ingresos generados por las entregas y recogidas en cada cliente menos los costos operativos de la flota utilizada. 

**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20FrescoDistribuci%C3%B3n.pdf" download>Enunciado Fresco Distribución</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Datos_FrescoDistribución.xlsx" download> Datos Fresco Distribución</a>

**Desarrollo:**

Paso 0 — Importación de las librerías necesarias.
 
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxIterations
```
Paso 1 — Carga de datos desde Excel
 
Se cargan las tres hojas del archivo `Datos_FrescoDistribución.xlsx`. A partir de los datos de los clientes se calcula el **ingreso por cliente** como:
 
$$I_i = g^{ent}_i \times q^{ent}_i + g^{rec}_i \times q^{rec}_i$$
 
Este ingreso se usará como `prize` en PyVRP, lo que permite que el solver decida si conviene o no visitar cada cliente.
 
```python
ARCHIVO = "Datos_FrescoDistribución.xlsx"
 
df_cli  = pd.read_excel(ARCHIVO, sheet_name="Clientes")
df_flo  = pd.read_excel(ARCHIVO, sheet_name="Flota")
df_dist = pd.read_excel(ARCHIVO, sheet_name="Distancias", index_col=0)
 
# Calcular ingreso por cliente
df_cli["Ingreso"] = (df_cli["Ganancia entrega ($/kg)"] * df_cli["Entrega (kg)"] +
                     df_cli["Ganancia recogida ($/kg)"] * df_cli["Recogida (kg)"])
 
``` 
Paso 2 — Escalado
 
PyVRP requiere valores enteros. Como los datos tienen decimales se usa un factor de escala `ESCALA = 100` para preservar precisión. Todos los tiempos y costos se multiplican por este factor antes de pasarlos al modelo, y al reportar se dividen por `ESCALA` o `ESCALA²` según corresponda.
 
- **Distancias:** enteras no se escalan
- **Duraciones:** `distancia / velocidad * 60` puede ser decimal → se escala por `ESCALA`
- **Tiempos** (`tw_early`, `tw_late`, `service_duration`, `shift_duration`): en minutos → se escalan por `ESCALA`
- **Costos unitarios** (`unit_distance_cost`, `fixed_cost`): decimales → se escalan por `ESCALA`
- **Prize:** decimal → se escala por `ESCALA`
  
Al reportar los resultados se desescala cada componente según cómo fue escalado: la duración se divide por `ESCALA`, el costo fijo y el prize también se dividen por `ESCALA` ya que fueron escalados una sola vez. Para los componentes de costo que dependen de una magnitud y un costo unitario, como el costo de distancia (distancia × costo_por_km) y el costo de duración (duración × costo_por_minuto), solo se debe escalar uno de los dos factores del producto, nunca ambos. Si se escalan los dos, el costo interno queda multiplicado por `ESCALA²` mientras que el prize y el costo fijo solo por `ESCALA`, rompiendo el balance del objetivo y haciendo que el solver nunca encuentre rentable visitar ningún cliente.
  
```python
ESCALA = 100
```
Paso 3 — Crear el modelo y agregar el depósito
 
Se crea el modelo y se agrega el depósito. El horario del depósito es de **06:00 a 18:00**, lo que en minutos desde medianoche corresponde a `[360, 1080]`. Ambos valores se escalan por `ESCALA`.
 
```python
m      = Model()
perfil = m.add_profile()
 
DEP_EARLY = int(6  * 60 * ESCALA)   # 06:00 
DEP_LATE  = int(18 * 60 * ESCALA)   # 18:00 
 
m.add_depot(
    x        = 0,
    y        = 0,
    tw_early = DEP_EARLY,
    tw_late  = DEP_LATE,
    name     = "Deposito"
)
```
Paso 4 — Agregar los clientes
 
Cada cliente se agrega con `required=False` y un prize igual a su ingreso, lo que le indica a `PyVRP` que la visita es opcional y que solo conviene realizarla si el ingreso que aporta supera el costo de incluirlo en la ruta. El `prize` es el ingreso calculado en el Paso 1, escalado por `ESCALA`. Las ventanas de tiempo se convierten a minutos y se escalan.
 
```python
for _, row in df_cli.iterrows():
    tw_early_min = int(row["Ventana inicio"].split(":")[0]) * 60
    tw_late_min  = int(row["Ventana fin"].split(":")[0])    * 60
 
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["Entrega (kg)"]),
        pickup           = int(row["Recogida (kg)"]),
        service_duration = int(row["Tiempo servicio (min)"] * ESCALA),
        tw_early         = int(tw_early_min * ESCALA),
        tw_late          = int(tw_late_min  * ESCALA),
        prize            = int(round(row["Ingreso"]) * ESCALA),
        required         = False,
        name             = str(row["Cliente"])
    )
```
Paso 5 — Agregar los arcos
 
Cada tipo de vehículo tiene su propia velocidad, por lo que la duración de cada arco es diferente según el vehículo. Se crea un **perfil por tipo de vehículo** y se agregan los arcos correspondientes a cada perfil con su propia duración.
 
Los arcos cuyo nodo de salida es igual al nodo de llegada tienen `distance=0` y `duration=0`. Para los demás arcos, la duración se calcula como `distancia / velocidad * 60` en minutos, escalada por `ESCALA`.
 
```python
nodos      = list(m.locations)
nodos_dist = ["Deposito"] + list(df_cli["Cliente"])
 
perfiles = {}
for _, row in df_flo.iterrows():
    tipo = row["Tipo"]
    vel  = float(row["Velocidad (Km/h)"])
    p    = m.add_profile(name=tipo)
    perfiles[tipo] = (p, vel)
 
    # Self loops
    for loc in nodos:
        m.add_edge(loc, loc, distance=0, duration=0, profile=p)
 
    # Arcos entre ubicaciones distintas
    for frm in nodos:
        for to in nodos:
            if frm.name == to.name:
                continue
            dist_km = df_dist.loc[frm.name, to.name]
            dur_min = (dist_km / vel) * 60
            m.add_edge(frm, to,
                       distance = int(dist_km),
                       duration = int(dur_min * ESCALA),
                       profile  = p)
```
 
Paso 6 — Agregar los tipos de vehículos
 
Cada tipo de vehículo se agrega con su capacidad, autonomía, costo fijo, costo por km, jornada y horario. Todos los tiempos y costos se escalan por `ESCALA`. El perfil asignado determina qué matriz de duraciones usa ese vehículo.
 
```python
for _, row in df_flo.iterrows():
    tipo     = row["Tipo"]
    perfil_v, vel = perfiles[tipo]
 
    m.add_vehicle_type(
        num_available      = int(row["Unidades disp."]),
        capacity           = int(row["Capacidad (kg)"]),
        max_distance       = int(row["Autonomía (km)"]),
        fixed_cost         = int(row["Costo fijo ($)"]    * ESCALA),
        unit_distance_cost = int(row["Costo por km ($/km)"] * ESCALA),
        tw_early           = DEP_EARLY,
        tw_late            = DEP_LATE,
        shift_duration     = int(row["Jornada (h)"] * 60 * ESCALA),
        profile            = perfil_v,
        name               = tipo
    )
```
 
Paso 7 — Resolver
 
Se resuelve con `MaxIterations(2000)` y `seed=42` para garantizar reproducibilidad.
 
```python
result = m.solve(stop=MaxIterations(2000), seed=42)
sol    = result.best
 
print(f"Factible : {sol.is_feasible()}")
```
 
Paso 8 — Reportar resultados globales
 
Se calculan los ingresos totales de los clientes visitados, el costo total de la flota y la utilidad neta. Los costos se desescalan dividiendo por `ESCALA` o `ESCALA²` según corresponda.
 
```python
def min_a_hora(minutos_esc):
    minutos = minutos_esc / ESCALA
    h = int(minutos) // 60
    m = int(minutos) % 60
    return f"{h:02d}:{m:02d}"
 
# Clientes visitados e ingresos
visitados     = set()
ingreso_total = 0
 
for route in sol.routes():
    for v in route.visits():
        loc = m.locations[v]
        if loc.name != "Deposito":
            visitados.add(loc.name)
 
for _, row in df_cli.iterrows():
    if row["Cliente"] in visitados:
        ingreso_total += row["Ingreso"]
 
# Costos
costo_distancia = sol.distance_cost() / ESCALA**2
costo_fijo      = sol.fixed_vehicle_cost() / ESCALA
costo_total     = costo_distancia + costo_fijo
utilidad_neta   = ingreso_total - costo_total
 
print(f"\n{'='*55}")
print(f"  RESULTADOS GLOBALES")
print(f"{'='*55}")
print(f"  Clientes visitados : {len(visitados)} de {len(df_cli)}")
print(f"  Ingresos totales   : ${ingreso_total:,.2f}")
print(f"  Costo de distancia : ${costo_distancia:,.2f}")
print(f"  Costo fijo         : ${costo_fijo:,.2f}")
print(f"  Costo total        : ${costo_total:,.2f}")
print(f"  Utilidad neta      : ${utilidad_neta:,.2f}")
print(f"{'='*55}")
```
 
Paso 9 — Reportar rutas 
 
Para cada ruta activa se imprime la secuencia de nodos visitados con los horarios de llegada y salida, la carga del vehículo en cada nodo, la distancia, la duración y los costos de la ruta.
 
```python
for route in sol.routes():
 
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
 
    # Calcular ingresos de la ruta
    ingreso_ruta = sum(
        df_cli[df_cli["Cliente"] == n]["Ingreso"].values[0]
        for n in nombres if n != "Deposito"
    )
 
    costo_dist_ruta = route.distance_cost() / ESCALA
    costo_fijo_ruta = vt.fixed_cost / ESCALA
    utilidad_ruta   = ingreso_ruta - costo_dist_ruta - costo_fijo_ruta
 
    print(f"\n{'─'*55}")
    print(f"  Vehículo   : {vt.name}")
    print(f"  Ruta       : {' → '.join(nombres)}")
    print(f"  Distancia  : {route.distance()} km")
    print(f"  Duración   : {route.duration() / ESCALA:.1f} min")
    print(f"  Inicio     : {min_a_hora(route.start_time())}")
    print(f"  Fin        : {min_a_hora(route.end_time())}")
    print(f"  Ingresos   : ${ingreso_ruta:,.2f}")
    print(f"  Costo dist : ${costo_dist_ruta:,.2f}")
    print(f"  Costo fijo : ${costo_fijo_ruta:,.2f}")
    print(f"  Utilidad   : ${utilidad_ruta:,.2f}")
 
    print(f"\n  {'Nodo':<12} {'Llega':>8} {'Sale':>8} {'Carga sal. (kg)':>16}")
    print(f"  {'─'*50}")
 
    carga = sum(df_cli["Entrega (kg)"])   # carga inicial = total de entregas
    for visit in route.schedule():
        loc  = m.locations[visit.location]
        llega = min_a_hora(visit.start_service)
        sale  = min_a_hora(visit.end_service)
 
        if loc.name != "Deposito":
            fila   = df_cli[df_cli["Cliente"] == loc.name].iloc[0]
            carga -= fila["Entrega (kg)"]    # entrega
            carga += fila["Recogida (kg)"]   # recogida
 
        print(f"  {loc.name:<12} {llega:>8} {sale:>8} {carga:>16.0f}")
 
print(f"\n{'─'*55}")
```
**Solución:**
Resumen global

| Parámetro | Resultado |
|-----------|-----------|
| Clientes visitados | 5 de 8 |
| Ingresos totales | $3,915.00 |
| Costo por distancia | $4.83 |
| Costo fijo | $270.00 |
| Costo total | $274.83 |
| Utilidad neta | $3,640.17 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración | Ingresos | Costo distancia | Costo fijo | Utilidad |
|----------|------|-----------|----------|----------|-----------------|------------|----------|
| Camión Grande | C5 → C2 → C1 | 125 km | 294.3 min | $2,237.50 | $312.50 | $200.00 | $1,725.00 |
| Furgoneta | C6 → C4 | 142 km | 201.8 min | $1,677.50 | $170.40 | $70.00 | $1,437.10 |

Detalle de recorrido por vehículo

Camión Grande

| Nodo | Llegada | Salida | Carga salida (kg) |
|------|---------|--------|-------------------|
| Depósito | 06:51 | 06:51 | 880 |
| C5 | 08:00 | 08:35 | 725 |
| C2 | 09:35 | 09:55 | 655 |
| C1 | 10:32 | 10:57 | 565 |
| Depósito | 11:45 | 11:45 | 565 |

Furgoneta

| Nodo | Llegada | Salida | Carga salida (kg) |
|------|---------|--------|-------------------|
| Depósito | 09:48 | 09:48 | 880 |
| C6 | 11:00 | 11:20 | 860 |
| C4 | 11:52 | 12:10 | 880 |
| Depósito | 13:10 | 13:10 | 880 |
</details>

<details>
<summary> Red de distribución FarmaExpress S.A.S </summary>
  
FarmaExpress S.A.S. distribuye productos farmacéuticos a 100 clientes desde un único depósito que opera entre las 6:00 a.m. y las 10:00 p.m. La flota es heterogénea y está compuesta por vehículos eléctricos de dos tipos: Pequeño y Grande, con diferentes capacidades, autonomías y velocidades. Cada cliente tiene una ventana horaria de atención y un tiempo de servicio fijo. Adicionalmente, ciertos tramos de la red vial están restringidos para todos los vehículos. El modelo minimiza el costo total de operación respetando todas las restricciones.

**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20FarmaExpress.pdf" download>Enunciado FarmaExpress</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Datos_FarmaExpress.xlsx" download> Datos FarmaExpress</a>

**Desarrollo:**

Paso 0 — Importación de librerías

```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
```

Paso 1 — Carga de datos desde Excel

Se cargan las tres hojas del archivo `Datos_FarmaExpress.xlsx`. Las ventanas de tiempo de los clientes ya vienen expresadas en minutos desde las 00:00, por lo que no requieren conversión.

```python
ARCHIVO = "Datos_FarmaExpress.xlsx"

df_cli  = pd.read_excel(ARCHIVO, sheet_name="Info_Clientes")
df_veh  = pd.read_excel(ARCHIVO, sheet_name="Vehiculos")
df_dist = pd.read_excel(ARCHIVO, sheet_name="Distancias_clientes", index_col=0)
```
Paso 2 — Escalado

Las distancias ya son números enteros y los costos también son enteros, por lo que no se escalan. Sin embargo, la duración de cada arco se calcula como `distancia / velocidad × 60` en minutos y resulta decimal. Para convertirla a entero sin perder precisión se escala por `ESCALA = 100`. Todos los tiempos del modelo deben escalarse por el mismo factor.

Al reportar: duración ÷ `ESCALA`. La distancia y los costos no se desescalan.

```python
ESCALA = 100
```
Paso 3 — Definir los segmentos restringidos

El enunciado indica que los vehículos no pueden transitar por los segmentos C3–C7, C10–C12, C15–C18, C2–C5, C8–C11 y C14–C20. Estos segmentos se restringen en ambas direcciones. Se almacenan como un conjunto de pares congelados para facilitar la consulta al construir los arcos.

```python
restringidos = {
    frozenset(["C3",  "C7"]),
    frozenset(["C10", "C12"]),
    frozenset(["C15", "C18"]),
    frozenset(["C2",  "C5"]),
    frozenset(["C8",  "C11"]),
    frozenset(["C14", "C20"]),
}
```

Paso 4 — Crear el modelo y los perfiles

Se crea el modelo y se agrega un perfil por cada tipo de vehículo. Los perfiles son necesarios porque cada tipo de vehículo tiene una velocidad diferente, lo que resulta en duraciones distintas para los mismos arcos. Se guarda en un diccionario el perfil y la velocidad de cada tipo para usarlos al construir los arcos y los tipos de vehículo.

```python
m = Model()
perfiles = {}

for _, row in df_veh.iterrows():
    tipo = row["Tipo_Vehiculo"]
    vel  = float(row["Velocidad_km_h"])
    p    = m.add_profile(name=tipo)
    perfiles[tipo] = (p, vel)
```
Paso 5 — Agregar el depósito

El depósito opera entre las 6:00 a.m. (360 min desde 00:00) y las 10:00 p.m. (1320 min desde 00:00). Ambos valores se escalan por `ESCALA`.

```python
fila_dep = df_cli[df_cli["Cliente"] == "DEPOT"].iloc[0]

m.add_depot(
    x        = 0,
    y        = 0,
    tw_early = int(fila_dep["Ventana_Inicio_min"] * ESCALA),
    tw_late  = int(fila_dep["Ventana_Fin_min"]    * ESCALA),
    name     = "DEPOT"
)
```

Paso 6 — Agregar los clientes

Las ventanas de tiempo y el tiempo de servicio se escalan por `ESCALA`. 

```python
for _, row in df_cli[df_cli["Cliente"] != "DEPOT"].iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["Demanda"]),
        service_duration = int(row["Tiempo_Servicio_min"] * ESCALA),
        tw_early         = int(row["Ventana_Inicio_min"]  * ESCALA),
        tw_late          = int(row["Ventana_Fin_min"]     * ESCALA),
        name             = str(row["Cliente"])
    )
```

Paso 7 — Agregar los arcos

Se construyen los arcos para cada perfil. Los arcos cuyo nodo de origen y destino es iguao tienen `distance=0` y `duration=0`. Para los arcos entre ubicaciones distintas se consulta si el par pertenece a los segmentos restringidos: si es así, se asigna `distance=INF` para que nunca se usen esos arcos. Si no está restringido, se calcula la duración usando la velocidad del perfil correspondiente y se escala.

Como las distancias no se escalan pero el `unit_distance_cost` tampoco se escala, el costo de distancia interno de PyVRP es `distancia × costo_km` directamente en unidades reales.

```python
INF  = 10**7
locs = list(m.locations)

# Self loops para todos los perfiles
for loc in locs:
    for p, vel in perfiles.values():
        m.add_edge(loc, loc, distance=0, duration=0, profile=p)

# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue

        dist_km = df_dist.loc[frm.name, to.name]
        par     = frozenset([frm.name, to.name])

        for p, vel in perfiles.values():
            if par in restringidos:
                m.add_edge(frm, to, distance=INF, duration=0, profile=p)
            else:
                dur_min = (dist_km / vel) * 60
                m.add_edge(frm, to,
                           distance = int(dist_km),
                           duration = int(dur_min * ESCALA),
                           profile  = p)
```

> **Nota:** los segmentos restringidos reciben `distance=INF` para que el solver los evite. La duración se pone en `0` porque el arco nunca será usado de todas formas.

Paso 8 — Agregar los tipos de vehículos

Cada tipo de vehículo se configura con su capacidad, autonomía, costos, ventana de operación y jornada. La autonomía se pasa sin escalar porque las distancias tampoco se escalaron. Los tiempos sí se escalan.

```python
jornada_esc = int((fila_dep["Ventana_Fin_min"] - fila_dep["Ventana_Inicio_min"]) * ESCALA)

for _, row in df_veh.iterrows():
    tipo   = row["Tipo_Vehiculo"]
    p, vel = perfiles[tipo]

    m.add_vehicle_type(
        num_available      = int(row["Cantidad_Disponible"]),
        capacity           = int(row["Capacidad_unidades"]),
        max_distance       = int(row["Autonomia_km"]),
        fixed_cost         = int(row["Costo_Fijo_COP"]),
        unit_distance_cost = int(row["Costo_por_km_COP"]),
        tw_early           = int(fila_dep["Ventana_Inicio_min"] * ESCALA),
        tw_late            = int(fila_dep["Ventana_Fin_min"]    * ESCALA),
        shift_duration     = jornada_esc,
        profile            = p,
        name               = tipo
    )
```

> **Nota:** la autonomía no se escala porque las distancias tampoco se escalaron.

Paso 9 — Resolver

Se resuelve con `MaxRuntime(30)` y `seed=42`.

```python
result = m.solve(stop=MaxRuntime(30), seed=42)
sol    = result.best

print(f"Factible : {sol.is_feasible()}")
print(f"Rutas    : {sol.num_routes()}")
```

Paso 10 — Identificar la ruta con menor distancia y reportar resultados

Se ordenan las rutas por distancia y se selecciona la menor. Se reporta su secuencia, duración, distancia, hora de inicio, hora de fin y autonomía restante.

```python
def min_a_hora(minutos_esc):
    minutos = minutos_esc / ESCALA
    h = int(minutos) // 60
    m = int(minutos) % 60
    return f"{h:02d}:{m:02d}"

# Ordenar rutas por distancia y tomar la menor
rutas_ordenadas = sorted(sol.routes(), key=lambda r: r.distance())
ruta_min        = rutas_ordenadas[0]

vt      = m.vehicle_types[ruta_min.vehicle_type()]
nombres = [m.locations[v].name for v in ruta_min.visits()]

autonomia_restante = vt.max_distance - ruta_min.distance()

print(f"\n{'='*55}")
print(f"  RUTA CON MENOR DISTANCIA")
print(f"{'='*55}")
print(f"  Vehículo           : {vt.name}")
print(f"  Secuencia          : DEPOT → {' → '.join(nombres)} → DEPOT")
print(f"  Distancia          : {ruta_min.distance()} km")
print(f"  Duración           : {ruta_min.duration() / ESCALA:.1f} min")
print(f"  Hora de inicio     : {min_a_hora(ruta_min.start_time())}")
print(f"  Hora de fin        : {min_a_hora(ruta_min.end_time())}")
print(f"  Autonomía restante : {autonomia_restante} km")
print(f"{'='*55}")
```
**Solución:**

Resumen de la ruta

| Parámetro | Resultado |
|-----------|-----------|
| Vehículo | Grande |
| Secuencia de ruta | DEPOT → C98 → C40 → C65 → C22 → C91 → C27 → C19 → C52 → C26 → C80 → C3 → C76 → DEPOT |
| Distancia total | 64 km |
| Duración total | 291.5 min |
| Hora de inicio | 12:20 |
| Hora de finalización | 17:11 |
| Autonomía restante | 106 km |

</details>

<details>
<summary> Red de distribución LogiMarket S.A.S. </summary>
  
LogiMarket S.A.S. distribuye insumos mayoristas a 100 clientes (distribuidores, oficinas y pequeños comercios) desde un único depósito que opera entre las 7:00 a.m. y las 11:00 p.m. La flota es homogénea: 8 vehículos eléctricos idénticos con capacidad de 125 unidades, velocidad de 36 km/h y autonomía de 55 km. A diferencia de otros ejercicios, los clientes no tienen ventanas de tiempo individuales, por lo que pueden ser atendidos en cualquier momento dentro del horario del depósito. El modelo minimiza la distancia total recorrida respetando las restricciones de capacidad y autonomía.

**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20LogiMarket.pdf" download>Enunciado LogiMarket</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Datos_LogiMarket.xlsx" download> Datos LogiMarket</a>

**Desarrollo:**

Paso 0 — Importación de librerías

```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
```
Paso 1 — Carga de datos desde Excel

Se cargan las dos hojas del archivo `Datos_LogiMarket.xlsx`. La hoja `Info_clientes` contiene la demanda y el tiempo de servicio de cada cliente. La hoja `Distancias_Clientes` contiene la matriz de distancias en kilómetros entre el depósito y los 100 clientes.

```python
ARCHIVO = "Datos_LogiMarket.xlsx"

df_cli  = pd.read_excel(ARCHIVO, sheet_name="Info_clientes")
df_dist = pd.read_excel(ARCHIVO, sheet_name="Distancias_Clientes", index_col=0)
```

Paso 2 — Parámetros del problema y escalado

Los parámetros de la flota son comunes a todos los vehículos y se definen directamente en el código. El depósito opera de 7:00 a.m. a 11:00 p.m., lo que en minutos desde las 00:00 corresponde a `[420, 1380]`.

Las distancias ya son enteros en kilómetros y no se escalan. Sin embargo, la duración de cada arco se calcula como `distancia / 36 × 60` en minutos, que resulta decimal. Se escala por `ESCALA = 100` para preservar precisión. Todos los tiempos del modelo se escalan por el mismo factor. Al reportar, la duración se desescala dividiendo por `ESCALA`.

```python
ESCALA    = 100
VELOCIDAD = 36.0   # km/h
AUTO      = 55     # km
CAP       = 125    # unidades
N_VEH     = 8

DEP_EARLY = int(7  * 60 * ESCALA)   # 07:00 → 4200
DEP_LATE  = int(23 * 60 * ESCALA)   # 23:00 → 13800
JORNADA   = int((23 - 7) * 60 * ESCALA)
```
Paso 3 — Crear el modelo, perfil y depósito

Se crea el modelo con un único perfil, ya que todos los vehículos son idénticos y comparten la misma velocidad. El depósito se agrega con el horario de operación escalado.

```python
m      = Model()
perfil = m.add_profile()

m.add_depot(
    x        = 0,
    y        = 0,
    tw_early = DEP_EARLY,
    tw_late  = DEP_LATE,
    name     = "DEPOT"
)
```

Paso 4 — Agregar los clientes

Los clientes no tienen ventanas de tiempo individuales, por lo que se les asigna el mismo horario del depósito: pueden ser atendidos en cualquier momento entre las 7:00 a.m. y las 11:00 p.m. El tiempo de servicio se escala por `ESCALA`.

```python
for _, row in df_cli.iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["Demanda"]),
        service_duration = int(row["Servicio_min"] * ESCALA),
        tw_early         = DEP_EARLY,
        tw_late          = DEP_LATE,
        name             = str(row["ID"])
    )
```

Paso 5 — Agregar el tipo de vehículo

Se agrega un único tipo de vehículo con los parámetros comunes a toda la flota. La autonomía no se escala porque las distancias tampoco se escalaron. Se usa `unit_distance_cost = 1` para que el solver minimice la distancia total.

```python
m.add_vehicle_type(
    num_available      = N_VEH,
    capacity           = CAP,
    max_distance       = AUTO,
    tw_early           = DEP_EARLY,
    tw_late            = DEP_LATE,
    shift_duration     = JORNADA,
    unit_distance_cost = 1,
    profile            = perfil,
    name               = "Electrico"
)
```

> **Nota:** `unit_distance_cost = 1` hace que el objetivo sea directamente la distancia total recorrida, lo que permite minimizarla sin necesidad de escalar costos.

Paso 6 — Agregar los arcos

La duración se calcula dividiendo la distancia entre la velocidad y multiplicando por 60 para obtener minutos, luego se escala.

```python
locs = list(m.locations)

# Self loops
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil)

# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
        dist_km = df_dist.loc[frm.name, to.name]
        dur_min = (dist_km / VELOCIDAD) * 60
        m.add_edge(frm, to,
                   distance = int(dist_km),
                   duration = int(dur_min * ESCALA),
                   profile  = perfil)
```

Paso 7 — Resolver

Se resuelve con `MaxRuntime(30)` y `seed=42`.

```python
result = m.solve(stop=MaxRuntime(30), seed=42)
sol    = result.best

print(f"Factible : {sol.is_feasible()}")
print(f"Rutas    : {sol.num_routes()}")
```

Paso 8 — Identificar la ruta con menor distancia y reportar

Se ordenan las rutas por distancia y se selecciona la de menor recorrido. Se reporta su secuencia, duración, distancia, hora de inicio, hora de fin y autonomía restante.

```python
def min_a_hora(minutos_esc):
    minutos = minutos_esc / ESCALA
    h = int(minutos) // 60
    m = int(minutos) % 60
    return f"{h:02d}:{m:02d}"

# Ordenar rutas por distancia y tomar la menor
rutas_ordenadas = sorted(sol.routes(), key=lambda r: r.distance())
ruta_min        = rutas_ordenadas[0]

vt      = m.vehicle_types[ruta_min.vehicle_type()]
nombres = [m.locations[v].name for v in ruta_min.visits()]

autonomia_restante = AUTO - ruta_min.distance()

print(f"\n{'='*55}")
print(f"  RUTA CON MENOR DISTANCIA")
print(f"{'='*55}")
print(f"  Secuencia          : DEPOT → {' → '.join(nombres)} → DEPOT")
print(f"  Distancia          : {ruta_min.distance()} km")
print(f"  Duración           : {ruta_min.duration() / ESCALA:.1f} min")
print(f"  Hora de inicio     : {min_a_hora(ruta_min.start_time())}")
print(f"  Hora de fin        : {min_a_hora(ruta_min.end_time())}")
print(f"  Autonomía restante : {autonomia_restante} km")
print(f"{'='*55}")
```
