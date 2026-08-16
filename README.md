# MongoDB & Neo4j Game Data Modeling

A NoSQL database project exploring **document-oriented modeling with MongoDB** and **graph-oriented modeling with Neo4j** using a video-game data domain based on **Jotun's Lair**.

The project demonstrates how the same complex game ecosystem can benefit from different NoSQL paradigms depending on the required access patterns.

It includes MongoDB aggregation pipelines, API-oriented queries, schema optimization patterns, Cypher graph queries, weapon-crafting analysis, and graph algorithms using **Neo4j Graph Data Science**.

---

## Overview

The repository is divided into two main sections:

1. **MongoDB** — document-oriented storage and query optimization for users, rooms, monsters, loot, dungeons, and comments.
2. **Neo4j** — graph modeling and analysis of items, monsters, recipes, weapons, crafting requirements, upgrades, and elemental damage.

The project explores not only how to query both databases, but also **how their data models should be designed according to expected application workloads**.

---

# MongoDB

The MongoDB section models the Jotun's Lair game environment using document-oriented collections.

The main datasets include:

```text
users
rooms
monsters
loot
```

Source data is provided as JSON files under:

```text
datos/
├── users.json
├── rooms.json
├── monster.json
└── loot.json
```

---

## MongoDB API Operations

Python functions implemented with **PyMongo** reproduce several REST-style operations.

The supported use cases include:

```text
GET    /user/{email}
GET    /room/{room_id}
GET    /dungeon/{dungeon_id}
POST   /comment
DELETE /monster/{monster_id}
```

Database-side operations are prioritized whenever possible instead of performing unnecessary calculations in Python.

---

## User Queries

The user query retrieves complete player information together with the **20 most recent comments** made by that user.

Comment information includes:

* Text
* Creation date
* Category
* Room ID
* Room name
* Dungeon ID
* Dungeon name

MongoDB operators such as array sorting and aggregation are used to process this information directly inside the database.

---

## Room Queries

Room-related queries combine information including:

* Monsters
* Loot
* Comments
* Dungeon information
* Gold values
* Comment categories

These operations demonstrate how nested documents and arrays can be manipulated efficiently using MongoDB.

---

## Dungeon Queries

Dungeon queries aggregate information across multiple rooms and related embedded structures.

The project explores how document-oriented modeling can reduce the number of database operations required for high-frequency application endpoints.

---

# MongoDB Aggregation Pipelines

Several analytical queries are implemented using MongoDB's aggregation framework.

Examples include:

### Monsters in the Rooms with the Most Bug Reports

The pipeline:

1. Counts `bug` comments in each room.
2. Sorts rooms by the number of bug reports.
3. Selects the top rooms.
4. Unwinds their monsters.
5. Returns the unique monsters involved.

---

### Dungeons with Above-Average Hint Activity

The system:

1. Counts `hint` comments per room.
2. Aggregates them by dungeon.
3. Calculates the global average.
4. Returns dungeons whose hint count is above that average.

---

### Dungeon-Level Analytics

Other analytical operations combine information such as:

* Comment categories
* Total gold
* Monster information
* Player levels
* User countries
* Room statistics

These queries demonstrate the use of:

* `$match`
* `$filter`
* `$size`
* `$addFields`
* `$unwind`
* `$group`
* `$project`
* `$sort`
* `$limit`
* `$expr`

---

# MongoDB Design Patterns

The project evaluates several MongoDB schema design patterns according to the expected application workload.

The analyzed system has significantly more reads than writes, including high-frequency endpoints such as:

```text
GET /dungeon    ~1M requests/day
GET /room       ~500K requests/day
GET /user       ~300K requests/day
```

while operations such as comment creation or monster deletion occur much less frequently.

This workload motivates several optimization strategies.

---

## Subset Pattern

Instead of storing every historical comment directly inside frequently accessed room documents, only the most relevant subset can remain embedded.

For example:

```text
20 most recent comments
```

Older comments can be moved into a separate collection.

This reduces document size while preserving fast access to commonly requested information.

---

## Computed Pattern

Frequently requested values can be calculated in advance and stored directly in the document.

Examples include:

```text
total_gold
monster_count_by_type
comments_by_category
```

This avoids repeatedly executing expensive aggregations for high-volume reads.

---

## Outlier Pattern

Some rooms may contain unusually large numbers of:

* Monsters
* Loot items
* Comments

These outlier documents can be handled differently from normal rooms to prevent them from negatively affecting common query performance.

---

# Comment Schema Redesign

The project also explores extracting embedded comments into an independent collection.

Instead of duplicating comments across user and room documents, a central:

```text
Hints
```

collection can store the relationship between:

* User
* Room
* Dungeon
* Comment
* Category
* Creation date

This reduces duplication and improves consistency.

The notebooks also analyze how existing API operations would need to change after this redesign.

---

# Neo4j

The second half of the project represents the game's crafting ecosystem as a graph.

The model includes concepts such as:

```text
Weapon
Item
Recipe
WeaponCraft
WeaponUpgrade
Monster
Element
```

and relationships including:

```text
NEEDS
PRODUCES
DROPS
DEALS
```

This graph representation makes it possible to naturally express complex relationships between weapons, crafting materials, monsters, recipes, and elemental damage.

---

# Cypher Queries

The Neo4j section contains several advanced Cypher queries.

## Unobtainable Crafting Materials

The graph identifies items that:

* Cannot be produced through a recipe.
* Cannot be dropped by a monster.
* Are nevertheless required for crafting or upgrading a weapon.

Conceptually:

```text
Item
 ├── NOT produced by Recipe
 ├── NOT dropped by Monster
 └── required by WeaponCraft / WeaponUpgrade
```

---

## Most Profitable Recipes

Recipes are ranked according to:

```text
profit =
value of generated items
-
value of required items
```

The query also filters recipes whose input materials are not themselves produced by other recipes.

The top five most profitable recipes can then be returned.

---

## Best Craftable Fire Weapon

The graph identifies the directly craftable weapon with the highest fire damage.

The query also determines:

* Monsters that must be defeated to obtain required materials.
* Additional materials not dropped by those monsters.

This demonstrates the advantage of traversing interconnected game relationships using a graph database.

---

## Maximum Damage by Weapon Type

For each weapon type, the project calculates:

```text
combined damage =
base damage
+
elemental damage
```

The strongest weapon or weapons of every type are then returned and ranked according to total damage.

---

# Graph Data Science

The project goes beyond Cypher queries and uses the **Neo4j Graph Data Science library** for graph analytics.

The Python client is used through:

```python
from graphdatascience import GraphDataScience
```

with a local Neo4j connection.

---

## Weapon Similarity Graph

Weapons are connected according to similarity in their crafting or upgrade requirements.

Two weapons can therefore be considered similar when they share common materials.

Conceptually:

```text
Weapon A
   │
   ├── needs Material X
   └── needs Material Y

Weapon B
   │
   ├── needs Material X
   └── needs Material Z

        ↓

Weapon A ── SIMILAR ── Weapon B
```

This transforms crafting information into a graph suitable for graph-learning and community-analysis algorithms.

---

# Leiden Community Detection

The **Leiden algorithm** is used to identify communities of weapons with similar crafting characteristics.

The goal is to group weapons that are structurally related according to their shared material requirements.

This allows the crafting ecosystem to be analyzed from a higher-level structural perspective instead of examining each weapon individually.

---

# PageRank

After detecting communities, **PageRank** is applied to identify representative or influential weapons within those groups.

The workflow is approximately:

```text
Weapon crafting data
        │
        ▼
Shared material relationships
        │
        ▼
SIMILAR graph
        │
        ▼
Leiden communities
        │
        ▼
PageRank
        │
        ▼
Representative weapons
```

This demonstrates how traditional graph algorithms can be applied to a game-data domain.

---

# Graph Model Design

The project also evaluates alternative Neo4j modeling decisions.

One example is whether:

```text
Weapon.kind
```

should remain a repeated string property or become an independent:

```text
(:WeaponKind)
```

node.

The dataset contains a small number of weapon kinds shared by many weapon nodes.

Representing weapon types as nodes can therefore provide advantages such as:

* Reduced duplication
* Explicit relationships
* More expressive graph traversals
* Natural many-to-many modeling
* Easier model evolution

The project discusses these trade-offs rather than treating graph modeling as purely mechanical.

---

# Project Structure

```text
mongodb-neo4j-game-data-modeling/
│
├── crafteo_de_armas/
│   ├── Neo4J_consultas.ipynb
│   ├── Neo4J_consultas.txt
│   ├── GraphDataScience.ipynb
│   ├── RecomendacionesDiseno.ipynb
│   │
│   └── import/
│       └── neo4j.dump
│
├── wiki/
│   ├── tarea_1.ipynb
│   ├── tarea_2.ipynb
│   ├── tarea-3.ipynb
│   ├── tarea-4.ipynb
│   │
│   └── resultados_mongodb/
│       ├── q1.json
│       ├── q2.json
│       ├── q3.json
│       ├── q4.json
│       ├── q5.json
│       ├── q6.json
│       └── q7.json
│
├── datos/
│   ├── users.json
│   ├── rooms.json
│   ├── monster.json
│   └── loot.json
│
├── scripts/
│   ├── iniciar_mongodb.txt
│   └── inicializar_neo4j.txt
│
├── docs/
│   ├── Bases de datos no relacionales.pdf
│   ├── Memoria_MongoDB.pdf
│   ├── Memoria_Neo4J.pdf
│   └── guion_practica_neo4j.pdf
│
├── pyproject.toml
├── uv.lock
├── .gitignore
├── LICENSE
└── README.md
```

---

# Requirements

The Python project requires Python 3.11 or newer.

Main dependencies include:

```text
pymongo
motor
jupyter
ipykernel
```

The Neo4j notebooks additionally use:

```text
neo4j
graphdatascience
pandas
```

---

# Installation

Using `uv`:

```bash
uv sync
```

Alternatively, create a Python virtual environment and install the required libraries manually.

For example:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Then install the main dependencies:

```bash
pip install pymongo motor jupyter neo4j graphdatascience pandas
```

---

# Running MongoDB

A MongoDB Docker container can be started locally.

From the repository root:

```bash
docker run --rm \
  --name mongodb_0 \
  -v ${PWD}/mongodata_0:/data/db \
  -v ${PWD}/datos:/datos \
  -p 27017:27017 \
  mongo:latest
```

MongoDB will then be available at:

```text
mongodb://localhost:27017
```

The project uses the database:

```text
jotuns_lair
```

The JSON datasets can be imported into their corresponding collections using `mongoimport`.

---

# Running Neo4j

The Neo4j database dump is stored in:

```text
crafteo_de_armas/import/neo4j.dump
```

The project uses Neo4j 5.25 with:

* APOC
* Graph Data Science

The Neo4j Browser is exposed on:

```text
http://localhost:7474
```

and Bolt on:

```text
bolt://localhost:7687
```

The local educational configuration uses:

```text
Username: neo4j
Password: password
```

These credentials are intended only for local development.

---

# Technologies

## Databases

* MongoDB
* Neo4j

## Query Languages

* MongoDB Query Language
* MongoDB Aggregation Framework
* Cypher

## Graph Analytics

* Neo4j Graph Data Science
* Leiden Community Detection
* PageRank
* Similarity graphs

## Python

* PyMongo
* Motor
* Neo4j Python Driver
* Graph Data Science Python Client
* Pandas
* Jupyter Notebook

## Infrastructure

* Docker

---

# Key Concepts

This project explores:

* NoSQL databases
* Document databases
* Graph databases
* Document embedding
* Data denormalization
* MongoDB aggregation pipelines
* MongoDB schema design patterns
* Query-driven modeling
* Cypher
* Graph traversal
* Graph modeling
* Community detection
* Centrality algorithms
* PageRank
* Leiden
* Similarity networks
* Data-model optimization

---

# MongoDB vs. Neo4j

The project illustrates why different data models are suitable for different problems.

| MongoDB                       | Neo4j                            |
| ----------------------------- | -------------------------------- |
| Document-oriented             | Graph-oriented                   |
| Efficient aggregate documents | Efficient relationship traversal |
| Embedded structures           | Explicit relationships           |
| Aggregation pipelines         | Cypher pattern matching          |
| API-oriented workloads        | Connectivity-oriented workloads  |
| Schema design patterns        | Graph modeling and algorithms    |

MongoDB is particularly useful for retrieving complete application entities and nested structures, while Neo4j provides a natural representation for highly interconnected crafting relationships.

---

# Academic Context

This project was developed as part of the **Databases II** coursework in the Bachelor's Degree in Data Science and Artificial Intelligence.

The objective was to explore NoSQL database design using both document and graph paradigms and to understand how database architecture should be adapted to different query patterns and analytical requirements.

# License

See the repository license for applicable terms.


---

## Spanish translation:

## BDDII Parte II - Practica MongoDB y Neo4j

Repositorio de la practica de Bases de Datos No Relacionales con dos bloques principales:
- MongoDB (consultas y resultados)
- Neo4j (consultas Cypher, GDS y recomendaciones de diseno)

## Estructura ordenada

```text
.
|- MongoDB/
|  |- tarea_1.ipynb
|  |- tarea_2.ipynb
|  |- tarea-3.ipynb
|  |- tarea-4.ipynb
|- neo4j/
|  |- Neo4J_consultas.ipynb
|  |- Neo4J_consultas.txt
|  |- GraphDataScience.ipynb
|  |- RecomendacionesDiseno.ipynb
|  |- import/
|- datos/
|  |- users.json
|  |- monster.json
|  |- rooms.json
|  |- loot.json
|- resultados_mongodb/
|  |- q1.json ... q7.json
|- docs/
|  |- Bases de datos no relacionales.pdf
|  |- guion_practica_neo4j.pdf
|  |- Memoria_MongoDB.pdf
|  |- Memoria_Neo4J.pdf
|- scripts/
|  |- iniciar_mongodb.txt
|  |- inicializar_neo4j.txt
|- LICENSE
|- pyproject.toml
|- uv.lock
```

## Notas de GitHub

Este repo ya ignora datos locales generados en ejecucion para evitar subir archivos pesados o temporales:
- mongodata/
- neo4j/data/
- neo4j/plugins/
- .venv/
- .DS_Store

## Ejecucion rapida

1. MongoDB:
- Usa los comandos de scripts/iniciar_mongodb.txt
- Ejecuta los notebooks en MongoDB/

2. Neo4j:
- Usa los comandos de scripts/inicializar_neo4j.txt
- Ejecuta los notebooks en neo4j/

## Archivos de trabajo en raiz

Existen notebooks en la raiz (por ejemplo, tarea-4.ipynb y Neo4J_consultas.ipynb) como versiones de trabajo personales.
Si quieres un repositorio aun mas estricto, se pueden mover a una carpeta de archivo en un siguiente paso.
