---
title: 'Uso del SDK de .NET para HBase: Azure HDInsight'
description: Use el SDK de .NET para HBase para crear y eliminar tablas, así como para leer y escribir datos.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/02/2019
ms.openlocfilehash: e8bce1ca10e9175b699bd548d9241b78bce3b5cf
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89504866"
---
# <a name="use-the-net-sdk-for-apache-hbase"></a>Uso del SDK de .NET para Apache HBase

[Apache HBase](apache-hbase-overview.md) proporciona dos opciones principales para trabajar con los datos: [las consultas de Apache Hive y las llamadas a la API RESTful de HBase](apache-hbase-tutorial-get-started-linux.md). Puede trabajar directamente con la API de REST con el comando `curl` o una utilidad similar.

Para las aplicaciones de C# y .NET, la [biblioteca cliente de REST de Microsoft HBase para .NET](https://www.nuget.org/packages/Microsoft.HBase.Client/) proporciona una biblioteca cliente, además de la API de REST de HBase.

## <a name="install-the-sdk"></a>Instalación del SDK

El SDK de .NET para HBase se proporciona como un paquete de NuGet, que puede instalarse desde la **Consola del Administrador de paquetes de NuGet** de Visual Studio con el siguiente comando:

```console
Install-Package Microsoft.HBase.Client
```

## <a name="instantiate-a-new-hbaseclient-object"></a>Crear una instancia de un objeto nuevo de HBaseClient

Para usar el SDK, cree instancias de un objeto nuevo de `HBaseClient`, pasando en `ClusterCredentials` compuesto por `Uri` para el clúster, y la contraseña y el nombre de usuario de Hadoop.

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net"), "USERNAME", "PASSWORD");
client = new HBaseClient(credentials);
```

Reemplace CLUSTERNAME por el nombre del clúster de HDInsight HBase y USERNAME y PASSWORD por las credenciales de Apache Hadoop especificadas durante la creación del clúster. El nombre de usuario de Hadoop predeterminado es **admin**.

## <a name="create-a-new-table"></a>Creación de una nueva tabla

HBase almacena los datos en tablas. Una tabla consta de un *Rowkey*, la clave principal y uno o varios grupos de columnas denominadas *familias de columnas*. Los datos de cada tabla se distribuyen horizontalmente por un intervalo de Rowkey en *regiones*. Cada región tiene una clave de inicio y de fin. Una tabla puede tener una o varias regiones. A medida que aumentan los datos de la tabla, HBase divide las regiones grandes en regiones más pequeñas. Las regiones se almacenan en *servidores de región*, donde un servidor de región puede almacenar varias regiones.

Los datos se almacenan físicamente en *HFiles*. Un único HFile contiene datos de una tabla, una región y una familia de columnas. Las filas de HFile se almacenan ordenadas en Rowkey. Cada HFile tiene un índice *Árbol B+* para la recuperación rápida de las filas.

Para crear una tabla nueva, especifique `TableSchema` y columnas. El código siguiente comprueba si la tabla "RestSDKTable" ya existe. Si no es así, se crea.

```csharp
if (!client.ListTablesAsync().Result.name.Contains("RestSDKTable"))
{
    // Create the table
    var newTableSchema = new TableSchema {name = "RestSDKTable" };
    newTableSchema.columns.Add(new ColumnSchema() {name = "t1"});
    newTableSchema.columns.Add(new ColumnSchema() {name = "t2"});

    await client.CreateTableAsync(newTableSchema);
}
```

Esta nueva tabla tiene familias de dos columnas, t1 y t2. Dado que las familias de columnas se almacenan por separado en HFiles diferentes, resulta útil tener una familia de columnas independiente para los datos consultados con más frecuencia. En el siguiente ejemplo para [insertar datos](#insert-data), se agregan columnas a la familia de columnas t1.

## <a name="delete-a-table"></a>Eliminar una tabla

Para eliminar una tabla:

```csharp
await client.DeleteTableAsync("RestSDKTable");
```

## <a name="insert-data"></a>Insertar datos

Para insertar datos, especifique una clave de fila única como identificador de la fila. Los datos se almacenan en una matriz de `byte[]`. El código siguiente define y agrega las columnas `title`, `director` y `release_date` a la familia de columnas t1, ya que es a estas columnas a las que se accede con más frecuencia. Las columnas `description` y `tagline` se agregan a la familia de columnas t2. Puede dividir los datos en familias de columnas según sea necesario.

```csharp
var key = "fifth_element";
var row = new CellSet.Row { key = Encoding.UTF8.GetBytes(key) };
var value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:title"),
    data = Encoding.UTF8.GetBytes("The Fifth Element")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:director"),
    data = Encoding.UTF8.GetBytes("Luc Besson")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:release_date"),
    data = Encoding.UTF8.GetBytes("1997")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:description"),
    data = Encoding.UTF8.GetBytes("In the colorful future, a cab driver unwittingly becomes the central figure in the search for a legendary cosmic weapon to keep Evil and Mr Zorg at bay.")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:tagline"),
    data = Encoding.UTF8.GetBytes("The Fifth is life")
};
row.values.Add(value);

var set = new CellSet();
set.rows.Add(row);

await client.StoreCellsAsync("RestSDKTable", set);
```

HBase implementa [Cloud BigTable](https://cloud.google.com/bigtable/), por lo que el formato de datos es similar al siguiente:

![Salida de datos de ejemplo de Apache HBase](./media/apache-hbase-rest-sdk/hdinsight-table-roles.png)

## <a name="select-data"></a>Selección de datos

Para leer los datos de una tabla de HBase, pase la clave de la fila y el nombre de la tabla para que el método `GetCellsAsync` devuelva `CellSet`.

```csharp
var key = "fifth_element";

var cells = await client.GetCellsAsync("RestSDKTable", key);
// Get the first value from the row.
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values[0].data));
// Get the value for t1:title
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:title").data));
// With the previous insert, it should yield: "The Fifth Element"
```

En este caso, el código solo devuelve la primera fila coincidente, ya que solo debe haber una fila para una clave única. El valor devuelto se cambia al formato `string` desde la matriz `byte[]`. También puede convertir el valor en otros tipos, como un número entero para la fecha de lanzamiento de la película:

```csharp
var releaseDateField = cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:release_date");
int releaseDate = 0;

if (releaseDateField != null)
{
    releaseDate = Convert.ToInt32(Encoding.UTF8.GetString(releaseDateField.data));
}
Console.WriteLine(releaseDate);
// Should return 1997
```

## <a name="scan-over-rows"></a>Análisis de filas

HBase utiliza `scan` para recuperar una o varias filas. En este ejemplo se solicitan varias filas en lotes de 10 y se recuperan datos cuyos valores clave están comprendidos entre 25 y 35. Después de recuperar todas las filas, elimine el escáner para limpiar los recursos.

```csharp
var tableName = "mytablename";

// Assume the table has integer keys and we want data between keys 25 and 35
var scanSettings = new Scanner()
{
    batch = 10,
    startRow = BitConverter.GetBytes(25),
    endRow = BitConverter.GetBytes(35)
};
RequestOptions scanOptions = RequestOptions.GetDefaultOptions();
scanOptions.AlternativeEndpoint = "hbaserest0/";
ScannerInformation scannerInfo = null;
try
{
    scannerInfo = await client.CreateScannerAsync(tableName, scanSettings, scanOptions);
    CellSet next = null;
    while ((next = client.ScannerGetNextAsync(scannerInfo, scanOptions).Result) != null)
    {
        foreach (var row in next.rows)
        {
            // ... read the rows
        }
    }
}
finally
{
    if (scannerInfo != null)
    {
        await client.DeleteScannerAsync(tableName, scannerInfo, scanOptions);
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a un ejemplo de Apache HBase en HDInsight](apache-hbase-tutorial-get-started-linux.md)
* Compilación de una aplicación de extremo a extremo mediante el artículo [Analizar opinión de Twitter en tiempo real con Apache HBase](../hdinsight-hbase-analyze-twitter-sentiment.md)
