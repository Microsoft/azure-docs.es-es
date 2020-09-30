---
title: 'Registro de una API web que llama a las API web: Plataforma de identidad de Microsoft | Azure'
description: Obtenga información sobre cómo compilar una API web que llame a API web de bajada (registro de la aplicación).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: c0b009d6d53a117aa11d0629fb649b2dd55559af
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89438201"
---
# <a name="a-web-api-that-calls-web-apis-app-registration"></a>API web que llama a API web: Registro de aplicación

Una API web que llama a API web de bajada tiene el mismo registro que una API web protegida. Por lo tanto, deberá seguir las instrucciones de [API web protegida: registro de aplicación](scenario-protected-web-api-app-registration.md).

Dado que la aplicación web ahora llama a las API web, se convierte en una aplicación cliente confidencial. Por eso se exige información de registro adicional: la aplicación tiene que compartir secretos (credenciales de cliente) con la Plataforma de identidad de Microsoft.

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>Permisos de API

Las aplicaciones web llaman a las API en nombre de los usuarios para los que se ha recibido el token de portador. Tienen que solicitar permisos delegados. Para más información, consulte [Adición de permisos para acceder a la API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API web que llama a las API web: configuración del código](scenario-web-api-call-api-app-configuration.md)
