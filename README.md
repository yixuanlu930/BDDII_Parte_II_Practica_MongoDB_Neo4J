# Parte_II_Pr-ctica_MongoDB_Neo4J

Proof-of-concept migration from a relational database to MongoDB for "Jotun's Lair", a fictional MMO wiki. Covers REST API endpoint implementation in Python, advanced aggregation pipelines (outlier detection, percentiles, graph data) and schema evolution with design patterns.


# Práctica MongoDB

# Práctica Neo4j - Jotun's Lair

Este proyecto contiene la parte de **Neo4j** de la práctica de Bases de Datos No Relacionales. El caso de estudio es el sistema de crafteo del videojuego ficticio **Jotun's Lair**, donde se modelan armas, monstruos, materiales, recetas, localizaciones y elementos mediante una base de datos de grafos.

La práctica se divide en tres bloques principales:

1. Consultas Cypher sobre el grafo.
2. Análisis de grafos con Graph Data Science.
3. Recomendaciones de diseño del modelo.

---

## 1. Estructura del proyecto

```text
.
├── Neo4J_consultas.ipynb
├── Neo4J_consultas.txt
├── GraphDataScience.ipynb
├── RecomendacionesDiseño.ipynb
├── inicializar_neo4j.txt
├── README.md
└── neo4j/
    ├── data/
    ├── import/
    │   └── neo4j.dump
    └── plugins/
```

### Archivos principales

- `Neo4J_consultas.ipynb`: notebook con las 10 consultas Cypher de la práctica.
- `Neo4J_consultas.txt`: versión en texto de las consultas y sus explicaciones.
- `GraphDataScience.ipynb`: notebook con el análisis de grafos usando Graph Data Science.
- `RecomendacionesDiseño.ipynb`: notebook con la parte de recomendaciones de diseño.
- `inicializar_neo4j.txt`: comandos para cargar el dump y arrancar Neo4j.
- `neo4j/import/neo4j.dump`: dump de la base de datos Neo4j, si se incluye localmente.

> Las carpetas `neo4j/data/` y `neo4j/plugins/` son generadas por Neo4j y no deberían subirse a GitHub si contienen archivos pesados.

---

## 2. Requisitos

Para ejecutar la práctica se necesita:

- Docker
- Python 3.10 o superior
- Jupyter Notebook
- Neo4j 5.25
- Plugin APOC
- Plugin Graph Data Science

Librerías de Python utilizadas:

```bash
pip install graphdatascience pandas neo4j jupyter notebook
```

---

## 3. Cargar la base de datos Neo4j

Primero se carga el archivo `neo4j.dump` dentro de la base de datos `neo4j`.

Desde la carpeta `neo4j/`, ejecutar:

```bash
docker run -it --rm \
  --volume="$PWD/data:/data" \
  --volume="$PWD/import:/backups" \
  neo4j:5.25 \
  neo4j-admin database load neo4j --from-path=/backups --overwrite-destination=true --verbose
```

Este comando carga el dump almacenado en `import/` y lo restaura dentro de la carpeta `data/`.

---

## 4. Arrancar Neo4j

Después de cargar la base de datos, se arranca Neo4j con los plugins necesarios:

```bash
docker run --rm \
  --name neo4j-jotuns \
  --volume="$PWD/data:/data" \
  --volume="$PWD/plugins:/plugins" \
  --publish=7474:7474 \
  --publish=7687:7687 \
  --env NEO4J_AUTH=neo4j/password \
  --env NEO4J_PLUGINS='["apoc","graph-data-science"]' \
  neo4j:5.25
```

Acceso a la interfaz web:

```text
http://localhost:7474
```

Credenciales:

```text
Usuario: neo4j
Contraseña: password
```

El puerto `7474` se usa para Neo4j Browser y el puerto `7687` para la conexión Bolt desde Python.

---

## 5. Consultas Cypher

El notebook `Neo4J_consultas.ipynb` contiene las 10 consultas pedidas en el enunciado.

Las consultas trabajan sobre el modelo de grafos de la práctica, que incluye:

- `Weapon`
- `WeaponCraft`
- `WeaponUpgrade`
- `Item`
- `Recipe`
- `Monster`
- `Location`
- `Element`

Relaciones principales:

- `PRODUCES`
- `NEEDS`
- `TRANSFORMS`
- `DROPS`
- `LIVES_IN`
- `WEAK_TO`
- `RESISTENT_TO`
- `DEALS`

Las consultas resuelven tareas como encontrar materiales sin forma directa de obtención, calcular recetas rentables, reconstruir cadenas de mejora, obtener monstruos necesarios para fabricar armas y recomendar armas según localizaciones y debilidades.

---

## 6. Análisis con Graph Data Science

El notebook `GraphDataScience.ipynb` implementa el apartado de análisis de grafos.

El objetivo es agrupar armas según sus materiales de fabricación o actualización.

### Paso 1: crear relaciones `SIMILAR`

Se crea una relación `SIMILAR` entre dos armas si comparten al menos un material.

El peso de la relación se guarda en la propiedad `weight` y se calcula mediante el índice de Jaccard:

```text
Jaccard = materiales comunes / materiales distintos totales
```

Ejemplo:

```text
Arma A: [Dragonita, Cola de Quematrice, Piedra de fuego]
Arma B: [Cristal de tierra, Machalita, Dragonita]

Intersección = 1
Unión = 5

Jaccard = 1 / 5 = 0.2
```

### Paso 2: detección de comunidades con Leiden

Después se proyecta el grafo de armas en GDS usando las relaciones `SIMILAR` como no dirigidas.

Se aplica Leiden para detectar comunidades de armas similares:

```python
gds.leiden.write(
    G,
    writeProperty="comunidad",
    relationshipWeightProperty="weight",
    randomSeed=19
)
```

El resultado se guarda en cada nodo `Weapon` mediante la propiedad `comunidad`.

### Paso 3: PageRank por comunidad

Finalmente se aplica PageRank dentro de cada comunidad para encontrar el arma o armas más centrales.

La relación `SIMILAR` se trata como no dirigida porque la similitud es simétrica: si un arma A es similar a un arma B, B también es similar a A.

Para cada comunidad se crea un subgrafo con `gds.graph.filter()` y se ejecuta PageRank usando `weight` como peso.

Las armas con mayor PageRank dentro de cada comunidad se consideran representantes de esa comunidad.

---

## 7. Recomendaciones de diseño

El notebook `RecomendacionesDiseño.ipynb` analiza si tiene sentido cambiar el diseño del modelo para convertir el tipo de arma (`kind`) en un nodo independiente `WeaponKind`.

Diseño actual:

```text
(:Weapon {kind: "sword"})
```

Diseño alternativo:

```text
(:Weapon)-[:HAS_KIND]->(:WeaponKind {name: "sword"})
```

La conclusión es que `kind` puede mantenerse como propiedad si solo se usa como categoría simple para filtrar o agrupar armas.

Sin embargo, tendría sentido convertirlo en nodo si:

- los tipos de arma tuvieran atributos propios;
- existieran subtipos o jerarquías;
- se consultaran muy frecuentemente como punto de entrada;
- se quisieran relacionar con otros nodos;
- se fueran a usar en análisis de grafos.

En la base actual, los tipos aparecen como valores planos (`bow`, `great-sword`, `hammer`, etc.), por lo que mantener `kind` como propiedad es suficiente.

---

## 8. Ejecución recomendada

1. Arrancar Neo4j.
2. Abrir `Neo4J_consultas.ipynb`.
3. Ejecutar las 10 consultas Cypher.
4. Abrir `GraphDataScience.ipynb`.
5. Ejecutar la creación de relaciones `SIMILAR`.
6. Ejecutar Leiden.
7. Ejecutar PageRank por comunidad.
8. Abrir `RecomendacionesDiseño.ipynb`.
9. Revisar la justificación del cambio `kind` a `WeaponKind`.

---

## 9. Limpieza

Para detener Neo4j:

```bash
Ctrl + C
```

Si se quiere eliminar el contenedor manualmente:

```bash
docker rm -f neo4j-jotuns
```

Si se quieren eliminar datos locales generados por Neo4j:

```bash
rm -rf neo4j/data
```

---

## 10. Notas sobre GitHub

No se recomienda subir a GitHub las carpetas generadas por Neo4j:

```gitignore
neo4j/data/
neo4j/plugins/
*.dump
```

Estas carpetas pueden contener archivos muy grandes, como transacciones internas de Neo4j o plugins `.jar`, que superan los límites recomendados por GitHub.

---

## Autores

Yixuan Lu Guo y Shengkai zu. Práctica realizada para la asignatura de Bases de Datos No Relacionales, centrada en Neo4j, consultas Cypher, Graph Data Science y recomendaciones de diseño de grafos.

