---
title: 'Paso a producción de una aplicación de demonio que llama a las API web: plataforma de identidad de Microsoft'
description: Aprenda a pasar a producción una aplicación de demonio que llama a las API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 72e6d0a8677f7a8175223b80541c99026a9eb16f
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119171"
---
# <a name="daemon-app-that-calls-web-apis---move-to-production"></a>Aplicación de demonio que llama a las API web: paso a producción

Ahora que sabe cómo adquirir y usar un token para una llamada de servicio a servicio, aprenda a pasar la aplicación a producción.

## <a name="deployment---multitenant-daemon-apps"></a>Implementación: aplicaciones de demonio multiinquilino

Si es un ISV que crea una aplicación de demonio que se pueda ejecutar en varios inquilinos, deberá asegurarse de que el administrador del inquilino:

- Aprovisione una entidad de servicio para la aplicación.
- Dé su consentimiento a la aplicación.

Deberá explicar a sus clientes cómo realizar estas operaciones. Para más información, consulte [Solicitud de consentimiento para un inquilino al completo](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Pasos siguientes

Estos son algunos vínculos para ayudarle a obtener más información:

# <a name="net"></a>[.NET](#tab/dotnet)

- Inicio rápido: [Adquisición de un token y llamada a Microsoft Graph API desde una aplicación de consola mediante la identidad de la aplicación](./quickstart-v2-netcore-daemon.md)
- Documentación de referencia para:
  - Creación de instancias de [ConfidentialClientApplication](/dotnet/api/microsoft.identity.client.confidentialclientapplicationbuilder).
  - Llamada a [AcquireTokenForClient](/dotnet/api/microsoft.identity.client.acquiretokenforclientparameterbuilder).
- Otros ejemplos y tutoriales:
  - [microsoft-identity-platform-console-daemon](https://github.com/Azure-Samples/microsoft-identity-platform-console-daemon) incluye una sencilla aplicación de consola de demonio de .NET Core que muestra los usuarios de un inquilino que consultan Microsoft Graph.

    ![Ejemplo de topología de aplicación de demonio](media/scenario-daemon-app/daemon-app-sample.svg)

    El ejemplo también ilustra una variación con certificados:

    ![Ejemplo de topología de aplicación de demonio: certificados](media/scenario-daemon-app/daemon-app-sample-with-certificate.svg)

  - [microsoft-identity-platform-aspnet-webapp-daemon](https://github.com/Azure-Samples/microsoft-identity-platform-aspnet-webapp-daemon) incluye una aplicación web de ASP.NET MVC que sincroniza los datos de Microsoft Graph mediante la identidad de la aplicación, en lugar hacerlo en nombre de un usuario. El ejemplo también ilustra el proceso de consentimiento del administrador.

    ![Topología](media/scenario-daemon-app/damon-app-sample-web.svg)

# <a name="python"></a>[Python](#tab/python)

Pruebe la guía de inicio rápido [Adquisición de un token y llamada a Microsoft Graph API desde una aplicación de consola en Python mediante la identidad de la aplicación](./quickstart-v2-python-daemon.md).

# <a name="java"></a>[Java](#tab/java)

MSAL Java está actualmente en versión preliminar pública. Para más información, consulte los [ejemplos de desarrollo de MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples).

---