---
title: Creación de un centro de IoT de Azure mediante una plantilla (.NET) | Microsoft Docs
description: Describe cómo usar una plantilla de Azure Resource Manager para crear un centro de IoT con el programa C#.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.openlocfilehash: b0a647fe3499590c0307b89d45d662ecf7e53392
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827783"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-net"></a>Creación de un centro de IoT con una plantilla de Azure Resource Manager (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Puede usar Azure Resource Manager para crear y administrar los centros de IoT de Azure mediante programación. En este tutorial se muestra cómo usar una plantilla de Azure Resource Manager para crear un IoT Hub desde un programa de C#.

> [!NOTE]
> Azure tiene dos modelos de implementación diferentes para crear recursos y trabajar con ellos:  [Azure Resource Manager y clásico](../azure-resource-manager/resource-manager-deployment-model.md).  Este artículo trata sobre el uso del modelo de implementación de Azure Resource Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este tutorial, necesitará lo siguiente:

* Visual Studio.
* Una cuenta de Azure activa. <br/>Si no tiene ninguna, puede crear una [cuenta gratuita][lnk-free-trial] en tan solo unos minutos.
* Una [cuenta de Azure Storage][lnk-storage-account] en la que pueda guardar los archivos de plantilla de Azure Resource Manager.
* [Azure PowerShell 1.0][lnk-powershell-install] o posterior.

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>Preparar su proyecto de Visual Studio

1. En Visual Studio, cree un proyecto de escritorio clásico de Windows de Visual C# usando la plantilla de proyecto **Aplicación de consola (.NET Framework)**. Asigne al proyecto el nombre **CreateIoTHub**.

2. En el Explorador de soluciones, haga clic con el botón secundario en su proyecto y luego haga clic en **Administrar paquetes de NuGet**.

3. En el Administrador de paquetes NuGet, active **Incluir versión preliminar** y en la página **Examinar** busque **Microsoft.Azure.Management.ResourceManager**. Seleccione el paquete, haga clic en **Instalar**, en **Revisar cambios**, haga clic en **Aceptar** y, luego, en **Acepto** para aceptar las licencias.

4. En el Administrador de paquetes NuGet, busque **Microsoft.IdentityModel.Clients.ActiveDirectory**.  Haga clic en **Instalar**, en **Revisar cambios**, haga clic en **Aceptar** y, luego, en **Acepto** para aceptar la licencia.

5. En Program.cs, reemplace las instrucciones **using** existentes por el siguiente código:

    ```csharp
    using System;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```

6. En Program.cs, agregue las siguientes variables estáticas reemplazando los valores de marcador de posición. Ha tomado nota de **ApplicationId**, **SubscriptionId**, **TenantId** y **Password** anteriormente en este tutorial. El **nombre de la cuenta de Azure Storage** es el nombre de la cuenta de Azure Storage en donde almacenará los archivos de la plantilla de Azure Resource Manager. El **nombre de grupo de recursos** es el nombre del grupo de recursos que usará al crear IoT Hub. El nombre puede ser un grupo de recursos ya existente u otro nuevo. El **Nombre de la implementación** es un nombre para la implementación, como **Deployment_01**.

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";
    static string storageAddress = "https://{Your storage account name}.blob.core.windows.net";
    static string rgName = "{Resource group name}";
    static string deploymentName = "{Deployment name}";
    ```

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="submit-a-template-to-create-an-iot-hub"></a>Enviar una plantilla para crear un centro de IoT

Use una plantilla de JSON y un archivo de parámetro para crear un centro de IoT en el grupo de recursos. También puede usar una plantilla de Azure Resource Manager para realizar cambios en un IoT Hub existente.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto, haga clic en **Agregar** y luego en **Nuevo elemento**. Agregue un archivo JSON denominado **template.json** a su proyecto.

2. Reemplace el contenido de **template.json** por la siguiente definición de recursos para agregar un centro de IoT estándar a la región **este de EE. UU.** Para ver una lista actualizada de las ubicaciones admitidas en IoT Hub, consulte [Estado de Azure][lnk-status]:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": {
          "type": "string"
        }
      },
      "resources": [
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs",
        "name": "[parameters('hubName')]",
        "location": "East US",
        "sku": {
          "name": "S1",
          "tier": "Standard",
          "capacity": 1
        },
        "properties": {
          "location": "East US"
        }
      }
      ],
      "outputs": {
        "hubKeys": {
          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
          "type": "object"
        }
      }
    }
    ```

3. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto, haga clic en **Agregar** y luego en **Nuevo elemento**. Agregue un archivo JSON denominado **parameters.json** a su proyecto.

4. Reemplace el contenido de **parameters.json** por la siguiente información de parámetro que establece el nombre del nuevo IoT Hub, como **{sus iniciales}miNuevoCentroDeIoT**. El nombre del centro de IoT debe ser único en todo el mundo, por lo que debe incluir su nombre o sus iniciales:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": { "value": "mynewiothub" }
      }
    }
    ```
   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

5. En el **Explorador de servidores**, conéctese a su suscripción de Azure y, en su cuenta de Azure Storage, cree un contenedor denominado **plantillas**. En el panel **Propiedades**, establezca los permisos de **Acceso de lectura público** para el contenedor de **plantillas** en **Blob**.

6. En el **Explorador de servidores**, haga clic con el botón derecho en el contenedor **plantillas** y luego haga clic en **Ver contenedor de blob**. Haga clic en el botón **Cargar blob**, seleccione los dos archivos, **parameters.json** y **templates.json**, y luego haga clic en **Abrir** para cargar los archivos JSON en el contenedor de **plantillas**. Las direcciones URL de los blobs que contienen los datos de JSON son:

    ```csharp
    https://{Your storage account name}.blob.core.windows.net/templates/parameters.json
    https://{Your storage account name}.blob.core.windows.net/templates/template.json
    ```
7. Agregue el método siguiente a Program.cs:

    ```csharp
    static void CreateIoTHub(ResourceManagementClient client)
    {

    }
    ```

8. Agregue el código siguiente al método **CreateIoTHub** para enviar los archivos de plantilla y parámetro a Azure Resource Manager:

    ```csharp
    var createResponse = client.Deployments.CreateOrUpdate(
        rgName,
        deploymentName,
        new Deployment()
        {
          Properties = new DeploymentProperties
          {
            Mode = DeploymentMode.Incremental,
            TemplateLink = new TemplateLink
            {
              Uri = storageAddress + "/templates/template.json"
            },
            ParametersLink = new ParametersLink
            {
              Uri = storageAddress + "/templates/parameters.json"
            }
          }
        });
    ```

9. Agregue el código siguiente al método **CreateIoTHub** que muestra el estado y las claves del nuevo centro de IoT:

    ```csharp
    string state = createResponse.Properties.ProvisioningState;
    Console.WriteLine("Deployment state: {0}", state);

    if (state != "Succeeded")
    {
      Console.WriteLine("Failed to create iothub");
    }
    Console.WriteLine(createResponse.Properties.Outputs);
    ```

## <a name="complete-and-run-the-application"></a>Completar y ejecutar la aplicación

Ahora puede completar la aplicación llamando al método **CreateIoTHub** antes de compilarla y ejecutarla.

1. Agregue el siguiente código al final del método **Main** :

    ```csharp
    CreateIoTHub(client);
    Console.ReadLine();
    ```

2. Haga clic en **Compilar** y luego en **Compilar solución**. Corrija los errores.

3. Haga clic en **Depurar** y luego en **Iniciar depuración** para ejecutar la aplicación. La ejecución de la implementación puede tardar varios minutos en completarse.

4. Para comprobar que la aplicación ha agregado el nuevo centro de IoT, visite [Azure Portal][lnk-azure-portal] y vea la lista de recursos. También puede usar el **Get AzResource** cmdlet de PowerShell.

> [!NOTE]
> Esta aplicación de ejemplo agrega un centro de IoT estándar S1 por el que se le cobrará. Puede eliminar el centro de IoT a través de la [portal de Azure] [ lnk-azure-portal] o mediante el **Remove-AzResource** cmdlet de PowerShell cuando haya terminado.

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha implementado un centro de IoT mediante una plantilla de Azure Resource Manager con un programa de C#, quizá desee seguir explorando:

* Consulte las funcionalidades de la [API de REST del proveedor de recursos de IoT Hub][lnk-rest-api].
* Para más información sobre las funcionalidades de Azure Resource Manager, consulte [Información general de Azure Resource Manager][lnk-azure-rm-overview].
* Para conocer la sintaxis y las propiedades JSON que se usan en las plantillas, consulte [Microsoft.Devices resource types](/azure/templates/microsoft.devices/iothub-allversions) (Tipos de recursos de Microsoft.Devices).

Para obtener más información sobre cómo desarrollar para IoT Hub, consulte los siguientes artículos:

* [Introducción a C SDK][lnk-c-sdk]
* [SDK de IoT de Azure][lnk-sdks]

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de Azure IoT Edge en un dispositivo simulado en Linux: versión preliminar][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azure/install-Az-ps
[lnk-rest-api]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md
[lnk-storage-account]:../storage/common/storage-create-storage-account.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/tutorial-simulate-device-linux.md
