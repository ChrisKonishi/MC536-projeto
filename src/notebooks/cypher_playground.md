MATCH ()-[E]-()
DELETE E;
MATCH(N)
DELETE N;

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/main/stage03/notebooks/countries_processed/countries.csv' AS line
CREATE (:PAIS {CODE: line.ALPHA3});

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/main/stage03/notebooks/WHR_processed/whr_processado.csv' AS LINE
MATCH (N:PAIS {CODE: LINE.ALPHA3})
SET N.HAPPY = toFloat(LINE.HAPPINESSSCORE);

MATCH (N:PAIS)
WHERE N.HAPPY>4
SET N:HAPPY;

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/chris/src/notebooks/database/pais_indice.csv' AS LINE
MATCH (P:PAIS {CODE: LINE.LOCAL})
SET P.INDICE = toFloat(LINE.INDICE);

MATCH (N:PAIS)
MATCH (O:PAIS)
WHERE N<>O AND N.INDICE > O.INDICE - 0.01 AND N.INDICE < O.INDICE + 0.01
CREATE (N)-[:SIMILAR]->(O);

MATCH (N:PAIS)-[E:SIMILAR]->(P:PAIS)
RETURN N,E,P;