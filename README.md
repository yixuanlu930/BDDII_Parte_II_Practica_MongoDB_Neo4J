# BDDII Parte II - Practica MongoDB y Neo4j

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
