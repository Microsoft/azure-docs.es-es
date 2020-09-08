---
title: Procedimiento para usar una identidad administrad asignada por el sistema para acceder a datos de Azure Cosmos DB
description: Aprenda a configurar una identidad administrada de Azure Active Directory (Azure AD) asignada por el sistema (identidad de servicio administrada) para acceder a claves desde Azure Cosmos DB.
author: j-patrick
ms.service: cosmos-db
ms.topic: how-to
ms.date: 03/20/2020
ms.author: justipat
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 25ec74f3638ce857e4472d73a51e45f24c4df5ec
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997734"
---
# <a name="use-system-assigned-managed-identities-to-access-azure-cosmos-db-data"></a>Uso de identidades administradas asignadas por el sistema para acceder a datos de Azure Cosmos DB

En este artículo, configurará una solución *sólida e independiente de la rotación de claves* para acceder a las claves de Azure Cosmos DB mediante [identidades administradas](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md). En el ejemplo de este artículo se usa Azure Functions, pero puede usar cualquier servicio que admita identidades administradas. 

Aprenderá a crear una aplicación de funciones que puede acceder a datos de Azure Cosmos DB sin necesidad de copiar ninguna clave de Azure Cosmos DB. La aplicación de funciones se reactivará cada minuto y registrará la temperatura actual de un acuario. Para obtener información sobre cómo configurar una aplicación de funciones desencadenada por un temporizador, vea el artículo [Creación de una función en Azure que se desencadena mediante un temporizador](../azure-functions/functions-create-scheduled-function.md).

Para simplificar el escenario, la limpieza de los documentos de temperatura más antiguos se controla mediante una opción [Período de vida](./time-to-live.md) ya configurada. 

## <a name="assign-a-system-assigned-managed-identity-to-a-function-app"></a>Asignación de una identidad administrada asignada por el sistema a una aplicación de funciones

En este paso, asignará una identidad administrada asignada por el sistema a la aplicación de funciones.

1. En [Azure Portal](https://portal.azure.com/), abra el panel **Función de Azure** y vaya a la aplicación de funciones. 

1. Abra la pestaña **Características de la plataforma** > **Identidad**: 

   :::image type="content" source="./media/managed-identity-based-authentication/identity-tab-selection.png" alt-text="Captura de pantalla que muestra las características de Plataforma y las opciones de Identidad de la aplicación de funciones.":::

1. En la pestaña **Identidad**, **active** el **Estado** de identidad del sistema y seleccione **Guardar**. El panel **Identidad** debe tener este aspecto:  

   :::image type="content" source="./media/managed-identity-based-authentication/identity-tab-system-managed-on.png" alt-text="Captura de pantalla que muestra el estado de identidad del sistema activado.":::

## <a name="grant-access-to-your-azure-cosmos-account"></a>Concesión de acceso a la cuenta de Azure Cosmos

En este paso, asignará un rol a la identidad administrada asignada por el sistema de la aplicación de funciones. Azure Cosmos DB tiene varios roles integrados que se pueden asignar a la identidad administrada. Para esta solución, usará los dos roles siguientes:

|Rol integrado  |Descripción  |
|---------|---------|
|[Colaborador de cuenta de DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)|Puede administrar cuentas de Azure Cosmos DB. Permite la recuperación de claves de lectura/escritura. |
|[Rol de lector de cuentas de Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)|Puede leer los datos de cuentas de Azure Cosmos DB. Permite la recuperación de claves de lectura. |

> [!IMPORTANT]
> La compatibilidad con el control de acceso basado en roles en Azure Cosmos DB se aplica solo a las operaciones del plano de control. Las operaciones del plano de datos se protegen mediante claves maestras o tokens de recursos. Para más información, vea el artículo [Protección del acceso a los datos](secure-access-to-data.md).

> [!TIP] 
> Al asignar roles, asigne solo el acceso necesario. Si el servicio solo requiere leer datos, asigne el rol **Lector de cuentas de Cosmos DB** a la identidad administrada. Para más información sobre la importancia del acceso con privilegios mínimos, vea el artículo [Menor exposición de las cuentas con privilegios](../security/fundamentals/identity-management-best-practices.md#lower-exposure-of-privileged-accounts).

En este escenario, la aplicación de funciones leerá la temperatura del acuario y, luego, reescribirá esos datos en un contenedor de Azure Cosmos DB. Dado que la aplicación de funciones debe escribir los datos, deberá asignar el rol **Colaborador de cuenta de DocumentDB**. 

### <a name="assign-the-role-using-azure-portal"></a>Asignación del rol mediante Azure Portal

1. Inicie sesión en Azure Portal y vaya a la cuenta de Azure Cosmos DB. Abra el panel **Control de acceso (IAM)** y, después, la pestaña **Asignaciones de roles**:

   :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab.png" alt-text="Captura de pantalla que muestra el panel Control de acceso y la pestaña Asignaciones de roles.":::

1. Seleccione **Agregar** > **Agregar asignación de roles**.

1. El panel **Agregar asignación de roles** se abre a la derecha:

   :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab-add-role-pane.png" alt-text="Captura de pantalla que muestra el panel Agregar asignación de roles.":::

   * **Rol**: seleccione **Colaborador de cuenta de DocumentDB**.
   * **Asignar acceso a**: en la subsección **Select system-assigned managed identity** (Seleccionar identidad administrada asignada por el sistema), seleccione **Aplicación de funciones**.
   * **Select**: el panel se rellenará con todas las aplicaciones de funciones de la suscripción que tengan una **Identidad del sistema administrada**. En este caso, seleccione la aplicación de funciones **FishTankTemperatureService**: 

      :::image type="content" source="./media/managed-identity-based-authentication/cosmos-db-iam-tab-add-role-pane-filled.png" alt-text="Captura de pantalla que muestra el panel Agregar asignación de roles con ejemplos.":::

1. Tras seleccionar la aplicación de funciones, seleccione **Guardar**.

### <a name="assign-the-role-using-azure-cli"></a>Asignación del rol mediante la CLI de Azure

Para asignar el rol mediante la CLI de Azure, abra Azure Cloud Shell y ejecute los siguientes comandos:

```azurecli-interactive

scope=$(az cosmosdb show --name '<Your_Azure_Cosmos_account_name>' --resource-group '<CosmosDB_Resource_Group>' --query id)

principalId=$(az webapp identity show -n '<Your_Azure_Function_name>' -g '<Azure_Function_Resource_Group>' --query principalId)

az role assignment create --assignee $principalId --role "DocumentDB Account Contributor" --scope $scope
```

## <a name="programmatically-access-the-azure-cosmos-db-keys"></a>Acceso mediante programación a las claves de Azure Cosmos DB

Ahora tenemos una aplicación de funciones que tiene una identidad administrada asignada por el sistema con el rol **Colaborador de cuenta de DocumentDB** en los permisos de Azure Cosmos DB. El código de la aplicación de funciones siguiente obtendrá las claves de Azure Cosmos DB, creará un objeto CosmosClient, obtendrá la temperatura del acuario y, después, la guardará en Azure Cosmos DB.

En este ejemplo se usa la [API List Keys](/rest/api/cosmos-db-resource-provider/2020-04-01/databaseaccounts/listkeys) para acceder a las claves de la cuenta de Azure Cosmos DB.

> [!IMPORTANT] 
> Si quiere [asignar el rol Lector de cuentas de Cosmos DB](#grant-access-to-your-azure-cosmos-account), tendrá que usar la [List Read Only Keys API](/rest/api/cosmos-db-resource-provider/2020-04-01/databaseaccounts/listreadonlykeys). Esto solamente rellenará las claves de solo lectura.

La API List Keys devuelve el objeto `DatabaseAccountListKeysResult`. Este tipo no está definido en las bibliotecas de C#. En el código siguiente se muestra la implementación de esta clase:  

```csharp 
namespace Monitor 
{
  public class DatabaseAccountListKeysResult
  {
      public string primaryMasterKey {get;set;}
      public string primaryReadonlyMasterKey {get; set;}
      public string secondaryMasterKey {get; set;}
      public string secondaryReadonlyMasterKey {get;set;}
  }
}
```

En el ejemplo también se usa un documento simple denominado "TemperatureRecord", que se define de esta forma:

```csharp
using System;

namespace Monitor
{
    public class TemperatureRecord
    {
        public string id { get; set; } = Guid.NewGuid().ToString();
        public DateTime RecordTime { get; set; }
        public int Temperature { get; set; }

    }
}
```

Usará la biblioteca [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) para obtener el token de identidad administrada asignada por el sistema. Para aprender otras formas de conseguir el token y obtener más información sobre la biblioteca `Microsoft.Azure.Service.AppAuthentication`, vea el artículo [Autenticación entre servicios](../key-vault/general/service-to-service-authentication.md).


```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.Azure.Cosmos;
using Microsoft.Azure.Services.AppAuthentication;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

namespace Monitor
{
    public static class FishTankTemperatureService
    {
        private static string subscriptionId =
        "<azure subscription id>";
        private static string resourceGroupName =
        "<name of your azure resource group>";
        private static string accountName =
        "<Azure Cosmos DB account name>";
        private static string cosmosDbEndpoint =
        "<Azure Cosmos DB endpoint>";
        private static string databaseName =
        "<Azure Cosmos DB name>";
        private static string containerName =
        "<container to store the temperature in>";

        [FunctionName("FishTankTemperatureService")]
        public static async Task Run([TimerTrigger("0 * * * * *")]TimerInfo myTimer, ILogger log)
        {
            log.LogInformation($"Starting temperature monitoring: {DateTime.Now}");

            // AzureServiceTokenProvider will help us to get the Service Managed token.
            var azureServiceTokenProvider = new AzureServiceTokenProvider();

            // Authenticate to the Azure Resource Manager to get the Service Managed token.
            string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");

            // Setup the List Keys API to get the Azure Cosmos DB keys.
            string endpoint = $"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/listKeys?api-version=2019-12-12";

            // Setup an HTTP Client and add the access token.
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            // Post to the endpoint to get the keys result.
            var result = await httpClient.PostAsync(endpoint, new StringContent(""));

            // Get the result back as a DatabaseAccountListKeysResult.
            DatabaseAccountListKeysResult keys = await result.Content.ReadAsAsync<DatabaseAccountListKeysResult>();

            log.LogInformation("Starting to create the client");

            CosmosClient client = new CosmosClient(cosmosDbEndpoint, keys.primaryMasterKey);

            log.LogInformation("Client created");

            var database = client.GetDatabase(databaseName);
            var container = database.GetContainer(containerName);

            log.LogInformation("Get the temperature.");

            var tempRecord = new TemperatureRecord() { RecordTime = DateTime.UtcNow, Temperature = GetTemperature() };

            log.LogInformation("Store temperature");

            await container.CreateItemAsync<TemperatureRecord>(tempRecord);

            log.LogInformation($"Ending temperature monitor: {DateTime.Now}");
        }

        private static int GetTemperature()
        {
            // Fake the temperature sensor for this demo.
            Random r = new Random(DateTime.UtcNow.Second);
            return r.Next(0, 120);
        }
    }
}
```

Ya está preparado para [implementar la aplicación de funciones](../azure-functions/functions-create-first-function-vs-code.md).

## <a name="next-steps"></a>Pasos siguientes

* [Autenticación basada en certificados con Azure Active Directory y Azure Cosmos DB](certificate-based-authentication.md)
* [Protección de las claves de Azure Cosmos DB con Azure Key Vault](access-secrets-from-keyvault.md)
* [Línea de base de seguridad para Azure Cosmos DB](security-baseline.md)
