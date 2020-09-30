---
title: Notificaciones en Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Envíe notificaciones a los usuarios de aplicaciones basadas en Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 5be2ff48ea5510c54c79e76e8bae082bd5085794
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945354"
---
# <a name="communication-services-notifications"></a>Notificaciones de Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Las bibliotecas cliente de llamadas y chat de Azure Communication Services crean un canal de mensajería en tiempo real que permite insertar mensajes de señalización en los clientes conectados de una manera eficaz y confiable. De este modo, puede incluir una funcionalidad de comunicación enriquecida y en tiempo real en sus aplicaciones sin necesidad de implementar una lógica complicada de sondeo HTTP. Sin embargo, en las aplicaciones móviles, este canal de señalización solo permanece conectado si la aplicación está activa en primer plano. Si quiere que los usuarios reciban llamadas entrantes o mensajes de chat mientras la aplicación está en segundo plano, debe usar las notificaciones push.

Las notificaciones push permiten enviar información de la aplicación a los dispositivos móviles de los usuarios. Puede usar las notificaciones push para mostrar un cuadro de diálogo, reproducir un sonido o mostrar la interfaz de usuario de llamadas entrantes. Azure Communication Services ofrece integraciones con [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/overview) y [Azure Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-overview) que permiten agregar notificaciones push a las aplicaciones.

## <a name="trigger-push-notifications-via-azure-event-grid"></a>Desencadenamiento de notificaciones a través de Azure Event Grid

Azure Communication Services se integra con [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) para ofrecer notificaciones de eventos en tiempo real de manera confiable, escalable y segura. Puede aprovechar esta integración para crear un servicio de notificaciones que entregue notificaciones push móviles a los usuarios al crear una suscripción a Event Grid que desencadene una [función de Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-overview) o un webhook.

:::image type="content" source="./media/notifications/acs-events-int.png" alt-text="Diagrama que muestra cómo se integra Communication Services con Event Grid.":::

Obtenga más información sobre el [control de eventos en Azure Communication Services](./event-handling.md).

## <a name="deliver-push-notifications-via-azure-notification-hubs"></a>Entrega de notificaciones push mediante Azure Notification Hubs

Puede conectar una instancia de Azure Notification Hubs a su recurso de Communication Services para enviar automáticamente notificaciones push al dispositivo móvil de un usuario cuando recibe una llamada entrante. Debe usar estas notificaciones push para reactivar la aplicación en segundo plano y mostrar la interfaz de usuario que permite al usuario aceptar o rechazar la llamada. 

:::image type="content" source="./media/notifications/acs-anh-int.png" alt-text="Diagrama que muestra cómo se integra Communication Services con Azure Notification Hubs.":::

Communication Services usa Azure Notification Hubs como un servicio de tránsito para comunicarse con los diversos servicios de notificaciones push específicos de cada plataforma mediante [Direct Send](https://docs.microsoft.com/rest/api/notificationhubs/direct-send) API. De este modo, podrá volver a usar los recursos y valores de configuración de Azure Notification Hubs existentes para ofrecer notificaciones de llamada confiables y de baja latencia a sus aplicaciones.

### <a name="notification-hub-provisioning"></a>Aprovisionamiento de una instancia de Notification Hubs 

Para enviar notificaciones push a dispositivos cliente mediante Notification Hubs, [cree una instancia de Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/create-notification-hub-portal) dentro de la misma suscripción en la que se encuentra el recurso de Communication Services. Azure Notification Hubs debe estar configurado para el servicio de notificaciones de plataforma que desea usar. Para información sobre cómo obtener notificaciones de inserción en la aplicación cliente desde Notification Hubs, consulte [Introducción a Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/ios-sdk-get-started) y seleccione la plataforma cliente de destino en la lista desplegable que aparece cerca de la parte superior de la página.

Una vez configurada la instancia de Notification Hubs, puede asociarla a su recurso de Communication Services; para ello, proporcione una cadena de conexión para el centro mediante el cliente de Azure Resource Manager o Azure Portal. La cadena de conexión debe contener permisos de "envío". Se recomienda crear otra directiva de acceso con permisos de "envío" únicamente para el centro. Más información sobre las [directivas de seguridad y acceso de Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-security)

> NOTA:  Para habilitar las notificaciones VoIP de Apple Push Notification Service, el nombre del centro de notificaciones se debe establecer en el id. de lote de aplicaciones con el sufijo `.voip`. Consulte [Uso de VoIP de APNS a través de Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/voip-apns).

#### <a name="using-the-azure-resource-manager-client-to-configure-the-notification-hub"></a>Uso del cliente de Azure Resource Manager para configurar la instancia de Notification Hubs

Para iniciar sesión en Azure Resource Manager, ejecute el siguiente código e inicie sesión con sus credenciales.

```console
armclient login
```

 Una vez que haya iniciado sesión correctamente, ejecute el siguiente código para aprovisionar el centro de notificaciones:

```console
armclient POST /subscriptions/<sub_id>/resourceGroups/<resource_group>/providers/Microsoft.Communication/CommunicationServices/<resource_id>/linkNotificationHub?api-version=2020-08-20-preview "{'connectionString': '<connection_string>','resourceId': '<resource_id>'}"
```

#### <a name="using-the-azure-portal-to-configure-the-notification-hub"></a>Uso de Azure Portal para configurar la instancia de Notification Hubs

En el portal, navegue hasta el recurso de Azure Communication Services. En el recurso de Communication Services, seleccione Notificaciones push en el menú de la izquierda de la página Communication Services y conecte la instancia de Notification Hubs que aprovisionó anteriormente. Debe especificar la cadena de conexión y el identificador de recurso aquí:

:::image type="content" source="./media/notifications/acs-anh-portal-int.png" alt-text="Captura de pantalla que muestra la configuración Notificaciones push en Azure Portal.":::

#### <a name="device-registration"></a>Registro de dispositivos 

Consulte la [guía de inicio rápido de llamada de voz](../quickstarts/voice-video-calling/getting-started-with-calling.md) para obtener información sobre cómo registrar el identificador del dispositivo con Communication Services.

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a Azure Event Grid, consulte [Introducción a Azure Event Grid](https://docs.microsoft.com/azure/event-grid/overview).
* Para más información sobre los conceptos de Azure Notification Hubs, consulte la [documentación de Azure Notification Hubs](https://docs.microsoft.com/azure/notification-hubs/)
