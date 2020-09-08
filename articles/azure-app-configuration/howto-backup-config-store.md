---
title: Copia de seguridad automática de valores de clave del almacén de Azure App Configuration
description: Obtenga información sobre cómo configurar una copia de seguridad automática de valores de clave entre los almacenes de App Configuration
services: azure-app-configuration
author: avanigupta
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.custom: devx-track-dotnet
ms.topic: how-to
ms.date: 04/27/2020
ms.author: avgupta
ms.openlocfilehash: a3c1699dd4b7b828c7dc652f14f431878f785061
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88207148"
---
# <a name="back-up-app-configuration-stores-automatically"></a>Copia de seguridad automática de almacenes de App Configuration

En este artículo, obtendrá información sobre cómo configurar una copia de seguridad automática de los valores de clave de un almacén de Azure App Configuration primario en un almacén secundario. La copia de seguridad automática aprovecha la integración de Azure Event Grid en App Configuration. 

Después de haber configurado la copia de seguridad automática, App Configuration publicará eventos en Azure Event Grid para cualquier cambio realizado en los valores de clave de un almacén de configuración. Event Grid es compatible con una gran variedad de servicios de Azure desde los que los usuarios pueden suscribirse a los eventos emitidos cada vez que se crean, actualizan o eliminan los valores de clave.

## <a name="overview"></a>Información general

En este tutorial, usará Azure Queue Storage para recibir eventos de Event Grid y usará un desencadenador de temporizador de Azure Functions para procesar en lotes los eventos de la cola. 

Cuando se desencadene una función, según los eventos, esta capturará los valores más recientes de las claves que han cambiado desde el almacén de App Configuration primario y actualizará el almacén secundario según corresponda. Esta configuración ayuda a combinar varios cambios que se producen en un breve período en una operación de copia de seguridad, lo que evita realizar solicitudes excesivas a los almacenes de App Configuration.

![Diagrama que muestra la arquitectura de la copia de seguridad de almacenes de App Configuration.](./media/config-store-backup-architecture.png)

## <a name="resource-provisioning"></a>Aprovisionamiento de recursos

El motivo de crear copias de seguridad de los almacenes de App Configuration es el uso de varios almacenes de configuración en diferentes regiones de Azure para aumentar la resistencia geográfica de la aplicación. Para lograrlo, los almacenes primario y secundario deben estar en regiones distintas de Azure. Todos los demás recursos creados en este tutorial se pueden aprovisionar en cualquier región que elija. Ya que si la región primaria está inactiva, no habrá nada nuevo en la copia de seguridad hasta que se pueda acceder a ella de nuevo.

En este tutorial, creará un almacén secundario en la región `centralus`, y todos los demás recursos en la región `westus`.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción de Azure. [cree una de forma gratuita](https://azure.microsoft.com/free/). 
- [Visual Studio 2019](https://visualstudio.microsoft.com/vs) con la carga de trabajo de desarrollo de Azure.
- [SDK de .NET Core](https://dotnet.microsoft.com/download).
- La versión más reciente de la CLI de Azure (2.3.1 o posterior). Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si usa la CLI de Azure, primero debe iniciar sesión con `az login`. También puede usar Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

El grupo de recursos de Azure es una colección lógica en la que se implementan y administran los recursos de Azure.

Cree un grupo de recursos con el comando [az group create](/cli/azure/group).

En el ejemplo siguiente, se crea un grupo de recursos denominado `<resource_group_name>` en la ubicación `westus`.  Reemplace `<resource_group_name>` por un nombre único para grupo de recursos.

```azurecli-interactive
resourceGroupName="<resource_group_name>"
az group create --name $resourceGroupName --location westus
```

## <a name="create-app-configuration-stores"></a>Creación de almacenes de App Configuration

Cree los almacenes de App Configuration primario y secundario en diferentes regiones.
Reemplace `<primary_appconfig_name>` y `<secondary_appconfig_name>` por nombres únicos para los almacenes de configuración. El nombre de cada almacén debe ser único, porque se usa como nombre DNS.

```azurecli-interactive
primaryAppConfigName="<primary_appconfig_name>"
secondaryAppConfigName="<secondary_appconfig_name>"
az appconfig create \
  --name $primaryAppConfigName \
  --location westus \
  --resource-group $resourceGroupName\
  --sku standard

az appconfig create \
  --name $secondaryAppConfigName \
  --location centralus \
  --resource-group $resourceGroupName\
  --sku standard
```

## <a name="create-a-queue"></a>Creación de una cola

Cree una cuenta de almacenamiento y una cola para recibir los eventos que publica Event Grid.

```azurecli-interactive
storageName="<unique_storage_name>"
queueName="<queue_name>"
az storage account create -n $storageName -g $resourceGroupName -l westus --sku Standard_LRS
az storage queue create --name $queueName --account-name $storageName --auth-mode login
```

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-your-app-configuration-store-events"></a>Suscripción a los eventos del almacén de App Configuration

Debe suscribirse a estos dos eventos del almacén de App Configuration primario:

- `Microsoft.AppConfiguration.KeyValueModified`
- `Microsoft.AppConfiguration.KeyValueDeleted`

El siguiente comando crea una suscripción de Event Grid para los dos eventos que se envían a la cola. El tipo de punto de conexión se establece en `storagequeue`, y el punto de conexión se establece en el identificador de la cola. Reemplace `<event_subscription_name>` por el nombre que quiera para la suscripción de eventos.

```azurecli-interactive
storageId=$(az storage account show --name $storageName --resource-group  $resourceGroupName --query id --output tsv)
queueId="$storageId/queueservices/default/queues/$queueName"
appconfigId=$(az appconfig show --name $primaryAppConfigName --resource-group $resourceGroupName --query id --output tsv)
eventSubscriptionName="<event_subscription_name>"
az eventgrid event-subscription create \
  --source-resource-id $appconfigId \
  --name $eventSubscriptionName \
  --endpoint-type storagequeue \
  --endpoint $queueId \
  --included-event-types Microsoft.AppConfiguration.KeyValueModified Microsoft.AppConfiguration.KeyValueDeleted 
```

## <a name="create-functions-for-handling-events-from-queue-storage"></a>Creación de funciones para controlar eventos de Queue Storage

### <a name="set-up-with-ready-to-use-functions"></a>Configuración con funciones listas para usar

En este artículo, trabajará con funciones de C# que tienen las siguientes propiedades:
- Pila en tiempo de ejecución de .NET Core 3.1
- Versión 3.x del entorno en tiempo de ejecución de Azure Functions
- Función que desencadena un temporizador cada 10 minutos

Para que le resulte más fácil empezar a realizar copias de seguridad de los datos, hemos [probado y publicado una función](https://github.com/Azure/AppConfiguration/tree/master/examples/ConfigurationStoreBackup) que puede usar sin modificar el código. Descargue los archivos del proyecto y [publíquelos en su propia aplicación de funciones de Azure desde Visual Studio](/azure/azure-functions/functions-develop-vs#publish-to-azure).

> [!IMPORTANT]
> No realice ningún cambio en las variables de entorno del código que ha descargado. Creará las opciones de configuración de la aplicación necesarias en la sección siguiente.
>

### <a name="build-your-own-function"></a>Creación de una función propia

Si el código de ejemplo proporcionado anteriormente no cumple sus requisitos, también puede crear su propia función. La función debe ser capaz de realizar las siguientes tareas para completar la copia de seguridad:
- Lea periódicamente el contenido de la cola para saber si contiene notificaciones de Event Grid. Consulte el [SDK de la cola de Storage](/azure/storage/queues/storage-quickstart-queues-dotnet) para conocer los detalles de implementación.
- Si la cola contiene [notificaciones de eventos de Event Grid](/azure/azure-app-configuration/concept-app-configuration-event?branch=pr-en-us-112982#event-schema), extraiga toda la información exclusiva de `<key, label>` de los mensajes de eventos. La combinación de clave y etiqueta es el identificador único para los cambios de pares clave-valor en el almacén primario.
- Lea toda la configuración del almacén primario. Actualice solo los valores del almacén secundario que tengan un evento correspondiente en la cola. Elimine toda la configuración del almacén secundario que estaba presente en la cola, pero no en el almacén primario. Puede usar el [SDK de App Configuration](https://github.com/Azure/AppConfiguration#sdks) para acceder a los almacenes de configuración mediante programación.
- Elimine los mensajes de la cola si no hubo ninguna excepción durante el procesamiento.
- Implemente el control de errores de acuerdo con sus necesidades. Consulte el ejemplo de código anterior para ver algunas excepciones comunes que quizá quiera administrar.

Para obtener más información sobre la creación de una función, consulte: [Cree una función en Azure que se desencadena mediante un temporizador](/azure/azure-functions/functions-create-scheduled-function) y [Desarrollo de Azure Functions con Visual Studio](/azure/azure-functions/functions-develop-vs).


> [!IMPORTANT]
> Elija con criterio la programación del temporizador en función de la frecuencia con la que realiza cambios en el almacén de configuración primario. Ejecutar la función con demasiada frecuencia podría limitar las solicitudes al almacén.
>


## <a name="create-function-app-settings"></a>Creación de la configuración de la aplicación de funciones

Si usa una función proporcionada, necesitará la siguiente configuración de la aplicación en la aplicación de funciones:
- `PrimaryStoreEndpoint`: Punto de conexión para el almacén de App Configuration primario. Un ejemplo es `https://{primary_appconfig_name}.azconfig.io`.
- `SecondaryStoreEndpoint`: Punto de conexión para el almacén de App Configuration secundario. Un ejemplo es `https://{secondary_appconfig_name}.azconfig.io`.
- `StorageQueueUri`: URI de la cola. Un ejemplo es `https://{unique_storage_name}.queue.core.windows.net/{queue_name}`.

El siguiente comando crea la configuración de la aplicación necesaria en la aplicación de funciones. Reemplace `<function_app_name>` por el nombre de la aplicación de función.

```azurecli-interactive
functionAppName="<function_app_name>"
primaryStoreEndpoint="https://$primaryAppConfigName.azconfig.io"
secondaryStoreEndpoint="https://$secondaryAppConfigName.azconfig.io"
storageQueueUri="https://$storageName.queue.core.windows.net/$queueName"
az functionapp config appsettings set --name $functionAppName --resource-group $resourceGroupName --settings StorageQueueUri=$storageQueueUri PrimaryStoreEndpoint=$primaryStoreEndpoint SecondaryStoreEndpoint=$secondaryStoreEndpoint
```


## <a name="grant-access-to-the-managed-identity-of-the-function-app"></a>Concesión de acceso a la identidad administrada de la aplicación de funciones

Use el comando siguiente o [Azure Portal](/azure/app-service/overview-managed-identity#add-a-system-assigned-identity) para agregar una identidad administrada asignada por el sistema para la aplicación de funciones.

```azurecli-interactive
az functionapp identity assign --name $functionAppName --resource-group $resourceGroupName
```

> [!NOTE]
> Para crear los recursos necesarios y administrar los roles, la cuenta debe tener permisos de `Owner` en el ámbito adecuado (en su suscripción o en un grupo de recursos). Si necesita ayuda con las asignaciones de roles, consulte [Incorporación o eliminación de asignaciones de roles mediante Azure Portal](/azure/role-based-access-control/role-assignments-portal).

Use los siguientes comandos o [Azure Portal](/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity#grant-access-to-app-configuration) para conceder a la identidad administrada de la aplicación de funciones acceso a los almacenes de App Configuration. Use estos roles:
- Asigne el rol `App Configuration Data Reader` al almacén de App Configuration primario.
- Asigne el rol `App Configuration Data Owner` al almacén de App Configuration secundario.

```azurecli-interactive
functionPrincipalId=$(az functionapp identity show --name $functionAppName --resource-group  $resourceGroupName --query principalId --output tsv)
primaryAppConfigId=$(az appconfig show -n $primaryAppConfigName --query id --output tsv)
secondaryAppConfigId=$(az appconfig show -n $secondaryAppConfigName --query id --output tsv)

az role assignment create \
    --role "App Configuration Data Reader" \
    --assignee $functionPrincipalId \
    --scope $primaryAppConfigId

az role assignment create \
    --role "App Configuration Data Owner" \
    --assignee $functionPrincipalId \
    --scope $secondaryAppConfigId
```

Use el siguiente comando o [Azure Portal](/azure/storage/common/storage-auth-aad-rbac-portal#assign-azure-roles-using-the-azure-portal) para conceder a la identidad administrada de la aplicación de funciones acceso a la cola. Asigne el rol `Storage Queue Data Contributor` en la cola.

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Contributor" \
    --assignee $functionPrincipalId \
    --scope $queueId
```

## <a name="trigger-an-app-configuration-event"></a>Desencadenamiento de un evento de App Configuration

Para probar que todo funciona, puede crear, actualizar o eliminar un valor de clave del almacén primario. Este cambio debería reflejarse automáticamente en el almacén secundario unos segundos después de que el temporizador desencadene la instancia de Azure Functions.

```azurecli-interactive
az appconfig kv set --name $primaryAppConfigName --key Foo --value Bar --yes
```

Ha desencadenado el evento. En unos instantes, Event Grid enviará la notificación de eventos a la cola. *Después de la siguiente ejecución programada de la función*, consulte los valores de configuración en el almacén secundario para comprobar si este contiene el valor de clave actualizado del almacén primario.

> [!NOTE]
> Puede [desencadenar manualmente la función](/azure/azure-functions/functions-manually-run-non-http) durante las pruebas y la solución de problemas sin necesidad de esperar al desencadenador de temporizador programado.

Después de asegurarse de que la función de copia de seguridad se ha ejecutado correctamente, podrá ver que la clave ya estará presente en el almacén secundario.

```azurecli-interactive
az appconfig kv show --name $secondaryAppConfigName --key Foo
```

```json
{
  "contentType": null,
  "etag": "eVgJugUUuopXnbCxg0dB63PDEJY",
  "key": "Foo",
  "label": null,
  "lastModified": "2020-04-27T23:25:08+00:00",
  "locked": false,
  "tags": {},
  "value": "Bar"
}
```

## <a name="troubleshooting"></a>Solución de problemas

Si la nueva configuración no se muestra en el almacén secundario:

- Asegúrese de que la función de copia de seguridad se haya desencadenado *después* de crear la configuración en el almacén primario.
- Es posible que Event Grid no haya podido enviar la notificación de eventos a la cola a tiempo. Compruebe si la cola aún contiene la notificación de eventos del almacén primario. Si es así, vuelva a desencadenar la función de copia de seguridad.
- Compruebe los [registros de Azure Functions](/azure/azure-functions/functions-create-scheduled-function#test-the-function) para comprobar si hay algún error o advertencia.
- Use [Azure Portal](/azure/azure-functions/functions-how-to-use-azure-function-app-settings#get-started-in-the-azure-portal) para asegurarse de que la aplicación de funciones de Azure contiene los valores correctos para la configuración de la aplicación que Azure Functions está intentando leer.
- También puede configurar la supervisión y las alertas de Azure Functions con [Azure Application Insights.](/azure/azure-functions/functions-monitoring?tabs=cmd) 


## <a name="clean-up-resources"></a>Limpieza de recursos
Si planea seguir trabajando con esta instancia de App Configuration y la suscripción de eventos, no elimine los recursos creados en este artículo. Si no va a continuar, use el siguiente comando para eliminar los recursos creados en este artículo.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo configurar la copia de seguridad automática de los valores de clave, obtenga más información sobre cómo puede aumentar la resistencia geográfica de su aplicación:

- [Resistencia y recuperación ante desastres](concept-disaster-recovery.md)
