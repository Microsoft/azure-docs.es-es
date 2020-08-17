---
title: Envío de eventos a un punto de conexión web mediante Azure App Configuration
description: Aprenda a configurar las suscripciones a eventos de Azure App Configuration para que envíen eventos de modificación de pares clave-valor a un punto de conexión web
services: azure-app-configuration
author: lisaguthrie
ms.assetid: ''
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: how-to
ms.date: 02/25/2020
ms.author: lcozzens
ms.custom: devx-track-azurecli
ms.openlocfilehash: dc816f54a690a4c72c44d70ecbf2cc0156ac84ed
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498317"
---
# <a name="route-azure-app-configuration-events-to-a-web-endpoint-with-azure-cli"></a>Enrutamiento de eventos de Azure App Configuration a un punto de conexión web con la CLI de Azure

En este artículo, aprenderá a configurar las suscripciones a eventos de Azure App Configuration para que envíen eventos de modificación de pares clave-valor a un punto de conexión web. Los usuarios de Azure App Configuration pueden suscribirse a eventos que se emiten cada vez que se modifican los pares clave-valor. Estos eventos pueden desencadenar webhooks, instancias de Azure Functions, colas de Azure Storage o cualquier otro controlador de eventos que sea compatible con Azure Event Grid. Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. Sin embargo, para simplificar en este artículo, los eventos se envían a una aplicación web que recopila y muestra los mensajes.

## <a name="prerequisites"></a>Prerequisites

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/). También puede usar Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para los fines de este artículo, es preciso que ejecute la versión más reciente de la CLI de Azure (2.0.70 o posterior). Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Si no usa Cloud Shell, primero debe iniciar sesión con `az login`.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los temas de Event Grid son recursos de Azure y se deben colocar en un grupo de recursos de Azure. El grupo de recursos de Azure es una colección lógica en la que se implementan y administran los recursos de Azure.

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). 

En el ejemplo siguiente se crea un grupo de recursos llamado `<resource_group_name>` en la ubicación *eastus*.  Reemplace `<resource_group_name>` por un nombre único para grupo de recursos.

```azurecli-interactive
az group create --name <resource_group_name> --location westus
```

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

Reemplace `<appconfig_name>` por el nombre único de la instancia del almacén de configuración y `<resource_group_name>` por el grupo de recursos que creó anteriormente. El nombre debe ser único porque se utiliza como un nombre DNS.

```azurecli-interactive
az appconfig create \
  --name <appconfig_name> \
  --location westus \
  --resource-group <resource_group_name> \
  --sku free
```

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Antes de suscribirse al tema, vamos a crear el punto de conexión para el mensaje de evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. Para simplificar esta guía de inicio rápido, se implementa una [aplicación web pregenerada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestra los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

Reemplace `<your-site-name>` por un nombre único para la aplicación web. El nombre de la aplicación web debe ser único, ya que es parte de la entrada DNS.

```azurecli-interactive
$sitename=<your-site-name>

az group deployment create \
  --resource-group <resource_group_name> \
  --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
  --parameters siteName=$sitename hostingPlanName=viewerhost
```

La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

Debería ver el sitio, que no muestra ningún mensaje actualmente.

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-your-app-configuration-store"></a>Suscripción al almacén de App Configuration

Suscríbase a un tema que indique a Event Grid los eventos cuyo seguimiento desea realizar y el lugar al que deben enviarse dichos eventos. En el ejemplo siguiente se suscribirá a la instancia de App Configuration que ha creado y pasará la dirección URL de la aplicación web como el punto de conexión de la notificación de eventos. Reemplace `<event_subscription_name>` por un nombre para la suscripción de eventos. En `<resource_group_name>` y `<appconfig_name>`, use los valores que creó anteriormente.

El punto de conexión de la aplicación web debe incluir el sufijo `/api/updates/`.

```azurecli-interactive
appconfigId=$(az appconfig show --name <appconfig_name> --resource-group <resource_group_name> --query id --output tsv)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --resource-id $appconfigId \
  --name <event_subscription_name> \
  --endpoint $endpoint
```

Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

![Visualización del evento de suscripción](./media/quickstarts/event-grid/view-subscription-event.png)

## <a name="trigger-an-app-configuration-event"></a>Desencadenamiento de un evento de App Configuration

Ahora, vamos a desencadenar un evento para ver cómo Event Grid distribuye el mensaje al punto de conexión. Cree un par clave-valor mediante `<appconfig_name>` que apareció anteriormente.

```azurecli-interactive
az appconfig kv set --name <appconfig_name> --key Foo --value Bar --yes
```

Ha desencadenado el evento y Event Grid ha enviado el mensaje al punto de conexión que configuró al realizar la suscripción. Vaya a la aplicación web para ver el evento que acaba de enviar.

```json
[{
  "id": "deb8e00d-8c64-4b6e-9cab-282259c7674f",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/eventDemoGroup/providers/microsoft.appconfiguration/configurationstores/{appconfig-name}",
  "subject": "https://{appconfig-name}.azconfig.io/kv/Foo",
  "data": {
    "key": "Foo",
    "etag": "a1LIDdNEIV6wCnfv3xaip7fMXD3"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueModified",
  "eventTime": "2019-05-31T18:59:54Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

## <a name="clean-up-resources"></a>Limpieza de recursos
Si planea seguir trabajando con esta instancia de App Configuration y la suscripción de eventos, no elimine los recursos creados en este artículo. Si no va a continuar, use el siguiente comando para eliminar los recursos creados en este artículo.

Sustituya `<resource_group_name>` por el nombre del grupo de recursos que ha creado.

```azurecli-interactive
az group delete --name <resource_group_name>
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo crear suscripciones a temas y eventos, obtenga más información acerca de los eventos de los pares clave-valor y lo que Event Grid puede ayudarle a hacer:

- [Reacción ante eventos de pares clave-valor](concept-app-configuration-event.md)
- [Una introducción a Azure Event Grid](../event-grid/overview.md)
- [Controladores de Azure Event Grid](../event-grid/event-handlers.md)
