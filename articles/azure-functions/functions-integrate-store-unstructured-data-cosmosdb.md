---
title: Almacenamiento de datos no estructurados mediante Azure Cosmos DB y Functions
description: Almacenamiento de datos no estructurados mediante Azure Functions y Cosmos DB
ms.topic: quickstart
ms.date: 04/14/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: a242f1ffc0a7738ce7cdf33aeeef214fb4f63e61
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88210882"
---
# <a name="store-unstructured-data-using-azure-functions-and-azure-cosmos-db"></a>Almacenamiento de datos no estructurados mediante Azure Functions y Cosmos DB

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) es una excelente manera de almacenar datos no estructurados y JSON. En combinación con Azure Functions, Cosmos DB facilita y agiliza el almacenamiento de datos con mucho menos código que el necesario para almacenar datos en una base de datos relacional.

> [!NOTE]
> En este momento, el desencadenador de Azure Cosmos DB y los enlaces de entrada y de salida solo funcionan con cuentas de SQL API y Graph API.

En Azure Functions, los enlaces de entrada y salida proporcionan una manera declarativa de conectarse a los datos de servicio externos desde su función. En este artículo, aprenda a actualizar una función existente para agregar un enlace de salida que almacene datos no estructurados en un documento de Azure Cosmos DB.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial:

[!INCLUDE [Previous quickstart note](../../includes/functions-quickstart-previous-topics.md)]

## <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Debe tener una cuenta de Azure Cosmos DB que use la API de SQL para poder crear el enlace de salida.

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-an-output-binding"></a>Adición de un enlace de salida

1. En Azure Portal, vaya a la aplicación de funciones que se ha creado anteriormente y selecciónela.

1. Seleccione **Funciones** y, después, seleccione la función HttpTrigger.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-select-http-function.png" alt-text="Seleccione su función Http en Azure Portal." border="true":::

1. Seleccione **Integración** y **+ Agregar salida**.

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-add-output-binding.png" alt-text="Adición de un enlace de salida de Azure Cosmos DB" border="true":::.

1. Use la configuración de **Crear salida**, como se especifica en la tabla:

     :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-configure-cosmosdb-binding.png" alt-text="Configuración del enlace de salida de Azure Cosmos DB" border="true":::.

    | Configuración      | Valor sugerido  | Descripción                                |
    | ------------ | ---------------- | ------------------------------------------ |
    | **Tipo de enlace** | Azure Cosmos DB | Nombre del tipo de enlace que se selecciona para crear el enlace de salida a Azure Cosmos DB. |
    | **Nombre del parámetro de documento** | taskDocument | Nombre que hace referencia al objeto de Cosmos DB en el código. |
    | **Nombre de la base de datos** | taskDatabase | Nombre de la base de datos para guardar documentos. |
    | **Nombre de colección** | taskCollection | Nombre de la colección de la base de datos. |
    | **Si es true, crea la base de datos y la colección de Cosmos DB** | Sí | La colección no existe, por lo que se crea. |
    | **Conexión de la cuenta de Cosmos DB** | Nueva configuración | Seleccione **Nuevo**, luego, **Cuenta de Azure Cosmos DB** y la **cuenta de la base de datos** que creó anteriormente y, después, seleccione **Aceptar**. Con esto se crea una configuración de la aplicación para la conexión de la cuenta. Esta configuración la usa el enlace para establecer la conexión a la base de datos. |

1. Seleccione **Aceptar** para crear el enlace.

## <a name="update-the-function-code"></a>Actualización del código de la función

Reemplace el código existente de la función por el código siguiente, en el lenguaje de su elección:

# <a name="c"></a>[C#](#tab/csharp)

Reemplace la función de C# existente por el código siguiente:

```csharp
#r "Newtonsoft.Json"

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

public static IActionResult Run(HttpRequest req, out object taskDocument, ILogger log)
{
    string name = req.Query["name"];
    string task = req.Query["task"];
    string duedate = req.Query["duedate"];

    // We need both name and task parameters.
    if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(task))
    {
        taskDocument = new
        {
            name,
            duedate,
            task
        };

        return (ActionResult)new OkResult();
    }
    else
    {
        taskDocument = null;
        return (ActionResult)new BadRequestResult();
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Reemplace la función de JavaScript existente por el código siguiente:

```js
module.exports = async function (context, req) {

    // We need both name and task parameters.
    if (req.query.name && req.query.task) {

        // Set the output binding data from the query object.
        context.bindings.taskDocument = req.query;

        // Success.
        context.res = {
            status: 200
        };
    }
    else {
        context.res = {
            status: 400,
            body: "The query options 'name' and 'task' are required."
        };
    }
};
```
---

Este ejemplo de código lee las cadenas de consulta de la solicitud HTTP y las asigna a los campos del objeto `taskDocument`. El enlace `taskDocument` envía los datos del objeto desde este parámetro de enlace para almacenarlos en la base de datos de documentos enlazada. La base de datos se crea la primera vez que se ejecuta la función.

## <a name="test-the-function-and-database"></a>Prueba de la función y la base de datos

1. Seleccione **Probar**. En **Consulta**, seleccione **+ Agregar parámetro** y agregue los siguientes parámetros a la cadena de consulta:

    + `name`
    + `task`
    + `duedate`

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function.png" alt-text="Probar la función." border="true":::


1. Seleccione **Ejecutar** y compruebe que se devuelve un estado 200.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-test-function-output.png" alt-text="Probar la función." border="true":::


1. En Azure Portal, busque y seleccione **Azure Cosmos DB**.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-search-cosmos-db.png" alt-text="Búsqueda del servicio Cosmos DB" border="true":::.

1. Elija la cuenta de Azure Cosmos DB y luego seleccione **Explorador de datos**.

1. Expanda los nodos de **TaskCollection**, seleccione el nuevo documento y confirme que el documento contiene los valores de la cadena de consulta, junto con metadatos adicionales.

    :::image type="content" source="./media/functions-integrate-store-unstructured-data-cosmosdb/functions-data-explorer-check-document.png" alt-text="Verificar los valores de cadena del documento." border="true":::

Ha agregado correctamente un enlace al desencadenador HTTP que almacena datos no estructurados en Azure Cosmos DB.

[!INCLUDE [Clean-up section](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el enlace a una base de datos de la base de datos de Cosmos, consulte [Enlaces de Cosmos DB en Azure Functions](functions-bindings-cosmosdb.md).

[!INCLUDE [functions-quickstart-next-steps](../../includes/functions-quickstart-next-steps-2.md)]
