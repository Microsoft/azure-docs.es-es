---
title: Las aplicaciones cliente (biblioteca de autenticación de Microsoft) | Azure
description: Obtenga información acerca de cliente público y cliente confidencial aplicaciones en la biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/25/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9e0300ec0ef4ee67b06acb85514ae898bbd0a830
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544286"
---
# <a name="public-client-and-confidential-client-applications"></a>Cliente público y las aplicaciones cliente confidencial
Biblioteca de autenticación de Microsoft (MSAL) define dos tipos de clientes: los clientes públicos y confidenciales. Los tipos de cliente de dos se distinguen por su capacidad de autenticar de forma segura con el servidor de autorización y mantener la confidencialidad de sus credenciales de cliente.  En cambio, Azure AD Authentication Library (ADAL) tiene el concepto de contexto de autenticación (que es una conexión a Azure AD).

- **Las aplicaciones cliente confidencial** son aplicaciones que se ejecutan en servidores (Web Apps, API Web o incluso las aplicaciones de servicio o demonio). Se consideran difícil para el acceso y, por tanto, ser capaz de mantener un secreto de aplicación. Los clientes confidenciales son capaz de almacenar secretos de configuración. Cada instancia del cliente tiene una configuración distinta (incluido el clientId y secreto). Estos valores son difíciles para los usuarios finales extraer. Una aplicación web es el cliente confidencial más comunes. El identificador de cliente se expone a través del explorador web, pero el secreto se pasa sólo en el canal y se exponen nunca directamente.

    Aplicaciones cliente confidencial: <BR>
    ![Aplicación Web](media/msal-client-applications/web-app.png) ![API Web](media/msal-client-applications/web-api.png) ![demonio/servicio](media/msal-client-applications/daemon-service.png)

- **Las aplicaciones de cliente público** son aplicaciones, que se ejecutan en dispositivos o equipos de escritorio o en un explorador web. No son de confianza para mantener de forma segura secretos de aplicación y, por lo tanto sólo tener acceso a las API Web en nombre del usuario (solo admiten flujos de cliente público). Los clientes públicos son no puede contener configuración secretos y no tiene como resultado ningún secreto de cliente.

    Aplicaciones de cliente público: <BR>
    ![Aplicación de escritorio](media/msal-client-applications/desktop-app.png) ![API Browserless](media/msal-client-applications/browserless-app.png) ![aplicación móvil](media/msal-client-applications/mobile-app.png)

> [!NOTE]
> En MSAL.js, no hay ninguna separación de las aplicaciones cliente públicos y confidenciales.  MSAL.js representa las aplicaciones cliente como aplicaciones basadas en el agente de usuario de un cliente público en el que se ejecuta el código de cliente en un agente de usuario como un explorador web.  Estos clientes no almacenan secretos, puesto que el contexto del explorador es accesible públicamente.

## <a name="comparing-the-client-types"></a>Comparación de los tipos de cliente
Hay algunas similitudes y diferencias entre el cliente público y cliente confidencial aplicaciones:

- Ambos tipos de aplicaciones de mantienen una caché de tokens de usuario y pueden adquirir un token de forma silenciosa (en casos donde el token ya está en la caché de tokens). Las aplicaciones de cliente confidencial también tienen una caché de tokens de aplicación para los tokens, que son para la propia aplicación.
- Ambos administrar cuentas de usuario y pueden obtener las cuentas de la caché de tokens de usuario, obtener una cuenta de su identificador o quitar una cuenta.
- Las aplicaciones de cliente público tienen cuatro maneras de adquirir un token (cuatro flujos de autenticación), mientras que las aplicaciones cliente confidencial tienen tres (y un método para calcular la dirección URL del proveedor de identidades autorizar un extremo). Para obtener más información, vea escenarios y adquisición de tokens.

Si ha usado ADAL en el pasado, es posible que tenga en cuenta que, al contrario que el contexto de autenticación de ADAL, en MSAL el cliente (también denominado identificador de aplicación o Id. de aplicación) de identificador se pasa una vez en la construcción de la aplicación y ya no es necesario repetir al adquirir un token. Este es el caso tanto para las aplicaciones cliente públicos y confidenciales. Los constructores de las aplicaciones cliente confidencial también se pasan las credenciales del cliente: el secreto que se comparten con el proveedor de identidades.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre lo siguiente:
- [Opciones de configuración de aplicación de cliente](msal-client-application-configuration.md)
- [Crear instancias de las aplicaciones de cliente mediante MSAL.NET](msal-net-initializing-client-applications.md).
- [Crear instancias de las aplicaciones cliente mediante MSAL.js](msal-js-initializing-client-applications.md).
