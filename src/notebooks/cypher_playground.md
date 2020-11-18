LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/main/stage03/notebooks/countries_processed/countries.csv' AS line
CREATE (:PAIS {CODE: line.ALPHA3);

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/ChrisKonishi/MC536-projeto/main/stage03/notebooks/WHR_processed/whr_processado.csv' AS LINE
MATCH (N:PAIS {CODE: LINE.ALPHA3})
SET N.HAPPY = toFloat(LINE.HAPPINESSSCORE);

MATCH (N:PAIS)
WHERE N.HAPPY>4
SET N:HAPPY;