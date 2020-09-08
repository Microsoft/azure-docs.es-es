---
title: Phoenix Query Server REST SDK en Azure HDInsight
description: Instale y utilice Phoenix Query Server REST SDK en Azure HDInsight.
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/01/2020
ms.openlocfilehash: 4ad6d16e4c6d91b44061d99ffa1c397fb0e0ec28
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019647"
---
# <a name="apache-phoenix-query-server-rest-sdk"></a>SDK de REST de Apache Phoenix Query Server

[Apache Phoenix](https://phoenix.apache.org/) es una capa de base de datos relacional de código abierto y masivamente paralela que se utiliza sobre [Apache HBase](apache-hbase-overview.md). Phoenix le permite usar consultas de tipo SQL con HBase mediante herramientas SSH como [SQLLine](apache-hbase-query-with-phoenix.md). Phoenix también proporciona un servidor HTTP, denominado Phoenix Query Server (PQS). Se trata de un cliente ligero que admite dos mecanismos de transporte para la comunicación de cliente: JSON y búferes de protocolo. Los búferes de protocolo son el mecanismo predeterminado y ofrecen una comunicación más eficaz que JSON.

En este artículo se describe cómo usar PQS REST SDK para crear tablas, realizar operaciones upsert individualmente y de forma masiva, y seleccionar datos con instrucciones SQL. En los ejemplos se usa el [controlador de Microsoft .NET para Apache Phoenix Query Server](https://www.nuget.org/packages/Microsoft.Phoenix.Client). Este SDK se basa en API de [Avatica de Apache Calcite](https://calcite.apache.org/avatica/), que usa exclusivamente los búferes de protocolo para el formato de serialización.

Para obtener más información, consulte [Apache Calcite Avatica Protocol Buffers Reference](https://calcite.apache.org/avatica/docs/protobuf_reference.html) (Referencia de los búferes de protocolo de Apache Calcite Avatica).

## <a name="install-the-sdk"></a>Instalación del SDK

El controlador Microsoft .NET para Apache Phoenix Query Server se proporciona como un paquete de NuGet, que puede instalarse desde la **Consola del Administrador de paquetes de NuGet** de Visual Studio con el siguiente comando:

```console
Install-Package Microsoft.Phoenix.Client
```

## <a name="instantiate-new-phoenixclient-object"></a>Crear una instancia nueva del objeto PhoenixClient

Para comenzar a usar la biblioteca, cree una instancia de un objeto nuevo `PhoenixClient`. Para ello, pase un objeto `ClusterCredentials` que contenga el `Uri` al clúster, así como la contraseña y el nombre de usuario de Apache Hadoop del clúster.

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net/"), "USERNAME", "PASSWORD");
client = new PhoenixClient(credentials);
```

Reemplace CLUSTERNAME por el nombre del clúster de HDInsight HBase y USERNAME y PASSWORD por las credenciales de Hadoop especificadas durante la creación del clúster. El nombre de usuario de Hadoop predeterminado es **admin**.

## <a name="generate-unique-connection-identifier"></a>Generar un identificador de conexión único

Para enviar una o varias solicitudes a PQS, debe incluir un identificador de conexión único para asociar las solicitudes con la conexión.

```csharp
string connId = Guid.NewGuid().ToString();
```

En primer lugar, cada ejemplo realiza una llamada al método `OpenConnectionRequestAsync`, pasando el identificador único de la conexión. A continuación, defina `ConnectionProperties` y `RequestOptions`, pasando esos objetos y el identificador de conexión generado para el método `ConnectionSyncRequestAsync`. El objeto `ConnectionSyncRequest` de PQS ayuda a garantizar que el cliente y el servidor disponen de una vista coherente de las propiedades de la base de datos.

## <a name="connectionsyncrequest-and-its-connectionproperties"></a>ConnectionSyncRequest y ConnectionProperties

Para llamar a `ConnectionSyncRequestAsync`, pase un objeto `ConnectionProperties`.

```csharp
ConnectionProperties connProperties = new ConnectionProperties
{
    HasAutoCommit = true,
    AutoCommit = true,
    HasReadOnly = true,
    ReadOnly = false,
    TransactionIsolation = 0,
    Catalog = "",
    Schema = "",
    IsDirty = true
};
await client.ConnectionSyncRequestAsync(connId, connProperties, options);
```

Estas son algunas propiedades que pueden interesarle:

| Propiedad | Descripción |
| -- | -- |
| AutoCommit | Valor booleano que indica si la propiedad `autoCommit` está habilitada para las transacciones de Phoenix. |
| ReadOnly | Valor booleano que indica si la conexión es de solo lectura. |
| TransactionIsolation | Valor entero que indica el nivel de aislamiento de la transacción por la especificación de JDBC, consulte la tabla siguiente.|
| Catálogo | Nombre del catálogo que se usará al capturar propiedades de conexión. |
| Schema | Nombre del esquema que se usará al capturar propiedades de conexión. |
| IsDirty | Valor booleano que indica si se han modificado las propiedades. |

A continuación, se muestran los valores `TransactionIsolation`:

| Valor de aislamiento | Descripción |
| -- | -- |
| 0 | Las transacciones no se admiten. |
| 1 | Pueden producirse lecturas de datos sucios, no repetibles y fantasma. |
| 2 | Se evitan las lecturas de datos sucios, pero pueden producirse lecturas no repetibles y fantasma. |
| 4 | Se evitan las lecturas de datos sucios y no repetibles, pero pueden producirse lecturas fantasma. |
| 8 | Se evitan las lecturas de datos sucios, no repetibles y fantasma. |

## <a name="create-a-new-table"></a>Creación de una nueva tabla

HBase, al igual que cualquier otro RDBMS, almacena los datos en tablas. Phoenix utiliza consultas SQL estándar para crear nuevas tablas, al mismo tiempo que define los tipos de columna y las claves principales.

En este ejemplo y en todos los siguientes, use el objeto `PhoenixClient` con instancias tal como se define en [Crear una instancia nueva del objeto PhoenixClient](#instantiate-new-phoenixclient-object).

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// You can set certain request options, such as timeout in milliseconds:
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
CreateStatementResponse createStatementResponse = null;
OpenConnectionResponse openConnResponse = null;

try
{
    // Opening connection
    var info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Create the statement
    createStatementResponse = client.CreateStatementRequestAsync(connId, options).Result;
    
    // Create the table if it does not exist
    string sql = "CREATE TABLE IF NOT EXISTS Customers (Id varchar(20) PRIMARY KEY, FirstName varchar(50), " +
        "LastName varchar(100), StateProvince char(2), Email varchar(255), Phone varchar(15))";
    await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    Console.WriteLine($"Table \"Customers\" created.");
}
catch (Exception e)
{
    Console.WriteLine(e);
    throw;
}
finally
{
    if (createStatementResponse != null)
    {
        client.CloseStatementRequestAsync(connId, createStatementResponse.StatementId, options).Wait();
        createStatementResponse = null;
    }

    if (openConnResponse != null)
    {
        client.CloseConnectionRequestAsync(connId, options).Wait();
        openConnResponse = null;
    }
}
```

En el ejemplo anterior se crea una nueva tabla denominada `Customers` con la opción `IF NOT EXISTS`. La llamada `CreateStatementRequestAsync` crea una nueva instrucción en el servidor Avitica (PQS). El bloque `finally` cierra los objetos devueltos `CreateStatementResponse` y `OpenConnectionResponse`.

## <a name="insert-data-individually"></a>Insertar datos individualmente

En este ejemplo se muestra una inserción de datos individual, que hace referencia a una colección `List<string>` de abreviaturas de los territorios y estados de América:

```csharp
var states = new List<string> { "AL", "AK", "AS", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FM", "FL", "GA", "GU", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MH", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "MP", "OH", "OK", "OR", "PW", "PA", "PR", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VI", "VA", "WA", "WV", "WI", "WY" };
```

El valor de la columna `StateProvince` de la tabla se usará en una operación de selección posterior.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = await client.PrepareRequestAsync(connId, sql, 100, options);
    statementHandle = prepareResponse.Statement;
    
    var r = new Random();

    // Insert 300 rows
    for (int i = 0; i < 300; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0,1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        Console.WriteLine("Inserting customer " + i);

        await client.ExecuteRequestAsync(statementHandle, list, long.MaxValue, true, options);
    }

    await client.CommitRequestAsync(connId, options);

    Console.WriteLine("Upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

La estructura para ejecutar una instrucción INSERT es similar a crear una nueva tabla. Al final del bloque `try`, la transacción se confirma explícitamente. En este ejemplo se repite una transacción de inserción 300 veces. En el ejemplo siguiente se muestra un proceso de inserción de lote más eficaz.

## <a name="batch-insert-data"></a>Datos de inserción de lote

El código siguiente es casi idéntico al código para insertar datos de forma individual. En este ejemplo se usa el objeto `UpdateBatch` en una llamada a `ExecuteBatchRequestAsync`, en lugar de llamar repetidamente a `ExecuteRequestAsync` con una instrucción preparada.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
CreateStatementResponse createStatementResponse = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Creating statement
    createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = client.PrepareRequestAsync(connId, sql, long.MaxValue, options).Result;
    var statementHandle = prepareResponse.Statement;
    var updates = new pbc.RepeatedField<UpdateBatch>();

    var r = new Random();

    // Insert 300 rows
    for (int i = 300; i < 600; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0, 1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        var batch = new UpdateBatch
        {
            ParameterValues = list
        };
        updates.Add(batch);

        Console.WriteLine($"Added customer {i} to batch");
    }

    var executeBatchResponse = await client.ExecuteBatchRequestAsync(connId, statementHandle.Id, updates, options);

    Console.WriteLine("Batch upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

En un entorno de prueba, insertar individualmente 300 registros nuevos tardó casi 2 minutos. En cambio, la inserción de 300 registros como un lote solo tardó 6 segundos.

## <a name="select-data"></a>Selección de datos

En este ejemplo se muestra cómo reutilizar una conexión para ejecutar varias consultas:

1. Seleccione todos los registros y, después, capture los registros restantes después de que se devuelva el valor máximo predeterminado de 100.
2. Utilice una instrucción SELECT de recuento total de filas para recuperar un único resultado escalar.
3. Ejecute una instrucción SELECT que devuelve el número total de clientes por estado o territorio.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;

try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);
    var createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "SELECT * FROM Customers";
    ExecuteResponse executeResponse = await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> rows = executeResponse.Results[0].FirstFrame.Rows;
    // Loop through all of the returned rows and display the first two columns
    for (int i = 0; i < rows.Count; i++)
    {
        Row row = rows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + " " + row.Value[1].ScalarValue.StringValue);
    }
    
    // 100 is hard-coded on the server side as the default firstframe size
    // FetchRequestAsync is called to get any remaining rows
    Console.WriteLine("");
    Console.WriteLine($"Number of rows: {rows.Count}");

    // Fetch remaining rows, offset is not used, simply set to 0
    // When FetchResponse.Frame.Done is true, all rows were fetched
    FetchResponse fetchResponse = await client.FetchRequestAsync(connId, createStatementResponse.StatementId, 0, int.MaxValue, options);
    Console.WriteLine($"Frame row count: {fetchResponse.Frame.Rows.Count}");
    Console.WriteLine($"Fetch response is done: {fetchResponse.Frame.Done}");
    Console.WriteLine("");

    // Running query 2
    string sql2 = "select count(*) from Customers";
    ExecuteResponse countResponse = await client.PrepareAndExecuteRequestAsync(connId, sql2, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);
    long count = countResponse.Results[0].FirstFrame.Rows[0].Value[0].ScalarValue.NumberValue;

    Console.WriteLine($"Total customer records: {count}");
    Console.WriteLine("");

    // Running query 3
    string sql3 = "select StateProvince, count(*) as Number from Customers group by StateProvince order by Number desc";
    ExecuteResponse groupByResponse = await client.PrepareAndExecuteRequestAsync(connId, sql3, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> stateRows = groupByResponse.Results[0].FirstFrame.Rows;
    for (int i = 0; i < stateRows.Count; i++)
    {
        Row row = stateRows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + ": " + row.Value[1].ScalarValue.NumberValue);
    }
}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

El resultado de las instrucciones `select` debe ser el siguiente:

```output
id0 first0
id1 first1
id10 first10
id100 first100
id101 first101
id102 first102
. . .
id185 first185
id186 first186
id187 first187
id188 first188

Number of rows: 100
Frame row count: 500
Fetch response is done: True

Total customer records: 600

NJ: 21
CA: 19
GU: 17
NC: 16
IN: 16
MA: 16
AZ: 16
ME: 16
IL: 15
OR: 15
. . . 
MO: 10
HI: 10
GA: 10
DC: 9
NM: 9
MD: 9
MP: 9
SC: 7
AR: 7
MH: 6
FM: 5
```

## <a name="next-steps"></a>Pasos siguientes

* [Apache Phoenix en HDInsight](../hdinsight-phoenix-in-hdinsight.md)
* [Uso del SDK de REST para Apache HBase](apache-hbase-rest-sdk.md)
