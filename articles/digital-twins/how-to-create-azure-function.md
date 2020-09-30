---
title: Configuración de una función de Azure para procesar datos
titleSuffix: Azure Digital Twins
description: Vea cómo crear una función de Azure a la que se pueda acceder y que se desencadene mediante gemelos digitales.
author: baanders
ms.author: baanders
ms.date: 8/27/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: a0f5a921ad9eba82f27b3a8945643737b6d76030
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89614070"
---
# <a name="connect-azure-functions-apps-for-processing-data"></a>Conexión de aplicaciones de Azure Functions para el procesamiento de datos

La actualización de gemelos digitales basados en los datos se controla mediante [**rutas de evento**](concepts-route-events.md) con recursos de proceso como [Azure Functions](../azure-functions/functions-overview.md). Se puede usar una función de Azure para actualizar un gemelo digital en respuesta a:
* Datos de telemetría de dispositivo procedentes de IoT Hub
* Cambio de propiedad u otros datos procedentes de otro gemelo digital dentro del grafo gemelo

Este artículo le guiará por la creación de una función de Azure para usarse con Azure Digital Twins. 

Se trata de una introducción a los pasos que contiene:

1. Creación de una aplicación de Azure Functions en Visual Studio
2. Escritura de una función de Azure con un [desencadenador de Event Grid](../event-grid/overview.md)
3. Incorporación de código de autenticación a la función (para poder acceder a Azure Digital Twins)
4. Publicación de la aplicación de funciones en Azure
5. Configuración del acceso de [seguridad](concepts-security.md) para la aplicación de funciones de Azure

## <a name="create-an-azure-functions-app-in-visual-studio"></a>Creación de una aplicación de Azure Functions en Visual Studio

En Visual Studio 2019, seleccione _Archivo > Nuevo > Proyecto_, busque la plantilla _Azure Functions_ y seleccione _Siguiente_.

:::image type="content" source="media/how-to-create-azure-function/create-azure-function-project.png" alt-text="Visual Studio: cuadro de diálogo Nuevo proyecto":::

Especifique un nombre para la aplicación de funciones y seleccione _Crear_.

:::image type="content" source="media/how-to-create-azure-function/configure-new-project.png" alt-text="Visual Studio: configuración de un proyecto nuevo":::

Seleccione el tipo de *desencadenador de Event Grid* de la aplicación de funciones y seleccione _Crear_.

:::image type="content" source="media/how-to-create-azure-function/eventgridtrigger-function.png" alt-text="Visual Studio: cuadro de diálogo del desencadenador del proyecto de función de Azure":::

Una vez creada la aplicación de funciones, Visual Studio tendrá un ejemplo de código rellenado automáticamente en **function.cs** en la carpeta del proyecto. Esta breve función de Azure se usa para registrar eventos.

:::image type="content" source="media/how-to-create-azure-function/visual-studio-sample-code.png" alt-text="Visual Studio: Ventana de proyecto con código de ejemplo":::

## <a name="write-an-azure-function-with-an-event-grid-trigger"></a>Escritura de una función de Azure con un desencadenador de Event Grid

Para escribir una función de Azure, agregue el SDK a la aplicación de funciones. La aplicación de funciones interactúa con Azure Digital Twins mediante la [biblioteca de cliente de Azure IoT Digital Twins para .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core). 

Para poder usar el SDK, debe incluir los siguientes paquetes en el proyecto. Puede instalar los paquetes mediante el administrador de paquetes NuGet de Visual Studio o agregar los paquetes mediante la herramienta de línea de comandos `dotnet`. Elija cualquiera de estos métodos: 

**Opción 1. Agregue paquetes con el administrador de paquetes de Visual Studio:**
    
Para ello, haga clic con el botón derecho en el proyecto y seleccione _Administrar paquetes NuGet_ en la lista. A continuación, en la ventana que se abre, seleccione la pestaña _Examinar_ y busque los siguientes paquetes. Seleccione _Instalar_ y _acepte_ el contrato de licencia para instalar los paquetes.

* `Azure.DigitalTwins.Core`
* `Azure.Identity` 

Para que la configuración de la canalización de Azure SDK se configure correctamente para Azure Functions, también necesitará los paquetes siguientes. Repita el mismo proceso que antes para instalar todos los paquetes.

* `System.Net.Http`
* `Azure.Core.Pipeline`

**Opción 2. Agregar paquetes mediante la herramienta de línea de comandos `dotnet`:**

```cmd/sh
dotnet add package Azure.DigitalTwins.Core --version 1.0.0-preview.3
dotnet add package Azure.identity --version 1.2.2
dotnet add package System.Net.Http
dotnet add package Azure.Core.Pipeline
```
Después, en el Explorador de soluciones de Visual Studio, abra el archivo _function.cs_ en el que tiene código de ejemplo y agregue lo siguiente _mediante_ instrucciones a la función de Azure. 

```csharp
using Azure.DigitalTwins.Core;
using Azure.Identity;
using System.Net.Http;
using Azure.Core.Pipeline;
```
## <a name="add-authentication-code-to-the-azure-function"></a>Incorporación de código de autenticación a la función de Azure

Ahora declarará variables de nivel de clase y agregará un código de autenticación que permitirá a la función acceder a Azure Digital Twins. Agregará lo siguiente a la función de Azure en el archivo {nombre de función}.cs.

* Lea la dirección URL del servicio ADT como variable de entorno. Se recomienda leer la dirección URL del servicio en una variable de entorno, en lugar de codificarla de forma rígida en la función.
```csharp     
private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
```
* Una variable estática para contener una instancia de HttpClient. HttpClient es relativamente costoso de crear y queremos evitar tener que hacerlo para cada invocación de función.
```csharp
private static readonly HttpClient httpClient = new HttpClient();
```
* Puede usar las credenciales de identidad administrada en una función de Azure.
```csharp
ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
```
* Agregue una variable local _DigitalTwinsClient_ dentro de la función para que contenga la instancia del cliente de Azure Digital Twins en el proyecto de función. *No* haga que esta variable sea estática dentro de la clase.
```csharp
DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
```
* Agregue una comprobación nula para _adtInstanceUrl_ y ajuste la lógica de la función en un bloque try catch para detectar cualquier excepción.

Después de estos cambios, el código de la función será similar al siguiente:

```csharp
// Default URL for triggering event grid function in the local environment.
// http://localhost:7071/runtime/webhooks/EventGrid?functionName={functionname}
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Azure.DigitalTwins.Core;
using Azure.Identity;
using System.Net.Http;
using Azure.Core.Pipeline;

namespace adtIngestFunctionSample
{
    public class Function1
    {
        //Your Digital Twin URL is stored in an application setting in Azure Functions
        private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static readonly HttpClient httpClient = new HttpClient();

        [FunctionName("TwinsFunction")]
        public void Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
            if (adtInstanceUrl == null) log.LogError("Application setting \"ADT_SERVICE_URL\" not set");
            try
            {
                //Authenticate with Digital Twins
                ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
                DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
                log.LogInformation($"ADT service client connection created.");
                /*
                * Add your business logic here
                */
            }
            catch (Exception e)
            {
                log.LogError(e.Message);
            }

        }
    }
}
```

## <a name="publish-the-function-app-to-azure"></a>Publicación de la aplicación de funciones en Azure

Para publicar la aplicación de funciones en Azure, haga clic con el botón derecho en el proyecto de función (no en la solución) en el Explorador de soluciones y elija **Publicar**.

> [!IMPORTANT] 
> La publicación de una función de Azure incurrirá en cargos adicionales por su suscripción, independientemente de Azure Digital Twins.

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function.png" alt-text="Visual Studio: publicar una función de Azure":::

Seleccione **Azure** como el destino de publicación y, a continuación, elija **Siguiente**.

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-1.png" alt-text="Visual Studio: cuadro de diálogo de publicación de función de Azure; seleccione Azure ":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-2.png" alt-text="Visual Studio: cuadro de diálogo de publicación de función de Azure; seleccione Aplicación de funciones de Azure (Windows) o (Linux) en función de su máquina":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-3.png" alt-text="Visual Studio: cuadro de diálogo de publicación de función; Creación de una nueva función de Azure":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-4.png" alt-text="Visual Studio: cuadro de diálogo de publicación de función; rellene los campos y seleccione Crear":::

:::image type="content" source="media/how-to-create-azure-function/publish-azure-function-5.png" alt-text="Visual Studio: cuadro de diálogo de publicación de función; seleccione la aplicación de funciones en la lista y, a continuación, Finalizar":::

En la página siguiente, escriba el nombre que desee para la nueva aplicación de funciones, un grupo de recursos y otros detalles.
Para que la aplicación de Functions pueda acceder a Azure Digital Twins, debe tener una identidad administrada por el sistema y disponer de permisos para acceder a la instancia de Azure Digital Twins.

A continuación, puede configurar el acceso de seguridad para la función mediante la CLI o Azure Portal. Elija cualquiera de estos métodos:

## <a name="set-up-security-access-for-the-azure-function-app"></a>Configuración del acceso de seguridad para la aplicación de funciones de Azure
Puede configurar el acceso de seguridad para la aplicación de funciones de Azure mediante una de estas opciones:

### <a name="option-1-set-up-security-access-for-the-azure-function-app-using-cli"></a>Opción 1: Configuración del acceso de seguridad para la aplicación de funciones de Azure mediante la CLI

El esqueleto de la función de Azure de los ejemplos anteriores requiere que se le pase un token de portador para poder autenticarse con Azure Digital Twins. Para asegurarse de que se pasa este token de portador, debe configurar [Managed Service Identity (MSI)](../active-directory/managed-identities-azure-resources/overview.md) para la aplicación de funciones. Esto solo debe realizarse una vez para cada aplicación de funciones.

Puede crear una identidad administrada por el sistema y asignar la identidad de la aplicación de funciones al rol _Propietario de Azure Digital Twins (versión preliminar)_ para la instancia de Azure Digital Twins. Esto proporcionará el permiso de la aplicación de funciones en la instancia para realizar actividades del plano de datos. A continuación, haga que la dirección URL de la instancia de Azure Digital Twins sea accesible para la función mediante la definición de una variable de entorno.

 Use [Azure Cloud Shell](https://shell.azure.com) para ejecutar los comandos.

Use el siguiente comando para crear la identidad administrada por el sistema. Anote el valor del campo _principalId_ de la salida.

```azurecli 
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>   
```
Use el valor de _principalId_ en el siguiente comando para asignar la identidad de la aplicación de funciones al rol _Propietario de Azure Digital Twins (versión preliminar)_ en la instancia de Azure Digital Twins.

```azurecli 
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Owner (Preview)"
```
Por último, puede hacer que la dirección URL de la instancia de Azure Digital Twins sea accesible para la función si establece una variable de entorno. Para obtener más información sobre la configuración de variables de entorno, consulte [*Variables de entorno*](https://docs.microsoft.com/sandbox/functions-recipes/environment-variables). 

> [!TIP]
> Para crear la dirección URL de la instancia de Azure Digital Twins, agregue *https://* al principio del *nombre de host* de la instancia de Azure Digital Twins. Para ver el nombre de host, junto con todas las propiedades de la instancia, puede ejecutar `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

```azurecli 
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=https://<your-Azure-Digital-Twins-instance-hostname>"
```
### <a name="option-2-set-up-security-access-for-the-azure-function-app-using-azure-portal"></a>Opción 2: Configuración del acceso de seguridad para la aplicación de funciones de Azure mediante Azure Portal

Una identidad administrada asignada por el sistema permite que los recursos de Azure se autentiquen en servicios en la nube (por ejemplo, Azure Key Vault) sin almacenar las credenciales en el código. Una vez habilitada, todos los permisos necesarios se pueden conceder mediante el control de acceso basado en roles de Azure. El ciclo de vida de este tipo de identidad administrada está ligado al ciclo de vida de este recurso. Además, cada recurso (por ejemplo, máquina virtual) solo puede tener una identidad administrada asignada por el sistema.

En [Azure Portal](https://portal.azure.com/), busque _aplicación de funciones_ en la barra de búsqueda con el nombre de la aplicación de funciones que creó anteriormente. Seleccione la *Aplicación de funciones* en la lista. 

:::image type="content" source="media/how-to-create-azure-function/portal-search-for-functionapp.png" alt-text="Azure Portal: buscar aplicaciones de funciones":::

En la ventana de la aplicación de funciones, seleccione _Identidad_ en la barra de navegación de la izquierda para habilitar la identidad administrada.
En la pestaña _Asignada por el sistema_, cambie el _Estado_ a Activado y _Guarde_. Verá un menú emergente para _Habilitar identidad administrada asignada por el sistema_.
Seleccione el botón _Sí_. 

:::image type="content" source="media/how-to-create-azure-function/enable-system-managed-identity.png" alt-text="Azure Portal: habilitar la identidad administrada por el sistema":::

Puede comprobar en las notificaciones que la función se ha registrado correctamente con Azure Active Directory.

:::image type="content" source="media/how-to-create-azure-function/notifications-enable-managed-identity.png" alt-text="Azure Portal: notificaciones":::

Anote también el **id. de objeto** que se muestra en la página _Identidad_ , ya que se usará en la sección siguiente.

:::image type="content" source="media/how-to-create-azure-function/object-id.png" alt-text="Copiar el id. de objeto para usarlo en el futuro":::

### <a name="assign-access-roles-using-azure-portal"></a>Asignación de roles de acceso mediante Azure Portal

Seleccione el botón _Asignaciones de roles de Azure_, que se abre en la página de asignaciones de roles de Azure. A continuación, seleccione _+ Agregar asignación de roles (versión preliminar)_ .

:::image type="content" source="media/how-to-create-azure-function/add-role-assignments.png" alt-text="Azure Portal: agregar asignación de roles":::

En la ventana _Agregar asignación de roles (versión preliminar)_ que se abre, seleccione

* _Ámbito_: grupo de recursos
* _Suscripción_: seleccione su suscripción de Azure.
* _Grupo de recursos_: seleccione el grupo de recursos en la lista desplegable.
* _Rol_: seleccione _Propietario de Azure Digital Twins (versión preliminar)_ en el menú desplegable

Para guardar los detalles, seleccione el botón _Guardar_.

:::image type="content" source="media/how-to-create-azure-function/add-role-assignment.png" alt-text="Azure Portal: agregar asignación de roles (versión preliminar) ":::

### <a name="configure-application-settings-using-azure-portal"></a>Configuración de las opciones de la aplicación mediante Azure Portal

Puede hacer que la dirección URL de la instancia de Azure Digital Twins sea accesible para la función si establece una variable de entorno. Para más información, consulte [*Variables de entorno*](https://docs.microsoft.com/sandbox/functions-recipes/environment-variables). La configuración de la aplicación se expone como variables de entorno para tener acceso a la instancia de Digital Twins. 

Necesitará ADT_INSTANCE_URL para crear una configuración de aplicación.

Para obtener ADT_INSTANCE_URL, anexe **_https://_** al nombre de host de la instancia. En Azure Portal, para encontrar el nombre del host de la instancia de Digital Twins, busque su instancia en la barra de búsqueda. A continuación, seleccione _Información general_ en la barra de navegación izquierda para ver el _Nombre de host_. Para crear una opción de aplicación, copie este valor.

:::image type="content" source="media/how-to-create-azure-function/adt-hostname.png" alt-text="Azure Portal: Información general -> Copie el nombre de host para usarlo en el campo _Value_.":::

Ahora puede seguir estos pasos para crear una configuración de la aplicación:

* Busque la función de Azure con el nombre de la función en la barra de búsqueda y seleccione la función en la lista.
* Seleccione _Configuración_ en la barra de navegación de la izquierda para crear una nueva configuración de la aplicación.
* En la pestaña _Configuración de la aplicación_, seleccione _+ Nueva configuración de la aplicación_

:::image type="content" source="media/how-to-create-azure-function/search-for-azure-function.png" alt-text="Azure Portal: buscar una función de Azure existente":::

:::image type="content" source="media/how-to-create-azure-function/application-setting.png" alt-text="Azure Portal: Configuración de la aplicación":::

En la ventana que se abre, use el valor copiado anteriormente para crear una configuración de la aplicación. \
_Nombre_: URL_SERVICIO_ADT \
_Valor_: https://{nombre-de-host-de-azure-digital-twins}

Seleccione _Aceptar_ para crear una configuración de la aplicación.

:::image type="content" source="media/how-to-create-azure-function/add-application-setting.png" alt-text="Azure Portal: agregar la configuración de la aplicación.":::

Puede ver la configuración de la aplicación con el nombre de la aplicación bajo el campo _Nombre_ . A continuación, para guardar la configuración de la aplicación, seleccione el botón _Guardar_.

:::image type="content" source="media/how-to-create-azure-function/application-setting-save-details.png" alt-text="Azure Portal: visualizar la aplicación creada y reiniciarla":::

Cualquier cambio en la configuración de la aplicación requiere que se reinicie la aplicación. Seleccione _Continuar_ para reiniciar la aplicación.

:::image type="content" source="media/how-to-create-azure-function/save-application-setting.png" alt-text="Azure Portal: guardar la configuración de la aplicación":::

Para ver que la configuración de la aplicación se actualiza, seleccione el icono _Notificaciones_. Si no se crea la configuración de la aplicación, puede volver a intentar agregar una configuración de aplicación siguiendo el proceso anterior.

:::image type="content" source="media/how-to-create-azure-function/notifications-update-web-app-settings.png" alt-text="Azure Portal: notificaciones para actualizar la configuración de la aplicación":::

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha seguido los pasos para configurar una función de Azure para usarla con Azure Digital Twins. A continuación, puede suscribir la función de Azure a Event Grid para escuchar en un punto de conexión. Este punto de conexión podría ser:
* Un punto de conexión de Event Grid asociado a Azure Digital Twins para procesar los mensajes procedentes del propio Azure Digital Twins (como los mensajes de cambio de propiedad, los mensajes de telemetría que genera [Digital Twins](concepts-twins-graph.md) en el grafo de gemelos o los mensajes del ciclo de vida).
* Los temas del sistema de IoT que IoT Hub usa para enviar telemetría y otros eventos de dispositivo
* Un punto de conexión de Event Grid que reciba los mensajes de otros servicios

A continuación, consulte cómo crear una función de Azure básica para ingerir datos de IoT Hub en Azure Digital Twins:
* [*Procedimiento: Ingesta de telemetría de IoT Hub*](how-to-ingest-iot-hub-data.md)