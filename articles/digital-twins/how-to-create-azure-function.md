---
title: Configuración de una función de Azure para procesar datos
titleSuffix: Azure Digital Twins
description: Vea cómo crear una función de Azure a la que se pueda acceder y que se desencadene mediante gemelos digitales.
author: cschormann
ms.author: cschorm
ms.date: 3/17/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 3e284bc76da9ca40341d72f772aa7ee947a11638
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87124313"
---
# <a name="connect-azure-functions-apps-for-processing-data"></a>Conexión de aplicaciones de Azure Functions para el procesamiento de datos

Durante la versión preliminar, la actualización de gemelos digitales basados en los datos se controla mediante [**rutas de evento**](concepts-route-events.md) con recursos de proceso como [Azure Functions](../azure-functions/functions-overview.md). Se puede usar una función de Azure para actualizar un gemelo digital en respuesta a:
* Datos de telemetría de dispositivo procedentes de IoT Hub
* Cambio de propiedad u otros datos procedentes de otro gemelo digital dentro del grafo gemelo

Este artículo le guiará por la creación de una función de Azure para usarse con Azure Digital Twins. 

Se trata de una introducción a los pasos que contiene:

1. Creación de una aplicación de Azure Functions en Visual Studio
2. Escritura de una función de Azure con un [desencadenador de Event Grid](../event-grid/overview.md)
3. Incorporación de código de autenticación a la función (para poder acceder a Azure Digital Twins)
4. Publicación de la aplicación de funciones en Azure
5. Configure el acceso de [seguridad](concepts-security.md). Para que la función de Azure reciba un token de acceso en tiempo de ejecución para acceder al servicio, deberá configurar Managed Service Identity para la aplicación de funciones.

El resto de este artículo le guía por los pasos de configuración de Azure Functions, de uno en uno.

## <a name="create-an-azure-functions-app-in-visual-studio"></a>Creación de una aplicación de Azure Functions en Visual Studio

En Visual Studio 2019, seleccione *Archivo -> Nuevo > Proyecto*. Busque la plantilla *Azure Functions*, selecciónela y presione "Siguiente".

:::image type="content" source="media/how-to-create-azure-function/visual-studio-new-project.png" alt-text="Visual Studio: cuadro de diálogo Nuevo proyecto":::

Especifique un nombre para la aplicación de funciones y presione "Crear".

:::image type="content" source="media/how-to-create-azure-function/visual-studio-project-config.png" alt-text="Visual Studio: cuadro de diálogo Configurar proyecto":::

Seleccione el *desencadenador de Event Grid* y presione "Crear".

:::image type="content" source="media/how-to-create-azure-function/visual-studio-project-trigger.png" alt-text="Visual Studio: cuadro de diálogo del desencadenador del proyecto de función de Azure":::

## <a name="write-an-azure-function-with-an-event-grid-trigger"></a>Escritura de una función de Azure con un desencadenador de Event Grid

El código siguiente crea una función corta de Azure que puede usar para registrar eventos: 

```csharp
// Default URL for triggering Event Grid function in the local environment
// http://localhost:7071/runtime/webhooks/EventGrid?functionName={functionname}
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;

namespace FunctionSample
{
    public static class FooFunction    {
        [FunctionName("Foo")]
        public static void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

Esta es la función básica de Azure.

### <a name="run-and-debug-the-azure-function-app"></a>Ejecución y depuración de la aplicación de funciones de Azure

Ahora puede compilar y ejecutar la función. Aunque las funciones de Azure están pensadas para ejecutarse en la nube, también puede ejecutarlas y depurarlas de forma local.

Para más información, consulte [*Depuración de un desencadenador de Event Grid de forma local*](../azure-functions/functions-debug-event-grid-trigger-local.md).

### <a name="add-the-azure-digital-twins-sdk-to-your-azure-function-app"></a>Incorporación del SDK de Azure Digital Twins a la aplicación de funciones de Azure

La aplicación de funciones interactúa con Azure Digital Twins mediante la [biblioteca de cliente de Azure IoT Digital Twins para .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core). Para poder usar el SDK, debe incluir los siguientes paquetes en el proyecto:
* `Azure.DigitalTwins.Core` (versión `1.0.0-preview.2`)
* `Azure.Identity`

Para que la configuración de la canalización de Azure SDK se configure correctamente para Azure Functions, también necesitará:
* `Azure.Net.Http`
* `Azure.Core`

En función de las herramientas que elija, puede hacerlo con el administrador de paquetes de Visual Studio o con la herramienta de línea de comandos `dotnet`. 

Agregue las siguientes instrucciones using a la función de Azure.

```csharp
using Azure.Identity;
using Azure.DigitalTwins.Core;
using System.Net.Http;
using Azure.Core.Pipeline;
```

## <a name="add-authentication-code-to-the-azure-function"></a>Incorporación de código de autenticación a la función de Azure

A continuación, agregue el código de autenticación que permitirá a la función acceder a Azure Digital Twins.

Agregue variables a la clase de función para estos valores: 
* El identificador de aplicación de Azure Digital Twins
* La dirección URL de la instancia de Azure Digital Twins. Se recomienda leer la dirección URL del servicio en una variable de entorno, en lugar de codificarla de forma rígida en la función.
* Una variable estática para contener una instancia de HttpClient. HttpClient es relativamente costoso de crear y queremos evitar tener que hacerlo para cada invocación de función.

Agregue también una variable local dentro de la función para que contenga la instancia del cliente de Azure Digital Twins en el proyecto de función. *No* haga que esta variable sea estática dentro de la clase.

Por último, cambie la signatura de función para que sea asincrónica.

Después de estos cambios, el código de la función debe ser similar al siguiente:

```csharp
namespace FunctionSample
{
    public static class FooFunction
    {
        const string adtAppId = "https://digitaltwins.azure.net";
        private static string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static HttpClient httpClient = new HttpClient();

        [FunctionName("Foo")]
        public static async Task Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            DigitalTwinsClient client = null;
            try
            {
                ManagedIdentityCredential cred = new ManagedIdentityCredential(adtAppId);
                DigitalTwinsClientOptions opts = 
                    new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
                client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, opts);
                                                
                log.LogInformation($"ADT service client connection created.");
            }
            catch (Exception e)
            {
                log.LogError($"ADT service client connection failed. " + e.ToString());
                return;
            }
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

Para que la aplicación de Functions pueda acceder a Azure Digital Twins, debe tener una identidad administrada por el sistema y disponer de permisos para acceder a la instancia de Azure Digital Twins.

Use el siguiente comando para crear la identidad administrada por el sistema. Anote el valor del campo *principalId* de la salida.

```azurecli
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>
```

Use el valor de *principalId* en el siguiente comando para asignar la identidad de la aplicación de funciones al rol *Propietario* de su instancia de Azure Digital Twins:

```azurecli
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Owner (Preview)"
```

Para más información sobre identidades administradas, consulte [*Procedimiento para usar identidades administradas para App Service y Azure Functions*](../app-service/overview-managed-identity.md).

Por último, puede hacer que la dirección URL de la instancia de Azure Digital Twins sea accesible para la función si establece una variable de entorno. Para más información, consulte [*Variables de entorno*](https://docs.microsoft.com/sandbox/functions-recipes/environment-variables).

> [!TIP]
> Para crear la dirección URL de la instancia de Azure Digital Twins, agregue *https://* al principio del *nombre de host* de la instancia de Azure Digital Twins. Para ver el nombre de host, junto con todas las propiedades de la instancia, puede ejecutar `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

```azurecli
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-URL>"
```

## <a name="publish-the-azure-function-app"></a>Publicación de la aplicación de funciones en Azure

Para publicar la aplicación de funciones en Azure, haga clic con el botón derecho en el proyecto de función (no en la solución) en el Explorador de soluciones y elija *Publicar()* .

Aparecerá la siguiente pestaña:

:::image type="content" source="media/how-to-create-azure-function/visual-studio-publish-1.png" alt-text="Visual Studio: cuadro de diálogo publicar función, página 1":::

Seleccione o cree un plan de App Service para usarlo con Azure Functions. Si tiene dudas, empiece a usar el plan de consumo predeterminado.

> [!IMPORTANT] 
> La publicación de una función de Azure incurrirá en cargos adicionales por su suscripción, independientemente de Azure Digital Twins.

:::image type="content" source="media/how-to-create-azure-function/visual-studio-publish-2.png" alt-text="Visual Studio: cuadro de diálogo publicar función, página 2":::

En la página siguiente, escriba el nombre que desee para la nueva aplicación de funciones, un grupo de recursos y otros detalles.

## <a name="set-up-security-access-for-the-azure-function-app"></a>Configuración del acceso de seguridad para la aplicación de funciones de Azure

El esqueleto de la función de Azure de los ejemplos anteriores requiere que se le pase un token de portador para poder autenticarse con Azure Digital Twins. Para asegurarse de que se pasa este token de portador, debe configurar [Managed Service Identity (MSI)](../active-directory/managed-identities-azure-resources/overview.md) para la aplicación de funciones. Esto solo debe realizarse una vez para cada aplicación de funciones.

Para configurarlo, abra [Azure Portal](https://portal.azure.com/) y vaya a la aplicación de funciones.

En la pestaña *Características de la plataforma*, seleccione *Identidad*:

:::image type="content" source="media/how-to-create-azure-function/visual-studio-msi-1.png" alt-text="Azure Portal: Selección de la identidad para una función de Azure":::

En la página de identidad, establezca el *Estado* en *Activado*. 

:::image type="content" source="media/how-to-create-azure-function/visual-studio-msi-2.png" alt-text="Azure Portal: Activación del estado de identidad":::

Tenga en cuenta también el **identificador de objeto** que se muestra en esta página, ya que se usará en la sección siguiente.

### <a name="assign-access-roles"></a>Asignación de roles de acceso

Dado que Azure Digital Twins usa el control de acceso basado en rol para administrar el acceso (consulte [*Conceptos: Seguridad de las soluciones de Azure Digital Twins*](concepts-security.md) para más información), también debe agregar un rol para cada aplicación de funciones a la que desee permitir el acceso a Azure Digital Twins.

Para asignar un rol, necesita el **identificador de recurso** de la instancia de Azure Digital Twins que ha creado. Si no lo grabó antes de crear la instancia, puede recuperarlo mediante este comando:

```bash
az dt show --name <your-instance-name> -g <your-resource-group-name>
```
El identificador de recurso formará parte de la salida, como una cadena larga denominada "id" que comienza con las letras "/subscriptions/...".

Use el identificador de recurso junto con el identificador de objeto de la función de Azure anterior en el siguiente comando:

```azurecli
az role assignment create --role "Azure Digital Twins Owner (Preview)" --assignee <object-ID> --scope <resource-ID>
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha seguido los pasos para configurar una función de Azure para usarla con Azure Digital Twins. A continuación, puede suscribir la función de Azure a Event Grid para escuchar en un punto de conexión. Este punto de conexión podría ser:
* Un punto de conexión de Event Grid asociado a Azure Digital Twins para procesar los mensajes procedentes del propio Azure Digital Twins (como los mensajes de cambio de propiedad, los mensajes de telemetría que genera [Digital Twins](concepts-twins-graph.md) en el grafo de gemelos o los mensajes del ciclo de vida).
* Los temas del sistema de IoT que IoT Hub usa para enviar telemetría y otros eventos de dispositivo
* Un punto de conexión de Event Grid que reciba los mensajes de otros servicios

A continuación, consulte cómo crear una función de Azure básica para ingerir datos de IoT Hub en Azure Digital Twins:
* [*Procedimiento: Ingesta de telemetría de IoT Hub*](how-to-ingest-iot-hub-data.md)