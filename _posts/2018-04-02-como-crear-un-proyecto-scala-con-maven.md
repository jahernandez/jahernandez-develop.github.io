---
title:		"Cómo crear un proyecto Scala con Maven"
description:	Describe paso a paso cómo crear un proyecto para código en Scala utilziando el gestor de dependencias Maven
tags:		scala maven
---

En esta entrada voy a describir paso a paso como crear un proyecto para código Scala utilizando el gestor de dependencias Maven y partiendo del arquetipo indicado por la documentación oficial.

Para la elaboración de esta guía se han empleado las versiones de las siguientes tecnologías:

* Java 1.8
* Maven 3.5.3
* Scala 2.11.12

# Generación del proyecto

Para generar la estructura básica del proyecto vamos a utilizar el arquetipo que proporciona el plugin para Maven de Scala. Un arquetipo no es más que una plantilla parametrizada con una estructura de proyecto predefinida. Indicaremos la última versión disponible hasta la fecha.


Para crear el proyecto a partir del arquetipo ejecutaremos:

```bash
mvn archetype:generate -B -DarchetypeGroupId=net.alchim31.maven -DarchetypeArtifactId=scala-archetype-simple -DarchetypeVersion=1.6 -DgroupId=com.josealbertohdez -DartifactId=scala-maven-project-example -Dversion=1.0.0-SNAPSHOT
```

donde los argumentos `groupId`, `artifactId` y `version` son los correspondientes a nuestro proyecto.

Una vez generado podemos ver que nos ha generado una estructura como ésta:

```
scala-maven-project-example/
├── pom.xml
└── src
    ├── main
    │   └── scala
    │       └── com
    │           └── josealbertohdez
    │               └── App.scala
    └── test
        └── scala
            └── samples
                ├── junit.scala
                ├── scalatest.scala
                └── specs.scala
```

# Configuración del proyecto generado

A continuación tenemos que realizar una serie de modificaciones en el `pom.xml` del proyecto.

Por un lado, hay que cambiar la versión de Java a la 1.8 de las propiedades `maven.compiler.source` y `maven.compiler.target`. A fecha de este artículo, todavía Scala no es compatible con Java 9. 

Por otro lado, la versión de Scala que vamos a usar en el proyecto, que en este caso he especificado la última versión de la 2.11.

```xml
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <encoding>UTF-8</encoding>
    <scala.version>2.11.12</scala.version>
    <scala.compat.version>2.11</scala.compat.version>
</properties>
```

En cuanto a la versión del plugin `scala-maven-plugin` conviene indicar la última versión estable publicada en su [página](https://davidb.github.com/scala-maven-plugin) que en este caso es la 3.3.2. También de su sección de configuración hay que eliminar la siguiente línea ya que si no, no compilará:

```xml
<arg>-make:transitive</arg>
```

Después hay que incluir la siguiente dependencia para que compilen los tests:

```xml
<dependency>
	<groupId>org.specs2</groupId>
	<artifactId>specs2-junit_${scala.compat.version}</artifactId>
	<version>2.4.16</version>
	<scope>test</scope>
</dependency>
```

Por último, solo queda ejecutar el siguiente comando para compilar y ejecutar los test a partir del cual empezaremos nuestro proyecto en Scala:

```bash
mvn package
```

# Notas finales

En esta entrada hemos visto como crear la estructura de un proyecto Scala con Maven a partir del arquetipo oficial. A pesar de ello, no está muy pulida y hay que modificar algunos aspectos del `pom.xml` para que el proyecto compile y ejecute los tests.

Asimismo, este arquetipo sólo crea proyectos con estructura simple. Si queremos crear uno multimódulo una opción es utilizarlo para crear cada uno de los módulos que vayan a ser proyecto Scala. Después crearemos el proyecto padre y unificaremos las definiciones repetidas y comunes en este proyecto.

El código fuente final se encuentra disponible en [GitHub](https://github.com/jahernandez/scala-maven-project-example).
