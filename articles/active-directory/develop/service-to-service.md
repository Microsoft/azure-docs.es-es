---
title: Aplicaciones servicio a servicio en Azure Active Directory
description: Se describe qué son las aplicaciones de servicio a servicio y los conceptos básicos sobre el flujo de protocolos, el registro y la expiración de tokens en este tipo de aplicación.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 683664b3172cb12ba6adf6c8006e9685a6d1ec35
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540324"
---
# <a name="service-to-service-apps"></a>Aplicaciones de se servicio a servicio

Las aplicaciones de servicio a servicio pueden ser una aplicación de demonio o de servidor que necesita obtener recursos de una API web. Hay dos subescenarios aplicables a esta sección:

- Un demonio que necesita llamar a una API web, que se basa en el tipo de concesión de credenciales de cliente de OAuth 2.0

    En este escenario, es importante entender varios aspectos. En primer lugar, la interacción con el usuario no es posible con una aplicación de demonio, que requiere que la aplicación tenga su propia identidad. Un ejemplo de aplicación de demonio es un trabajo por lotes o un servicio de sistema operativo que se ejecuta en segundo plano. Este tipo de aplicación solicita un token de acceso con su identidad de aplicación y presenta el identificador de la aplicación, las credenciales (contraseña o certificado) y el identificador URI del identificador de la aplicación a Azure AD. Tras la correcta autenticación, el demonio recibe un token de acceso de Azure AD, que se usa posteriormente para llamar a la API web.

- Una aplicación de servidor (por ejemplo, una API web) que necesita llamar a una API web, basada en la especificación "On-Behalf-Of" de OAuth 2.0

    En este escenario, imagine que un usuario se ha autenticado en una aplicación nativa y que la aplicación nativa necesita llamar a una API web. Azure AD emite un token de acceso de JWT para llamar a la API web. Si la API web necesita llamar a otra API web de nivel inferior, puede usar el flujo "On-Behalf-Of" para delegar la identidad del usuario y autenticarlo en la API web de segundo nivel.

## <a name="diagram"></a>Diagrama

![Diagrama de aplicación de servidor o de demonio en API web](./media/authentication-scenarios/daemon_server_app_to_web_api.png)

## <a name="dprotocol-flow"></a>Flujo DProtocol

### <a name="application-identity-with-oauth-20-client-credentials-grant"></a>Identidad de aplicación con concesión de credenciales de cliente OAuth 2.0

1. En primer lugar, la aplicación de servidor necesita autenticarse con Azure AD por sí misma, sin ninguna interacción como, por ejemplo, un cuadro de diálogo de inicio de sesión interactivo. Esta realiza una solicitud al punto de conexión de token de Azure AD y proporciona las credenciales, el identificador de la aplicación y el identificador URI del identificador de la aplicación.
1. Azure AD autentica la aplicación y devuelve un token de acceso de JWT que se usa para llamar a la API web.
1. A través de HTTPS, la aplicación web usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación “Bearer” en el encabezado Authorization de la solicitud a la API web. A continuación, la API web valida el token de JWT y, si la validación es correcta, devuelve el recurso deseado.

### <a name="delegated-user-identity-with-oauth-20-on-behalf-of-draft-specification"></a>Identidad de usuario delegado con la especificación provisional "On-Behalf-Of" de OAuth 2.0

En el flujo descrito a continuación se asume que un usuario se ha autenticado en otra aplicación (por ejemplo, una aplicación nativa) y se ha usado su identidad de usuario para adquirir un token de acceso para la API web de primer nivel.

1. La aplicación nativa envía el token de acceso a la API web de primer nivel.
1. La API web de primer nivel envía una solicitud al punto de conexión de token de Azure AD y proporciona el identificador de la aplicación y las credenciales, así como el token de acceso del usuario. Además, la solicitud se envía con un parámetro on_behalf_of que indica que la API web solicita nuevos tokens para llamar a una API web de nivel inferior en nombre del usuario original.
1. Azure AD comprueba que la API web de primer nivel tiene permisos de acceso a la API web de segundo nivel, valida la solicitud y devuelve un token de acceso de JWT y un token de actualización de JWT a la API web de primer nivel.
1. La API web de primer nivel llama después a la API web de segundo nivel a través de HTTPS. Para ello, anexa la cadena de token en el encabezado Authorization de la solicitud. La API web de primer nivel puede seguir llamando a la API web de segundo nivel siempre que el token de acceso y los tokens de actualización sean válidos.

## <a name="code-samples"></a>Ejemplos de código

Vea los ejemplos de código para escenarios de aplicación de servidor o de demonio en API web. Agregamos nuevos ejemplos continuamente, así que no olvide consultarlos con frecuencia. [Aplicación de servidor o de demonio en API web](sample-v1-code.md#daemon-applications-accessing-web-apis-with-the-applications-identity)

## <a name="app-registration"></a>Registro de aplicación

* Un solo inquilino: tanto en el caso de la identidad de aplicación como en el de la identidad de usuario delegado, la aplicación de demonio o de servidor debe estar registrada en el mismo directorio de Azure AD. La API web puede configurarse para exponer un conjunto de permisos, que se usan para limitar el acceso de la aplicación de demonio o de servidor a sus recursos. Si se usa un tipo de identidad de usuario delegado, es necesario que la aplicación de servidor seleccione los permisos deseados en el menú desplegable "Permisos para otras aplicaciones" de Azure Portal. Este paso no es necesario si se usa el tipo de identidad de aplicación.
* Multiempresa (varios inquilinos): en primer lugar, la aplicación de demonio o de servidor se configura para indicar los permisos que requiere para ser funcional. La lista de permisos necesarios se muestra en un cuadro de diálogo cuando un usuario o administrador del directorio de destino da su consentimiento a la aplicación, lo que la pone a disposición de la organización. Algunas aplicaciones solo requieren permisos de nivel de usuario, para los que cualquier usuario de la organización puede dar el consentimiento. Otras aplicaciones requieren permisos de nivel de administrador, para los que un usuario de la organización no puede dar su consentimiento. Solo un administrador de directorio puede dar su consentimiento a las aplicaciones que requieran este nivel de permisos. Cuando el usuario o el administrador dan su consentimiento, las dos API web se registran en el directorio.

## <a name="token-expiration"></a>Expiración del token

Cuando la primera aplicación usa su código de autorización para obtener un token de acceso de JWT, también recibe un token de actualización de JWT. Cuando el token de acceso expira, el token de actualización se puede usar para autenticar al usuario sin pedir credenciales. Este token de actualización se usa a continuación para autenticar al usuario, lo que se traduce en un token de acceso y un token de actualización nuevos.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre otros [tipos y escenarios de aplicaciones](app-types.md).
- Más información sobre los [conceptos básicos de autenticación](authentication-scenarios.md) de Azure AD.
