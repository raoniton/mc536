## Lab08 - PubChem e DRON com XQuery/XPath

## Tarefas com Publicações

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~sql
SELECT individuo.nome 
FROM fichario
WHERE individuo.idade > 20
~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]

return {data($i/@nome)}
~~~
### Resolução
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

return {data ($fichariodoc//individuo[idade > 17]/@nome)}
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]

return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT individuo.nome 
FROM fichario 
WHERE individuo.idade > 17
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $publications := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

return {count($publications//publication[year > 2011])}
~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $publications := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

return $publications//category[label="e-Science Domain"]'
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $publications := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

(:
essa variavel var foi usada para testar e encontrar a key referente a label e-science domain
let $var := data($publications//category[label="e-Science Domain"]/@key)
:)

(: no return eu uso a sentenca inteira ao inves da var, pois assim foi pedido no exercício :)
return {$publications//publication[key = data($publications//category[label="e-Science Domain"]/@key)]}
~~~

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')

for $d in ($dron//drug/drug/drug[drug/@name])
let $sub_dir_name := $d/@name

return {data($sub_dir_name), '&#xa;'}
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')

for $d in ($dron//drug[drug/drug/@name="VITAMIN D2"])
let $name := $d/@name
group by $name

return {data($name), '&#xa;'}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $pubchem := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $i in ($pubchem//Information)
where $i//Synonym[contains(text(), "CHEBI")]

let $result := substring($i//Synonym[contains(text(), "CHEBI")]/text(),7)

return {$result, '&#xa;'}

~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $pubchem := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $i in ($pubchem//Information)
where $i//Synonym[contains(text(), "CHEBI")]

let $result := substring($i//Synonym[contains(text(), "CHEBI:28838")]/text(),7)

return {concat('CHEBI: ', data($result), ', Synonym: ', data($i/Synonym[1]) ), '&#xa;'}

~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

#### Resolução
~~~xquery
let $pubchem := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')

return
<pubchem_dron>{

   for $p in ($pubchem//Information),
       $d in ($dron//drug)
   where concat('http://purl.obolibrary.org/obo/CHEBI_', substring($p//Synonym[contains(text(), "CHEBI")]/text(),7) ) = $d/@id

   let $synonym := ($p//Synonym)
   let $nameDron := $d/@name

   group by $nameDron
   order by $nameDron

   return
   <Name>
     {$nameDron}
      <Synonym>{ data($synonym/text()) } </Synonym> &#xa;
   </Name>
}
</pubchem_dron>

~~~