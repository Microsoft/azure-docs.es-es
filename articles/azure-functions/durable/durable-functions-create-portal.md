---
title: Creación de Durable Functions mediante Azure Portal
description: Obtenga información sobre cómo instalar la extensión Durable Functions para Azure Functions para el desarrollo del portal.
ms.topic: conceptual
ms.date: 04/10/2020
ms.reviewer: azfuncdf
ms.openlocfilehash: b029fa246977dfe4210f6e8df242415f7e4103f1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081923"
---
# <a name="create-durable-functions-using-the-azure-portal"></a>Creación de Durable Functions mediante Azure Portal

La extensión [Durable Functions](durable-functions-overview.md) para Azure Functions se proporciona en el paquete NuGet [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask). Esta extensión debe instalarse en la aplicación de función. En este artículo se muestra cómo instalar este paquete para que pueda desarrollar funciones duraderas en Azure Portal.

> [!NOTE]
> 
> * Si está desarrollando Durable Functions en C#, debería tener en cuenta el [desarrollo de Visual Studio 2019](durable-functions-create-first-csharp.md).
> * Si está desarrollando Durable Functions en JavaScript, debería tener en cuenta el [desarrollo de Visual Studio Code](./quickstart-js-vscode.md).

## <a name="create-a-function-app"></a>Creación de una aplicación de función

Debe tener una aplicación de función para hospedar la ejecución de cualquier función. Una aplicación de funciones permite agrupar funciones como una unidad lógica para facilitar la administración, la implementación, el escalado y el uso compartido de recursos. Puede crear una aplicación .NET o JavaScript.

[!INCLUDE [Create function app Azure portal](../../../includes/functions-create-function-app-portal.md)]

De forma predeterminada, la aplicación de función creada utiliza la versión 2.x del entorno de ejecución de Azure Functions. La extensión de Durable Functions funciona en ambas versiones 1.x y 2.x del entorno de ejecución de Azure Functions en C# y en la versión 2.x de JavaScript. Sin embargo, las plantillas solo están disponibles cuando la versión del entorno de ejecución de destino es la 2.x, con independencia del lenguaje elegido.

## <a name="install-the-durable-functions-npm-package-javascript-only"></a>Instalación del paquete npm de Durable Functions (solo para JavaScript)

Si va a crear Durable Functions de JavaScript, deberá instalar el [`durable-functions`paquete npm](https://www.npmjs.com/package/durable-functions):

1. En la página de la aplicación de funciones, seleccione **Herramientas avanzadas** en **Herramientas de desarrollo** en el panel izquierdo.

   :::image type="content" source="./media/durable-functions-create-portal/function-app-platform-features-choose-kudu.png" alt-text="Selección de Kudu en las características de la plataforma de funciones":::

2. En la página **Herramientas avanzadas**, seleccione **Ir**.

3. En la consola de Kudu, seleccione **Consola de depuración** y, a continuación, **CMD**.

   :::image type="content" source="./media/durable-functions-create-portal/kudu-choose-debug-console.png" alt-text="Consola de depuración de Kudu":::

3. Debería aparecer la estructura de directorios de archivos de la aplicación de funciones. Vaya a la carpeta `site/wwwroot`. Desde allí, puede cargar un archivo `package.json` al arrastrarlo y colocarlo en la ventana del directorio de archivo. A continuación se muestra el ejemplo `package.json`:

    ```json
    {
      "dependencies": {
        "durable-functions": "^1.3.1"
      }
    }
    ```

   :::image type="content" source="./media/durable-functions-create-portal/kudu-choose-debug-console.png" alt-text="package.json cargado en Kudu":::

4. Una vez cargado el archivo `package.json`, ejecute el comando `npm install` en la consola de ejecución remota de Kudu.

   ![Ejecución de la instalación de npm en Kudu](./media/durable-functions-create-portal/kudu-npm-install.png)
   
## <a name="create-an-orchestrator-function"></a>Creación de una función de orquestador

1. En la aplicación de funciones, seleccione **Funciones** en el panel izquierdo y, a continuación, seleccione **Agregar** en el menú superior. 

1. En el campo de búsqueda de la página **Nueva función**, escriba `durable` y elija la plantilla **Inicio HTTP de Durable Functions**.

   :::image type="content" source="./media/durable-functions-create-portal/durable-functions-http-starter-template.png" alt-text="Selección de Inicio de HTTP de Durable Functions":::

1. Para el nombre de la **Nueva función**, escriba `HttpStart` y seleccione **Crear función**.

   La función creada se usa para iniciar la orquestación.

1. Cree otra función en la aplicación de funciones, esta vez mediante la plantilla **Orquestador de Durable Functions**. Asigne un nombre a la nueva función de orquestación `HelloSequence`.

1. Cree una tercera función llamada `Hello` mediante la plantilla **Actividad de Durable Functions**.

## <a name="test-the-durable-function-orchestration"></a>Prueba de la orquestación de función duradera

1. Vuelva a la función **HttpStart**, elija **Obtener la dirección URL de la función** y seleccione el icono **Copiar al Portapapeles** para copiar la dirección URL. Use esta dirección URL para iniciar la función **HelloSequence**.

1. Use una herramienta HTTP como Postman o cURL para enviar una solicitud POST a la dirección URL que copió. El ejemplo siguiente es un comando de cURL que envía una solicitud POST a la función duradera:

    ```bash
    curl -X POST https://{your-function-app-name}.azurewebsites.net/api/orchestrators/HelloSequence --header "Content-Length: 0"
    ```

    En este ejemplo, `{your-function-app-name}` es el dominio que es el nombre de la aplicación de función. El mensaje de respuesta contiene un conjunto de puntos de conexión URI que puede usar para supervisar y administrar la ejecución, que es similar al ejemplo siguiente:

    ```json
    {  
       "id":"10585834a930427195479de25e0b952d",
       "statusQueryGetUri":"https://...",
       "sendEventPostUri":"https://...",
       "terminatePostUri":"https://...",
       "rewindPostUri":"https://..."
    }
    ```

1. Llame al URI del punto de conexión `statusQueryGetUri` y verá el estado actual de la función duradera, lo que podría parecerse a este ejemplo:

    ```json
        {
            "runtimeStatus": "Running",
            "input": null,
            "output": null,
            "createdTime": "2017-12-01T05:37:33Z",
            "lastUpdatedTime": "2017-12-01T05:37:36Z"
        }
    ```

1. Continúe con la llamada al punto de conexión `statusQueryGetUri` hasta que el estado cambie a **Completado**, y verá una respuesta similar al ejemplo siguiente:

    ```json
    {
            "runtimeStatus": "Completed",
            "input": null,
            "output": [
                "Hello Tokyo!",
                "Hello Seattle!",
                "Hello London!"
            ],
            "createdTime": "2017-12-01T05:38:22Z",
            "lastUpdatedTime": "2017-12-01T05:38:28Z"
        }
    ```

La primera función duradera ahora está en funcionamiento en Azure.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre los patrones comunes de funciones durables](durable-functions-overview.md#application-patterns)
