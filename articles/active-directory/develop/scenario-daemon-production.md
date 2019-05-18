---
title: 'Llamada de la aplicación de demonio web API (pasar a producción): la plataforma de identidad de Microsoft'
description: Obtenga información sobre cómo compilar una aplicación demonio que llama a web API (pasar a producción)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 627dab0cb23800664c5fb5b3df9c61f5071d4b87
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545400"
---
# <a name="daemon-app-that-calls-web-apis---move-to-production"></a>Aplicación de demonio que llama a web API: pasar a producción

Ahora que sabe cómo adquirir y usar un token para una llamada a servicios, obtenga información sobre cómo mover la aplicación a producción.

## <a name="deployment---case-of-multi-tenant-daemon-apps"></a>Implementación - caso de aplicaciones de demonio de varios inquilinos

Si es un ISV que QUIERE crear una aplicación demonio que se puede ejecutar en varios inquilinos, deberá asegurarse de que los administradores de inquilinos:

- Aprovisiona a una entidad de servicio para la aplicación
- Concede consentimiento a la aplicación

Necesitará explicar a sus clientes cómo realizar estas operaciones. Para obtener más información, consulte [solicitud de consentimiento para un inquilino al completo](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Pasos siguientes

Estos son algunos vínculos para obtener más información:

### <a name="net"></a>.NET

- Si aún no lo tiene, pruebe el inicio rápido [adquirir un token y llamar a Microsoft Graph API desde una aplicación de consola con la identidad de la aplicación](./quickstart-v2-netcore-daemon.md).
- Documentación de referencia para:
  - Creación de instancias [ConfidentialClientApplication](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.confidentialclientapplicationbuilder)
  - Una llamada a [AcquireTokenForClient](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.acquiretokenforclientparameterbuilder)
- Otros ejemplos y tutoriales:
  - [Microsoft identity platform consola demonio](https://github.com/Azure-Samples/microsoft-identity-platform-console-daemon) características de una aplicación de consola de demonio de .NET Core sencilla que muestra los usuarios de un inquilino consultar Microsoft Graph.

    ![Topología](media/scenario-daemon-app/daemon-app-sample.svg)

    El mismo ejemplo también muestra la variación con certificados.

    ![Topología](media/scenario-daemon-app/daemon-app-sample-with-certificate.svg)

  - [Microsoft-Identity-Platform-ASPNET-WebApp-daemon](https://github.com/Azure-Samples/microsoft-identity-platform-aspnet-webapp-daemon) características de una aplicación web de ASP.NET MVC que se sincroniza los datos de Microsoft Graph mediante la identidad de la aplicación en lugar del nombre del usuario. El ejemplo también muestra el proceso de consentimiento del administrador.

    ![Topología](media/scenario-daemon-app/damon-app-sample-web.svg)

### <a name="python"></a>Python

MSAL Python está actualmente en versión preliminar pública. Para obtener más información, consulte [ejemplo de en el repositorio de credenciales de cliente de Python de MSAL](https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/dev/sample/client_credentials_sample.py).

### <a name="java"></a>Java

MSAL Python está actualmente en versión preliminar pública. Para obtener más información, consulte [en repositorio ejemplos de Java de MSAL](https://github.com/AzureAD/azure-activedirectory-library-for-java/tree/dev/src/samples).