## Tarefa de análises feitas no Cypher

## Exercício 1

Calcule o Pagerank do exemplo da Wikipedia em Cypher:

Carrego o grafo para o Neo4J:
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/network/pagerank/pagerank-wikipedia.csv' AS line
MERGE (p1:Page {name:line.source})
MERGE (p2:Page {name:line.target})
CREATE (p1)-[:LINKS]->(p2)
~~~
Exibo o grafo:
~~~
MATCH (n:Page)
RETURN n
~~~
Crio uma projeção do grafo na memória.
~~~
CALL gds.graph.create(
  'prGraph',
  'Page',
  'LINKS'
)
~~~

~~~
CALL gds.pageRank.stream('prGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC, name ASC
~~~

~~~
CALL gds.pageRank.stream('prGraph')
YIELD nodeId, score
MATCH (p:Page {name: gds.util.asNode(nodeId).name})
SET p.pagerank = score
~~~

~~~
CALL gds.pageRank.stream('prGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score AS pagerank
~~~
![PageRank](images/graph.png)
~~~cypher
name | score
--|--------
B |3.422628361545502
C |3.0444085506722334
E |0.7503553031226835
D |0.36260066517154566
F |0.36260066517154566
A |0.3041052853903018
G |0.15000000000000002
H |0.15000000000000002
I |0.15000000000000002
J |0.15000000000000002
K |0.15000000000000002
~~~

## Exercício 2

Departing from a Drug-Drug graph created in a previous lab, whose relationship determines drugs taken together, apply a community detection in it to see the results:

Criando uma nova projeção:
~~~cypher
CALL gds.graph.create(
  'DrugGraph',
  'Drug',
  {
    Relates: {
      orientation: 'UNDIRECTED'
    }
  }
)
~~~
Detectando uma comunidade utilizando Louvain Algorithm:
~~~
CALL gds.louvain.stream('DrugGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId
ORDER BY communityId ASC
~~~
Transferindo a comunidade para os nós:
~~~
CALL gds.louvain.stream('DrugGraph')
YIELD nodeId, communityId
MATCH (d:Drug {name: gds.util.asNode(nodeId).name})
SET d.community = communityId
~~~
Exportando a comunidade para visualização:
~~~
CALL gds.louvain.stream('DrugGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId
~~~
