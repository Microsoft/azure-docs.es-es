---
title: Uso de una función definida por el usuario (UDF) de Java con Apache Hive Azure HDInsight
description: Aprenda cómo crear una función basada en Java y definida por el usuario (UDF) que funcione con Apache Hive. Esta función de ejemplo definida por el usuario convierte una tabla de cadenas de texto en minúsculas.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-java
ms.date: 11/20/2019
ms.openlocfilehash: 15a86fd191eefc5882150fb276228d18efd6ff5d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323798"
---
# <a name="use-a-java-udf-with-apache-hive-in-hdinsight"></a>Utilización de una función definida por el usuario de Java con Apache Hive en HDInsight

Aprenda cómo crear una función basada en Java y definida por el usuario (UDF) que funcione con Apache Hive. La función de Java definida por el usuario de este ejemplo convierte una tabla de cadenas de texto a caracteres en minúscula.

## <a name="prerequisites"></a>Prerequisites

* Un clúster de Hadoop en HDInsight. Consulte [Introducción a HDInsight en Linux](./apache-hadoop-linux-tutorial-get-started.md).
* [Kit de desarrolladores de Java (JDK), versión 8](https://aka.ms/azure-jdks)
* [Apache Maven](https://maven.apache.org/download.cgi) correctamente [instalado](https://maven.apache.org/install.html) según Apache.  Maven es un sistema de compilación de proyectos de Java.
* El [esquema de URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme) para el almacenamiento principal de clústeres. Sería wasb:// para Azure Storage, abfs:// para Azure Data Lake Storage Gen2 o adl:// para Azure Data Lake Storage Gen1. Si se habilita la transferencia segura para Azure Storage, el identificador URI sería `wasbs://`.  Consulte también el artículo acerca de la [transferencia segura](../../storage/common/storage-require-secure-transfer.md).

* Un editor de texto o IDE de Java

    > [!IMPORTANT]  
    > Si crea los archivos de Python en un cliente Windows, debe usar un editor que emplee LF como final de línea. Si no está seguro de si el editor usa LF o CRLF, vea la sección [Solución de problemas](#troubleshooting) para conocer los pasos a seguir para quitar el carácter CR.

## <a name="test-environment"></a>Entorno de prueba

El entorno usado en este artículo fue un equipo donde se ejecuta Windows 10.  Los comandos se ejecutaron en un símbolo del sistema, y los distintos archivos se editaron con el Bloc de notas. Realice las modificaciones según corresponda en su entorno.

Desde un símbolo del sistema, escriba los siguientes comandos para crear un entorno de trabajo:

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-an-example-java-udf"></a>Crear una función de Java definida por el usuario de ejemplo

1. Utilice el comando siguiente para crear un nuevo proyecto de Maven:

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=ExampleUDF -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

    Este comando crea un directorio denominado `exampleudf`, que contiene el proyecto de Maven.

2. Una vez creado el proyecto, elimine el directorio `exampleudf/src/test` que se creó como parte del proyecto. Para ello, escriba el siguiente comando:

    ```cmd
    cd ExampleUDF
    rmdir /S /Q "src/test"
    ```

3. Especifique el siguiente comando para abrir `pom.xml`:

    ```cmd
    notepad pom.xml
    ```

    Reemplace la entrada `<dependencies>` existente por el siguiente código XML:

    ```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.3</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>1.2.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    ```

    Estas entradas especifican la versión de Hadoop y Hive incluida con el clúster de HDInsight 3.6. Puede encontrar información sobre las versiones de Hadoop y Hive proporcionadas con HDInsight en el artículo [¿Cuáles son los diferentes componentes de Hadoop disponibles con HDInsight?](../hdinsight-component-versioning.md) .

    Agregue una sección `<build>` antes de la línea `</project>` al final del archivo. Esta sección debe contener el siguiente XML:

    ```xml
    <build>
        <plugins>
            <!-- build for Java 1.8. This is required by HDInsight 3.6  -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <!-- Keep us from getting a can't overwrite file error -->
                    <transformers>
                        <transformer
                                implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                        </transformer>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer">
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
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
        </plugins>
    </build>
    ```

    Estas entradas definen cómo compilar el proyecto. Específicamente, la versión de Java que el proyecto usa y cómo compilar un archivo uberjar para implementarlo en el clúster.

    Guarde el archivo una vez hechos los cambios.

4. Escriba el comando siguiente para crear un archivo `ExampleUDF.java` y abrirlo:

    ```cmd
    notepad src/main/java/com/microsoft/examples/ExampleUDF.java
    ```

    Luego, copie y pegue el código Java siguiente en el nuevo archivo. y ciérrelo.

    ```java
    package com.microsoft.examples;

    import org.apache.hadoop.hive.ql.exec.Description;
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.*;

    // Description of the UDF
    @Description(
        name="ExampleUDF",
        value="returns a lower case version of the input string.",
        extended="select ExampleUDF(deviceplatform) from hivesampletable limit 10;"
    )
    public class ExampleUDF extends UDF {
        // Accept a string input
        public String evaluate(String input) {
            // If the value is null, return a null
            if(input == null)
                return null;
            // Lowercase the input string and return it
            return input.toLowerCase();
        }
    }
    ```

    Este código implementa una función definida por el usuario que acepta un valor de cadena y devuelve una versión en minúsculas de esta.

## <a name="build-and-install-the-udf"></a>Compilación e instalación de la función definida por el usuario

En los siguientes comandos, reemplace `sshuser` por el nombre de usuario real, si es distinto. Reemplace `mycluster` por el nombre de clúster real.

1. Escriba el siguiente comando para compilar y empaquetar la función definida por el usuario:

    ```cmd
    mvn compile package
    ```

    Este comando compila y empaqueta la función definida por el usuario en el archivo `exampleudf/target/ExampleUDF-1.0-SNAPSHOT.jar`.

2. Use el comando `scp` para copiar el archivo en el clúster de HDInsight. Para ello, escriba el siguiente comando:

    ```cmd
    scp ./target/ExampleUDF-1.0-SNAPSHOT.jar sshuser@mycluster-ssh.azurehdinsight.net:
    ```

3. Conéctese al clúster mediante SSH con el comando siguiente:

    ```cmd
    ssh sshuser@mycluster-ssh.azurehdinsight.net
    ```

4. Desde la sesión de SSH abierta, copie el archivo .jar en el almacenamiento de HDInsight.

    ```bash
    hdfs dfs -put ExampleUDF-1.0-SNAPSHOT.jar /example/jars
    ```

## <a name="use-the-udf-from-hive"></a>Uso de la función definida por el usuario desde Hive

1. Escriba el siguiente comando desde la sesión de SSH para iniciar el cliente de Beeline:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

    Este comando supone que usó el valor predeterminado de **admin** para la cuenta de inicio de sesión del clúster.

2. Una vez alcanzado el aviso `jdbc:hive2://localhost:10001/>` , escriba lo siguiente para agregar la función definida por el usuario a Hive y exponerla como una función.

    ```hiveql
    ADD JAR wasbs:///example/jars/ExampleUDF-1.0-SNAPSHOT.jar;
    CREATE TEMPORARY FUNCTION tolower as 'com.microsoft.examples.ExampleUDF';
    ```

3. Use la función definida por el usuario para convertir los valores recuperados de una tabla a cadenas de minúsculas.

    ```hiveql
    SELECT tolower(state) AS ExampleUDF, state FROM hivesampletable LIMIT 10;
    ```

    Esta consulta selecciona el estado de la tabla, convierte la cadena a minúsculas y, a continuación, las muestra junto con el nombre sin modificar. La salida es similar al siguiente texto:

    ```output
    +---------------+---------------+--+
    |  exampleudf   |     state     |
    +---------------+---------------+--+
    | california    | California    |
    | pennsylvania  | Pennsylvania  |
    | pennsylvania  | Pennsylvania  |
    | pennsylvania  | Pennsylvania  |
    | colorado      | Colorado      |
    | colorado      | Colorado      |
    | colorado      | Colorado      |
    | utah          | Utah          |
    | utah          | Utah          |
    | colorado      | Colorado      |
    +---------------+---------------+--+
    ```

## <a name="troubleshooting"></a>Solución de problemas

Al ejecutar el trabajo de Hive, es posible que se produzca un error similar al texto siguiente:

```output
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.
```

Este problema puede deberse a los finales de línea del archivo de Python. De forma predeterminada, muchos editores de Windows usan CRLF como final de línea, pero las aplicaciones Linux normalmente esperan caracteres LF.

Puede usar las siguientes instrucciones de PowerShell para quitar los caracteres CR antes de cargar el archivo en HDInsight:

```PowerShell
# Set $original_file to the python file path
$text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
[IO.File]::WriteAllText($original_file, $text)
```

## <a name="next-steps"></a>Pasos siguientes

Para conocer otras formas de trabajar con Hive, consulte [Uso de Apache Hive con HDInsight](hdinsight-use-hive.md).

Para más información sobre las funciones definidas por el usuario de Hive, consulte la sección [Apache Hive Operators and User-Defined Functions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF) (Operadores de Apache Hive y funciones definidas por el usuario) de la wiki de Hive en apache.org.
