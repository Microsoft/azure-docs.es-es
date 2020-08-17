---
title: Bibliotecas de administración de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo usar las bibliotecas de administración de Azure Service Bus para aprovisionar dinámicamente las entidades y los espacios de nombres de Service Bus.
ms.devlang: dotnet
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 63a8f0f8ef299f9d27d3c1be2746052536c4e5e8
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88066175"
---
# <a name="service-bus-management-libraries"></a>Bibliotecas de administración de Service Bus

Las bibliotecas de administración de Azure Service Bus pueden aprovisionar dinámicamente las entidades y los espacios de nombres de Service Bus. Esto posibilita los escenarios complejos de implementación y mensajería, y hace posible determinar mediante programación qué entidades aprovisionar. Estas bibliotecas están actualmente disponibles para .NET.

## <a name="supported-functionality"></a>Funcionalidad admitida

* Creación, actualización y eliminación de espacios de nombres
* Creación, actualización y eliminación de colas
* Creación, actualización y eliminación de temas
* Creación, actualización y eliminación de suscripciones

## <a name="prerequisites"></a>Prerrequisitos

Para comenzar a usar las bibliotecas de administración de Service Bus, debe autenticarse con el servicio Azure Active Directory (Azure AAD). Azure AAD requiere que se autentique como una entidad de servicio que proporciona acceso a los recursos de Azure. Para más información sobre cómo crear una entidad de servicio, consulte uno de los siguientes artículos:  

* [Uso de Azure Portal para crear una aplicación de Active Directory y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md)
* [Uso de Azure PowerShell para crear a una entidad de servicio para acceder a recursos](../active-directory/develop/howto-authenticate-service-principal-powershell.md)
* [Uso de la CLI de Azure para crear a una entidad de servicio para acceder a recursos](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

Estos tutoriales le proporcionan valores para `AppId` (identificador de cliente), `TenantId` y `ClientSecret` (clave de autenticación), que usan las bibliotecas de administración con fines de autenticación. Debe tener como mínimo los permisos [**propietario de datos**](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) o [**colaborador de Azure Service Bus**](../role-based-access-control/built-in-roles.md#contributor) para el grupo de recursos en el que quiere realizar la ejecución.

## <a name="programming-pattern"></a>Modelo de programación

El patrón para manipular los recursos de Service Bus sigue un protocolo común:

1. Obtenga un token de Azure AD utilizando la biblioteca **Microsoft.IdentityModel.Clients.ActiveDirectory**:
   ```csharp
   var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

   var result = await context.AcquireTokenAsync("https://management.azure.com/", new ClientCredential(clientId, clientSecret));
   ```
2. Cree el objeto `ServiceBusManagementClient`:

   ```csharp
   var creds = new TokenCredentials(token);
   var sbClient = new ServiceBusManagementClient(creds)
   {
       SubscriptionId = SettingsCache["SubscriptionId"]
   };
   ```
3. Establezca los parámetros de `CreateOrUpdate` en los valores especificados:

   ```csharp
   var queueParams = new QueueCreateOrUpdateParameters()
   {
       Location = SettingsCache["DataCenterLocation"],
       EnablePartitioning = true
   };
   ```
4. Ejecute la llamada:

   ```csharp
   await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, QueueName, queueParams);
   ```

## <a name="complete-code-to-create-a-queue"></a>Código completo para crear una cola
Este es el código completo para crear una cola de Service Bus: 

```csharp
using System;
using System.Threading.Tasks;

using Microsoft.Azure.Management.ServiceBus;
using Microsoft.Azure.Management.ServiceBus.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;

namespace SBusADApp
{
    class Program
    {
        static void Main(string[] args)
        {
            CreateQueue().GetAwaiter().GetResult();
        }

        private static async Task CreateQueue()
        {
            try
            {
                var subscriptionID = "<SUBSCRIPTION ID>";
                var resourceGroupName = "<RESOURCE GROUP NAME>";
                var namespaceName = "<SERVICE BUS NAMESPACE NAME>";
                var queueName = "<NAME OF QUEUE YOU WANT TO CREATE>";

                var token = await GetToken();

                var creds = new TokenCredentials(token);
                var sbClient = new ServiceBusManagementClient(creds)
                {
                    SubscriptionId = subscriptionID,
                };

                var queueParams = new SBQueue();

                Console.WriteLine("Creating queue...");
                await sbClient.Queues.CreateOrUpdateAsync(resourceGroupName, namespaceName, queueName, queueParams);
                Console.WriteLine("Created queue successfully.");
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not create a queue...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

        private static async Task<string> GetToken()
        {
            try
            {
                var tenantId = "<AZURE AD TENANT ID>";
                var clientId = "<APPLICATION/CLIENT ID>";
                var clientSecret = "<CLIENT SECRET>";

                var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

                var result = await context.AcquireTokenAsync(
                    "https://management.azure.com/",
                    new ClientCredential(clientId, clientSecret)
                );

                // If the token isn't a valid string, throw an error.
                if (string.IsNullOrEmpty(result.AccessToken))
                {
                    throw new Exception("Token result is empty!");
                }

                return result.AccessToken;
            }
            catch (Exception e)
            {
                Console.WriteLine("Could not get a token...");
                Console.WriteLine(e.Message);
                throw e;
            }
        }

    }
}
```

> [!IMPORTANT]
> Para ver un ejemplo completo, consulte el [ejemplo de administración de .NET en GitHub](https://github.com/Azure-Samples/service-bus-dotnet-management/). 

## <a name="next-steps"></a>Pasos siguientes
[Microsoft.Azure.Management.ServiceBus API reference](/dotnet/api/Microsoft.Azure.Management.ServiceBus) (Referencia de API de Microsoft.Azure.Management.ServiceBus)