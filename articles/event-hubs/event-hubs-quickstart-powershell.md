---
title: 'Inicio rápido: Creación de un centro de eventos con PowerShell: Azure Event Hubs'
description: En esta guía de inicio rápido se describe cómo crear un centro de eventos mediante Azure PowerShell y, después, enviar y recibir eventos mediante el SDK de .NET Standard.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 1c81024a9195e463eeaaf384d5cbbd1e0054c485
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933977"
---
# <a name="quickstart-create-an-event-hub-using-azure-powershell"></a>Inicio rápido: Creación de un centro de eventos mediante Azure PowerShell

Azure Event Hubs es una plataforma de streaming de macrodatos y servicio de ingesta de eventos de gran escalabilidad capaz de recibir y procesar millones de eventos por segundo. Event Hubs puede procesar y almacenar eventos, datos o telemetría generados por dispositivos y software distribuido. Los datos enviados a un centro de eventos se pueden transformar y almacenar con cualquier proveedor de análisis en tiempo real o adaptadores de procesamiento por lotes y almacenamiento. Para más información sobre Event Hubs, consulte [Introducción a Event Hubs](event-hubs-about.md) y [Características de Event Hubs](event-hubs-features.md).

En esta guía de inicio rápido se crea un centro de eventos mediante Azure PowerShell.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este tutorial, asegúrese de disponer de los siguientes elementos:

- Suscripción de Azure. Si no tiene una, [cree una cuenta gratuita][] antes de empezar.
- [Visual Studio 2019](https://www.visualstudio.com/vs).
- [SDK de .NET Standard](https://www.microsoft.com/net/download/windows), versión 2.0 o posterior.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si está usando PowerShell localmente, tiene que ejecutar la versión más reciente de PowerShell para completar esta guía de inicio rápido. Si necesita instalarlo o actualizarlo, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es una recopilación lógica de recursos de Azure, para crear un centro de eventos necesita un grupo de recursos. 

En el siguiente ejemplo, se crea un grupo de recursos en la región Este de EE. UU. Sustituya `myResourceGroup` por el nombre del grupo de recursos que quiere utilizar:

```azurepowershell-interactive
New-AzResourceGroup –Name myResourceGroup –Location eastus
```

## <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs

Una vez que se crea el grupo de recursos, cree un espacio de nombres de Event Hubs dentro de ese grupo de recursos. Un espacio de nombres de Event Hubs proporciona un nombre de dominio completo y exclusivo en el que puede crear el centro de eventos. Reemplace `namespace_name` por un nombre único para el espacio de nombres:

```azurepowershell-interactive
New-AzEventHubNamespace -ResourceGroupName myResourceGroup -NamespaceName namespace_name -Location eastus
```

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Ahora que ya tiene un espacio de nombres de Event Hubs, cree un centro de eventos dentro de ese espacio de nombres:  
El período permitido para `MessageRetentionInDays` está comprendido entre 1 y 7 días.

```azurepowershell-interactive
New-AzEventHub -ResourceGroupName myResourceGroup -NamespaceName namespace_name -EventHubName eventhub_name -MessageRetentionInDays 3
```

Felicidades. Ha usado Azure PowerShell para crear un espacio de nombres de Event Hubs y un centro de eventos dentro de ese espacio de nombres. 

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado el espacio de nombres de Event Hubs y ha usado aplicaciones de ejemplo para enviar eventos al centro de eventos y recibirlos de este. Para encontrar instrucciones paso a paso sobre cómo enviar eventos a un centro de eventos o recibirlos de este, consulte los tutoriales sobre **envío y recepción de eventos**: 

- [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
- [Java](event-hubs-java-get-started-send.md)
- [Python](event-hubs-python-get-started-send.md)
- [JavaScript](event-hubs-node-get-started-send.md)
- [Go](event-hubs-go-get-started-send.md)
- [C (solo enviar)](event-hubs-c-getstarted-send.md)
- [Apache Storm (solo recibir)](event-hubs-storm-getstarted-receive.md)


[cree una cuenta gratuita]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[Install and Configure Azure PowerShell]: /powershell/azure/install-az-ps
[New-AzResourceGroup]: /powershell/module/az.resources/new-azresourcegroup
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[3]: ./media/event-hubs-quickstart-powershell/sender1.png
[4]: ./media/event-hubs-quickstart-powershell/receiver1.png
[5]: ./media/event-hubs-quickstart-powershell/metrics.png
