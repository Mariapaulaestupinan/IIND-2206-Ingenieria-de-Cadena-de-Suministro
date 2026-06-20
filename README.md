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

**Literal b — Menor costo**

  Ruta (ciudades) : Atlanta  →  Houston  →  Phoenix  →  Las Vegas
  
  Arcos           : 3
  
  Costo total     : $16,791.55
  
  Distancia total : 3,434.0 km
  
  Tiempo total    : 25.33 h  (1520 min)
  
**Literal c — Menor tiempo**

Ruta (ciudades) : Atlanta  →  Seattle  →  Dallas  →  Washington DC  →  Las Vegas

Arcos           : 4

Costo total     : $52,342.65

Distancia total : 11,722.0 km

Tiempo total    : 21.27 h  (1276 min)

</details>
<details>
<summary> Red Vial en Bogotá </summary>
Una empresa de mensajería urbana en Bogotá necesita calcular la ruta de menor costo para un mensajero en moto que parte desde la intersección 1 y debe llegar a la intersección 12. El costo de cada tramo no depende únicamente de la distancia recorrida, sino también de la velocidad del tráfico según la hora del día, la presencia de peajes en la vía y la cantidad de semáforos.
    
**Enunciado:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Enunciado%20Red%20vial%20en%20Bogot%C3%A1.pdf" download>Enunciado Red Vial en Bogotá</a>

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/datos_red_vial_bogota.xlsx" download>Base de Datos Red vial en Bogotá</a>

**Solución:** 

</details>

<details>
<summary>🔴 Nivel Difícil — Próximamente</summary>

> En construcción

</details>




