---
title: Filtro de datos mediante la aceleración de consultas de Azure Data Lake Storage (versión preliminar) | Microsoft Docs
description: Use la aceleración de consultas (versión preliminar) para recuperar un subconjunto de datos de la cuenta de almacenamiento.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 04/21/2020
ms.author: normesta
ms.reviewer: jamsbak
ms.custom: devx-track-csharp
ms.openlocfilehash: 6de6661e5c970c7c3cbfc944b8539060b8844a36
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89005231"
---
# <a name="filter-data-by-using-azure-data-lake-storage-query-acceleration-preview"></a>Filtro de datos mediante la aceleración de consultas de Azure Data Lake Storage (versión preliminar)

En este artículo se muestra cómo usar la aceleración de consultas (versión preliminar) para recuperar un subconjunto de datos de la cuenta de almacenamiento. 

La aceleración de consultas (versión preliminar) es una nueva capacidad de Azure Data Lake Storage que permite a las aplicaciones y los marcos de análisis optimizar considerablemente el procesamiento de datos mediante la recuperación únicamente de los datos necesarios para una operación determinada. Para obtener más información, consulte [Aceleración de consultas de Azure Data Lake Storage (versión preliminar)](data-lake-storage-query-acceleration.md).

> [!NOTE]
> La característica de aceleración de consultas está en versión preliminar pública y está disponible en las regiones Centro de Canadá y Centro de Francia. Para conocer las limitaciones, consulte el artículo [Problemas conocidos](data-lake-storage-known-issues.md). Para inscribirse en la versión preliminar, visite [este formulario](https://aka.ms/adls/qa-preview-signup).  

## <a name="prerequisites"></a>Requisitos previos

### <a name="net"></a>[.NET](#tab/dotnet)

- Para acceder a Azure Storage, necesitará una suscripción de Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Una cuenta de almacenamiento de **uso general v2**. Vea [Creación de una cuenta de Azure Storage](../common/storage-quickstart-create-account.md).

- [.NET SDK](https://dotnet.microsoft.com/download). 

### <a name="java"></a>[Java](#tab/java)

- Para acceder a Azure Storage, necesitará una suscripción de Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Una cuenta de almacenamiento de **uso general v2**. Vea [Creación de una cuenta de Azure Storage](../common/storage-quickstart-create-account.md).

- [Kit de desarrollo de Java (JDK)](/java/azure/jdk/?view=azure-java-stable), versión 8 o posterior.

- [Apache Maven](https://maven.apache.org/download.cgi). 

  > [!NOTE] 
  > En este artículo se considera que ha creado un proyecto de Java mediante Apache Maven. Para obtener un ejemplo de cómo crear un proyecto con Apache Maven, consulte [Instalación](storage-quickstart-blobs-java.md#setting-up).
  
---

## <a name="install-packages"></a>Instalar paquetes 

### <a name="net"></a>[.NET](#tab/dotnet)

1. Descargue los paquetes de aceleración de consultas. Puede obtener un archivo ZIP comprimido que contenga estos paquetes mediante este vínculo: [https://aka.ms/adls/qqsdk/.net](https://aka.ms/adls/qqsdk/.net). 

2. Extraiga el contenido de este archivo en el directorio del proyecto.

3. Abra el archivo del proyecto ( *.csproj*) en un editor de texto y agregue estas referencias de paquete dentro del elemento \<Project\>.

   ```xml
   <ItemGroup>
       <PackageReference Include="Azure.Storage.Blobs" Version="12.5.0-preview.1" />
       <PackageReference Include="Azure.Storage.Common" Version="12.4.0-preview.1" />
       <PackageReference Include="Azure.Storage.QuickQuery" Version="12.0.0-preview.1" />
   </ItemGroup>
   ```

4. Restaure los paquetes del SDK de versión preliminar. Este comando de ejemplo restaura los paquetes del SDK de versión preliminar mediante el comando `dotnet restore`. 

   ```console
   dotnet restore --source C:\Users\contoso\myProject
   ```

5. Restaure las demás dependencias desde el repositorio público de NuGet.

   ```console
   dotnet restore
   ```

### <a name="java"></a>[Java](#tab/java)

1. Cree un directorio en la raíz del proyecto. El directorio raíz es el directorio que contiene el archivo **pom.xml**.

   > [!NOTE]
   > En los ejemplos de este artículo se considera que el nombre del directorio es **lib**.

2. Descargue los paquetes de aceleración de consultas. Puede obtener un archivo ZIP comprimido que contenga estos paquetes mediante este vínculo: [https://aka.ms/adls/qqsdk/java](https://aka.ms/adls/qqsdk/java). 

3. Extraiga los archivos de este archivo ZIP en el directorio que ha creado. En nuestro ejemplo, ese directorio se denomina **lib**. 

4. Abra el archivo *pom.xml* en el editor de texto. Agregue los siguientes elementos de dependencia al grupo de dependencias. 

   ```xml
   <!-- Request static dependencies from Maven -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-core</artifactId>
       <version>1.3.0</version>
   </dependency>
   <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-core-http-netty</artifactId>
      <version>1.3.0</version>
   </dependency>
   <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>1.9.2</version>
   </dependency>
   <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.8</version>
   </dependency>
   <!-- Local dependencies -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-storage-blob</artifactId>
       <version>12.5.0-beta.1</version>
       <scope>system</scope>
       <systemPath>${project.basedir}/lib/azure-storage-blob-12.5.0-beta.1.jar</systemPath>
   </dependency>
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-storage-common</artifactId>
       <version>12.5.0-beta.1</version>
       <scope>system</scope>
       <systemPath>${project.basedir}/lib/azure-storage-common-12.5.0-beta.1.jar</systemPath>
   </dependency>
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-storage-quickquery</artifactId>
       <version>12.0.0-beta.1</version>
       <scope>system</scope>
       <systemPath>${project.basedir}/lib/azure-storage-quickquery-12.0.0-beta.1.jar</systemPath>
   </dependency>
   ```

---

## <a name="add-statements"></a>Adición de instrucciones


### <a name="net"></a>[.NET](#tab/dotnet)

Agregue estas instrucciones `using` al inicio del archivo de código.

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;
using Azure.Storage.QuickQuery;
using Azure.Storage.QuickQuery.Models;
```

La aceleración de consultas recupera datos con formato CSV y JSON. Por lo tanto, asegúrese de agregar instrucciones USING para las bibliotecas de análisis de CSV o JSON que decida usar. En los ejemplos que aparecen en este artículo se analiza un archivo CSV mediante la biblioteca de [CsvHelper](https://www.nuget.org/packages/CsvHelper/) que está disponible en NuGet. Por lo tanto, se agregan estas instrucciones `using` al inicio del archivo de código.

```csharp
using CsvHelper;
using CsvHelper.Configuration;
```

Para compilar los ejemplos presentados en este artículo, también deberá agregar estas instrucciones `using`.

```csharp
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
using System.Threading;
using System.Linq;
```

### <a name="java"></a>[Java](#tab/java)

Agregue estas instrucciones `import` al inicio del archivo de código.

```java
import com.azure.storage.blob.*;
import com.azure.storage.blob.models.*;
import com.azure.storage.common.*;
import com.azure.storage.quickquery.*;
import com.azure.storage.quickquery.models.*;
import java.io.*;
import org.apache.commons.csv.*;
```

---

## <a name="retrieve-data-by-using-a-filter"></a>Recuperación de datos mediante un filtro

Puede usar SQL para especificar los predicados de filtro de fila y las proyecciones de columna en una solicitud de aceleración de consultas. En el código siguiente se consulta un archivo CSV en el almacenamiento y se devuelven todas las filas de datos en las que la tercera columna coincide con el valor `Hemingway, Ernest`. 

- En la consulta SQL, se usa la palabra clave `BlobStorage` para indicar el archivo que se consulta.

- Las referencias de columna se especifican como `_N`, donde la primera columna es `_1`. Si el archivo de código fuente contiene una fila de encabezado, puede hacer referencia a las columnas por el nombre especificado en la fila de encabezado. 

### <a name="net"></a>[.NET](#tab/dotnet)

El método asincrónico `BlobQuickQueryClient.QueryAsync` envía la consulta a la API de aceleración de consultas y, a continuación, vuelve a transmitir los resultados a la aplicación como un objeto [Stream](https://docs.microsoft.com/dotnet/api/system.io.stream?view=netframework-4.8).

```cs
static async Task QueryHemingway(BlockBlobClient blob)
{
    string query = @"SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest'";
    await DumpQueryCsv(blob, query, false);
}

private static async Task DumpQueryCsv(BlockBlobClient blob, string query, bool headers)
{
    try
    {
        using (var reader = new StreamReader((await blob.GetQuickQueryClient().QueryAsync(query,
                new CsvTextConfiguration() { HasHeaders = headers }, 
                new CsvTextConfiguration() { HasHeaders = false }, 
                new ErrorHandler(),
                new BlobRequestConditions(), 
                new ProgressHandler(),
                CancellationToken.None)).Value.Content))
        {
            using (var parser = new CsvReader(reader, new CsvConfiguration(CultureInfo.CurrentCulture) 
            { HasHeaderRecord = false }))
            {
                while (await parser.ReadAsync())
                {
                    parser.Context.Record.All(cell =>
                    {
                        Console.Out.Write(cell + "  ");
                        return true;
                    });
                    Console.Out.WriteLine();
                }
            }
        }
    }
    catch (Exception ex)
    {
        Console.Error.WriteLine("Exception: " + ex.ToString());
    }
}

class ErrorHandler : IBlobQueryErrorReceiver
{
    public void ReportError(BlobQueryError err)
    {
        Console.Error.WriteLine($"Error: {err.Name}:{ err.Description }");
    }
}

class ProgressHandler : IProgress<long>
{
    public void Report(long value)
    {
        Console.Error.WriteLine("Bytes scanned: " + value.ToString());
    }
}

```

### <a name="java"></a>[Java](#tab/java)

El método `BlobQuickQueryClient.openInputStream()` envía la consulta a la API de aceleración de consultas y, a continuación, vuelve a transmitir los resultados a la aplicación como un objeto `InputStream` que se puede leer como cualquier otro objeto InputStream.

```java
static void QueryHemingway(BlobClient blobClient) {
    String expression = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest'";
    DumpQueryCsv(blobClient, expression, false);
}

static void DumpQueryCsv(BlobClient blobClient, String query, Boolean headers) {
    try {
    
        BlobQuickQueryDelimitedSerialization input = new BlobQuickQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(headers)
            .setFieldQuote('\0')
            .setEscapeChar('\\');

        BlobQuickQueryDelimitedSerialization output = new BlobQuickQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(false)
            .setFieldQuote('\0')
            .setEscapeChar('\n');
                
        BlobRequestConditions requestConditions = null;
        /* ErrorReceiver determines what to do on errors. */
        ErrorReceiver<BlobQuickQueryError> errorReceiver = System.out::println;

        /* ProgressReceiver details how to log progress*/
        com.azure.storage.common.ProgressReceiver progressReceiver = System.out::println;
    
        /* Create a query acceleration client to the blob. */
        BlobQuickQueryClient qqClient = new BlobQuickQueryClientBuilder(blobClient)
            .buildClient();
        /* Open the query input stream. */
        InputStream stream = qqClient.openInputStream(query, input, output, requestConditions, errorReceiver, progressReceiver);
            
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(stream))) {
            /* Read from stream like you normally would. */
            for (CSVRecord record : CSVParser.parse(reader, CSVFormat.EXCEL)) {
                System.out.println(record.toString());
            }
        }
    } catch (Exception e) {
        System.err.println("Exception: " + e.toString());
    }
}
```

---

## <a name="retrieve-specific-columns"></a>Recuperación de columnas específicas

Puede limitar los resultados a un subconjunto de columnas. De este modo, solo recuperará las columnas necesarias para realizar un cálculo determinado. Esto mejora el rendimiento de la aplicación y reduce los costos porque se transfieren menos datos a través de la red. 

Este código solo recupera la columna `PublicationYear` para todos los libros del conjunto de datos. También utiliza la información de la fila de encabezado del archivo de origen para hacer referencia a las columnas de la consulta.


### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryPublishDates(BlockBlobClient blob)
{
    string query = @"SELECT PublicationYear FROM BlobStorage";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryPublishDates(BlobClient blobClient)
{
    String expression = "SELECT PublicationYear FROM BlobStorage";
    DumpQueryCsv(blobClient, expression, true);
}
```

---

El código siguiente combina el filtrado de filas y las proyecciones de columnas en la misma consulta. 

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryMysteryBooks(BlockBlobClient blob)
{
    string query = @"SELECT BibNum, Title, Author, ISBN, Publisher FROM BlobStorage WHERE Subjects LIKE '%Mystery%'";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryMysteryBooks(BlobClient blobClient)
{
    String expression = "SELECT BibNum, Title, Author, ISBN, Publisher FROM BlobStorage WHERE Subjects LIKE '%Mystery%'";
    DumpQueryCsv(blobClient, expression, true);
}
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Formulario de inscripción para la aceleración de consultas](https://aka.ms/adls/qa-preview-signup)    
- [Aceleración de consultas de Azure Data Lake Storage (versión preliminar)](data-lake-storage-query-acceleration.md)
- [Referencia del lenguaje SQL de aceleración de consultas (versión preliminar)](query-acceleration-sql-reference.md)
