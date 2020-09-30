---
title: 'Inicio rápido: Creación y administración de tokens de acceso'
titleSuffix: An Azure Communication Services quickstart
description: Aprenda a administrar identidades y tokens de acceso mediante la biblioteca cliente de administración de Azure Communication Services.
author: matthewrobertson
manager: jken
services: azure-communication-services
ms.author: marobert
ms.date: 08/20/2020
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: a76000ecacdf78196ec1b80a60940484f6421641
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945739"
---
# <a name="quickstart-create-and-manage-access-tokens"></a>Inicio rápido: Creación y administración de tokens de acceso

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Para empezar a usar Azure Communication Services, utilice la biblioteca cliente de administración de Communication Services para aprovisionar y administrar sus tokens de acceso. Los tokens de acceso permiten que las aplicaciones cliente de chat y llamada se autentiquen directamente en Azure Communication Services. Estos tokens se generan en un servicio de aprovisionamiento de token del lado servidor que implemente. A continuación, se usan para inicializar las bibliotecas cliente de Communication Services en los dispositivos cliente.

Tenga en cuenta que los precios que se muestran en las imágenes de este tutorial son ejemplos solamente.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/user-access-token-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/user-access-token-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/user-access-token-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/user-access-token-java.md)]
::: zone-end

La salida de la aplicación describe cada acción que se completa:
<!---cSpell:disable --->
```console
Azure Communication Services - Access Tokens Quickstart

Issued a access token with 'voip' scope for identity with ID: 8:acs:fecfaddf-bf2c-4a0e-b52f-7d918c9536e6_65012b-1400da9050:
<token signature here>

Issued a access token with 'chat' scope for identity with ID: 8:acs:fecfaddf-bf2c-4a0e-b52f-7d918c9536e6_65012b-1400da9050:
<token signature here>

Successfully deleted the identity with ID: 8:acs:fecfaddf-bf2c-4a0e-b52f-7d918c9536e6_65012b-1400da9050

Deleted the identity with ID: 8:acs:fecfaddf-bf2c-4a0e-b52f-7d918c9536e6_65012b-1400da9050
```
<!---cSpell:enable --->

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y quitar una suscripción a Communication Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él. Obtenga más información sobre la [limpieza de recursos](./create-communication-resource.md#clean-up-resources).


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a:

> [!div class="checklist"]
> * Administrar identidades
> * Emitir tokens de acceso
> * Usar la biblioteca cliente de administración de Communication Services


> [!div class="nextstepaction"]
> [Adición de la llamada de voz a una aplicación](./voice-video-calling/getting-started-with-calling.md)

Puede que también le interese:

 - [Información sobre la autenticación](../concepts/authentication.md)
 - [Incorporación de chat a una aplicación](./chat/get-started.md)
 - [Información sobre la arquitectura de cliente y servidor](../concepts/client-and-server-architecture.md)
