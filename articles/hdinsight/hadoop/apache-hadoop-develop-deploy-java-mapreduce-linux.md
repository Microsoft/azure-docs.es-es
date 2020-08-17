---
title: 'Creación de programas MapReduce de Java para Apache Hadoop: Azure HDInsight'
description: Aprenda a usar Apache Maven para crear una aplicación MapReduce basada en Java y ejecutarla en Hadoop de Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-java
ms.date: 01/16/2020
ms.openlocfilehash: 84d9253b865ddac6d97395af3d8632e29cc2ea24
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323815"
---
# <a name="develop-java-mapreduce-programs-for-apache-hadoop-on-hdinsight"></a>Desarrollo de programas MapReduce de Java para Apache Hadoop en HDInsight

Aprenda a usar Apache Maven para crear una aplicación MapReduce basada en Java y ejecutarla con Apache Hadoop en Azure HDInsight.

## <a name="prerequisites"></a>Prerrequisitos

* [Kit de desarrolladores de Java (JDK), versión 8](https://aka.ms/azure-jdks).

* [Apache Maven](https://maven.apache.org/download.cgi) correctamente [instalado](https://maven.apache.org/install.html) según Apache.  Maven es un sistema de compilación de proyectos de Java.

## <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

El entorno usado en este artículo fue un equipo donde se ejecuta Windows 10. Los comandos se ejecutaron en un símbolo del sistema, y los distintos archivos se editaron con el Bloc de notas. Realice las modificaciones según corresponda en su entorno.

Desde un símbolo del sistema, escriba los siguientes comandos para crear un entorno de trabajo:

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Creación de un proyecto de Maven

1. Especifique el siguiente comando para crear un proyecto de Maven llamado **wordcountjava**:

   ```bash
   mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

    Esta acción creará un directorio en el directorio actual, con el nombre especificado por el parámetro `artifactID` (**hbaseapp** en este ejemplo). Este directorio raíz contiene los siguientes elementos:

    * `pom.xml`: el [modelo de objetos de proyectos (POM)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html) que contiene la información y los detalles de configuración para compilar el proyecto.
    * src\main\java\org\apache\hadoop\examples: Contiene el código de la aplicación.
    * src\test\java\org\apache\hadoop\examples: Contiene pruebas para la aplicación.

1. Quite el código de ejemplo generado. Especifique los siguientes comandos para eliminar los archivos de aplicación y de prueba generados (`AppTest.java` y `App.java`):

    ```cmd
    cd wordcountjava
    DEL src\main\java\org\apache\hadoop\examples\App.java
    DEL src\test\java\org\apache\hadoop\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Actualización del modelo de objetos de proyectos

Para obtener una referencia completa del archivo pom.xml, vea https://maven.apache.org/pom.html. Especifique el siguiente comando para abrir `pom.xml`:

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Adición de dependencias

En la sección `pom.xml`, agregue el siguiente texto a la sección `<dependencies>`:

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-examples</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-client-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
```

Esto define las bibliotecas requeridas (enumeradas en &lt;artifactId\>) de una versión específica (enumerada en &lt;version\>). En tiempo de compilación, estas dependencias se descargan del repositorio de Maven predeterminado. Puede usar la [búsqueda del repositorio de Maven](https://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar) para ver más información.

`<scope>provided</scope>` indica a Maven que no hace falta que empaquete esas dependencias con la aplicación, puesto que las proporciona el clúster de HDInsight en tiempo de ejecución.

> [!IMPORTANT]
> La versión utilizada debe coincidir con la versión de Hadoop presente en el clúster. Para obtener más información sobre las versiones, vea el documento de [versiones de componentes de HDInsight](../hdinsight-component-versioning.md).

### <a name="build-configuration"></a>Configuración de compilación

Los complementos de Maven permiten personalizar las fases de compilación del proyecto, Esta sección se usa para agregar complementos, recursos y otras opciones de configuración de compilación.

Agregue el código siguiente al archivo `pom.xml` y, después, guárdelo y ciérrelo. Este texto debe estar dentro de las etiquetas `<project>...</project>` en el archivo; por ejemplo, entre `</dependencies>` y `</project>`.

```xml
<build>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.3</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
                <goals>
                <goal>shade</goal>
                </goals>
        </execution>
        </executions>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.6.1</version>
        <configuration>
        <source>1.8</source>
        <target>1.8</target>
        </configuration>
    </plugin>
    </plugins>
</build>
```

En esta sección se configura Apache Maven Compiler Plugin y Apache Maven Shade Plugin. El complemento compiler se usa para compilar la topología. El complemento shade se usa para evitar la duplicación de licencias en el paquete JAR compilado por Maven. Este complemento se usa para evitar errores por "archivos de licencia duplicados" en tiempo de ejecución en el clúster de HDInsight. El uso del complemento maven-shade-plugin con la implementación de `ApacheLicenseResourceTransformer` evita este error.

El complemento maven-shade-plugin también producirá un uberjar, que contiene todas las dependencias que necesita la aplicación.

Guarde el archivo `pom.xml`.

## <a name="create-the-mapreduce-application"></a>Creación de la aplicación MapReduce

1. Escriba el comando siguiente para crear un archivo `WordCount.java` y abrirlo. Seleccione **Sí** en el símbolo del sistema para crear un archivo.

    ```cmd
    notepad src\main\java\org\apache\hadoop\examples\WordCount.java
    ```

2. Luego, copie y pegue el código Java siguiente en el nuevo archivo. y ciérrelo.

    ```java
    package org.apache.hadoop.examples;

    import java.io.IOException;
    import java.util.StringTokenizer;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.util.GenericOptionsParser;

    public class WordCount {

        public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
                        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                            Context context
                            ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
            sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }
    ```

    Tenga en cuenta que el nombre del paquete es `org.apache.hadoop.examples` y el nombre de clase es `WordCount`. Estos nombres se utilizan al enviar el trabajo de MapReduce.

## <a name="build-and-package-the-application"></a>Compilación y empaquetado de la aplicación

En el directorio `wordcountjava`, use el siguiente comando para compilar un archivo JAR que contenga la aplicación:

```cmd
mvn clean package
```

Este comando elimina los artefactos de compilación anteriores, descarga las dependencias que aún no se han instalado y luego compila y empaqueta la aplicación.

Una vez que finaliza el comando, el directorio `wordcountjava/target` contiene un archivo denominado `wordcountjava-1.0-SNAPSHOT.jar`.

> [!NOTE]
> El archivo `wordcountjava-1.0-SNAPSHOT.jar` es un uberjar que contiene no solo el trabajo WordCount, sino también las dependencias que el trabajo necesita en tiempo de ejecución.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Carga del archivo JAR y ejecución de trabajos (SSH)

En los siguientes pasos se usa `scp` para copiar el archivo JAR en el nodo primario principal de Apache HBase en el clúster de HDInsight. El comando `ssh`, se usa para conectarse al clúster y ejecutar el ejemplo directamente en el nodo principal.

1. Cargue el archivo JAR en el clúster. Reemplace `CLUSTERNAME` por su nombre de clúster de HDInsight y, después, escriba el siguiente comando:

    ```cmd
    scp target/wordcountjava-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Conéctese al clúster. Reemplace `CLUSTERNAME` por su nombre de clúster de HDInsight y, después, escriba el siguiente comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. En la sesión SSH, use el siguiente comando para ejecutar la aplicación MapReduce:

   ```bash
   yarn jar wordcountjava-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /example/data/gutenberg/davinci.txt /example/data/wordcountout
   ```

    Este comando inicia la aplicación WordCount MapReduce. El archivo de entrada es `/example/data/gutenberg/davinci.txt`, y el directorio de salida, `/example/data/wordcountout`. Tanto el archivo de entrada como el de salida se almacenan en el almacenamiento predeterminado del clúster.

1. Una vez completado el trabajo, use el siguiente comando para ver los resultados:

   ```bash
   hdfs dfs -cat /example/data/wordcountout/*
   ```

    Debería ver una lista de palabras y números con valores similares a los siguientes:

    ```output
    zeal    1
    zelus   1
    zenith  2
    ```

## <a name="next-steps"></a>Pasos siguientes

Gracias a este documento, ha aprendido a desarrollar un trabajo MapReduce de Java. Consulte los siguientes documentos para obtener información acerca de otras formas de trabajar con HDInsight.

* [Uso de Apache Hive con HDInsight](hdinsight-use-hive.md)
* [Uso de MapReduce con HDInsight](hdinsight-use-mapreduce.md)
* [Centro para desarrolladores de Java](https://azure.microsoft.com/develop/java/)
