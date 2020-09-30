---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: 258908ed1118b0463e8c824cd8c699fb460dfff2
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945931"
---
## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet/).
- La versión más reciente de la [biblioteca cliente de .NET Core](https://dotnet.microsoft.com/download/dotnet-core) para su sistema operativo.
- Obtenga la versión más reciente de la [biblioteca cliente de .NET Identity](https://docs.microsoft.com/dotnet/api/azure.identity?view=azure-dotnet).
- Obtenga la versión más reciente de la [biblioteca cliente de administración de .NET](../../concepts/sdk-options.md).

## <a name="installing-the-client-library"></a>Instalación de la biblioteca cliente

En primer lugar, incluya la biblioteca cliente de administración de Communication Services en el proyecto de C#:

```csharp
using Azure.ResourceManager.Communication;
```

## <a name="subscription-id"></a>Id. de suscripción

Deberá saber el identificador de su suscripción a Azure. Se puede adquirir en el portal:

1.  Inicie sesión en la cuenta de Azure.
2.  Seleccione Suscripciones en la barra lateral izquierda.
3.  Seleccione la suscripción que necesite.
4.  Haga clic en Información general.
5.  Seleccione el identificador de la suscripción.

En este inicio rápido, se supone que ha almacenado el identificador de la suscripción en una variable de entorno denominada `AZURE_SUBSCRIPTION_ID`.

## <a name="authentication"></a>Autenticación

Para comunicarse con Azure Communication Services, primero debe autenticarse en Azure. Normalmente, lo hará mediante la identidad de entidad de servicio.

### <a name="option-1-managed-identity"></a>Opción 1: Identidad administrada

Si el código se ejecuta como un servicio de Azure, la manera más sencilla de autenticarse es adquirir una identidad administrada de Azure. Obtenga más información sobre las [identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

[Servicios de Azure que admiten identidades administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities)

[Cómo usar identidades administradas para App Service y Azure Functions](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet)

#### <a name="system-assigned-managed-identity"></a>[Identidad administrada asignada por el sistema](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet#add-a-system-assigned-identity)

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var acsClient = new CommunicationManagementClient(subscriptionId, new ManagedIdentityCredential());
```

#### <a name="user-assigned-managed-identity"></a>[Identidad administrada asignada por el usuario](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet#add-a-user-assigned-identity)

El valor de ClientId de la identidad administrada que creó debe pasarse al elemento `ManagedIdentityCredential` explícitamente.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = "AZURE_SUBSCRIPTION_ID";
var managedIdentityCredential = new ManagedIdentityCredential("AZURE_CLIENT_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, managedIdentityCredential);
```

### <a name="option-2-service-principal"></a>Opción 2: Entidad de servicio

En lugar de usar una identidad administrada, es posible que quiera autenticarse en Azure con una entidad de servicio que administre. Obtenga más información con la documentación sobre la [creación y administración de una entidad de servicio en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal).

Una vez que haya creado la entidad de servicio, deberá recopilar la siguiente información sobre esta en Azure Portal:

- **Id. de cliente**
- **Secreto de cliente**
- **Id. de inquilino**

Almacene estos valores en las variables de entorno denominadas `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` y `AZURE_TENANT_ID`, respectivamente. A continuación, puede crear un cliente de administración de Communication Services como el siguiente:

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var acsClient = new CommunicationManagementClient(subscriptionId, new EnvironmentCredential());
```

### <a name="option-3-user-identity"></a>Opción 3: Identidad del usuario

Si quiere llamar a Azure en nombre de un usuario interactivo, en lugar de usar una identidad de servicio, puede usar el código siguiente para crear un cliente de administración de Azure Communication Services. Se abrirá una ventana del explorador para pedir al usuario que especifique sus credenciales de MSA o Azure AD.

```csharp
using Azure.Identity;
using Azure.ResourceManager.Communication;
using Azure.ResourceManager.Communication.Models;
using System;
...
var subscriptionId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
var communicationServiceClient = new CommunicationManagementClient(subscriptionId, new InteractiveBrowserCredential());
```

## <a name="managing-communication-services-resources"></a>Administración de recursos de Communication Services

### <a name="interacting-with-azure-resources"></a>Interacción con los recursos de Azure

Ahora que se autenticó, puede usar el cliente de administración para realizar llamadas a la API.

En cada uno de los ejemplos siguientes, asignaremos los recursos de Communication Services a un grupo de recursos existente.

Si necesita crear un grupo de recursos, puede hacerlo mediante [Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal) o la [biblioteca cliente de Azure Resource Manager](https://github.com/Azure/azure-sdk-for-net/blob/master/doc/mgmt_preview_quickstart.md).

### <a name="create-and-manage-a-communication-services-resource"></a>Creación y administración de un recurso de Communication Services

La instancia del cliente de la biblioteca cliente de administración de Communication Services (``Azure.ResourceManager.Communication.CommunicationManagementClient``) se puede usar para realizar operaciones en los recursos de Communication Services.

#### <a name="create-a-communication-services-resource"></a>Creación de un recurso de Communication Services

Al crear un recurso de Communication Services, especificará el nombre del grupo de recursos y el nombre del recurso. Tenga en cuenta que la propiedad `Location` siempre estará `global` y, durante la versión preliminar pública, el valor de `DataLocation` debe ser `UnitedStates`.

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates"  };
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="update-a-communication-services-resource"></a>Actualización de un recurso de Communication Services

```csharp
...
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var resource = new CommunicationServiceResource { Location = "Global", DataLocation = "UnitedStates" };
resource.Tags.Add("environment","test");
resource.Tags.Add("department","tech");
// Use existing resource name and new resource object
var operation = await acsClient.CommunicationService.StartCreateOrUpdateAsync(resourceGroupName, resourceName, resource);
await operation.WaitForCompletionAsync();
```

#### <a name="list-all-communication-services-resources"></a>Enumeración de todos los recursos de Communication Services

```csharp
var resources = acsClient.CommunicationService.ListBySubscription();
foreach (var resource in resources)
{
    Console.WriteLine(resource.Name);
}
```

#### <a name="delete-a-communication-services-resource"></a>Eliminación de un recurso de Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
await acsClient.CommunicationService.StartDeleteAsync(resourceGroupName, resourceName);
```

## <a name="managing-keys-and-connection-strings"></a>Administración de las claves y las cadenas de conexión

Cada recurso de Communication Services tiene un par de claves de acceso y las cadenas de conexión correspondientes. Se puede acceder a estas claves con la biblioteca cliente de administración y, a continuación, pueden usarlas otras bibliotecas cliente de Communication Services para autenticarse en Azure Communication Services.

#### <a name="get-access-keys-for-a-communication-services-resource"></a>Obtención de las claves de acceso de un recurso de Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keys = await acsClient.CommunicationService.ListKeysAsync(resourceGroupName, resourceName);

Console.WriteLine(keys.Value.PrimaryConnectionString);
Console.WriteLine(keys.Value.SecondaryConnectionString);
```

#### <a name="regenerate-an-access-key-for-a-communication-services-resource"></a>Regeneración de una clave de acceso para un recurso de Communication Services

```csharp
var resourceGroupName = "myResourceGroupName";
var resourceName = "myResource";
var keyParams = new RegenerateKeyParameters { KeyType = KeyType.Primary };
var keys = await acsClient.CommunicationService.RegenerateKeyAsync(resourceGroupName, resourceName, keyParams);

Console.WriteLine(keys.Value.PrimaryKey);
```
