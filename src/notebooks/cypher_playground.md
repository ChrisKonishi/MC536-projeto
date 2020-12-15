## Carregar e gerar o grafo

~~~cypher
MATCH ()-[E]-()
DELETE E;
MATCH(N)
DELETE N;

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/chris/src/data/processed/countries_processed/countries.csv' AS line
CREATE (:PAIS {CODE: line.ALPHA3});
CREATE INDEX ON :PAIS(CODE);


LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/develop/src/data/processed/cypher/happiness_norm.csv' AS LINE
MATCH (N:PAIS {CODE: LINE.COUNTRYCODE})
SET N.HAPPY = toFloat(LINE.HAPPINESSSCORE);

MATCH (N:PAIS)
WHERE N.HAPPY>=0.75
SET N:HAPPY;

MATCH (N:PAIS)
WHERE N.HAPPY>=0.5 AND N.HAPPY<0.75
SET N:SOMEWHATHAPPY;

MATCH (N:PAIS)
WHERE N.HAPPY>=0.25 AND N.HAPPY < 0.5
SET N:SOMEWHATNOTHAPPY;

MATCH (N:PAIS)
WHERE N.HAPPY<0.25
SET N:NOTHAPPY;

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/develop/src/data/processed/cypher/pais_indice.csv' AS LINE
MATCH (P:PAIS {CODE: LINE.LOCAL})
SET P.INDICE = toFloat(LINE.INDICE);

MATCH (N:PAIS)
MATCH (O:PAIS)
WHERE N<>O AND N.INDICE > O.INDICE - 0.01 AND N.INDICE < O.INDICE
CREATE (N)-[:SIMILAR]->(O);

MATCH (N)
WHERE NOT (N:PAIS)-[:SIMILAR]-(:PAIS)
DELETE N;
~~~

## Visualizar grafo

~~~cypher
MATCH (N:PAIS)-[E:SIMILAR]->(P:PAIS)
RETURN N,E,P;
~~~

## Encontrar comunidades

~~~cypher
CALL gds.graph.create(
  'countryhappy',
  'PAIS',
  {
    SIMILAR: {
      orientation: 'UNDIRECTED'
    }
  }
);
~~~

## Vértices

~~~cypher
CALL gds.louvain.stream('countryhappy6')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).CODE AS id, communityId AS modularity_class, gds.util.asNode(nodeId).HAPPY as happy
ORDER BY communityId ASC;
~~~

## Arestas

~~~cypher
match (n:PAIS)-[e:SIMILAR]->(o:PAIS)
return n.CODE as source, o.CODE as target;
~~~