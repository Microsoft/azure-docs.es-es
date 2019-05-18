---
title: Creación de una definición de OpenAPI para una función con Azure API Management
description: Cree una definición de OpenAPI que permita que otras aplicaciones y servicios llamen a la función en Azure.
services: functions
keywords: OpenAPI, Swagger, aplicaciones de nube, servicios en la nube,
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.topic: tutorial
ms.date: 05/08/2019
ms.author: glenga
ms.reviewer: sunayv
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: fc724e241849f4519a0e353cb6789d3f83eaf4b9
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510488"
---
# <a name="create-an-openapi-definition-for-a-function-with-azure-api-management"></a>Creación de una definición de OpenAPI para una función con Azure API Management

Las API de REST a menudo se describen con una definición de OpenAPI. Esta definición contiene información sobre las operaciones que están disponibles en una API y cómo se deben estructurar los datos de la solicitud y la respuesta para la API.

En este tutorial se crea una función que determina si la reparación de emergencia de una turbina eólica es rentable. Luego se crea una definición de OpenAPI para la aplicación de función mediante [Azure API Management](../api-management/api-management-key-concepts.md), para poder llamar a la función desde otras aplicaciones y servicios.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una función en Azure
> * Generar una definición de OpenAPI con Azure API Management
> * Probar la definición mediante una llamada a la función
> * Descarga de la definición de OpenAPI

## <a name="create-a-function-app"></a>Creación de una aplicación de función

Debe tener una Function App para hospedar la ejecución de las funciones. Una aplicación de función permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación, el escalado y el uso compartido de recursos.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-the-function"></a>Creación de la función

Este tutorial usa una función desencadenada por HTTP que adopta dos parámetros:

* El tiempo estimado para realizar la reparación de una turbina, en horas.
* La capacidad de la turbina, en kilovatios. 

La función luego calcula cuánto costará la reparación y cuántos ingresos podría generar la turbina en un período de 24 horas. PARA crear la función desencadenada por HTTP en [Azure Portal](https://portal.azure.com).

1. Expanda la instancia de Function App y haga clic en el botón **+**, que se encuentra junto a **Funciones**. Seleccione **En el portal** > **Continuar**.

1. Seleccione **Más plantillas...** y luego **Finalizar y ver plantillas**.

1. Seleccione el desencadenador HTTP, escriba `TurbineRepair` en el campo **Nombre** de la función, elija `Function` en **[Nivel de autenticación](functions-bindings-http-webhook.md#http-auth)** y, a continuación, seleccione **Crear**.  

    ![Creación de una función HTTP para OpenAPI](media/functions-openapi-definition/select-http-trigger-openapi.png)

1. Reemplace el contenido del archivo de script de C# run.csx por el código siguiente y, después, haga clic en **Guardar**:

    ```csharp
    #r "Newtonsoft.Json"
    
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    const double revenuePerkW = 0.12;
    const double technicianCost = 250;
    const double turbineCost = 100;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        // Get query strings if they exist
        int tempVal;
        int? hours = Int32.TryParse(req.Query["hours"], out tempVal) ? tempVal : (int?)null;
        int? capacity = Int32.TryParse(req.Query["capacity"], out tempVal) ? tempVal : (int?)null;
    
        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
    
        // Use request body if a query was not sent
        capacity = capacity ?? data?.capacity;
        hours = hours ?? data?.hours;
    
        // Return bad request if capacity or hours are not passed in
        if (capacity == null || hours == null){
            return new BadRequestObjectResult("Please pass capacity and hours on the query string or in the request body");
        }
        // Formulas to calculate revenue and cost
        double? revenueOpportunity = capacity * revenuePerkW * 24;  
        double? costToFix = (hours * technicianCost) +  turbineCost;
        string repairTurbine;
    
        if (revenueOpportunity > costToFix){
            repairTurbine = "Yes";
        }
        else {
            repairTurbine = "No";
        };
    
        return (ActionResult)new OkObjectResult(new{
            message = repairTurbine,
            revenueOpportunity = "$"+ revenueOpportunity,
            costToFix = "$"+ costToFix
        });
    }
    ```

    Este código de función devuelve un mensaje de `Yes` o `No` para indicar si una reparación de emergencia es rentable, así como la oportunidad de ingresos que representa la turbina, además del costo de repararla.

1. Para probar la función, haga clic en **Probar** a la derecha para expandir la pestaña de pruebas. Escriba el valor siguiente en el **Cuerpo de la solicitud**, y, luego, haga clic en **Ejecutar**.

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

    ![Prueba de la función en Azure Portal](media/functions-openapi-definition/test-function.png)

    El valor siguiente se devuelve en el cuerpo de la respuesta.

    ```json
    {"message":"Yes","revenueOpportunity":"$7200","costToFix":"$1600"}
    ```

Ahora ya existe una función que determina la rentabilidad de las reparaciones de emergencia. A continuación, se genera una definición de OpenAPI para la aplicación de función.

## <a name="generate-the-openapi-definition"></a>Generación de la definición de OpenAPI

Ahora el usuario está listo para generar la definición de OpenAPI.

1. Seleccione la aplicación de función; después, en **Características de la plataforma**, elija **API Management** y seleccione **Crear nuevo** en **API Management**.

    ![Elección de API Management en las características de la plataforma](media/functions-openapi-definition/select-all-settings-openapi.png)

1. Utilice la configuración de API Management que se especifica en la tabla debajo de la imagen.

    ![Creación de un nuevo servicio de API Management](media/functions-openapi-definition/new-apim-service-openapi.png)

    | Configuración      | Valor sugerido  | Descripción                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nombre** | Nombre único globalmente | Se genera un nombre basado en el nombre de la aplicación de función. |
    | **Suscripción** | Su suscripción | Suscripción en la que se crea este nuevo recurso. |  
    | **[Grupo de recursos](../azure-resource-manager/resource-group-overview.md)** |  myResourceGroup | El mismo recurso que la aplicación de función, que debe establecerse automáticamente. |
    | **Ubicación** | Oeste de EE. UU. | Elija la ubicación Oeste de EE. UU. |
    | **Nombre de la organización** | Contoso | El nombre de la organización utilizado en el portal para desarrolladores y para notificaciones de correo electrónico. |
    | **Correo electrónico del administrador** | Su correo electrónico | Correo electrónico que recibe las notificaciones del sistema desde API Management. |
    | **Plan de tarifa** | Consumo (versión preliminar) | El plan de consumo se encuentra en versión preliminar y no está disponible en todas las regiones. Para obtener información completa sobre los precios, vea la [página de precios de API Management](https://azure.microsoft.com/pricing/details/api-management/). |

1. Elija **Crear** para crear la instancia de API Management, que puede tardar varios minutos.

1. Seleccione **Habilitar Application Insights** para enviar registros al mismo lugar que la aplicación de función y, a continuación, acepte los demás valores predeterminados y seleccione **Vincular API**.

1. Se abre **Importar Azure Functions** con la función **TurbineRepair** resaltada. Haga clic en **Seleccionar** para continuar.

    ![Importación de Azure Functions en API Management](media/functions-openapi-definition/import-function-openapi.png)

1. En la página **Crear a partir de Function App**, acepte los valores predeterminados y seleccione **Crear**.

    ![Crear a partir de Function App](media/functions-openapi-definition/create-function-openapi.png)

La API ya se ha creado para la función.

## <a name="test-the-api"></a>Prueba de la API

Antes de usar la definición de OpenAPI, debe comprobar que la API funciona.

1. En la pestaña **Probar** de la función, seleccione la operación **POST**.

1. Escriba los valores de **horas** y **capacidad**.

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

1. Haga clic en **Enviar** y vea la respuesta HTTP.

    ![Prueba de la API de función](media/functions-openapi-definition/test-function-api-openapi.png)

## <a name="download-the-openapi-definition"></a>Descarga de la definición de OpenAPI

Si la API funciona según lo previsto, puede descargar la definición de OpenAPI.

1. Seleccione **Descargar definición de OpenAPI** en la parte superior de la página.
   
   ![Descarga de la definición de OpenAPI](media/functions-openapi-definition/download-definition.png)

2. Abra el archivo JSON descargado y revise la definición.

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

Ha utilizado la integración de API Management para generar una definición de OpenAPI de las funciones. Ahora puede editar la definición de API Management en el portal. También puede obtener [más información acerca de API Management](../api-management/api-management-key-concepts.md).

> [!div class="nextstepaction"]
> [Edición de la definición de OpenAPI en API Management](../api-management/edit-api.md)
