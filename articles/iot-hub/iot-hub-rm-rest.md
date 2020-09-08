---
title: Creación de un centro de IoT de Azure mediante la API de REST del proveedor de recursos | Microsoft Docs
description: Aprenda a usar la API REST de C# del proveedor de recursos para crear y administrar una instancia de IoT Hub mediante programación.
author: robinsh
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: fe143b1c172c5d89a7dba33731a378df988b4a47
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014615"
---
# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>Creación de un centro de IoT mediante la API de REST del proveedor de recursos (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Puede usar la [API de REST del proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource) para crear y administrar los centros de IoT de Azure mediante programación. En este tutorial se muestra cómo usar la API de REST del proveedor de recursos de IoT Hub para crear un IoT Hub desde un programa de C#.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este tutorial, necesitará lo siguiente:

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* [Azure PowerShell 1.0](https://docs.microsoft.com/powershell/azure/install-Az-ps) o posterior.

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>Preparar su proyecto de Visual Studio

1. En Visual Studio, cree un proyecto de escritorio clásico de Windows de Visual C# usando la plantilla de proyecto **Aplicación de consola (.NET Framework)** . Asigne al proyecto el nombre **CreateIoTHubREST**.

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

Utilice la [API de REST del proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource) para crear un centro de IoT en el grupo de recursos. También puede utilizar la API de REST del proveedor de recursos para efectuar cambios en un centro de IoT existente.

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

3. Agregue el código siguiente al método **CreateIoTHub**. Este código describe el centro de IoT que se creará y genera una representación JSON. Para ver una lista actualizada de las ubicaciones admitidas en IoT Hub, consulte [Estado de Azure](https://azure.microsoft.com/status/).

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

4. Para comprobar que la aplicación ha agregado la nueva instancia de IoT Hub, visite [Azure Portal](https://portal.azure.com/) y vea la lista de recursos. Como alternativa, use el cmdlet de PowerShell **Get-AzResource**.

> [!NOTE]
> Esta aplicación de ejemplo agrega un centro de IoT estándar S1 por el que se le cobrará. Cuando haya terminado, podrá eliminar el centro de IoT a través de [Azure Portal](https://portal.azure.com/) o con el cmdlet **Remove-AzResource** de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado un centro de IoT con la API de REST del proveedor de recursos, es posible que quiera profundizar más en este tema:

* Consulte las funcionalidades de la [API de REST del proveedor de recursos de IoT Hub](https://docs.microsoft.com/rest/api/iothub/iothubresource).

* Para más información sobre las funcionalidades de Azure Resource Manager, consulte [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md).

Para obtener más información sobre cómo desarrollar para IoT Hub, consulte los siguientes artículos:

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)