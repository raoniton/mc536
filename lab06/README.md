
## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1

Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
CREATE (:Drug{name:"Metamizole",drugbank:" DB04817"})
~~~

## Exercício 2

Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
MATCH (d1:Drug{name:"Dipyrone"})
MATCH (d2:Drug{name:"Metamizole"})
CREATE (d1)-[:SameAs]->(d2)
~~~


## Exercício 3
Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
MATCH (d1:Drug{name:"Dipyrone"})-[r]->(d2:Drug{name:"Metamizole"})
DELETE r
~~~

## Exercício 4
Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)-[a]->(d:Drug)<-[b]-(d2:Pathology)
WHERE a.weight > 20 AND b.weight > 20
MERGE (p1)<-[Sm:SameDrug]->(p2)
ON CREATE SET Sm.weight=1
ON MATCH SET Sm.weight=Sm.weight+1
~~~

## Exercício 5
Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (DR:Drug {code: line.codedrug})
MATCH (PA:Pathology {code: line.codepathology})
MERGE (DR)-[t:Treats]->(PA)
ON CREATE SET t.weight=1
ON MATCH SET t.weight=t.weight+1

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (PE:Person {code: line.idperson})
MATCH (DR:Drug {code: line.codedrug})
MERGE (PE)-[u:Uses]->(PA)
ON CREATE SET u.weight=1
ON MATCH SET u.weight=u.weight+1

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MATCH (PE:Person {code: line.idPerson})
MATCH (PA:Pathology {code: line.codePathology})
MERGE (PE)-[H:HAVE]->(PA)
ON CREATE SET H.weight=1
ON MATCH SET H.weight=H.weight+1
~~~

## Exercício 6

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
~~~cypher
(escreva aqui a resolução em Cypher)
~~~
