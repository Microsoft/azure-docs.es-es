---
title: Inicio rápido de Azure App Configuration con Azure Functions | Microsoft Docs
description: Inicio rápido para el uso de Azure App Configuration con Azure Functions.
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: quickstart
ms.tgt_pltfrm: Azure Functions
ms.workload: tbd
ms.date: 02/24/2019
ms.author: yegu
ms.openlocfilehash: a0c677e25be86e0f82b821a7931bda0af2b35127
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408654"
---
# <a name="quickstart-create-an-azure-function-with-app-configuration"></a>Inicio rápido: Creación de una función de Azure con App Configuration

Azure App Configuration es un servicio de configuración administrado de Azure. Puede usarlo para almacenar y administrar fácilmente toda la configuración de una aplicación en un solo lugar que esté separado del código. En este inicio rápido se muestra cómo incorporar el servicio a una función de Azure. 

Para realizar los pasos de este inicio rápido, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

![Inicio rápido completo local](./media/quickstarts/dotnet-core-function-launch-local.png)

## <a name="prerequisites"></a>Requisitos previos

Para realizar este inicio rápido, instale [Visual Studio 2017](https://visualstudio.microsoft.com/vs). Asegúrese de que la carga de trabajo de **Desarrollo de Azure** también está instalada. Instale también las [herramientas de Azure Functions más recientes](../azure-functions/functions-develop-vs.md#check-your-tools-version).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de configuración de aplicaciones

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Explorador de configuración** > **+ Crear** para agregar los siguientes pares clave-valor:

    | Clave | Valor |
    |---|---|
    | TestApp:Settings:Message | Datos de Azure App Configuration |

    Deje **Etiqueta** y **Tipo de contenido** en blanco, por ahora.

## <a name="create-a-function-app"></a>Creación de una aplicación de función

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de configuración de aplicaciones

1. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**. En la pestaña **Examinar**, busque y agregue los siguientes paquetes NuGet al proyecto. Si no los encuentra, seleccione la casilla **Incluir versión preliminar**.

    ```
    Microsoft.Extensions.Configuration.AzureAppConfiguration 1.0.0 preview or later
    ```

2. Abra *Function1.cs* y agregue una referencia al proveedor de configuración de la aplicación .NET Core.

    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

3. Actualice el método `Run` para usar App Configuration mediante la llamada a `builder.AddAzureAppConfiguration()`.

    ```csharp
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));
        var config = builder.Build();
        string message = config["TestApp:Settings:Message"];
        message = message ?? req.Query["message"];

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        message = message ?? data?.message;

        return message != null
            ? (ActionResult) new OkObjectResult(message)
            : new BadRequestObjectResult("Please pass a message from a configuration store, on the query string or in the request body");
    }
    ```

## <a name="test-the-function-locally"></a>Prueba local de la función

1. Establezca una variable de entorno llamada **ConnectionString** y defínala como la clave de acceso a su almacén de configuración de aplicaciones. Si usa el símbolo del sistema de Windows, ejecute el siguiente comando y reinícielo para que se aplique el cambio:

        setx ConnectionString "connection-string-of-your-app-configuration-store"

    Si usa Windows PowerShell, ejecute el siguiente comando:

        $Env:ConnectionString = "connection-string-of-your-app-configuration-store"

    Si usa macOS o Linux, ejecute el siguiente comando:

        export ConnectionString='connection-string-of-your-app-configuration-store'

2. Para probar la función, presione F5. Si se le solicita, acepte la solicitud de Visual Studio para descargar e instalar las herramientas de **Azure Functions Core (CLI)**. También es preciso que habilite una excepción de firewall para que las herramientas para controlen las solicitudes de HTTP.

3. Copie la dirección URL de la función de los resultados del runtime de Azure Functions.

    ![Inicio rápido: depuración de funciones en VS](./media/quickstarts/function-visual-studio-debugging.png)

4. Pegue la dirección URL de la solicitud HTTP en la barra de direcciones del explorador. La siguiente imagen muestra la respuesta en el explorador para la solicitud GET local devuelta por la función.

    ![Inicio rápido: inicio de funciones locales](./media/quickstarts/dotnet-core-function-launch-local.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un nuevo almacén de configuración de aplicaciones y lo ha usado con una función de Azure. Para más información acerca del uso de App Configuration, continúe con el siguiente tutorial, ya que en él se muestra cómo realizar la autenticación.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
