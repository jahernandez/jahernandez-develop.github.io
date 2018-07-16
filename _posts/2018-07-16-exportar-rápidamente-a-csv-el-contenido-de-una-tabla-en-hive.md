---
title:		"Exportar rápidamente a CSV el contenido de una tabla en Hive"
description:	Describe las diferentes opciones que tenemos para exportar a CSV cierta información de una tabla en Hive
tags:		hive impala
---

En diversas ocasiones necesitamos exportar a CSV el resultado de una consulta sobre una tabla en Hive. Aunque lo habitual es ir al interfaz web de Hive de la distribución que estemos usando, realizar la consulta y exportar los resultados a CSV, también podemos hacerlo vía shell.


Como tabla en Hive vamos a crear una que va a contener el padrón municipal de habitantes de la ciudad de Madrid de Junio de 2018 en formato CSV [link](https://datos.madrid.es/egob/catalogo/209163-112-padron-municipal-historico.csv):

```sql
CREATE EXTERNAL TABLE padron_municipal_madrid_raw (
  cod_distrito string,
  desc_distrito string,
  cod_dist_barrio string,
  desc_barrio string,
  cod_barrio string,
  cod_dist_seccion string,
  cod_seccion string,
  cod_edad_int string,
  espanoleshombres string,
  espanolesmujeres string,
  extranjeroshombres string,
  extranjerosmujeres string)
PARTITIONED BY (
  fecha string)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  'quoteChar'='\"',
  'separatorChar'='\;')
STORED AS TEXTFILE
TBLPROPERTIES (
  'skip.header.line.count'='1'
);
```

Creamos la partición y subimos a HDFS el fichero descargado con el padrón.

```sql
ALTER TABLE padron_municipal_madrid_raw ADD PARTITION(fecha='2018-06-01');
```

Nos creamos la tabla final a partir de la que realizaremos la extracción:

```sql
CREATE EXTERNAL TABLE padron_municipal_madrid (
  cod_distrito string,
  desc_distrito string,
  cod_dist_barrio string,
  desc_barrio string,
  cod_barrio string,
  cod_dist_seccion string,
  cod_seccion string,
  cod_edad_int int,
  espanoleshombres int,
  espanolesmujeres int,
  extranjeroshombres int,
  extranjerosmujeres int)
PARTITIONED BY (
  fecha string)
STORED AS PARQUET;
```

La rellenamos con datos creando la partición por fecha automáticamente:

```sql
SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT OVERWRITE table padron_municipal_madrid PARTITION(fecha) SELECT * FROM padron_municipal_madrid_raw;
```

Nos creamos un fichero `query.hql` donde incluiremos la consulta a realizar, la cual obtiene el número total de mujeres y hombres segmentado por distrito.

```sql
SELECT TRIM(desc_distrito) as distrito, CAST(SUM(espanoleshombres + espanolesmujeres + extranjeroshombres + extranjerosmujeres) AS INT) AS total FROM padron_municipal_madrid where fecha = '2018-06-01' group by cod_distrito, desc_distrito order by distrito; 
```
## Exportación a CSV a través de Beeline

Llegados a este punto podemos generar un fichero en formato CSV con el resultado de la consulta anterior:

```bash
beeline -u jdbc:hive2://localhost:10000 --showHeader=true --outputformat=csv2 --silent=true -f query.sql > informe.csv
```

Resultando el siguiente contenido:

```
distrito,total
ARGANZUELA,52543
BARAJAS,17904
CARABANCHEL,133187
CENTRO,76960
CHAMARTIN,44841
CHAMBERI,43955
CIUDAD LINEAL,84392
FUENCARRAL-EL PARDO,68838
HORTALEZA,74133
LATINA,93766
MONCLOA-ARAVACA,39910
MORATALAZ,20071
PUENTE DE VALLECAS,111632
RETIRO,25414
SALAMANCA,49777
SAN BLAS-CANILLEJAS,58174
TETUAN,83243
USERA,82236
VICALVARO,27959
VILLA DE VALLECAS,53475
VILLAVERDE,80333
```

## Exportación a CSV via impala-shell

Si estamos utilizando la distribución de Cloudera, podemos además utilizar Impala para llevar a cabo la misma extracción:

```bash
impala-shell -i localhost:21000 -f query.sql --quiet --print_header -B --output_delimiter=',' -o informe.csv
```

## Notas finales

En este entrada se ha planteado un caso práctico en el que se ha descrito cómo realizar desde la shell una extracción a CSV de los resultados de una consulta sobre una tabla Hive. Para ello se ha mostrado cómo utilizar la interfaz de comandos Beeline presente en cualquier distribución así como Impala disponible en Cloudera. 

