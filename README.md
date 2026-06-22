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
  - [Ejercicios](#ejercicios)
- [Dijkstra](#dijkstra)
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

### Ejercicios

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

#### Pseudocódigo del algoritmo de Dijkstra

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
#### Ejercicios 
En esta sección se desarrollan ejercicios de ruta más corta que requieren implementar el algoritmo de Dijkstra de forma manual, adaptando su lógica de exploración a restricciones que no pueden resolverse con librerías estándar como NetworkX. Cada ejercicio introduce una modificación diferente al estado del algoritmo, lo que permite comprender cómo extender Dijkstra más allá de su formulación clásica para abordar problemas reales con condiciones dinámicas sobre los arcos y los nodos.

<details>
<summary> Red de Distribución RiskShield </summary>
RiskShield Logistics es una empresa especializada en el transporte de sustancias químicas peligrosas que debe entregar un cargamento de peróxido de hidrógeno industrial desde Seattle hasta Houston. La empresa opera una red de rutas terrestres y ferroviarias a lo largo de Estados Unidos, y debe encontrar la ruta de menor costo garantizando que la exposición al riesgo operativo a lo largo del recorrido no supere los límites establecidos para cada ciudad intermedia.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20de%20Distribuci%C3%B3n%20RiskShield.pdf" download>Enunciado Red de Distribución RiskShield</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/red_RiskShield.xlsx" download>red_RiskShield</a>

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

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/red_FreshChain.xlsx" download>red_FreshChain</a>

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
    dist = float(row["Distancia_Km"])
    vel  = float(row["Velocidad_KmH"])
    modo = row["Modo_Transporte"]
    pea  = float(row["Peaje_USD"])

    tij = dist / vel
    cij = Um[modo] * dist + CO * tij + pea

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

Como el destino puede haberse alcanzado a través de distintos modos de transporte, pueden existir múltiples estados `(Cartagena, modo)` dentro del diccionario `costo`, cada uno con un costo acumulado diferente dependiendo de la ruta recorrida y los transbordos realizados.

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
