---
title: 'Inicio rápido: Incorporación de chat a una aplicación'
titleSuffix: An Azure Communication Services quickstart
description: En esta guía de inicio rápido se muestra cómo agregar un chat de Communication Services a la aplicación.
author: fanche
manager: phans
services: azure-communication-services
ms.author: mikben
ms.date: 03/10/2020
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: ed40f91c760d7c83466078b3523748175a1701ad
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945635"
---
# <a name="quickstart-add-chat-to-your-app"></a>Inicio rápido: Incorporación de chat a una aplicación

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include.md)]

Comience a usar Azure Communication Services mediante la biblioteca cliente de chat de Communication Services para agregar la función de chat en tiempo real a su aplicación. En esta guía de inicio rápido, usaremos la biblioteca cliente de chat para crear subprocesos de chat que permitan a los usuarios mantener conversaciones entre sí. Para obtener más información acerca de los conceptos de chat, visite la [documentación conceptual de chat](../../concepts/chat/concepts.md).

::: zone pivot="programming-language-javascript"
[!INCLUDE [Chat with JavaScript client library](./includes/chat-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Chat with Python client library](./includes/chat-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Chat with Java client library](./includes/chat-java.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Chat with C# client library](./includes/chat-csharp.md)]
::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](../create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

> [!div class="checklist"]
> * Creación de un cliente de chat
> * Creación de un subproceso con dos usuarios
> * Envío de un mensaje al subproceso
> * Recepción de mensajes de un subproceso
> * Eliminación de usuarios de un subproceso

> [!div class="nextstepaction"]
> [Prueba de la aplicación de elemento principal de chat](../../samples/chat-hero-sample.md)

Puede que también le interese:

 - Obtener más información sobre los [conceptos de chat](../../concepts/chat/concepts.md).
 - Familiarizarse con la [biblioteca cliente de chat](../../concepts/chat/sdk-features.md).
