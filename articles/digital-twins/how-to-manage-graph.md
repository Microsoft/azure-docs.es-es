---
title: Administración del grafo de gemelos con relaciones
titleSuffix: Azure Digital Twins
description: Consulte cómo administrar un grafo de gemelos digitales conectándolos con relaciones.
author: baanders
ms.author: baanders
ms.date: 4/10/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 7f7239e0c13478af712d8e8d9dad8fda23fe42c7
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125539"
---
# <a name="manage-a-graph-of-digital-twins-using-relationships"></a>Administración de un grafo de gemelos digitales con relaciones

El corazón de Azure Digital Twins es el [grafo de gemelos](concepts-twins-graph.md) que representa todo el entorno. El grafo de gemelos se compone de gemelos digitales individuales conectados a través de **relaciones**.

Una vez que tenga una [instancia de Azure Digital Twins](how-to-set-up-instance-scripted.md) en funcionamiento y que haya configurado el código de [autenticación](how-to-authenticate-client.md) para la aplicación cliente, puede usar las [**API DigitalTwins**](how-to-use-apis-sdks.md) para crear, modificar y eliminar gemelos digitales y sus relaciones en una instancia de Azure Digital Twins. También puede usar el [SDK de .NET ( C# )](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) o la [CLI de Azure Digital Twins](how-to-use-cli.md).

Este artículo está centrado en la administración de las relaciones y el grafo en conjunto; para trabajar con gemelos digitales individuales, consulte [*Procedimiento: Administración de Digital Twins*](how-to-manage-twin.md).

[!INCLUDE [visualizing with Azure Digital Twins explorer](../../includes/digital-twins-visualization.md)]

## <a name="create-relationships"></a>Crear relaciones

Las relaciones describen cómo los diferentes gemelos digitales se conectan entre sí, lo que constituye la base del grafo de gemelos.

Las relaciones se crean mediante la llamada a `CreateRelationship`. 

Para crear una relación, debe especificar:
* El id. del gemelo de origen (el gemelo en el que se origina la relación)
* El id. del gemelo de destino (el gemelo al que llega la relación)
* Un nombre de relación
* Un id. de relación

El id. de relación debe ser único dentro del gemelo de origen especificado. No es necesario que sea único globalmente.
Por ejemplo, para el gemelo *foo*, cada id. de relación específico debe ser único. Sin embargo, otro gemelo *bar* puede tener una relación de salida que coincida con el mismo id. de relación de *foo*. 

En el ejemplo de código siguiente se muestra cómo agregar una relación a la instancia de Azure Digital Twins.

```csharp
public async static Task CreateRelationship(DigitalTwinsClient client, string srcId, string targetId)
{
    var relationship = new BasicRelationship
    {
        TargetId = targetId,
        Name = "contains"
    };

    try
    {
        string relId = $"{srcId}-contains->{targetId}";
        await client.CreateRelationshipAsync(srcId, relId, JsonSerializer.Serialize(relationship));
        Console.WriteLine("Created relationship successfully");
    }
    catch (RequestFailedException rex) {
        Console.WriteLine($"Create relationship error: {rex.Status}:{rex.Message}");
    }
}
```

Para obtener más información sobre la clase auxiliar `BasicRelationship`, consulte [*Procedimiento: Uso de las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md).

## <a name="list-relationships"></a>Enumeración de las relaciones

Para tener acceso a la lista de relaciones de un gemelo determinado en el grafo, puede usar:

```csharp
await client.GetRelationshipsAsync(id);
```

Devuelve un elemento `Azure.Pageable<T>` o `Azure.AsyncPageable<T>`, en función de si se usa la versión sincrónica o asincrónica de la llamada.

Este es un ejemplo completo que recupera una lista de relaciones:

```csharp
public async Task<List<BasicRelationship>> FindOutgoingRelationshipsAsync(string dtId)
{
    // Find the relationships for the twin
    try
    {
        // GetRelationshipsAsync will throw if an error occurs
        AsyncPageable<string> relsJson = client.GetRelationshipsAsync(dtId);
        List<BasicRelationship> results = new List<BasicRelationship>();
        await foreach (string relJson in relsJson)
        {
            var rel = System.Text.Json.JsonSerializer.Deserialize<BasicRelationship>(relJson);
            results.Add(rel);
        }
        return results;
    }
    catch (RequestFailedException ex)
    {
        Log.Error($"*** Error {ex.Status}/{ex.ErrorCode} retrieving relationships for {dtId} due to {ex.Message}");
        return null;
    }
}
```

Puede usar las relaciones recuperadas para navegar a otros gemelos del grafo. Para ello, lea el campo `target` de la relación que se devuelve y úselo como id. de la siguiente llamada a `GetDigitalTwin`. 

### <a name="find-relationships-to-a-digital-twin"></a>Búsqueda de relaciones para un gemelo digital

Azure Digital Twins también dispone de una API para buscar todas las relaciones de entrada para un gemelo determinado. Suele ser útil para la navegación inversa o cuando se elimina un gemelo.

El ejemplo de código anterior se centró en buscar relaciones de salida. El ejemplo siguiente es similar, pero busca las relaciones de entrada en su lugar. También las elimina cuando se encuentran.

Tenga en cuenta que las llamadas a `IncomingRelationship` no devuelven todo el cuerpo de la relación.

```csharp
async Task<List<IncomingRelationship>> FindIncomingRelationshipsAsync(string dtId)
{
    // Find the relationships for the twin
    try
    {
        // GetRelationshipsAsync will throw an error if a problem occurs
        AsyncPageable<IncomingRelationship> incomingRels = client.GetIncomingRelationshipsAsync(dtId);

        List<IncomingRelationship> results = new List<IncomingRelationship>();
        await foreach (IncomingRelationship incomingRel in incomingRels)
            results.Add(incomingRel);
    }
    catch (RequestFailedException ex)
    {
        Log.Error($"*** Error {ex.Status}/{ex.ErrorCode} retrieving incoming relationships for {dtId} due to {ex.Message}");
    }
}
```

## <a name="delete-relationships"></a>Eliminar relaciones

Puede eliminar relaciones mediante `DeleteRelationship(source, relId);`.

El primer parámetro especifica el gemelo de origen (el gemelo en el que se origina la relación). El otro parámetro es el id. de relación. Necesita tanto el id. de gemelo como el id. de relación, ya que los id. de relación solo son únicos dentro del ámbito de un gemelo.

## <a name="create-a-twin-graph"></a>Creación de un grafo de gemelos 

En el siguiente fragmento de código se usan las operaciones de relación de este artículo para crear un grafo de gemelos con gemelos digitales y relaciones.

```csharp
static async Task CreateTwins()
{
    // Create twins - see utility functions below 
    await CreateRoom("Room01", 68, 50, false, "");
    await CreateRoom("Room02", 70, 66, true, "EId-00124");
    await CreateFloorOrBuilding("Floor01", makeFloor:true);

    // Create relationships
    await AddRelationship("Floor01", "contains", "Floor-to-Room01", "Room01");
    await AddRelationship("Floor01", "contains", "Floor-to-Room02", "Room02");
}

static async Task<bool> AddRelationship(string source, string relationship, string id, string target)
{
    var relationship = new BasicRelationship
    {
        TargetId = target,
        Name = relationship
    };

    try
    {
        string relId = $"{source}-contains->{target}";
        await client.CreateRelationshipAsync(source, relId, JsonSerializer.Serialize(relationship));
        Console.WriteLine("Created relationship successfully");
        return true;
    }
    catch (RequestFailedException rex) {
        Console.WriteLine($"Create relationship error: {rex.Status}:{rex.Message}");
        return false;
    }
}

static async Task<bool> CreateRoom(string id, double temperature, double humidity)
{
    BasicDigitalTwin twin = new BasicDigitalTwin();
    twin.Metadata = new DigitalTwinMetadata();
    twin.Metadata.ModelId = "dtmi:com:contoso:Room;2";
    // Initialize properties
    Dictionary<string, object> props = new Dictionary<string, object>();
    props.Add("Temperature", temperature);
    props.Add("Humidity", humidity);
    twin.CustomProperties = props;
    
    try
    {
        client.CreateDigitalTwin(id, JsonSerializer.Serialize<BasicDigitalTwin>(twin)); 
        return true;       
    }
    catch (ErrorResponseException e)
    {
        Console.WriteLine($"*** Error creating twin {id}: {e.Response.StatusCode}"); 
        return false;
    }
}

static async Task<bool> CreateFloorOrBuilding(string id, bool makeFloor=true)
{
    string type = "dtmi:com:contoso:Building;3";
    if (makeFloor==true)
        type = "dtmi:com:contoso:Floor;2";
    BasicDigitalTwin twin = new BasicDigitalTwin();
    twin.Metadata = new DigitalTwinMetadata();
    twin.Metadata.ModelId = type;
    // Initialize properties
    Dictionary<string, object> props = new Dictionary<string, object>();
    props.Add("AverageTemperature", 0);
    twin.CustomProperties = props;
    
    try
    {
        client.CreateDigitalTwin(id, JsonSerializer.Serialize<BasicDigitalTwin>(twin));  
        return true;      
    }
    catch (ErrorResponseException e)
    {
        Console.WriteLine($"*** Error creating twin {id}: {e.Response.StatusCode}"); 
        return false;
    }
}
```

### <a name="create-a-twin-graph-from-a-spreadsheet"></a>Creación de un grafo de gemelos a partir de una hoja de cálculo

En casos de uso prácticos, las jerarquías de gemelos se crean a menudo a partir de los datos almacenados en una base de datos diferente o quizás en una hoja de cálculo. En esta sección se muestra cómo se puede analizar una hoja de cálculo.

Tenga en cuenta la siguiente tabla de datos, que describe un conjunto de gemelos digitales y las relaciones que se van a crear.

| Modelo    | ID | Parent | Nombre de relación | Otros datos |
| --- | --- | --- | --- | --- |
| floor    | Floor01 | | | … |
| room    | Room10 | Floor01 | contains | … |
| room    | Room11 | Floor01 | contains | … |
| room    | Room12 | Floor01 | contains | … |
| floor    | Floor02 | | | … |
| room    | Room21 | Floor02 | contains | … |
| room    | Room22 | Floor02 | contains | … |

En el código siguiente se usa [Microsoft Graph API](https://docs.microsoft.com/graph/overview) para leer una hoja de cálculo y construir un grafo de gemelos de Azure Digital Twins a partir de los resultados.

```csharp
var range = msftGraphClient.Me.Drive.Items["BuildingsWorkbook"].Workbook.Worksheets["Building"].usedRange;
JsonDocument data = JsonDocument.Parse(range.values);
List<BasicRelationship> RelationshipRecordList = new List<BasicRelationship>();
foreach (JsonElement row in data.RootElement.EnumerateArray())
{
    string type = row[0].GetString();
    string id = row[1].GetString();
    string relSource = row[2].GetString();
    string relName = row[3].GetString();
    // Parse spreadsheet extra data into a JSON string to initialize the digital twin
    // Left out for compactness
    Dictionary<string, object> initData = new Dictionary<string, object>() { ... };

    if (relSource != null)
    {
        BasicRelationship br = new BasicRelationship()
        {
            SourceId = relSource,
            TargetId = id,
            Name = relName
        };
        RelationshipRecordList.Add(br);
    }

    BasicDigitalTwin twin = new BasicDigitalTwin();
    twin.CustomProperties = initData;
    // Set the type of twin to be created
    switch (type)
    {
        case "room":
            twin.Metadata = new DigitalTwinMetadata() { ModelId = "dtmi:com:contoso:Room;2" };
            break;
        case "floor":
            twin.Metadata = new DigitalTwinMetadata() { ModelId = "dtmi:com:contoso:Floor;2" };
            break;
        ... handle additional types
    }
    try
    {
        client.CreateDigitalTwin(id, JsonSerializer.Serialize<BasicDigitalTwin>(twin));
    }
    catch (RequestFailedException e)
    {
        Log.Error($"Error {e.Status}: {e.Message}");
    }
    foreach (BasicRelationship rec in RelationshipRecordList)
    { 
        try { 
            client.CreateRelationship(rec.SourceId, Guid.NewGuid().ToString(), JsonSerializer.Serialize<BasicRelationship>(rec));
        }
        catch (RequestFailedException e)
        {
            Log.Error($"Error {e.Status}: {e.Message}");
        }
    }
}
```
## <a name="manage-relationships-with-cli"></a>Administrar relaciones con la CLI

Los gemelos y sus relaciones también se pueden administrar mediante la CLI de Azure Digital Twins. Los comandos se pueden encontrar en [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo consultar un grafo de gemelos de Azure Digital Twins:
* [*Conceptos: Lenguaje de consulta*](concepts-query-language.md)
* [*Procedimiento: Consulta del grafo gemelo*](how-to-query-graph.md)