---
title: Gemelos digitales y el grafo de gemelos
titleSuffix: Azure Digital Twins
description: Comprenda el concepto de un gemelo digital y cómo sus relaciones forman un grafo.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 5821a1d1f6713ef39d7475fb004164e7c0fd71ec
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87062056"
---
# <a name="understand-digital-twins-and-their-twin-graph"></a>Explicación del concepto de gemelos digitales y su grafo gemelo

En una solución de Azure Digital Twins, las entidades de su entorno se representan mediante **gemelos digitales** de Azure. Un gemelo digital es una instancia de uno de sus [modelos](concepts-models.md) definidos de forma personalizada. Se puede conectar a otros gemelos digitales a través de **relaciones** para formar una **grafo de gemelos**: este grafo de gemelos es la representación de todo el entorno.

> [!TIP]
> "Azure Digital Twins" hace referencia a este servicio de Azure en conjunto. "Gemelos digitales" o simplemente "gemelos" hace referencia a nodos gemelos individuales dentro de la instancia del servicio.

## <a name="digital-twins"></a>Gemelos digitales

Para poder crear un gemelo digital en la instancia de Azure Digital Twins, debe tener un *modelo* cargado en el servicio. Un modelo describe el conjunto de propiedades, los mensajes de telemetría y las relaciones que puede tener un gemelo determinado, entre otras cosas. Para conocer los tipos de información que se definen en un modelo, consulte [*Conceptos: Modelos personalizados*](concepts-models.md).

Después de crear y cargar un modelo, la aplicación cliente puede crear una instancia del tipo, que es un gemelo digital. Por ejemplo, después de crear un modelo de *Floor*, puede crear uno o varios gemelos digitales que usen este tipo (como un gemelo de tipo *Floor* denominado *GroundFloor*, otro denominado *Floor2*, etc.). 

## <a name="relationships-a-graph-of-digital-twins"></a>Relaciones: un grafo de gemelos digitales

Los gemelos se conectan a un grafo de gemelos a través de sus relaciones. Las relaciones que un gemelo puede tener se definen como parte de su modelo.  

Por ejemplo, el modelo *Floor* podría definir una relación *contains* dirigida a gemelos de tipo *Room*. Con esta definición, Azure Digital Twins le permitirá crear relaciones *contains* desde cualquier gemelo *Floor* en cualquier gemelo *Room* (incluidos los gemelos que son de subtipos *Room*). 

El resultado de este proceso es un conjunto de nodos (los gemelos digitales) conectado a través de bordes (sus relaciones) en un grafo.

[!INCLUDE [visualizing with Azure Digital Twins explorer](../../includes/digital-twins-visualization.md)]

## <a name="create-with-the-apis"></a>Creación con las API

En esta sección se muestra en qué se basa la creación de gemelos digitales y relaciones desde una aplicación cliente. Contiene ejemplos de código de .NET que usan las [API de DigitalTwins](how-to-use-apis-sdks.md), para proporcionar contexto adicional sobre lo que sucede dentro de cada uno de estos conceptos.

### <a name="create-digital-twins"></a>Creación de gemelos digitales

A continuación se muestra un fragmento de código de cliente que usa las [API de DigitalTwins](how-to-use-apis-sdks.md) para crear una instancia de un gemelo de tipo *Room*.

En la versión preliminar actual de Azure Digital Twins, todas las propiedades de un gemelo se deben inicializar para poder crear el gemelo. Esto se hace mediante la creación de un documento JSON que proporcione los valores de inicialización necesarios.

```csharp
public Task<boolean> CreateRoom(string id, double temperature, double humidity) 
{
    // Define the model for the twin to be created
    Dictionary<string, object> meta = new Dictionary<string, object>()
    {
      { "$model", "dtmi:com:contoso:Room;2" }
    };
    // Initialize the twin properties
    Dictionary<string, object> initData = new Dictionary<string, object>()
    {
      { "$metadata", meta },
      { "Temperature", temperature},
      { "Humidity", humidity},
    };
    try
    {
      await client.DigitalTwins.AddAsync(id, initData);
      return true;
    }
    catch (ErrorResponseException e)
    {
      Console.WriteLine($"*** Error creating twin {id}: {e.Response.StatusCode}");
      return false;
    }
}
```

### <a name="create-relationships"></a>Crear relaciones

A continuación se incluye un ejemplo de código de cliente que usa las [API de DigitalTwins](how-to-use-apis-sdks.md) para crear una relación entre un gemelo digital de tipo *Floor* denominado *GroundFloor* y un gemelo digital de tipo *Room* denominado *Cafe*.

```csharp
// Create Twins, using functions similar to the previous sample
await CreateRoom("Cafe", 70, 66);
await CreateFloor("GroundFloor", averageTemperature=70);
// Create relationships
Dictionary<string, object> targetrec = new Dictionary<string, object>()
{
    { "$targetId", "Cafe" }
};
try
{
    await client.DigitalTwins.AddEdgeAsync("GroundFloor", "contains", "GF-to-Cafe", targetrec);
} catch(ErrorResponseException e)
{
    Console.WriteLine($"*** Error creating relationship: {e.Response.StatusCode}");
}
```

## <a name="json-representations-of-graph-elements"></a>Representaciones JSON de elementos de grafo

Los datos de gemelos digitales y los datos de relaciones se almacenan en formato JSON. Esto significa que, cuando [consulte el grafo de gemelos](how-to-query-graph.md) en la instancia de Azure Digital Twins, el resultado será una representación JSON de los gemelos digitales y las relaciones que ha creado.

### <a name="digital-twin-json-format"></a>Formato JSON de gemelo digital

Si se representa como un objeto JSON, un gemelo digital mostrará los campos siguientes:

| Nombre del campo | Descripción |
| --- | --- |
| `$dtId` | Cadena proporcionada por el usuario que representa el identificador del gemelo digital. |
| `$etag` | Campo HTTP estándar asignado por el servidor web. |
| `$conformance` | Enumeración que contiene el estado de conformidad de este gemelo digital (*conforme*, *no conforme* o *desconocido*) |
| `{propertyName}` | Valor de una propiedad en formato JSON (`string`, tipo de número u objeto). |
| `$relationships` | Dirección URL de la ruta de acceso a la colección de relaciones. Este campo no está presente si el gemelo digital no tiene bordes de relación salientes. |
| `$metadata.$model` | [Opcional] Identificador de la interfaz del modelo que caracteriza al gemelo digital. |
| `$metadata.{propertyName}.desiredValue` | [Solo para propiedades grabables] Valor deseado de la propiedad especificada. |
| `$metadata.{propertyName}.desiredVersion` | [Solo para propiedades grabables] Versión del valor deseado. |
| `$metadata.{propertyName}.ackVersion` | Versión confirmada por la aplicación del dispositivo que implementa el gemelo digital. |
| `$metadata.{propertyName}.ackCode` | [Solo para propiedades grabables] Código `ack` devuelto por la aplicación del dispositivo que implementa el gemelo digital. |
| `$metadata.{propertyName}.ackDescription` | [Solo para propiedades grabables] Descripción `ack` devuelta por la aplicación del dispositivo que implementa el gemelo digital. |
| `{componentName}` | Objeto JSON que contiene los valores de propiedad y los metadatos del componente, similares a los del objeto raíz. Este objeto existe aunque el componente no tenga propiedades. |
| `{componentName}.{propertyName}` | Valor de la propiedad del componente en formato JSON (`string`, tipo de número u objeto). |
| `{componentName}.$metadata` | Información de metadatos del componente, similar al nivel de raíz `$metadata`. |

A continuación se muestra un ejemplo de gemelo digital formateado como un objeto JSON:

```json
{
  "$dtId": "Cafe",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "Temperature": 72,
  "Location": {
    "x": 101,
    "y": 33
  },
  "component": {
    "TableOccupancy": 1,
    "$metadata": {
      "TableOccupancy": {
        "desiredValue": 1,
        "desiredVersion": 3,
        "ackVersion": 2,
        "ackCode": 200,
        "ackDescription": "OK"
      }
    }
  },
  "$metadata": {
    "$model": "dtmi:com:contoso:Room;1",
    "Temperature": {
      "desiredValue": 72,
      "desiredVersion": 5,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "OK"
    },
    "Location": {
      "desiredValue": {
        "x": 101,
        "y": 33,
      },
      "desiredVersion": 8,
      "ackVersion": 8,
      "ackCode": 200,
      "ackDescription": "OK"
    }
  }
}
```

### <a name="relationship-json-format"></a>Formato JSON de relación

Si se representa como un objeto JSON, una relación de un gemelo digital mostrará los campos siguientes:

| Nombre del campo | Descripción |
| --- | --- |
| `$relationshipId` | Cadena proporcionada por el usuario que representa el identificador de esta relación. Esta cadena es única en el contexto del gemelo digital de origen, lo que también significa que `sourceId` + `relationshipId` es único en el contexto de la instancia de Azure Digital Twins. |
| `$etag` | Campo HTTP estándar asignado por el servidor web. |
| `$sourceId` | Identificador del gemelo digital de origen. |
| `$targetId` | Identificador del gemelo digital de destino. |
| `$relationshipName` | Nombre de la relación. |
| `{propertyName}` | [Opcional] Valor de una propiedad de esta relación en formato JSON (`string`, tipo de número u objeto). |

A continuación se muestra un ejemplo de relación formateada como un objeto JSON:

```json
{
  "$relationshipId": "relationship-01",
  "$etag": "W/\"506e8391-2b21-4ac9-bca3-53e6620f6a90\"",
  "$sourceId": "GroundFloor",
  "$targetId": "Cafe",
  "$relationshipName": "contains",
  "startDate": "2020-02-04"
}
```

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo administrar elementos de grafo con las API de Azure Digital Twins:
* [*Procedimiento: Administración de Digital Twins*](how-to-manage-twin.md)
* [*Procedimiento: Administración del grafo de gemelos con relaciones*](how-to-manage-graph.md)

O bien, consulte el grafo de gemelos de Azure Digital Twins para obtener información:
* [*Conceptos: Lenguaje de consulta*](concepts-query-language.md)