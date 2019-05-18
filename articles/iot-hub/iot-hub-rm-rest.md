---
title: Creación de un centro de IoT de Azure mediante la API de REST del proveedor de recursos | Microsoft Docs
description: Describe cómo usar la API de REST del proveedor de recursos para crear un centro de IoT.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.openlocfilehash: 6d91f5e61dfd7c3cb4d1869edf0c6cb8c2c85190
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827493"
---
# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>Creación de un centro de IoT mediante la API de REST del proveedor de recursos (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Puede usar el [API de REST de proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource) para crear y administrar centros de IoT de Azure mediante programación. En este tutorial se muestra cómo usar la API de REST del proveedor de recursos de IoT Hub para crear un IoT Hub desde un programa de C#.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este tutorial, necesitará lo siguiente:

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* [Azure PowerShell 1.0](https://docs.microsoft.com/powershell/azure/install-Az-ps) o una versión posterior.

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>Preparar su proyecto de Visual Studio

1. En Visual Studio, cree un proyecto de escritorio clásico de Windows de Visual C# usando la plantilla de proyecto **Aplicación de consola (.NET Framework)**. Asigne al proyecto el nombre **CreateIoTHubREST**.

2. En el Explorador de soluciones, haga clic con el botón secundario en su proyecto y luego haga clic en **Administrar paquetes de NuGet**.

3. En el Administrador de paquetes NuGet, active **Incluir versión preliminar** y en la página **Examinar** busque **Microsoft.Azure.Management.ResourceManager**. Seleccione el paquete, haga clic en **Instalar**, en **Revisar cambios**, haga clic en **Aceptar** y, luego, en **Acepto** para aceptar las licencias.

4. En el Administrador de paquetes NuGet, busque **Microsoft.IdentityModel.Clients.ActiveDirectory**.  Haga clic en **Instalar**, en **Revisar cambios**, haga clic en **Aceptar** y, luego, en **Acepto** para aceptar la licencia.

5. En Program.cs, reemplace las instrucciones **using** existentes por el siguiente código:

    ```csharp
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    ```

6. En Program.cs, agregue las siguientes variables estáticas reemplazando los valores de marcador de posición. Ha tomado nota de **ApplicationId**, **SubscriptionId**, **TenantId** y **Password** anteriormente en este tutorial. El **nombre de grupo de recursos** es el nombre del grupo de recursos que usará al crear IoT Hub. Puede ser un grupo de recursos ya existente u otro nuevo. **Nombre de IoT Hub** es el nombre de la instancia de IoT Hub que creó como, por ejemplo, **MyIoTHub**. El nombre de su instancia de IoT Hub debe única globalmente. El **Nombre de la implementación** es un nombre para la implementación, como **Deployment_01**.

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";

    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```
   
    [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="use-the-resource-provider-rest-api-to-create-an-iot-hub"></a>Uso de la API de REST del proveedor de recursos para crear un centro de IoT

Use la [API de REST de proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource) para crear un centro de IoT en el grupo de recursos. También puede utilizar la API de REST del proveedor de recursos para efectuar cambios en un centro de IoT existente.

1. Agregue el método siguiente a Program.cs:

    ```csharp
    static void CreateIoTHub(string token)
    {

    }
    ```

2. Agregue el código siguiente al método **CreateIoTHub**. Este código crea un objeto **HttpClient** con el token de autenticación de los encabezados:

    ```csharp
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```

3. Agregue el código siguiente al método **CreateIoTHub**. Este código describe el centro de IoT que se creará y genera una representación JSON. Para obtener la lista actual de las ubicaciones que admiten IoT Hub consulte [estado Azure](https://azure.microsoft.com/status/):

    ```csharp
    var description = new
    {
      name = iotHubName,
      location = "East US",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };

    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```

4. Agregue el código siguiente al método **CreateIoTHub**. Este código envía la solicitud REST a Azure. El código comprueba la respuesta y recupera la URL con la que se puede supervisar el estado de la tarea de implementación:

    ```csharp
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.azure.com/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;

    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }

    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```

5. Agregue el siguiente código al final del método **CreateIoTHub**. Este código usa la dirección **asyncStatusUri** recuperada del paso anterior para esperar a que finalice la implementación:

    ```csharp
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    } while (body == "{\"status\":\"Running\"}");
    ```

6. Agregue el siguiente código al final del método **CreateIoTHub**. Este código recupera las claves del centro de IoT que se crea y las imprime en la consola:

    ```csharp
    var listKeysUri = string.Format("https://management.azure.com/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;

    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```

## <a name="complete-and-run-the-application"></a>Completar y ejecutar la aplicación

Ahora puede completar la aplicación llamando al método **CreateIoTHub** antes de compilarla y ejecutarla.

1. Agregue el siguiente código al final del método **Main** :

    ```csharp
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```

2. Haga clic en **Compilar** y luego en **Compilar solución**. Corrija los errores.

3. Haga clic en **Depurar** y luego en **Iniciar depuración** para ejecutar la aplicación. La ejecución de la implementación puede tardar varios minutos en completarse.

4. Para comprobar que la aplicación ha agregado el nuevo centro de IoT, visite la [portal Azure](https://portal.azure.com/) y ver la lista de recursos. También puede usar el **Get AzResource** cmdlet de PowerShell.

> [!NOTE]
> Esta aplicación de ejemplo agrega un centro de IoT estándar S1 por el que se le cobrará. Cuando haya terminado, puede eliminar el centro de IoT a través de la [portal de Azure](https://portal.azure.com/) o mediante el **Remove-AzResource** cmdlet de PowerShell cuando haya terminado.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado un centro de IoT con la API de REST del proveedor de recursos, es posible que quiera profundizar más en este tema:

* Obtenga información sobre las capacidades de la [API de REST de proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource).

* Lectura [Introducción a Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) para obtener más información sobre las funcionalidades de Azure Resource Manager.

Para obtener más información sobre cómo desarrollar para IoT Hub, consulte los siguientes artículos:

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)