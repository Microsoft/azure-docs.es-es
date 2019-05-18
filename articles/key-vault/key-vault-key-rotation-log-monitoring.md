---
title: Configuración de Azure Key Vault con la auditoría y la rotación de claves de un extremo a otro | Microsoft Docs
description: Use esta guía de procedimientos que le ayudarán a configurar la rotación de claves y supervise los registros del almacén de claves.
services: key-vault
author: barclayn
manager: barbkess
tags: ''
ms.service: key-vault
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: barclayn
ms.openlocfilehash: 20a170963ff4a8ff9cb69d3397e66e12c1047d16
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65561185"
---
# <a name="set-up-azure-key-vault-with-key-rotation-and-auditing"></a>Configuración de Azure Key Vault con la auditoría y la rotación de claves

## <a name="introduction"></a>Introducción

Cuando disponga de un almacén de claves, puede empezar a usarlo para almacenar claves y secretos. Ya no será necesario que las claves y los secretos se guarden en las aplicaciones, sino que estas los solicitarán al almacén cuando sea preciso. Un almacén de claves permite actualizar las claves y secretos sin afectar al comportamiento de la aplicación, que abre un abanico de posibilidades para la clave y la administración de secretos.

>[!IMPORTANT]
> Los ejemplos de este artículo se proporcionan únicamente con fines ilustrativos. No está pensados para su uso en producción. 

Este artículo ofrece lo siguiente:

- Un ejemplo de uso de Azure Key Vault para almacenar un secreto. En este artículo, el secreto almacenado es una aplicación tiene acceso a la clave de cuenta de almacenamiento de Azure. 
- Cómo implementar una rotación programada de dicha clave.
- Supervisión de la clave del almacén de registros de auditoría y generar alertas cuando se realizan solicitudes inesperadas.

> [!NOTE]
> En este artículo no se explica en detalle la configuración inicial del almacén de claves. Para obtener más información, consulte [¿Qué es Azure Key Vault?](key-vault-overview.md). Para obtener instrucciones de la interfaz de línea de comandos multiplataforma, consulte [administrar mediante la CLI de Azure Key Vault](key-vault-manage-with-cli2.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="set-up-key-vault"></a>Configuración de Key Vault

Para que una aplicación pueda recuperar un secreto de Key Vault, primero debe crear el secreto y guardarlo en el almacén.

Inicie una sesión de PowerShell de Azure e inicie sesión en su cuenta de Azure con el siguiente comando:

```powershell
Connect-AzAccount
```

En la ventana emergente del explorador, escriba el nombre de usuario y la contraseña para la cuenta de Azure. PowerShell obtendrá todas las suscripciones asociadas a esta cuenta. PowerShell utiliza la primera de forma predeterminada.

Si tiene varias suscripciones, es posible que deba especificar la que se usó para crear el almacén de claves. Escriba lo siguiente para ver las suscripciones de su cuenta:

```powershell
Get-AzSubscription
```

Para especificar la suscripción que está asociado con el almacén de claves que se iniciará sesión, escriba:

```powershell
Set-AzContext -SubscriptionId <subscriptionID>
```

Dado que en este artículo se explica cómo se guarda una clave de cuenta de almacenamiento como un secreto, debe obtener dicha clave.

```powershell
Get-AzStorageAccountKey -ResourceGroupName <resourceGroupName> -Name <storageAccountName>
```

Después de recuperar el secreto (en este caso, la clave de cuenta de almacenamiento), debe convertir esa clave en una cadena segura y, a continuación, crear un secreto con ese valor en el almacén de claves.

```powershell
$secretvalue = ConvertTo-SecureString <storageAccountKey> -AsPlainText -Force

Set-AzKeyVaultSecret -VaultName <vaultName> -Name <secretName> -SecretValue $secretvalue
```

A continuación, obtenga el identificador URI para el secreto que creó. Necesitará este URI en un paso posterior para llamar a key vault y recuperar el secreto. Ejecute el siguiente comando de PowerShell y anote el valor de identificador, que es el URI del secreto:

```powershell
Get-AzKeyVaultSecret –VaultName <vaultName>
```

## <a name="set-up-the-application"></a>Configuración de la aplicación

Ahora que tiene un secreto almacenado, puede usar código para recuperar y utilizarlo después de realizar algunos pasos más.

En primer lugar, debe registrar la aplicación con Azure Active Directory. A continuación, indicar a Key Vault la información de la aplicación para que pueda permitir que las solicitudes de la aplicación.

> [!NOTE]
> La aplicación debe crearse en el mismo inquilino de Azure Active Directory que el del almacén de claves.

1. Abra **Azure Active Directory**.
2. Seleccione **App registrations** (Registros de aplicaciones). 
3. Seleccione **nuevo registro de aplicaciones** para agregar una aplicación a Azure Active Directory.

    ![Abra las aplicaciones de Azure Active Directory](./media/keyvault-keyrotation/azure-ad-application.png)

4. En **crear**, deje el tipo de aplicación como **aplicación Web / API** y asigne un nombre de la aplicación. Asigne a la aplicación un **dirección URL de inicio de sesión**. Esta dirección URL puede ser cualquier cosa que desee para esta demostración.

    ![Crear registro de aplicación](./media/keyvault-keyrotation/create-app.png)

5. Después de agregar la aplicación a Azure Active Directory, se abre la página de aplicación. Seleccione **configuración**y, a continuación, seleccione **propiedades**. Copie el valor **Id. de aplicación**. La necesitará en pasos posteriores.

A continuación, genere una clave para la aplicación para que pueda interactuar con Azure Active Directory. Para crear una clave, seleccione **claves** en **configuración**. Tome nota de la clave recién generada para la aplicación de Azure Active Directory. La necesitará en otro paso más adelante. La clave no estará disponible después de salir de esta sección. 

![Claves de aplicación de Azure Active Directory](./media/keyvault-keyrotation/create-key.png)

Antes de establecer las llamadas desde la aplicación en el almacén de claves, debe indicar el almacén de claves de la aplicación y sus permisos. El siguiente comando usa el nombre del almacén y el identificador de aplicación de la aplicación de Azure Active Directory para conceder a la aplicación **obtener** acceso al almacén de claves.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName <vaultName> -ServicePrincipalName <clientIDfromAzureAD> -PermissionsToSecrets Get
```

Ahora está listo para comenzar a crear las llamadas a la aplicación. En la aplicación, debe instalar los paquetes de NuGet necesarios para interactuar con Azure Key Vault y Azure Active Directory. En la Consola del Administrador de paquetes de Visual Studio, escriba los siguientes comandos. Cuando se redactó este artículo, la versión actual del paquete de Azure Active Directory es 3.10.305231913, por lo que confirme la versión más reciente y actualizar según sea necesario.

```powershell
Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 3.10.305231913

Install-Package Microsoft.Azure.KeyVault
```

En el código de la aplicación, cree una clase que contenga el método de autenticación de Azure Active Directory. En este ejemplo, la clase se llama **Utils**. Agregue la siguiente instrucción `using` :

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

A continuación, agregue el método siguiente para recuperar el token JWT de Azure Active Directory. Para el mantenimiento, puede mover los valores de cadena codificados de forma rígida en la configuración web o aplicación.

```csharp
public async static Task<string> GetToken(string authority, string resource, string scope)
{
    var authContext = new AuthenticationContext(authority);

    ClientCredential clientCred = new ClientCredential("<AzureADApplicationClientID>","<AzureADApplicationClientKey>");

    AuthenticationResult result = await authContext.AcquireTokenAsync(resource, clientCred);

    if (result == null)

    throw new InvalidOperationException("Failed to obtain the JWT token");

    return result.AccessToken;
}
```

Agregue el código necesario para llamar a Key Vault y recuperar el valor del secreto. En primer lugar, debe agregar lo siguiente `using` instrucción:

```csharp
using Microsoft.Azure.KeyVault;
```

Agregue las llamadas de método para invocar a Key Vault y recuperar el secreto. En este método, proporcione el URI del secreto que guardó en un paso anterior. Tenga en cuenta el uso de la **GetToken** método desde el **Utils** clase que creó anteriormente.

```csharp
var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(Utils.GetToken));

var sec = kv.GetSecretAsync(<SecretID>).Result.Value;
```

Cuando ejecute la aplicación, debería estar autenticado en Azure Active Directory y recuperar el valor del secreto de Azure Key Vault.

## <a name="key-rotation-using-azure-automation"></a>Rotación de claves mediante Azure Automation

> [!IMPORTANT]
> Runbooks de automatización de Azure siguen requiriendo el uso de la `AzureRM` módulo.

Ahora está listo para configurar una estrategia de rotación de los valores almacenados como secretos de Key Vault. Los secretos se pueden girar de varias maneras:

- Como parte de un proceso manual
- Mediante programación usando las llamadas de API
- A través de un script de automatización de Azure

Para los fines de este artículo, usará PowerShell y Azure Automation para cambiar la clave de acceso de una cuenta de almacenamiento de Azure. A continuación, actualizará un secreto de almacén de claves con esa nueva clave.

Para permitir que Azure Automation pueda establecer los valores de secreto del almacén de claves, debe obtener el identificador de cliente para la conexión denominada **AzureRunAsConnection**. Esta conexión se creó al establecer la instancia de Azure Automation. Para buscar este Id., seleccione **activos** desde la instancia de Azure Automation. Desde allí, seleccionar **conexiones**y, a continuación, seleccione el **AzureRunAsConnection** entidad de servicio. Tome nota de la **ApplicationId** valor.

![Identificador de cliente de Azure Automation](./media/keyvault-keyrotation/Azure_Automation_ClientID.png)

En **activos**, seleccione **módulos**. Seleccione **galería**, a continuación, buscar e importar versiones actualizadas de cada uno de los siguientes módulos:

    Azure
    Azure.Storage
    AzureRM.Profile
    AzureRM.KeyVault
    AzureRM.Automation
    AzureRM.Storage

> [!NOTE]
> Cuando se redactó este artículo, los módulos que aparecen arriba eran los únicos que debían actualizarse para el script que se muestra a continuación. Si se produce un error en el trabajo de automatización, confirme que ha importado todos los módulos necesarios y sus dependencias.

Una vez que haya recuperado el identificador de aplicación para la conexión de Azure Automation, debe indicar que el almacén de claves que la aplicación tiene permiso para actualizar los secretos del almacén. Use el siguiente comando de PowerShell:

```powershell
Set-AzKeyVaultAccessPolicy -VaultName <vaultName> -ServicePrincipalName <applicationIDfromAzureAutomation> -PermissionsToSecrets Set
```

A continuación, seleccione **Runbooks** en su instancia de Azure Automation y, a continuación, seleccione **agregar un Runbook**. Seleccione **Creación rápida**. Nombre del runbook y seleccione **PowerShell** como el tipo de runbook. Puede agregar una descripción. Por último, seleccione **Crear**.

![Creación de runbook](./media/keyvault-keyrotation/Create_Runbook.png)

En el panel del editor del nuevo runbook, pegue el siguiente script de PowerShell:

```powershell
$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "AzureRunAsConnection"
    $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         

    "Logging in to Azure..."
    Connect-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint
    "Login complete."
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

# Optionally you can set the following as parameters
$StorageAccountName = <storageAccountName>
$RGName = <storageAccountResourceGroupName>
$VaultName = <keyVaultName>
$SecretName = <keyVaultSecretName>

#Key name. For example key1 or key2 for the storage account
New-AzureRmStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName -KeyName "key2" -Verbose
$SAKeys = Get-AzureRmStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName

$secretvalue = ConvertTo-SecureString $SAKeys[1].Value -AsPlainText -Force

$secret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $SecretName -SecretValue $secretvalue
```

En el panel del editor, seleccione **panel prueba** para probar la secuencia de comandos. Después de la secuencia de comandos se ejecuta sin errores, puede seleccionar **publicar**, y, a continuación, se puede aplicar una programación para el runbook en el panel de configuración.

## <a name="key-vault-auditing-pipeline"></a>Canalización de auditorías de Key Vault

Cuando configure un almacén de claves, puede habilitar la auditoría para que se recopilen registros de las solicitudes de acceso a dicho almacén. Estos registros se almacenan en una cuenta de almacenamiento de Azure designada y pueden extraerse, supervisarse y analizarse. El siguiente escenario usa las funciones de Azure, Azure logic apps y los registros de auditoría del almacén de claves para crear una canalización que envía un correo electrónico cuando una aplicación que no coincide con el identificador de aplicación de la aplicación web recupera los secretos del almacén.

En primer lugar, debe habilitar los registros en el almacén de claves. Use los siguientes comandos de PowerShell. (Puede ver los detalles completos de [en este artículo sobre el registro de key vault](key-vault-logging.md).)

```powershell
$sa = New-AzStorageAccount -ResourceGroupName <resourceGroupName> -Name <storageAccountName> -Type Standard\_LRS -Location 'East US'
$kv = Get-AzKeyVault -VaultName '<vaultName>'
Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Category AuditEvent
```

Después de habilita el registro, los registros de auditoría inicio que se almacenan en la cuenta de almacenamiento designada. Estos registros incluirán eventos acerca de qué usuario, cómo y cuándo ha obtenido acceso a los almacenes de claves.

> [!NOTE]
> Puede acceder a la información de registro 10 minutos después de la operación del almacén de claves. A menudo estará disponible antes de.

El siguiente paso consiste en la [creación de una cola de Azure Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md). Esta cola es donde se insertan los registros de auditoría del almacén de claves. Cuando los mensajes de registro de auditoría en la cola, la aplicación lógica las recoge y actúa sobre ellos. Cree una instancia de Service Bus con los pasos siguientes:

1. Crear un espacio de nombres de Service Bus (si ya tiene uno de los que desea usar, vaya al paso 2).
2. Vaya a la instancia de Service Bus en Azure portal y seleccione el espacio de nombres que desea crear la cola.
3. Seleccione **crear un recurso** > **Enterprise Integration Pack** > **Service Bus**y, a continuación, especifique los detalles necesarios.
4. Buscar la información de conexión de Service Bus, seleccione el espacio de nombres y, a continuación, seleccione **información de conexión**. Necesitará esta información para la sección siguiente.

A continuación, [crear una función de Azure](../azure-functions/functions-create-first-azure-function.md) para sondear los registros del almacén de claves dentro de la cuenta de almacenamiento y recopilar nuevos eventos. Esta función se desencadena según una programación.

Para crear una aplicación de función de Azure, seleccione **crear un recurso**, busque en marketplace **Function App**y, a continuación, seleccione **crear**. Durante el proceso de creación, puede usar un plan de hospedaje existente o crear uno nuevo. También puede optar por hospedaje dinámico. Para obtener más información acerca de las opciones de alojamiento para Azure Functions, consulte [cómo escalar Azure Functions](../azure-functions/functions-scale.md).

Una vez creada la aplicación de función de Azure, vaya a él y seleccione el **temporizador** escenario y **C\#**  para el idioma. A continuación, seleccione **crear esta función**.

![Hoja de inicio de Azure Functions](./media/keyvault-keyrotation/Azure_Functions_Start.png)

En la pestaña **Desarrollar**, sustituya el código de run.csx por el siguiente:

```csharp
#r "Newtonsoft.Json"

using System;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.ServiceBus.Messaging;
using System.Text;

public static void Run(TimerInfo myTimer, TextReader inputBlob, TextWriter outputBlob, TraceWriter log)
{
    log.Info("Starting");

    CloudStorageAccount sourceStorageAccount = new CloudStorageAccount(new StorageCredentials("<STORAGE_ACCOUNT_NAME>", "<STORAGE_ACCOUNT_KEY>"), true);

    CloudBlobClient sourceCloudBlobClient = sourceStorageAccount.CreateCloudBlobClient();

    var connectionString = "<SERVICE_BUS_CONNECTION_STRING>";
    var queueName = "<SERVICE_BUS_QUEUE_NAME>";

    var sbClient = QueueClient.CreateFromConnectionString(connectionString, queueName);

    DateTime dtPrev = DateTime.UtcNow;
    if(inputBlob != null)
    {
        var txt = inputBlob.ReadToEnd();

        if(!string.IsNullOrEmpty(txt))
        {
            dtPrev = DateTime.Parse(txt);
            log.Verbose($"SyncPoint: {dtPrev.ToString("O")}");
        }
        else
        {
            dtPrev = DateTime.UtcNow;
            log.Verbose($"Sync point file didnt have a date. Setting to now.");
        }
    }

    var now = DateTime.UtcNow;

    string blobPrefix = "insights-logs-auditevent/resourceId=/SUBSCRIPTIONS/<SUBSCRIPTION_ID>/RESOURCEGROUPS/<RESOURCE_GROUP_NAME>/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/<KEY_VAULT_NAME>/y=" + now.Year +"/m="+now.Month.ToString("D2")+"/d="+ (now.Day).ToString("D2")+"/h="+(now.Hour).ToString("D2")+"/m=00/";

    log.Info($"Scanning:  {blobPrefix}");

    IEnumerable<IListBlobItem> blobs = sourceCloudBlobClient.ListBlobs(blobPrefix, true);

    log.Info($"found {blobs.Count()} blobs");

    foreach(var item in blobs)
    {
        if (item is CloudBlockBlob)
        {
            CloudBlockBlob blockBlob = (CloudBlockBlob)item;

            log.Info($"Syncing: {item.Uri}");

            string sharedAccessUri = GetContainerSasUri(blockBlob);

            CloudBlockBlob sourceBlob = new CloudBlockBlob(new Uri(sharedAccessUri));

            string text;
            using (var memoryStream = new MemoryStream())
            {
                sourceBlob.DownloadToStream(memoryStream);
                text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
            }

            dynamic dynJson = JsonConvert.DeserializeObject(text);

            //Required to order by time as they might not be in the file
            var results = ((IEnumerable<dynamic>) dynJson.records).OrderBy(p => p.time);

            foreach (var jsonItem in results)
            {
                DateTime dt = Convert.ToDateTime(jsonItem.time);

                if(dt>dtPrev){
                    log.Info($"{jsonItem.ToString()}");

                    var payloadStream = new MemoryStream(Encoding.UTF8.GetBytes(jsonItem.ToString()));
                    //When sending to ServiceBus, use the payloadStream and set keeporiginal to true
                    var message = new BrokeredMessage(payloadStream, true);
                    sbClient.Send(message);
                    dtPrev = dt;
                }
            }
        }
    }
    outputBlob.Write(dtPrev.ToString("o"));
}

static string GetContainerSasUri(CloudBlockBlob blob)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();

    sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5);
    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
    sasConstraints.Permissions = SharedAccessBlobPermissions.Read;

    //Generate the shared access signature on the container, setting the constraints directly on the signature.
    string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

    //Return the URI string for the container, including the SAS token.
    return blob.Uri + sasBlobToken;
}
```

> [!NOTE]
> Cambiar las variables en el código anterior para que apunte a la cuenta de almacenamiento donde se escriben los registros del almacén de claves, en la instancia de Service Bus que creó anteriormente y a la ruta de acceso específica a los registros de almacenamiento de almacén de claves.

La función selecciona el archivo de registro más reciente de la cuenta de almacenamiento en la que se escriben los registros del almacén de claves, recopila los últimos eventos de ese archivo y los inserta en una cola de Service Bus. 

Dado que un único archivo puede tener varios eventos, debe crear un archivo sync.txt que la función también se examina para determinar la marca de tiempo del último evento seleccionado. Con este archivo, se garantiza que no se inserta el mismo evento varias veces. 

El archivo sync.txt contiene una marca de tiempo para el evento ha encontrado el último. Cuando se cargan los registros, se debe ordenar en función de sus marcas de tiempo para asegurarse de que está ordenados correctamente.

Para esta función, se hacen referencia a un par bibliotecas adicionales que no están disponibles de fábrica en Azure Functions. Para incluir estas bibliotecas, necesitamos que Azure Functions las extraiga mediante NuGet. En el **código** cuadro, seleccione **ver archivos**.

![Opción "Ver archivos"](./media/keyvault-keyrotation/Azure_Functions_ViewFiles.png)

Agregue un archivo denominado project.json con el siguiente contenido:

```json
    {
      "frameworks": {
        "net46":{
          "dependencies": {
                "WindowsAzure.Storage": "7.0.0",
                "WindowsAzure.ServiceBus":"3.2.2"
          }
        }
       }
    }
```

Después de seleccionar **guardar**, Azure Functions descargará los archivos binarios necesarios.

Cambie a la pestaña **Integrar** y asigne al parámetro del temporizador un nombre descriptivo para usarlo en la función. En el código anterior, la función espera que el temporizador para llamarse *myTimer*. Especifique un [expresión CRON](../app-service/webjobs-create.md#CreateScheduledCRON) para el temporizador, como se indica a continuación: `0 * * * * *`. Esta expresión hará que la función que se ejecuta una vez por minuto.

En el mismo **integrar** pestaña, agregue una entrada del tipo **almacenamiento de blobs de Azure**. Esta entrada apuntará al archivo sync.txt que contiene la marca de tiempo del último evento que consultó la función. Esta entrada se accederá dentro de la función con el nombre de parámetro. En el código anterior, la entrada de almacenamiento de blobs de Azure espera que el parámetro name sea *inputBlob*. Seleccione la cuenta de almacenamiento donde se encontrará el archivo sync.txt (podría ser la misma o una cuenta de almacenamiento diferente). En el campo ruta de acceso, proporcione la ruta de acceso al archivo en el formato `{container-name}/path/to/sync.txt`.

Agregar una salida del tipo **almacenamiento de blobs de Azure**. Esta salida apuntará al archivo sync.txt que definió en la entrada. Esta salida se usa la función para escribir la marca de tiempo del último evento que consultó. En el código anterior, el nombre esperado para esta parámetro es *outputBlob*.

La función ya está lista. No olvide volver a la pestaña **Desarrollar** y guardar el código. La ventana de salida para los errores de compilación y corríjalos según sea necesario. Si compila el código, a continuación, el código debe ahora se comprobando los registros de almacén de claves de cada minuto e insertar los nuevos eventos en la cola de Service Bus definida. Cada vez que la función se desencadena, debería aparecer la información de los registros en la ventana del registro.

### <a name="azure-logic-app"></a>Aplicación lógica de Azure

A continuación, debe crear una aplicación de lógica de Azure que recoge los eventos que la función se inserta en la cola de Service Bus, analiza el contenido y envía un correo electrónico según una condición que se coteja.

[Crear una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md) seleccionando **crear un recurso** > **integración** > **aplicación lógica**.

Una vez creada la aplicación lógica, vaya a él y seleccione **editar**. En el editor de aplicación lógica, seleccione **cola de Service Bus** y escriba sus credenciales de Service Bus para conectarla a la cola.

![Service Bus de la aplicación lógica de Azure](./media/keyvault-keyrotation/Azure_LogicApp_ServiceBus.png)

Seleccione **Agregar una condición**. En la condición, cambie al editor avanzado y escriba el código siguiente. Reemplace *valor APP_ID* con el identificador de aplicación real de la aplicación web:

```
@equals('<APP_ID>', json(decodeBase64(triggerBody()['ContentData']))['identity']['claim']['appid'])
```

Esta expresión devuelve esencialmente **false** si el *appid* del evento entrante (que es el cuerpo del mensaje de Service Bus) no es el *appid* de la aplicación.

Ahora, cree una acción en **si NO, no haga nada**.

![Azure Logic Apps elegir acción](./media/keyvault-keyrotation/Azure_LogicApp_Condition.png)

Para la acción, seleccione **Office 365 - enviar correo electrónico**. Rellene los campos para crear el correo electrónico que se enviará cuando la condición definida devuelva **false**. Si no tiene Office 365, busque otras alternativas para conseguir los mismos resultados.

Ahora dispone de una canalización de extremo a otro que busca nuevos registros de auditoría del almacén de claves una vez por minuto. Inserta nuevos registros que se encuentra en una cola de Service Bus. La aplicación lógica se activa cuando un nuevo mensaje llega a la cola. Si el *appid* dentro del evento no coincide con el identificador de aplicación de la aplicación que llama, envía un correo electrónico.
