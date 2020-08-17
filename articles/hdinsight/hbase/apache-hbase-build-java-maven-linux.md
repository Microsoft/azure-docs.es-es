---
title: Uso de Apache Maven para crear un cliente de HBase de Java para Azure HDInsight
description: Aprenda a usar Apache Maven para compilar una aplicación de Apache HBase basada en Java e implementarla después en HBase en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seodec18, devx-track-java
ms.date: 12/24/2019
ms.openlocfilehash: 7cd368df1f2a94c8f49454530e7f5997f2659a32
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323781"
---
# <a name="build-java-applications-for-apache-hbase"></a>Compilar aplicaciones Java para Apache HBase

Aprenda a crear una aplicación [HBase de Apache](https://hbase.apache.org/) en Java. Luego, use esa aplicación con HBase en Azure HDInsight.

En los pasos descritos en este documento se usa [Apache Maven](https://maven.apache.org/) para crear y compilar el proyecto. Maven es una herramienta de administración y comprensión de proyectos de software que le permite compilar software, documentación e informes para proyectos Java.

## <a name="prerequisites"></a>Prerrequisitos

* Un clúster de Apache HBase en HDInsight. Vea [Introducción a un ejemplo de Apache HBase en HDInsight](./apache-hbase-tutorial-get-started-linux.md).

* [Kit de desarrolladores de Java (JDK), versión 8](https://aka.ms/azure-jdks).

* [Apache Maven](https://maven.apache.org/download.cgi) correctamente [instalado](https://maven.apache.org/install.html) según Apache.  Maven es un sistema de compilación de proyectos de Java.

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Si usa PowerShell, necesitará el [Módulo AZ](https://docs.microsoft.com/powershell/azure/).

* Un editor de texto. En este artículo se usa el Bloc de notas de Microsoft.

## <a name="test-environment"></a>Entorno de prueba

El entorno usado en este artículo fue un equipo donde se ejecuta Windows 10.  Los comandos se ejecutaron en un símbolo del sistema, y los distintos archivos se editaron con el Bloc de notas. Realice las modificaciones según corresponda en su entorno.

Desde un símbolo del sistema, escriba los siguientes comandos para crear un entorno de trabajo:

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Creación de un proyecto de Maven

1. Especifique el siguiente comando para crear un proyecto de Maven llamado **hbaseapp**:

    ```cmd
    mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    cd hbaseapp
    mkdir conf
    ```

    Este comando crea un directorio denominado `hbaseapp` en la ubicación actual, que contiene un proyecto de Maven básico. El segundo comando cambia el directorio de trabajo a `hbaseapp`. El tercer comando crea un directorio, `conf`, que se usará más adelante. El directorio `hbaseapp` contiene los siguientes elementos:

    * `pom.xml`:  el modelo de objetos de proyectos ([POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) contiene la información y los detalles de configuración usados para compilar el proyecto.
    * `src\main\java\com\microsoft\examples`: Contiene el código de la aplicación.
    * `src\test\java\com\microsoft\examples`: Contiene pruebas para la aplicación.

2. Quite el código de ejemplo generado. Especifique los siguientes comandos para eliminar los archivos de aplicación y de prueba generados (`AppTest.java` y `App.java`):

    ```cmd
    DEL src\main\java\com\microsoft\examples\App.java
    DEL src\test\java\com\microsoft\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Actualización del modelo de objetos de proyectos

Para obtener una referencia completa del archivo pom.xml, vea https://maven.apache.org/pom.html.  Especifique el siguiente comando para abrir `pom.xml`:

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Adición de dependencias

En la sección `pom.xml`, agregue el siguiente texto a la sección `<dependencies>`:

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-shaded-client</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.14.1-HBase-1.1</version>
</dependency>
```  

En esta sección se indica que el proyecto necesita los componentes **hbase-client** y **phoenix-core**. En tiempo de compilación, estas dependencias se descargan del repositorio de Maven predeterminado. Puede usar la [búsqueda del repositorio central de Maven](https://search.maven.org/artifact/org.apache.hbase/hbase-client/1.1.2/jar) para ver más información sobre esta dependencia.

> [!IMPORTANT]  
> El número de versión de hbase-client debe coincidir con la versión de Apache HBase que se proporciona con el clúster de HDInsight. Utilice la siguiente tabla para buscar el número de versión correcto.

| Versión del clúster de HDInsight | Versión de Apache HBase que se va a utilizar |
| --- | --- |
| 3.6 | 1.1.2 |
| 4.0 | 2.0.0 |

Para más información sobre las versiones y componentes de HDInsight, consulte [¿Cuáles son los diferentes componentes de Hadoop disponibles con HDInsight?](../hdinsight-component-versioning.md)

### <a name="build-configuration"></a>Configuración de compilación

Los complementos de Maven permiten personalizar las fases de compilación del proyecto, Esta sección se usa para agregar complementos, recursos y otras opciones de configuración de compilación.

Agregue el código siguiente al archivo `pom.xml` y, después, guárdelo y ciérrelo. Este texto debe estar dentro de las etiquetas `<project>...</project>` en el archivo; por ejemplo, entre `</dependencies>` y `</project>`.

```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <resources>
    <resource>
        <directory>${basedir}/conf</directory>
        <filtering>false</filtering>
        <includes>
        <include>hbase-site.xml</include>
        </includes>
    </resource>
    </resources>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>3.2.1</version>
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
    </plugins>
</build>
```

Esta sección configura un recurso (`conf/hbase-site.xml`) que contiene información de configuración de HBase.

> [!NOTE]  
> También puede establecer los valores de configuración mediante código. Vea los comentarios en el ejemplo `CreateTable`.

Esta sección también configura los complementos [Apache Maven Compiler](https://maven.apache.org/plugins/maven-compiler-plugin/) y [Apache Maven Shade](https://maven.apache.org/plugins/maven-shade-plugin/). El complemento compiler se usa para compilar la topología. El complemento shade se usa para evitar la duplicación de licencias en el paquete JAR compilado por Maven. Este complemento se usa para evitar errores por "archivos de licencia duplicados" en tiempo de ejecución en el clúster de HDInsight. El uso del complemento maven-shade-plugin con la implementación de `ApacheLicenseResourceTransformer` evita este error.

El complemento maven-shade-plugin también producirá un uberjar, que contiene todas las dependencias que necesita la aplicación.

### <a name="download-the-hbase-sitexml"></a>Descarga del archivo hbase-site.xml

Use el siguiente comando para copiar la configuración de HBase desde el clúster HBase en el directorio `conf`. Reemplace `CLUSTERNAME` por su nombre de clúster de HDInsight y, después, escriba el comando:

```cmd
scp sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml
```

## <a name="create-the-application"></a>Creación de la aplicación

### <a name="implement-a-createtable-class"></a>Implementar una clase CreateTable

Escriba el comando siguiente para crear un archivo `CreateTable.java` y abrirlo. Seleccione **Sí** en el símbolo del sistema para crear un archivo.

```cmd
notepad src\main\java\com\microsoft\examples\CreateTable.java
```

Luego, copie y pegue el código Java siguiente en el nuevo archivo. y ciérrelo.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.util.Bytes;

public class CreateTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Example of setting zookeeper values for HDInsight
    // in code instead of an hbase-site.xml file
    //
    // config.set("hbase.zookeeper.quorum",
    //            "zookeepernode0,zookeepernode1,zookeepernode2");
    //config.set("hbase.zookeeper.property.clientPort", "2181");
    //config.set("hbase.cluster.distributed", "true");
    //
    //NOTE: Actual zookeeper host names can be found using Ambari:
    //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts"

    //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
    config.set("zookeeper.znode.parent","/hbase-unsecure");

    // create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // create the table...
    HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
    // ... with two column families
    tableDescriptor.addFamily(new HColumnDescriptor("name"));
    tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
    admin.createTable(tableDescriptor);

    // define some people
    String[][] people = {
        { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
        { "2", "Franklin", "Holtz", "franklin@contoso.com" },
        { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
        { "4", "Rae", "Schroeder", "rae@contoso.com" },
        { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
        { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

    HTable table = new HTable(config, "people");

    // Add each person to the table
    //   Use the `name` column family for the name
    //   Use the `contactinfo` column family for the email
    for (int i = 0; i< people.length; i++) {
        Put person = new Put(Bytes.toBytes(people[i][0]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
        person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
        person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
        table.put(person);
    }
    // flush commits and close the table
    table.flushCommits();
    table.close();
    }
}
```

Este código es la clase `CreateTable`, que creará una tabla llamada `people` y la rellenará con algunos usuarios predefinidos.

### <a name="implement-a-searchbyemail-class"></a>Implementar una clase SearchByEmail

Escriba el comando siguiente para crear un archivo `SearchByEmail.java` y abrirlo. Seleccione **Sí** en el símbolo del sistema para crear un archivo.

```cmd
notepad src\main\java\com\microsoft\examples\SearchByEmail.java
```

Luego, copie y pegue el código Java siguiente en el nuevo archivo. y ciérrelo.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.filter.RegexStringComparator;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.util.GenericOptionsParser;

public class SearchByEmail {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Use GenericOptionsParser to get only the parameters to the class
    // and not all the parameters passed (when using WebHCat for example)
    String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
    if (otherArgs.length != 1) {
        System.out.println("usage: [regular expression]");
        System.exit(-1);
    }

    // Open the table
    HTable table = new HTable(config, "people");

    // Define the family and qualifiers to be used
    byte[] contactFamily = Bytes.toBytes("contactinfo");
    byte[] emailQualifier = Bytes.toBytes("email");
    byte[] nameFamily = Bytes.toBytes("name");
    byte[] firstNameQualifier = Bytes.toBytes("first");
    byte[] lastNameQualifier = Bytes.toBytes("last");

    // Create a regex filter
    RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
    // Attach the regex filter to a filter
    //   for the email column
    SingleColumnValueFilter filter = new SingleColumnValueFilter(
        contactFamily,
        emailQualifier,
        CompareOp.EQUAL,
        emailFilter
    );

    // Create a scan and set the filter
    Scan scan = new Scan();
    scan.setFilter(filter);

    // Get the results
    ResultScanner results = table.getScanner(scan);
    // Iterate over results and print  values
    for (Result result : results ) {
        String id = new String(result.getRow());
        byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
        String firstName = new String(firstNameObj);
        byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
        String lastName = new String(lastNameObj);
        System.out.println(firstName + " " + lastName + " - ID: " + id);
        byte[] emailObj = result.getValue(contactFamily, emailQualifier);
        String email = new String(emailObj);
        System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
    }
    results.close();
    table.close();
    }
}
```

La clase `SearchByEmail` se puede usar para consultar filas por dirección de correo electrónico. Dado que esta clase usa un filtro de expresiones regulares, puede proporcionar una cadena o una expresión regular cuando la utilice.

### <a name="implement-a-deletetable-class"></a>Implementar una clase DeleteTable

Escriba el comando siguiente para crear un archivo `DeleteTable.java` y abrirlo. Seleccione **Sí** en el símbolo del sistema para crear un archivo.

```cmd
notepad src\main\java\com\microsoft\examples\DeleteTable.java
```

Luego, copie y pegue el código Java siguiente en el nuevo archivo. y ciérrelo.

```java
package com.microsoft.examples;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;

public class DeleteTable {
    public static void main(String[] args) throws IOException {
    Configuration config = HBaseConfiguration.create();

    // Create an admin object using the config
    HBaseAdmin admin = new HBaseAdmin(config);

    // Disable, and then delete the table
    admin.disableTable("people");
    admin.deleteTable("people");
    }
}
```

La clase `DeleteTable` limpia las tablas de HBase creadas en este ejemplo. Para ello, primero se deshabilita y luego se elimina la tabla creada por la clase `CreateTable`.

## <a name="build-and-package-the-application"></a>Compilación y empaquetado de la aplicación

1. En el directorio `hbaseapp`, use el siguiente comando para compilar un archivo JAR que contenga la aplicación:

    ```cmd
    mvn clean package
    ```

    Este comando compila y empaqueta la aplicación en un archivo .jar.

2. Cuando el comando finalice, el directorio `hbaseapp/target` contendrá un archivo denominado `hbaseapp-1.0-SNAPSHOT.jar`.

   > [!NOTE]  
   > El archivo `hbaseapp-1.0-SNAPSHOT.jar` es un archivo uber-jar. Contiene todas las dependencias necesarias para ejecutar la aplicación.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Carga del archivo JAR y ejecución de trabajos (SSH)

En los siguientes pasos se usa `scp` para copiar el archivo JAR en el nodo primario principal de Apache HBase en el clúster de HDInsight. El comando `ssh`, se usa para conectarse al clúster y ejecutar el ejemplo directamente en el nodo principal.

1. Cargue el archivo JAR en el clúster. Reemplace `CLUSTERNAME` por su nombre de clúster de HDInsight y, después, escriba el siguiente comando:

    ```cmd
    scp ./target/hbaseapp-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:hbaseapp-1.0-SNAPSHOT.jar
    ```

2. Conéctese al clúster de HBase. Reemplace `CLUSTERNAME` por su nombre de clúster de HDInsight y, después, escriba el siguiente comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Use el siguiente comando en la conexión SSH abierta para crear una tabla HBase por medio de la aplicación Java:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable
    ```

    Este comando crea una tabla HBase denominada **people**, que se rellenará con datos.

4. Use el siguiente comando para buscar direcciones de correo electrónico almacenadas en la tabla:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com
    ```

    Recibirá los siguientes resultados:

    ```console
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

5. Para eliminar la tabla, use el comando siguiente:

    ```bash
    yarn jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable
    ```

## <a name="upload-the-jar-and-run-jobs-powershell"></a>Carga del archivo JAR y ejecución de trabajos (PowerShell)

En los siguientes pasos se usa el [módulo AZ](https://docs.microsoft.com/powershell/azure/new-azureps-module-az) de Azure PowerShell para cargar el archivo JAR en el almacenamiento predeterminado del clúster de Apache HBase. Los cmdlets de HDInsight se usan para ejecutar los ejemplos de forma remota.

1. Tras instalar y configurar el módulo AZ, cree un archivo denominado `hbase-runner.psm1`. Use el texto siguiente como contenido de este archivo:

   ```powershell
    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.CreateTable"
    -clusterName "MyHDInsightCluster"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "contoso.com"

    .EXAMPLE
    Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
    -clusterName "MyHDInsightCluster"
    -emailRegex "^r" -showErr
    #>

    function Start-HBaseExample {
    [CmdletBinding(SupportsShouldProcess = $true)]
    param(
    #The class to run
    [Parameter(Mandatory = $true)]
    [String]$className,

    #The name of the HDInsight cluster
    [Parameter(Mandatory = $true)]
    [String]$clusterName,

    #Only used when using SearchByEmail
    [Parameter(Mandatory = $false)]
    [String]$emailRegex,

    #Use if you want to see stderr output
    [Parameter(Mandatory = $false)]
    [Switch]$showErr
    )

    Set-StrictMode -Version 3

    # Is the Azure module installed?
    FindAzure

    # Get the login for the HDInsight cluster
    $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

    # The JAR
    $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

    # The job definition
    $jobDefinition = New-AzHDInsightMapReduceJobDefinition `
        -JarFile $jarFile `
        -ClassName $className `
        -Arguments $emailRegex

    # Get the job output
    $job = Start-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
    if($showErr)
    {
    Write-Host "STDERR"
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds `
                -DisplayOutputType StandardError
    }
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzHDInsightJobOutput `
                -Clustername $clusterName `
                -JobId $job.JobId `
                -HttpCredential $creds
    }

    <#
    .SYNOPSIS
    Copies a file to the primary storage of an HDInsight cluster.
    .DESCRIPTION
    Copies a file from a local directory to the blob container for
    the HDInsight cluster.
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    .EXAMPLE
    Add-HDInsightFile -localPath "C:\temp\data.txt"
    -destinationPath "example/data/data.txt"
    -ClusterName "MyHDInsightCluster"
    -Container "MyContainer"
    #>

    function Add-HDInsightFile {
        [CmdletBinding(SupportsShouldProcess = $true)]
        param(
            #The path to the local file.
            [Parameter(Mandatory = $true)]
            [String]$localPath,

            #The destination path and file name, relative to the root of the container.
            [Parameter(Mandatory = $true)]
            [String]$destinationPath,

            #The name of the HDInsight cluster
            [Parameter(Mandatory = $true)]
            [String]$clusterName,

            #If specified, overwrites existing files without prompting
            [Parameter(Mandatory = $false)]
            [Switch]$force
        )

        Set-StrictMode -Version 3

        # Is the Azure module installed?
        FindAzure

        # Get authentication for the cluster
        $creds=Get-Credential

        # Does the local path exist?
        if (-not (Test-Path $localPath))
        {
            throw "Source path '$localPath' does not exist."
        }

        # Get the primary storage container
        $storage = GetStorage -clusterName $clusterName

        # Upload file to storage, overwriting existing files if -force was used.
        Set-AzStorageBlobContent -File $localPath `
            -Blob $destinationPath `
            -force:$force `
            -Container $storage.container `
            -Context $storage.context
    }

    function FindAzure {
        # Is there an active Azure subscription?
        $sub = Get-AzSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Connect-AzAccount
        }
    }

    function GetStorage {
        param(
            [Parameter(Mandatory = $true)]
            [String]$clusterName
        )
        $hdi = Get-AzHDInsightCluster -ClusterName $clusterName
        # Does the cluster exist?
        if (!$hdi)
        {
            throw "HDInsight cluster '$clusterName' does not exist."
        }
        # Create a return object for context & container
        $return = @{}
        $storageAccounts = @{}

        # Get storage information
        $resourceGroup = $hdi.ResourceGroup
        $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
        $container=$hdi.DefaultStorageContainer
        $storageAccountKey=(Get-AzStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
        # Get the resource group, in case we need that
        $return.resourceGroup = $resourceGroup
        # Get the storage context, as we can't depend
        # on using the default storage context
        $return.context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
        # Get the container, so we know where to
        # find/store blobs
        $return.container = $container
        # Return storage accounts to support finding all accounts for
        # a cluster
        $return.storageAccount = $storageAccountName
        $return.storageAccountKey = $storageAccountKey

        return $return
    }
    # Only export the verb-phrase things
    export-modulemember *-*
   ```

    Este archivo contiene dos módulos:

   * **Add-HDInsightFile**: se usa para cargar archivos en el clúster.
   * **Start-HBaseExample**: se usa para ejecutar las clases creadas anteriormente.

2. Guarde el archivo `hbase-runner.psm1` en el directorio `hbaseapp`.

3. Registre los módulos con Azure PowerShell. Abra una nueva ventana de PowerShell de Azure y modifique el siguiente comando, reemplazando `CLUSTERNAME` por el nombre del clúster. Después, escriba los comandos siguientes:

    ```powershell
    cd C:\HDI\hbaseapp
    $myCluster = "CLUSTERNAME"
    Import-Module .\hbase-runner.psm1
    ```

4. Use el siguiente comando para cargar `hbaseapp-1.0-SNAPSHOT.jar` en el clúster.

    ```powershell
    Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName $myCluster
    ```

    Cuando se le solicite, escriba el nombre de inicio de sesión del clúster (administrador) y su contraseña. El comando carga el archivo `hbaseapp-1.0-SNAPSHOT.jar` en la ubicación `example/jars` en el almacenamiento principal del clúster.

5. Use el siguiente comando para crear una tabla mediante `hbaseapp`:

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName $myCluster
    ```

    Cuando se le solicite, escriba el nombre de inicio de sesión del clúster (administrador) y su contraseña.

    Con este comando se crea una tabla denominada **people** en HBase en el clúster de HDInsight. Este comando no muestra ninguna salida en la ventana de la consola.

6. Para buscar entradas en la tabla, use el siguiente comando:

    ```powershell
    Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName $myCluster -emailRegex contoso.com
    ```

    Cuando se le solicite, escriba el nombre de inicio de sesión del clúster (administrador) y su contraseña.

    Este comando usa la clase `SearchByEmail` para buscar filas donde la familia de columnas `contactinformation` y la columna `email` contengan la cadena `contoso.com`. Debe recibir los siguientes resultados:

    ```output
    Franklin Holtz - ID: 2
    Franklin Holtz - franklin@contoso.com - ID: 2
    Rae Schroeder - ID: 4
    Rae Schroeder - rae@contoso.com - ID: 4
    Gabriela Ingram - ID: 6
    Gabriela Ingram - gabriela@contoso.com - ID: 6
    ```

    Si se usa **fabrikam.com** como valor de `-emailRegex` se devolverán los usuarios que tienen **fabrikam.com** en el campo de correo electrónico. También puede utilizar expresiones regulares en el término de búsqueda. Por ejemplo, **^ r** devuelve las direcciones de correo que comienzan por la letra "r".

7. Para eliminar la tabla, use el comando siguiente:

    ```PowerShell
    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName $myCluster
    ```

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>Sin resultados o resultados inesperados al usar Start-HBaseExample

Use el parámetro `-showErr` para ver el error estándar (STDERR) producido mientras se ejecutaba el trabajo.

## <a name="next-steps"></a>Pasos siguientes

[Aprenda a usar SQLLine con Apache HBase](apache-hbase-query-with-phoenix.md)
