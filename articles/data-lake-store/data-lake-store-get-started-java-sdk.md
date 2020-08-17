---
title: 'SDK de Java para operaciones del sistema de archivos en Data Lake Storage Gen1: Azure'
description: Use el SDK de Java de Azure Data Lake Storage Gen1 para realizar operaciones de sistema de archivos en Data Lake Storage Gen1, como crear carpetas, y cargar y descargar archivos de datos.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.custom: devx-track-java
ms.author: twooley
ms.openlocfilehash: a2c55a2d3277bbb6c3cf72f5ea703780d2a5e9bd
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87318851"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-java-sdk"></a>Operaciones de sistema de archivos en Azure Data Lake Storage Gen1 mediante el uso del SDK de Java
> [!div class="op_single_selector"]
> * [SDK de .NET](data-lake-store-data-operations-net-sdk.md)
> * [SDK de Java](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

Aprenda a utilizar el SDK de Java de Azure Data Lake Storage Gen1 para realizar operaciones básicas como crear carpetas, cargar y descargar archivos de datos, etc. Para más información sobre Data Lake Storage Gen1, consulte [Azure Data Lake Storage Gen1](data-lake-store-overview.md).

Puede acceder a los documentos de la API del SDK de Java para Data Lake Storage Gen1 en los [documentos de la API de Java de Azure Data Lake Storage Gen1](https://azure.github.io/azure-data-lake-store-java/javadoc/).

## <a name="prerequisites"></a>Requisitos previos
* Kit de desarrollo de Java (JDK 7 o superior, con Java versión 1.7 o posterior).
* Cuenta de Data Lake Storage Gen1. Siga las instrucciones de [Introducción a Azure Data Lake Storage Gen1 con Azure Portal](data-lake-store-get-started-portal.md).
* [Maven](https://maven.apache.org/install.html). Este tutorial usa Maven para crear las dependencias de un proyecto. Aunque es posible generarlas sin utilizar un sistema como Maven o Gradle, estos sistemas facilitan mucho la administración de las dependencias.
* (Opcional) Y un IDE como [IntelliJ IDEA](https://www.jetbrains.com/idea/download/), [Eclipse](https://www.eclipse.org/downloads/) o similar.

## <a name="create-a-java-application"></a>Creación de una aplicación Java
El ejemplo de código disponible [en GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/) le guía a través del proceso de creación de archivos en el almacén, concatenación de archivos, descarga de un archivo y eliminación de algunos archivos en el almacén. Esta sección del artículo le guía a través de las principales partes del código.

1. Cree un proyecto de Maven mediante [mvn archetype](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) en la línea de comandos o con un entorno de desarrollo integrado. Para ver instrucciones sobre cómo crear un proyecto de Java mediante IntelliJ, consulte [este artículo](https://www.jetbrains.com/help/idea/2016.1/creating-and-running-your-first-java-application.html). Para ver instrucciones sobre cómo crear un proyecto con Eclipse, consulte [este artículo](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm). 

2. Agregue las siguientes dependencias a su archivo **pom.xml** de Maven. Agregue el siguiente fragmento de código antes de la etiqueta **\</project>** :
   
    ```xml
    <dependencies>
        <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-data-lake-store-sdk</artifactId>
        <version>2.1.5</version>
        </dependency>
        <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-nop</artifactId>
        <version>1.7.21</version>
        </dependency>
    </dependencies>
    ```
   
    La primera dependencia es el uso del SDK de Data Lake Storage Gen1 (`azure-data-lake-store-sdk`) desde el repositorio de Maven. La segunda dependencia consiste en especificar qué plataforma de registro (`slf4j-nop`) se usará para esta aplicación. El SDK de Data Lake Storage Gen1 usa la fachada de registro [SLF4J](https://www.slf4j.org/), que permite elegir entre una serie de plataformas de registro populares, como Log4J, registro de Java, Logback, etc., o no registrarse. En este ejemplo, se deshabilita el registro; por tanto, se usa el enlace **slf4j-nop**. Para usar otras opciones de registro en su aplicación, consulte [este artículo](https://www.slf4j.org/manual.html#projectDep).

3. Agregue las siguientes instrucciones de importación a la aplicación.

    ```java
    import com.microsoft.azure.datalake.store.ADLException;
    import com.microsoft.azure.datalake.store.ADLStoreClient;
    import com.microsoft.azure.datalake.store.DirectoryEntry;
    import com.microsoft.azure.datalake.store.IfExists;
    import com.microsoft.azure.datalake.store.oauth2.AccessTokenProvider;
    import com.microsoft.azure.datalake.store.oauth2.ClientCredsTokenProvider;

    import java.io.*;
    import java.util.Arrays;
    import java.util.List;
    ```

## <a name="authentication"></a>Authentication

* Para la autenticación del usuario final para la aplicación, consulte [End-user authentication with Data Lake Store using Java](data-lake-store-end-user-authenticate-java-sdk.md) (Autenticación del usuario final con Data Lake Storage Gen1 mediante Java).
* Para la autenticación entre servicios para la aplicación, consulte [Service-to-service authentication with Data Lake Store using Java](data-lake-store-service-to-service-authenticate-java.md) (Autenticación entre servicios con Data Lake Storage Gen1 mediante Java).

## <a name="create-a-data-lake-storage-gen1-client"></a>Creación de un cliente de Data Lake Storage Gen1
La creación de un objeto [ADLStoreClient](https://azure.github.io/azure-data-lake-store-java/javadoc/) requiere la especificación del nombre de la cuenta de Data Lake Storage Gen1 y del proveedor de tokens que se generó con la autenticación en Data Lake Storage Gen1 (consulte la sección [Autenticación](#authentication)). El nombre de la cuenta de Data Lake Storage Gen1 debe ser un nombre de dominio completo. Por ejemplo, reemplace **FILL-IN-HERE** por algo similar a **mydatalakestoragegen1.azuredatalakestore.net**.

```java
private static String accountFQDN = "FILL-IN-HERE";  // full account FQDN, not just the account name
ADLStoreClient client = ADLStoreClient.createClient(accountFQDN, provider);
```

Los fragmentos de código de las secciones siguientes contienen ejemplos de operaciones de sistema de archivos comunes. Puede buscar en todos los [documentos de la API de SDK de Java de Data Lake Storage Gen1](https://azure.github.io/azure-data-lake-store-java/javadoc/) del objeto **ADLStoreClient** para ver otras operaciones.

## <a name="create-a-directory"></a>Creación de un directorio

El fragmento de código siguiente crea una estructura de directorios en la raíz de la cuenta de Data Lake Storage Gen1 especificada.

```java
// create directory
client.createDirectory("/a/b/w");
System.out.println("Directory created.");
```

## <a name="create-a-file"></a>Creación de un archivo

El fragmento de código siguiente crea un archivo (c.txt) en la estructura de directorios y escribe datos en él.

```java
// create file and write some content
String filename = "/a/b/c.txt";
OutputStream stream = client.createFile(filename, IfExists.OVERWRITE  );
PrintStream out = new PrintStream(stream);
for (int i = 1; i <= 10; i++) {
    out.println("This is line #" + i);
    out.format("This is the same line (%d), but using formatted output. %n", i);
}
out.close();
System.out.println("File created.");
```

También puede crear un archivo (d.txt) con matrices de bytes.

```java
// create file using byte arrays
stream = client.createFile("/a/b/d.txt", IfExists.OVERWRITE);
byte[] buf = getSampleContent();
stream.write(buf);
stream.close();
System.out.println("File created using byte array.");
```

La definición de la función `getSampleContent` utilizada en el fragmento de código anterior está disponible como parte del ejemplo [en GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/). 

## <a name="append-to-a-file"></a>Anexión a un archivo

El fragmento de código siguiente anexa contenido a un archivo existente.

```java
// append to file
stream = client.getAppendStream(filename);
stream.write(getSampleContent());
stream.close();
System.out.println("File appended.");
```

La definición de la función `getSampleContent` utilizada en el fragmento de código anterior está disponible como parte del ejemplo [en GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="read-a-file"></a>Lectura de un archivo

El fragmento de código siguiente lee el contenido de un archivo de cuenta de Data Lake Storage Gen1.

```java
// Read File
InputStream in = client.getReadStream(filename);
BufferedReader reader = new BufferedReader(new InputStreamReader(in));
String line;
while ( (line = reader.readLine()) != null) {
    System.out.println(line);
}
reader.close();
System.out.println();
System.out.println("File contents read.");
```

## <a name="concatenate-files"></a>Concatenación de archivos

El fragmento de código siguiente concatena dos archivos en una cuenta de Data Lake Storage Gen1. Si se realiza correctamente, el archivo concatenado reemplaza los dos existentes.

```java
// concatenate the two files into one
List<String> fileList = Arrays.asList("/a/b/c.txt", "/a/b/d.txt");
client.concatenateFiles("/a/b/f.txt", fileList);
System.out.println("Two files concatenated into a new file.");
```

## <a name="rename-a-file"></a>Cambio del nombre de un archivo

El fragmento de código cambia el nombre de un archivo en una cuenta de Data Lake Storage Gen1.

```java
//rename the file
client.rename("/a/b/f.txt", "/a/b/g.txt");
System.out.println("New file renamed.");
```

## <a name="get-metadata-for-a-file"></a>Obtención de los metadatos para un archivo

El fragmento de código siguiente recupera los metadatos de un archivo en una cuenta de Data Lake Storage Gen1.

```java
// get file metadata
DirectoryEntry ent = client.getDirectoryEntry(filename);
printDirectoryInfo(ent);
System.out.println("File metadata retrieved.");
```

## <a name="set-permissions-on-a-file"></a>Obtención de permisos en un archivo

El fragmento de código siguiente obtiene permisos en el archivo que se creó en la sección anterior.

```java
// set file permission
client.setPermission(filename, "744");
System.out.println("File permission set.");
```

## <a name="list-directory-contents"></a>Lista del contenido del directorio

El fragmento de código siguiente enumera el contenido de un directorio, repetidamente.

```java
// list directory contents
List<DirectoryEntry> list = client.enumerateDirectory("/a/b", 2000);
System.out.println("Directory listing for directory /a/b:");
for (DirectoryEntry entry : list) {
    printDirectoryInfo(entry);
}
System.out.println("Directory contents listed.");
```

La definición de la función `printDirectoryInfo` utilizada en el fragmento de código anterior está disponible como parte del ejemplo [en GitHub](https://azure.microsoft.com/documentation/samples/data-lake-store-java-upload-download-get-started/).

## <a name="delete-files-and-folders"></a>Eliminación de archivos y carpetas

El fragmento de código siguiente elimina los archivos y las carpetas especificados de una cuenta de Data Lake Storage Gen1, repetidamente.

```java
// delete directory along with all the subdirectories and files in it
client.deleteRecursive("/a");
System.out.println("All files and folders deleted recursively");
promptEnterKey();
```

## <a name="build-and-run-the-application"></a>Compilación y ejecución de la aplicación
1. Para ejecutarla desde un IDE, localícela y presione el botón **Ejecutar**. Para ejecutarla desde Maven, utilice [exec:exec](https://www.mojohaus.org/exec-maven-plugin/exec-mojo.html).
2. Para generar un archivo jar independiente que puede ejecutar desde la línea de comandos, genere el archivo jar con todas las dependencias incluidas mediante el [complemento de ensamblado de Maven](https://maven.apache.org/plugins/maven-assembly-plugin/usage.html). El archivo pom.xml del [código fuente de ejemplo de GitHub](https://github.com/Azure-Samples/data-lake-store-java-upload-download-get-started/blob/master/pom.xml) contiene un ejemplo.

## <a name="next-steps"></a>Pasos siguientes
* [Explorar JavaDoc para el SDK de Java](https://azure.github.io/azure-data-lake-store-java/javadoc/)
* [Protección de datos en Data Lake Storage Gen1](data-lake-store-secure-data.md)


