---
title: Supervisión de eventos de Azure Media Services con Event Grid mediante la CLI | Microsoft Docs
description: En este artículo se muestra cómo puede suscribirse a Event Grid para supervisar eventos de Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: b840f00f23db1590422bae853fee070c3c43f899
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296761"
---
# <a name="create-and-monitor-media-services-events-with-event-grid-using-the-azure-cli"></a>Crear y supervisar eventos de Media Services con Event Grid mediante la CLI de Azure

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Event Grid es un servicio de eventos para la nube. Este servicio usa las [suscripciones a eventos](../../event-grid/concepts.md#event-subscriptions) para enrutar los mensajes de eventos a los suscriptores. Los eventos de Media Services contienen toda la información necesaria para responder a cualquier cambio que se produzca en los datos. Puede identificar un evento de Media Services porque la propiedad eventType comienza por "Microsoft.Media". Para más información, consulte el artículo sobre los [esquemas de eventos de Media Services](media-services-event-schemas.md).

En este artículo se usa la CLI de Azure para suscribirse a eventos para su cuenta de Azure Media Services. A continuación, deberá desencadenar esos eventos para ver el resultado. Por lo general, se envían eventos a un punto de conexión que procesa los datos del evento y realiza acciones. En este artículo, los eventos se envían a una aplicación web que recopila y muestra los mensajes.

## <a name="prerequisites"></a>Prerrequisitos

- Una suscripción de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
- Instale y use la CLI localmente, para este artículo es preciso usar la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión que tiene. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

    Actualmente no todos los comandos de la [CLI de Media Services v3](https://aka.ms/ams-v3-cli-ref) funcionan en Azure Cloud Shell. Se recomienda usar la CLI localmente.

- [Cree una cuenta de Media Services](./create-account-howto.md).

    Asegúrese de recordar los valores que usó para el nombre de la cuenta de Media Services y el nombre del grupo de recursos.

## <a name="create-a-message-endpoint"></a>Creación de un punto de conexión de mensaje

Antes de suscribirse a los eventos de la cuenta de Media Services, vamos a crear el punto de conexión para el mensaje del evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. En este artículo debe implementar una [aplicación web precompilada](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestre los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. Seleccione **Deploy to Azure** (Implementar en Azure) para implementar la solución en su suscripción. En Azure Portal, proporcione valores para los parámetros.

   [![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json)

1. La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

Si cambia al sitio "Visor de Azure Event Grid", verá que no tiene todavía ningún evento.
   
[!INCLUDE [event-grid-register-provider-portal.md](../../../includes/event-grid-register-provider-portal.md)]

## <a name="set-the-azure-subscription"></a>Establecimiento de la suscripción de Azure

En el siguiente comando, proporcione el identificador de suscripción de Azure que quiere usar para la cuenta de Media Services. Para ver una lista de las suscripciones a las que tiene acceso, vaya a [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

```azurecli
az account set --subscription mySubscriptionId
```

## <a name="subscribe-to-media-services-events"></a>Suscribirse a los eventos de Media Services

Suscríbase a un artículo para indicar a Event Grid los eventos cuyo seguimiento desea realizar. En el ejemplo siguiente se realiza la suscripción a la cuenta de Media Services que creó y se pasa la dirección URL del sitio web que creó como punto de conexión para la notificación de eventos. 

Reemplace `<event_subscription_name>` por un nombre único para la suscripción de eventos. Para `<resource_group_name>` y `<ams_account_name>`, use los valores que usó al crear la cuenta de Media Services. En el caso de `api/updates`, especifique la dirección URL de la aplicación web y agregue `<endpoint_URL>` a la dirección URL de la página principal. Al especificar el punto de conexión cuando se realiza la suscripción, Event Grid controla el enrutamiento de los eventos a dicho punto de conexión. 

1. Obtenga el identificador del recurso

    ```azurecli
    amsResourceId=$(az ams account show --name <ams_account_name> --resource-group <resource_group_name> --query id --output tsv)
    ```

    Por ejemplo:

    ```
    amsResourceId=$(az ams account show --name amsaccount --resource-group amsResourceGroup --query id --output tsv)
    ```

2. Suscríbase a los eventos

    ```azurecli
    az eventgrid event-subscription create \
    --source-resource-id $amsResourceId \
    --name <event_subscription_name> \
    --endpoint <endpoint_URL>
    ```

    Por ejemplo:

    ```
    az eventgrid event-subscription create --source-resource-id $amsResourceId --name amsTestEventSubscription --endpoint https://amstesteventgrid.azurewebsites.net/api/updates/
    ```    

    > [!TIP]
    > Es posible que reciba una advertencia del protocolo de enlace de validación. Espere unos minutos y el protocolo de enlace debería realizar la validación.

A continuación, desencadenaremos algunos eventos para ver cómo Event Grid distribuye el mensaje al punto de conexión.

## <a name="send-an-event-to-your-endpoint"></a>Envío de un evento al punto de conexión

Puede desencadenar eventos de la cuenta de Media Services si ejecuta un trabajo de codificación. Puede seguir [este inicio rápido](stream-files-dotnet-quickstart.md) para codificar un archivo y empezar a enviar eventos. 

Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. El punto de conexión debe establecer `validationResponse` en `validationCode`. Para más información, vea [Event Grid security and authentication](../../event-grid/security-authentication.md) (Seguridad y autenticación de Event Grid). Puede ver el código de la aplicación web para ver cómo valida la suscripción.

> [!TIP]
> Seleccione el icono del ojo para expandir los datos del evento. No actualice la página si quiere ver todos los eventos.

![Visualización del evento de suscripción](./media/monitor-events-portal/view-subscription-event.png)

## <a name="next-steps"></a>Pasos siguientes

[Carga, codificación y streaming](stream-files-tutorial-with-api.md)
