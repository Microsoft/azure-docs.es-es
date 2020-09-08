---
title: Administración de Digital Twins
titleSuffix: Azure Digital Twins
description: Vea cómo recuperar, actualizar y eliminar relaciones y gemelos individuales.
author: baanders
ms.author: baanders
ms.date: 4/10/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: b88d017110f4d7b9859f2d801c5405ecee1589c5
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297474"
---
# <a name="manage-digital-twins"></a>Administración de Digital Twins

Las entidades de su entorno se representan mediante [gemelos digitales](concepts-twins-graph.md). La administración de los gemelos digitales puede incluir las operaciones de creación, modificación y eliminación. Para realizar estas operaciones, puede usar las [**API de DigitalTwins**](how-to-use-apis-sdks.md), el [SDK de .NET ( C# )](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) o la [CLI de Azure Digital Twins](how-to-use-cli.md).

Este artículo se centra en la administración de gemelos digitales; para trabajar con relaciones y con el [grafo de gemelos](concepts-twins-graph.md) en conjunto, consulte [*Procedimiento: Administración del grafo de gemelos con relaciones*](how-to-manage-graph.md).

> [!TIP]
> Todas las funciones del SDK cuentan con versiones sincrónicas y asincrónicas.

## <a name="create-a-digital-twin"></a>Creación de un gemelo digital

Para crear un gemelo, use el método `CreateDigitalTwin` en el cliente de servicio de la siguiente manera:

```csharp
await client.CreateDigitalTwinAsync("myNewTwinID", initData);
```

Para crear un gemelo digital, debe proporcionar lo siguiente:
* El identificador deseado para el gemelo digital.
* El [modelo](concepts-models.md) que quiere usar. 

Opcionalmente, puede proporcionar los valores iniciales de todas las propiedades del gemelo digital. 

Los valores de las propiedades del modelo e iniciales se proporcionan a través del parámetro `initData`, que es una cadena JSON que contiene los datos pertinentes. Para más información sobre cómo estructurar este objeto, vaya a la sección siguiente.

> [!TIP]
> Después de crear o actualizar un gemelo, puede haber una latencia de hasta 10 segundos antes de que los cambios se reflejen en las [consultas](how-to-query-graph.md). La API de `GetDigitalTwin` (que se describe [más adelante en este artículo](#get-data-for-a-digital-twin)) no experimenta este retraso, por lo que debe usar la llamada API en lugar de realizar una consulta para ver los gemelos recién creados si necesita una respuesta instantánea. 

### <a name="initialize-model-and-properties"></a>Inicialización del modelo y las propiedades

La API de creación de gemelos acepta un objeto serializado en una descripción JSON válida de las propiedades gemelas. Consulte [*Conceptos: Gemelos digitales y el grafo de gemelos*](concepts-twins-graph.md) para obtener una descripción del formato JSON de un gemelo. 

Primero, creará un objeto de datos para representar el gemelo y sus datos de propiedad. A continuación, puede usar `JsonSerializer` para pasar una versión serializada de este a la llamada API para el parámetro `initdata`.

Puede crear un objeto de parámetro manualmente o mediante una clase auxiliar proporcionada. A continuación se muestra un ejemplo de cada opción.

#### <a name="create-twins-using-manually-created-data"></a>Creación de gemelos con datos creados manualmente

Sin el uso de ninguna clase auxiliar personalizada, puede representar las propiedades de un gemelo en un elemento `Dictionary<string, object>`, donde `string` es el nombre de la propiedad y `object` es un objeto que representa la propiedad y su valor.

```csharp
// Define the model type for the twin to be created
Dictionary<string, object> meta = new Dictionary<string, object>()
{
    { "$model", "dtmi:com:contoso:Room;1" }
};
// Initialize the twin properties
Dictionary<string, object> twin = new Dictionary<string, object>()
{
    { "$metadata", meta },
    { "Temperature", temperature},
    { "Humidity", humidity},
};
client.CreateDigitalTwin("myNewRoomID", JsonSerializer.Serialize<Dictionary<string, object>>(twin));
```

#### <a name="create-twins-with-the-helper-class"></a>Creación de gemelos con la clase auxiliar

La clase auxiliar `BasicDigitalTwin` permite almacenar los campos de propiedades en un objeto "gemelo" más directamente. Aún puede crear la lista de propiedades mediante un elemento `Dictionary<string, object>`, que luego se puede agregar al objeto gemelo como su `CustomProperties` directamente.

```csharp
BasicDigitalTwin twin = new BasicDigitalTwin();
twin.Metadata = new DigitalTwinMetadata();
twin.Metadata.ModelId = "dtmi:example:Room;1";
// Initialize properties
Dictionary<string, object> props = new Dictionary<string, object>();
props.Add("Temperature", 25.0);
props.Add("Humidity", 50.0);
twin.CustomProperties = props;

client.CreateDigitalTwin("myNewRoomID", JsonSerializer.Serialize<BasicDigitalTwin>(twin));
```

>[!NOTE]
> Los objetos `BasicDigitalTwin` contienen un campo `Id`. Puede dejar este campo vacío, pero si agrega un valor de id., debe coincidir con el parámetro de id. pasado a la llamada `CreateDigitalTwin`. Para el ejemplo anterior, sería:
>
>```csharp
>twin.Id = "myNewRoomID";
>```

## <a name="get-data-for-a-digital-twin"></a>Obtención de datos para un gemelo digital

Puede acceder a los datos completos de cualquier gemelo digital mediante una llamada a:

```csharp
object result = await client.GetDigitalTwin(id);
```

Esta llamada devuelve los datos de gemelos como una cadena JSON. 

Solo se devuelven las propiedades que se han establecido al menos una vez cuando se recupera un gemelo con `GetDigitalTwin`.

>[!TIP]
>El `displayName` para un gemelo es parte de sus metadatos del modelo, por lo que no se mostrará al obtener los datos de la instancia gemela. Para ver este valor, puede [recuperarlo del modelo](how-to-manage-model.md#retrieve-models).

Para recuperar varios gemelos mediante una única llamada API, consulte los ejemplos de la API de consulta en [*Procedimiento: Consulta del grafo de gemelos*](how-to-query-graph.md).

Tenga en cuenta el siguiente modelo, escrito en el [lenguaje de definición de gemelos digitales (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/tree/master/DTDL), que define un objeto *Moon*:

```json
{
    "@id": " dtmi:com:contoso:Moon;1",
    "@type": "Interface",
    "@context": "dtmi:dtdl:context;2",
    "contents": [
        {
            "@type": "Property",
            "name": "radius",
            "schema": "double",
            "writable": true
        },
        {
            "@type": "Property",
            "name": "mass",
            "schema": "double",
            "writable": true
        }
    ]
}
```

El resultado de llamar a `object result = await client.DigitalTwins.GetByIdAsync("my-moon");` en un gemelo de tipo *Moon* puede ser similar al siguiente:

```json
{
  "$dtId": "myMoon-001",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "radius": 1737.1,
  "mass": 0.0734,
  "$metadata": {
    "$model": "dtmi:com:contoso:Moon;1",
    "radius": {
      "desiredValue": 1737.1,
      "desiredVersion": 5,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "OK"
    },
    "mass": {
      "desiredValue": 0.0734,
      "desiredVersion": 8,
      "ackVersion": 8,
      "ackCode": 200,
      "ackDescription": "OK"
    }
  }
}
```

Las propiedades definidas del gemelo digital se devuelven como propiedades de nivel superior en el gemelo digital. Los metadatos o la información del sistema que no forman parte de la definición de DTDL se devuelven con un prefijo `$`. Las propiedades de metadatos incluyen:
* El identificador del gemelo digital en esta instancia de Azure Digital Twins, como `$dtId`.
* `$etag`, un campo HTTP estándar asignado por el servidor web.
* Otras propiedades de una sección `$metadata`. Entre ellas se incluyen las siguientes:
    - DTMI del modelo del gemelo digital.
    - Estado de sincronización de cada propiedad grabable. Esto es útil principalmente para los dispositivos, donde es posible que el servicio y el dispositivo tengan estados divergentes (por ejemplo, cuando un dispositivo está sin conexión). Actualmente, esta propiedad solo se aplica a dispositivos físicos conectados a IoT Hub. Los datos de la sección de metadatos permiten comprender el estado completo de una propiedad, así como las últimas marcas de tiempo modificadas. Para obtener más información sobre el estado de sincronización, consulte [este tutorial de IoT Hub](../iot-hub/tutorial-device-twins.md) sobre la sincronización del estado del dispositivo.
    - Metadatos específicos del servicio, como de IoT Hub o Azure Digital Twins. 

Puede analizar el JSON devuelto para el gemelo mediante una biblioteca de análisis de JSON de su elección, como `System.Text.Json`.

También puede usar la clase auxiliar de serialización `BasicDigitalTwin` que se incluye con el SDK, que devolverá los metadatos y las propiedades de gemelos principales en formato analizado previamente. Este es un ejemplo:

```csharp
Response<string> res = client.GetDigitalTwin(twin_id);
BasicDigitalTwin twin = JsonSerializer.Deserialize<BasicDigitalTwin>(res.Value);
Console.WriteLine($"Model id: {twin.Metadata.ModelId}");
foreach (string prop in twin.CustomProperties.Keys)
{
    if (twin.CustomProperties.TryGetValue(prop, out object value))
        Console.WriteLine($"Property '{prop}': {value}");
}
```

Puede obtener más información sobre las clases auxiliares de serialización en [*Procedimiento: las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md).

## <a name="update-a-digital-twin"></a>Actualización de un gemelo digital

Para actualizar las propiedades de un gemelo digital, escriba la información que quiere reemplazar en formato de [revisión de JSON](http://jsonpatch.com/). De esta manera, puede reemplazar varias propiedades a la vez. A continuación, pase el documento de revisión de JSON a un método `Update`:

```csharp
await client.UpdateDigitalTwin(id, patch);
```

Una llamada de revisión puede actualizar tantas propiedades de un solo gemelo como se quiera (incluso todas ellas). Si necesita actualizar las propiedades en varios gemelos, necesitará una llamada de actualización independiente para cada uno.

> [!TIP]
> Después de crear o actualizar un gemelo, puede haber una latencia de hasta 10 segundos antes de que los cambios se reflejen en las [consultas](how-to-query-graph.md). La API de `GetDigitalTwin` (que se describe [anteriormente en este artículo](#get-data-for-a-digital-twin)) no experimenta este retraso, por lo que debe usar la llamada API en lugar de realizar una consulta para ver los gemelos recién actualizados si necesita una respuesta instantánea. 

Este es un ejemplo de código de revisión de JSON. En este documento se reemplazan los valores de las propiedades *mass* y *radius* del gemelo digital al que se aplica.

```json
[
  {
    "op": "replace",
    "path": "/mass",
    "value": 0.0799
  },
  {
    "op": "replace",
    "path": "/radius",
    "value": 0.800
  }
]
```

Puede crear revisiones manualmente o mediante una clase auxiliar de serialización en el [SDK](how-to-use-apis-sdks.md). A continuación se muestra un ejemplo de cada opción.

#### <a name="create-patches-manually"></a>Creación manual de revisiones

```csharp
List<object> twinData = new List<object>();
twinData.Add(new Dictionary<string, object>() {
    { "op", "add"},
    { "path", "/Temperature"},
    { "value", 25.0}
});

await client.UpdateDigitalTwinAsync(twinId, JsonConvert.SerializeObject(twinData));
```

#### <a name="create-patches-using-the-helper-class"></a>Creación de revisiones con la clase auxiliar

```csharp
UpdateOperationsUtility uou = new UpdateOperationsUtility();
uou.AppendAddOp("/Temperature", 25.0);
await client.UpdateDigitalTwinAsync(twinId, uou.Serialize());
```

### <a name="update-properties-in-digital-twin-components"></a>Actualización de propiedades en componentes de gemelos digitales

Recuerde que un modelo puede contener componentes, lo que permite que esté formado por otros modelos. 

Para revisar las propiedades de los componentes de un gemelo digital, usará la sintaxis de la ruta de acceso en la revisión de JSON:

```json
[
  {
    "op": "replace",
    "path": "/mycomponentname/mass",
    "value": 0.0799
  }
]
```

### <a name="update-a-digital-twins-model"></a>Actualización de un modelo de gemelo digital

La función `Update` también puede usarse para migrar un gemelo digital a un modelo diferente. 

Por ejemplo, tenga en cuenta el siguiente documento de revisión de JSON que reemplaza el campo `$model` de los metadatos del gemelo digital:

```json
[
  {
    "op": "replace",
    "path": "/$metadata/$model",
    "value": "dtmi:com:contoso:foo;1"
  }
]
```

Esta operación solo se realizará correctamente si el gemelo digital que la revisión está modificando se ajusta al nuevo modelo. 

Considere el ejemplo siguiente:
1. Imagine un gemelo digital con un modelo de *foo_old*. *foo_old* define una propiedad *mass* necesaria.
2. El nuevo modelo *foo_new* define una propiedad mass y agrega una nueva propiedad *temperature* necesaria.
3. Después de la revisión, el gemelo digital debe tener ambas propiedades mass y temperature. 

La revisión para esta situación debe actualizar tanto el modelo como la propiedad temperature del gemelo, de la siguiente manera:

```json
[
  {
    "op": "replace",
    "path": "$metadata.$model",
    "value": "dtmi:com:contoso:foo_new"
  },
  {
    "op": "add",
    "path": "temperature",
    "value": 60
  }
]
```

### <a name="handle-conflicting-update-calls"></a>Control de las llamadas de actualización en conflicto

Azure Digital Twins garantiza que todas las solicitudes entrantes se procesan una tras otra. Esto significa que, incluso para el caso de que varias funciones intenten actualizar la misma propiedad en un gemelo al mismo tiempo, **no es necesario** que escriba código de bloqueo explícito para controlar el conflicto.

Este comportamiento se basa en cada gemelo. 

Por ejemplo, imagine un escenario en el que llegan estas tres llamadas al mismo tiempo: 
*   Escribir propiedad A en *Gemelo1*
*   Escribir propiedad B en *Gemelo1*
*   Escribir propiedad A en *Gemelo2*

Las dos llamadas que modifican *Gemelo1* se ejecutan una tras otra, y se generan mensajes para cada cambio. La llamada para modificar *Gemelo2* puede ejecutarse simultáneamente sin ningún conflicto, en cuanto llega.

## <a name="delete-a-digital-twin"></a>Eliminación de un gemelo digital

Puede eliminar gemelos con `DeleteDigitalTwin(ID)`. Sin embargo, solo puede eliminar un gemelo si este no tiene más relaciones. Primero debe eliminar todas las relaciones. 

A continuación se muestra un ejemplo del código para realizar esta operación:

```csharp
static async Task DeleteTwin(string id)
{
    await FindAndDeleteOutgoingRelationshipsAsync(id);
    await FindAndDeleteIncomingRelationshipsAsync(id);
    try
    {
        await client.DeleteDigitalTwin(id);
    } catch (RequestFailedException exc)
    {
        Console.WriteLine($"*** Error:{exc.Error}/{exc.Message}");
    }
}

public async Task FindAndDeleteOutgoingRelationshipsAsync(string dtId)
{
    // Find the relationships for the twin

    try
    {
        // GetRelationshipsAsync will throw an error if a problem occurs
        AsyncPageable<string> relsJson = client.GetRelationshipsAsync(dtId);

        await foreach (string relJson in relsJson)
        {
            var rel = System.Text.Json.JsonSerializer.Deserialize<BasicRelationship>(relJson);
            await client.DeleteRelationshipAsync(dtId, rel.Id).ConfigureAwait(false);
            Log.Ok($"Deleted relationship {rel.Id} from {dtId}");
        }
    }
    catch (RequestFailedException ex)
    {
        Log.Error($"*** Error {ex.Status}/{ex.ErrorCode} retrieving or deleting relationships for {dtId} due to {ex.Message}");
    }
}

async Task FindAndDeleteIncomingRelationshipsAsync(string dtId)
{
    // Find the relationships for the twin

    try
    {
        // GetRelationshipssAsync will throw an error if a problem occurs
        AsyncPageable<IncomingRelationship> incomingRels = client.GetIncomingRelationshipsAsync(dtId);

        await foreach (IncomingRelationship incomingRel in incomingRels)
        {
            await client.DeleteRelationshipAsync(incomingRel.SourceId, incomingRel.RelationshipId).ConfigureAwait(false);
            Log.Ok($"Deleted incoming relationship {incomingRel.RelationshipId} from {dtId}");
        }
    }
    catch (RequestFailedException ex)
    {
        Log.Error($"*** Error {ex.Status}/{ex.ErrorCode} retrieving or deleting incoming relationships for {dtId} due to {ex.Message}");
    }
}
```

### <a name="delete-all-digital-twins"></a>Eliminación de todos los gemelos digitales

Para obtener un ejemplo de cómo eliminar todos los gemelos a la vez, descargue la aplicación de ejemplo que se usa en el [*Tutorial: Exploración de los conceptos básicos con una aplicación cliente de ejemplo*](tutorial-command-line-app.md). El archivo *CommandLoop.cs* realiza esta acción en una función `CommandDeleteAllTwins`.

## <a name="manage-twins-with-cli"></a>Administración de gemelos con la CLI

Los gemelos también se pueden administrar con la CLI de Azure Digital Twins. Los comandos se pueden encontrar en [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo crear y administrar relaciones entre los gemelos digitales:
* [*Procedimiento: Administración del grafo de gemelos con relaciones*](how-to-manage-graph.md)