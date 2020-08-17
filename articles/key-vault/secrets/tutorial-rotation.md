---
title: Tutorial de rotación para recursos con un conjunto de credenciales de autenticación almacenado en Azure Key Vault
description: Use este tutorial para aprender a automatizar la rotación de secretos para los recursos que usan un conjunto de credenciales de autenticación.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 01/26/2020
ms.author: mbaldwin
ms.openlocfilehash: b61ba7f160d012cc3d9ad9f477e969a626fdc38e
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87541426"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-use-one-set-of-authentication-credentials"></a>Automatización de la rotación de un secreto para recursos que usan un conjunto de credenciales de autenticación

Aunque la mejor forma de realizar la autenticación en los servicios de Azure es mediante una [identidad administrada](../general/managed-identity.md), hay algunos escenarios en los que no es posible. En estos casos, se usan claves de acceso o secretos. Tanto las claves de acceso como los secretos deben rotarse periódicamente.

En este tutorial se muestra cómo automatizar la rotación periódica de secretos de bases de datos y servicios que usan un conjunto de credenciales de autenticación. Específicamente, en este tutorial se rotan las contraseñas de SQL Server almacenadas en Azure Key Vault mediante el uso de una función desencadenada por una notificación de Azure Event Grid:

![Diagrama de la solución de rotación](../media/rotate1.png)

1. Treinta días antes de la fecha de expiración de un secreto, Key Vault publica el evento de "expiración cercana" en Event Grid.
1. Event Grid comprueba las suscripciones del evento y usa HTTP POST para llamar al punto de conexión de la aplicación de funciones suscrita al evento.
1. La aplicación de funciones recibe la información del secreto, genera una nueva contraseña aleatoria y crea una versión del secreto con la contraseña nueva en Key Vault.
1. La aplicación de funciones actualiza SQL Server con la nueva contraseña.

> [!NOTE]
> Podría haber un retraso entre los pasos 3 y 4. Durante ese tiempo, el secreto de Key Vault no podrá autenticarse en SQL Server. En caso de que se produzca un error en cualquiera de los pasos, Event Grid hace reintentos durante dos horas.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure Key Vault
* SQL Server

Si no tiene una instancia de Key Vault y SQL Server, puede usar el vínculo de implementación siguiente:

[![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmaster%2Farm-templates%2FInitial-Setup%2Fazuredeploy.json)

1. En **Grupo de recursos**, seleccione **Crear nuevo**. Asigne al grupo el nombre **akvrotation**.
1. En **Inicio de sesión de administrador de SQL**, escriba el nombre de inicio de sesión del administrador de SQL. 
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**

    ![Crear un grupo de recursos](../media/rotate2.png)

Ahora ya tiene una instancia de Key Vault, una instancia de SQL Server y una base de datos SQL. Puede comprobar esta configuración en la CLI de Azure mediante la ejecución del comando siguiente:

```azurecli
az resource list -o table
```

El resultado tendrá un aspecto similar a la salida siguiente:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv          akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotation-sql         akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql/master  akvrotation      eastus      Microsoft.Sql/servers/databases
```

## <a name="create-and-deploy-sql-server-password-rotation-function"></a>Creación e implementación de la función de rotación de contraseñas de SQL Server

A continuación, cree una aplicación de funciones con una identidad administrada por el sistema, además de los otros componentes necesarios, e implemente las funciones de rotación de contraseña de SQL Server.

La aplicación de funciones requiere estos componentes:
- Un plan de Azure App Service
- Una aplicación de funciones con funciones de rotación de contraseñas SQL con desencadenador de eventos y desencadenador http. 
- Una cuenta de almacenamiento necesaria para la administración del desencadenador de la aplicación de funciones
- Una directiva de acceso para la identidad de Function App para acceder a los secretos de Key Vault.
- Una suscripción a los eventos de EventGrid para el evento **SecretNearExpiry**.

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmaster%2Farm-templates%2FFunction%2Fazuredeploy.json)

1. En la lista **Grupo de recursos**, seleccione **akvrotation**.
1. En **Nombre de SQL Server**, escriba el nombre de la instancia de SQL Server con la contraseña que se va a rotar.
1. En **Nombre del almacén de claves**, escriba el nombre del almacén de claves.
1. En **Nombre de la aplicación de funciones**, escriba el nombre de la aplicación de funciones.
1. En **Nombre del secreto**, escriba el nombre del secreto en el que se almacenará la contraseña.
1. En **Dirección URL del repositorio**, escriba la ubicación del código de la función en GitHub ( **https://github.com/jlichwa/KeyVault-Rotation-SQLPassword-Csharp.git** ).
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.

   ![Selección de Revisar y crear](../media/rotate3.png)

Tras completar los pasos anteriores tendrá una cuenta de almacenamiento, una granja de servidores y una aplicación de funciones. Puede comprobar esta configuración en la CLI de Azure mediante la ejecución del comando siguiente:

```azurecli
az resource list -o table
```

El resultado tendrá un aspecto similar al de la salida siguiente:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation       eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation       eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation       eastus      Microsoft.Sql/servers/databases
cfogyydrufs5wazfunctions akvrotation       eastus      Microsoft.Storage/storageAccounts
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/serverFarms
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/sites
akvrotation-fnapp        akvrotation       eastus      Microsoft.insights/components
```

Para obtener información sobre cómo crear una aplicación de funciones y usar una identidad administrada para acceder a Key Vault, consulte [Creación de una aplicación de funciones desde Azure Portal](../../azure-functions/functions-create-function-app-portal.md) y [Autenticación de Key Vault con una identidad administrada](../general/managed-identity.md).

### <a name="rotation-function"></a>Función de rotación
La función implementada en el paso anterior utiliza un evento para desencadenar la rotación de un secreto mediante la actualización de Key Vault y la base de datos SQL. 

#### <a name="function-trigger-event"></a>Evento de desencadenador de la función

Esta función lee los datos del evento y ejecuta la lógica de la rotación:

```csharp
public static class SimpleRotationEventHandler
{
   [FunctionName("AKVSQLRotation")]
   public static void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
   {
      log.LogInformation("C# Event trigger function processed a request.");
      var secretName = eventGridEvent.Subject;
      var secretVersion = Regex.Match(eventGridEvent.Data.ToString(), "Version\":\"([a-z0-9]*)").Groups[1].ToString();
      var keyVaultName = Regex.Match(eventGridEvent.Topic, ".vaults.(.*)").Groups[1].ToString();
      log.LogInformation($"Key Vault Name: {keyVaultName}");
      log.LogInformation($"Secret Name: {secretName}");
      log.LogInformation($"Secret Version: {secretVersion}");

      SecretRotator.RotateSecret(log, secretName, keyVaultName);
   }
}
```

#### <a name="secret-rotation-logic"></a>Lógica de rotación de secretos
Este método de rotación lee la información de la base de datos del secreto, crea una nueva versión del secreto y actualiza la base de datos con el nuevo secreto:

```csharp
    public class SecretRotator
    {
        private const string CredentialIdTag = "CredentialId";
        private const string ProviderAddressTag = "ProviderAddress";
        private const string ValidityPeriodDaysTag = "ValidityPeriodDays";

        public static void RotateSecret(ILogger log, string secretName, string keyVaultName)
        {
            //Retrieve Current Secret
            var kvUri = "https://" + keyVaultName + ".vault.azure.net";
            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());
            KeyVaultSecret secret = client.GetSecret(secretName);
            log.LogInformation("Secret Info Retrieved");

            //Retrieve Secret Info
            var credentialId = secret.Properties.Tags.ContainsKey(CredentialIdTag) ? secret.Properties.Tags[CredentialIdTag] : "";
            var providerAddress = secret.Properties.Tags.ContainsKey(ProviderAddressTag) ? secret.Properties.Tags[ProviderAddressTag] : "";
            var validityPeriodDays = secret.Properties.Tags.ContainsKey(ValidityPeriodDaysTag) ? secret.Properties.Tags[ValidityPeriodDaysTag] : "";
            log.LogInformation($"Provider Address: {providerAddress}");
            log.LogInformation($"Credential Id: {credentialId}");

            //Check Service Provider connection
            CheckServiceConnection(secret);
            log.LogInformation("Service  Connection Validated");
            
            //Create new password
            var randomPassword = CreateRandomPassword();
            log.LogInformation("New Password Generated");

            //Add secret version with new password to Key Vault
            CreateNewSecretVersion(client, secret, randomPassword);
            log.LogInformation("New Secret Version Generated");

            //Update Service Provider with new password
            UpdateServicePassword(secret, randomPassword);
            log.LogInformation("Password Changed");
            log.LogInformation($"Secret Rotated Successfully");
        }
}
```
Puede encontrar el código completo en [GitHub](https://github.com/jlichwa/KeyVault-Rotation-SQLPassword-Csharp).

## <a name="add-the-secret-to-key-vault"></a>Adición del secreto a Key Vault
Establezca una directiva de acceso para conceder a los usuarios el permiso para *administrar secretos*.

```azurecli
az keyvault set-policy --upn <email-address-of-user> --name akvrotation-kv --secret-permissions set delete get list
```

Cree un nuevo secreto con etiquetas que contengan el identificador de recurso de SQL Server, el nombre del inicio de sesión de SQL Server y el período de validez del secreto en días. Proporcione el nombre del secreto, la contraseña inicial de la base de datos SQL (en nuestro ejemplo, "Simple123") e incluya una fecha de expiración establecida para mañana.

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name sqlPassword --vault-name akvrotation-kv --value "Simple123" --tags "CredentialId=sqlAdmin" "ProviderAddress=<sql-database-resource-id>" "ValidityPeriodDays=90" --expires $tomorrowDate
```

La creación de un secreto con una fecha de expiración corta publicará un evento `SecretNearExpiry` en un plazo de 15 minutos, lo que a su vez desencadenará la función para rotar el secreto.

## <a name="test-and-verify"></a>Prueba y comprobación

Para comprobar que el secreto ha rotado, vaya a **Key Vault** > **Secretos**:

![Ir a Secretos](../media/rotate8.png)

Abra el secreto **sqlPassword** y vea la versión original y la rotada:

![Abrir el secreto sqluser](../media/rotate9.png)

### <a name="create-a-web-app"></a>Creación de una aplicación web

Para comprobar las credenciales de SQL, cree una aplicación web. Esta aplicación web obtendrá el secreto de Key Vault, extraerá la información y las credenciales de la base de datos SQL del secreto, y probará la conexión con SQL Server.

La aplicación web requiere estos componentes:
- Una aplicación web con una identidad administrada por el sistema.
- Una directiva de acceso para acceder a los secretos de Key Vault mediante la identidad administrada de la aplicación web.

1. Seleccione el vínculo de implementación de la plantilla de Azure: 

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-SQLPassword-Csharp-WebApp%2Fmaster%2Farm-templates%2FWeb-App%2Fazuredeploy.json)

1. Seleccione el grupo de recursos **akvrotation**.
1. En **Nombre de SQL Server**, escriba el nombre de la instancia de SQL Server con la contraseña que se va a rotar.
1. En **Nombre del almacén de claves**, escriba el nombre del almacén de claves.
1. En **Nombre del secreto**, escriba el nombre del secreto en el que se almacena la contraseña.
1. En **Dirección URL del repositorio**, escriba la ubicación del código de la aplicación web en GitHub ( **https://github.com/jlichwa/KeyVault-Rotation-SQLPassword-Csharp-WebApp.git** ).
1. Seleccione **Revisar + crear**.
1. Seleccione **Crear**.


### <a name="open-the-web-app"></a>Abrir la aplicación web

Vaya a la dirección URL de la aplicación implementada:
 
https://akvrotation-app.azurewebsites.net/

Cuando la aplicación se abra en el explorador, verá el **valor de secreto generado** y un valor de **Base de datos conectada** de *true*.

## <a name="learn-more"></a>Más información

- Tutorial: [Tutorial de rotación para recursos con dos conjuntos de credenciales](tutorial-rotation-dual.md).
- Introducción: [Supervisión de Key Vault con Azure Event Grid (versión preliminar)](../general/event-grid-overview.md)
- Procedimientos: [Recibir un correo electrónico al cambiar un secreto del almacén de claves](../general/event-grid-logicapps.md)
- [Esquema de eventos de Azure Event Grid para Azure Key Vault (versión preliminar)](../../event-grid/event-schema-key-vault.md)
