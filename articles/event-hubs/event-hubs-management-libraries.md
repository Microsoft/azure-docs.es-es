---
title: Bibliotecas de administración de Azure Event Hubs | Microsoft Docs
description: En este artículo se proporciona información sobre la biblioteca que puede usar para administrar las entidades y espacios de nombres de Azure Event Hubs desde. NET.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 74392fbf0b2c0b81898410af8027a4f13fc52b67
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89014003"
---
# <a name="event-hubs-management-libraries"></a>Bibliotecas de administración de Event Hubs

Puede usar las bibliotecas de administración de Azure Event Hubs para aprovisionar de forma dinámica las entidades y los espacios de nombres de Event Hubs. Esta naturaleza dinámica posibilita escenarios complejos de implementación y mensajería, lo que permite determinar mediante programación qué entidades aprovisionar. Estas bibliotecas están actualmente disponibles para .NET.

## <a name="supported-functionality"></a>Funcionalidad admitida

* Creación, actualización y eliminación de espacios de nombres
* Creación, actualización y eliminación de Event Hubs
* Creación, actualización y eliminación de grupos de consumidores

## <a name="prerequisites"></a>Prerequisites

Para comenzar a usar las bibliotecas de administración de Event Hubs, debe autenticarse con Azure Active Directory (AAD). AAD requiere que se autentique como una entidad de servicio que proporciona acceso a los recursos de Azure. Para más información sobre cómo crear una entidad de servicio, consulte uno de los siguientes artículos:  

* [Uso de Azure Portal para crear una aplicación de Active Directory y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md)
* [Uso de Azure PowerShell para crear a una entidad de servicio para acceder a recursos](../active-directory/develop/howto-authenticate-service-principal-powershell.md)
* [Uso de la CLI de Azure para crear a una entidad de servicio para acceder a recursos](/cli/azure/create-an-azure-service-principal-azure-cli)

Estos tutoriales le proporcionan valores para `AppId` (identificador de cliente), `TenantId` y `ClientSecret` (clave de autenticación), que usan las bibliotecas de administración con fines de autenticación. Debe tener permisos de **Propietario** en el grupo de recursos en el que desea realizar la ejecución.

## <a name="programming-pattern"></a>Modelo de programación

El patrón para manipular los recursos de Event Hubs sigue un protocolo común:

1. Obtenga un token de AAD mediante la biblioteca `Microsoft.IdentityModel.Clients.ActiveDirectory`.
    ```csharp
    var context = new AuthenticationContext($"https://login.microsoftonline.com/{tenantId}");

    var result = await context.AcquireTokenAsync(
        "https://management.core.windows.net/",
        new ClientCredential(clientId, clientSecret)
    );
    ```

1. Cree el objeto `EventHubManagementClient`.
    ```csharp
    var creds = new TokenCredentials(token);
    var ehClient = new EventHubManagementClient(creds)
    {
        SubscriptionId = SettingsCache["SubscriptionId"]
    };
    ```

1. Establezca los parámetros de `CreateOrUpdate` en los valores especificados.
    ```csharp
    var ehParams = new EventHubCreateOrUpdateParameters()
    {
        Location = SettingsCache["DataCenterLocation"]
    };
    ```

1. Ejecute la llamada.
    ```csharp
    await ehClient.EventHubs.CreateOrUpdateAsync(resourceGroupName, namespaceName, EventHubName, ehParams);
    ```

## <a name="next-steps"></a>Pasos siguientes
* [Ejemplo de administración de .NET](https://github.com/Azure-Samples/event-hubs-dotnet-management/)
* [Referencia de Microsoft.Azure.Management.EventHub](/dotnet/api/Microsoft.Azure.Management.EventHub) 
