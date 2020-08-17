---
title: Introducción a la integración de Azure Service Bus en Event Grid | Microsoft Docs
description: En este artículo se proporciona una descripción de cómo se integra la mensajería de Azure Service Bus con Azure Event Grid.
documentationcenter: .net
author: spelluru
ms.topic: conceptual
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurecli
ms.openlocfilehash: 641cae751b0b87d93235e1272bc5f46f7797d834
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88065138"
---
# <a name="azure-service-bus-to-event-grid-integration-overview"></a>Introducción a la integración de Azure Service Bus en Event Grid

Azure Service Bus ha iniciado una nueva integración en Azure Event Grid. El principal escenario de esta característica es que aquellas colas o suscripciones de Service Bus que tengan un bajo volumen de mensajes, no tienen que tener un sondeo de receptor de mensajes en todo momento. 

Service Bus puede ahora emitir eventos a Event Grid cuando haya mensajes en una cola o suscripción y no haya ningún receptor presente. Puede crear suscripciones de Event Grid para los espacios de nombres de Service Bus, escuchar estos eventos y reaccionar a ellos iniciando un receptor. Con esta característica, puede usar Service Bus en modelos de programación reactiva.

Para habilitar la característica necesita los siguientes elementos:

* Un espacio de nombres Premium de Service Bus con al menos una cola de Service Bus o un tema de Service Bus que tenga como mínimo una suscripción.
* Acceso de colaborador al espacio de nombres de Service Bus.
* Además, se necesita una suscripción a Event Grid para el espacio de nombres de Service Bus. Esta suscripción recibe una notificación de Event Grid que indica que hay mensajes que se deben recopilar. Los suscriptores típicos pueden ser la característica Logic Apps de Azure App Service, Azure Functions, o un webhook que se ponga en contacto con una aplicación web. El suscriptor, a continuación, procesa los mensajes. 

![19][]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="verify-that-you-have-contributor-access"></a>Comprobación de que dispone de acceso de colaborador
Vaya al espacio de nombres de Service Bus, seleccione **Control de acceso (IAM)** y, a continuación, seleccione la pestaña **Asignaciones de roles**. Compruebe que dispone de acceso de colaborador al espacio de nombres. 

### <a name="events-and-event-schemas"></a>Eventos y esquemas de evento

Service Bus envía actualmente eventos para dos escenarios:

* [ActiveMessagesWithNoListenersAvailable](#active-messages-available-event)
* DeadletterMessagesAvailable

Además, Service Bus usa la seguridad estándar de Event Grid y [mecanismos de autenticación](../event-grid/security-authentication.md).

Para más información, consulte [Esquema de eventos de Azure Event Grid](../event-grid/event-schema.md).

#### <a name="active-messages-available-event"></a>Evento de mensajes activos disponibles

Este evento se genera si hay mensajes activos en una cola o suscripción y no hay receptores a la escucha.

El esquema para este evento es el siguiente:

```JSON
{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}
```

#### <a name="dead-letter-messages-available-event"></a>Evento de mensajes fallidos disponibles

Obtiene un evento como mínimo por cada cola de mensajes fallidos que contenga mensajes y ningún receptor activo.

El esquema para este evento es el siguiente:

```JSON
[{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="how-many-events-are-emitted-and-how-often"></a>¿Cuál es la frecuencia y la cantidad de eventos que se emiten?

Si tiene varias colas y temas o suscripciones en el espacio de nombres, obtendrá al menos un evento por cola y uno por suscripción. Los eventos se emiten inmediatamente si no hay ningún mensaje en la entidad de Service Bus y llega un mensaje nuevo. O bien, los eventos se emiten cada dos minutos, a menos que Service Bus detecte un receptor activo. El examen de los mensajes no interrumpe los eventos.

De forma predeterminada, Service Bus emite eventos para todas las entidades del espacio de nombres. Si desea obtener eventos solo para entidades concretas, consulte la siguiente sección.

### <a name="use-filters-to-limit-where-you-get-events-from"></a>Use filtros para limitar de dónde se obtienen los eventos

Por ejemplo, si desea obtener eventos solo para una cola o suscripción del espacio de nombres, puede utilizar los filtros *Begins with* (Comienza por) o *Ends with* (Termina con) que proporciona Event Grid. En algunas interfaces, los filtros se denominan *Prefijo* y *Sufijo*. Si desea obtener eventos para varias colas y suscripciones (pero no para todas), puede crear varias suscripciones de Event Grid y proporcionar un filtro para cada una.

## <a name="create-event-grid-subscriptions-for-service-bus-namespaces"></a>Creación de suscripciones de Event Grid para espacios de nombres de Service Bus

Puede crear suscripciones de Event Grid para los espacios de nombres de Service Bus de tres formas diferentes:

* En el Portal de Azure
* En la [CLI de Azure](#azure-cli-instructions)
* En [PowerShell](#powershell-instructions)

## <a name="azure-portal-instructions"></a>Instrucciones de Azure Portal

Para crear una nueva suscripción de Event Grid, haga lo siguiente:
1. En Azure Portal, vaya hasta su espacio de nombres.
2. En el panel izquierdo, seleccione **Event Grid**. 
3. Seleccione **Suscripción de eventos**.  

   La siguiente imagen muestra un espacio de nombres que tiene una suscripción de Event Grid:

   ![Suscripciones de Event Grid](./media/service-bus-to-event-grid-integration-concept/sbtoeventgridportal.png)

   La siguiente imagen muestra cómo suscribirse a una función o a un webhook sin ningún filtro concreto:

   ![21][]

## <a name="azure-cli-instructions"></a>Instrucciones para la CLI de Azure

En primer lugar, asegúrese de que dispone de la CLI de Azure versión 2.0 o posterior instalada. [Descargue el instalador](/cli/azure/install-azure-cli?view=azure-cli-latest). Seleccione **Windows + X** y abra una nueva consola de PowerShell con permisos de administrador. Alternativamente, puede usar un shell de comandos en Azure Portal.

Ejecute el código siguiente:

 ```azurecli-interactive
az login

az account set -s "<Azure subscription name>"

namespaceid=$(az resource show --namespace Microsoft.ServiceBus --resource-type namespaces --name "<service bus namespace>" --resource-group "<resource group that contains the service bus namespace>" --query id --output tsv

az eventgrid event-subscription create --resource-id $namespaceid --name "<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>" --endpoint "<your_function_url>" --subject-ends-with "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
```

Si está usando BASH 

## <a name="powershell-instructions"></a>Instrucciones para PowerShell

Asegúrese de que tiene instalado Azure PowerShell. [Descargue el instalador](/powershell/azure/install-Az-ps). Seleccione **Windows + X** y abra una nueva consola de PowerShell con permisos de administrador. Alternativamente, puede usar un shell de comandos en Azure Portal.

```powershell-interactive
Connect-AzAccount

Select-AzSubscription -SubscriptionName "<YOUR SUBSCRIPTION NAME>"

# This might be installed already
Install-Module Az.ServiceBus

$NSID = (Get-AzServiceBusNamespace -ResourceGroupName "<YOUR RESOURCE GROUP NAME>" -Na
mespaceName "<YOUR NAMESPACE NAME>").Id

New-AzEVentGridSubscription -EventSubscriptionName "<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>" -ResourceId $NSID -Endpoint "<YOUR FUNCTION URL>” -SubjectEndsWith "<YOUR SERVICE BUS SUBSCRIPTION NAME>"
```

Desde aquí, puede explorar las demás opciones de configuración o comprobar que los eventos fluyen.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga [ejemplos](service-bus-to-event-grid-integration-example.md) de Service Bus y Event Grid.
* Más información acerca de [Event Grid](../event-grid/index.yml).
* Más información acerca de [Azure Functions](../azure-functions/index.yml).
* Más información acerca de [Logic Apps](../logic-apps/index.yml).
* Más información acerca de [Service Bus](/azure/service-bus/).

[1]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgrid1.png
[19]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgriddiagram.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png