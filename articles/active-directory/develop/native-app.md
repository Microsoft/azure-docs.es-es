---
title: Aplicaciones nativas en Azure Active Directory
description: Se describe qué son las aplicaciones nativas y los conceptos básicos sobre el flujo de protocolos, el registro y la expiración de tokens en este tipo de aplicación.
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
ms.openlocfilehash: a6bf24124c4b072a64ef59500b2f723ff6abbb0e
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545838"
---
# <a name="native-apps"></a>Aplicaciones nativas

Las aplicaciones nativas son aplicaciones que llaman a una API web en nombre de un usuario. Este escenario se basa en el tipo de concesión de código de autorización OAuth 2.0 con un cliente público, tal como se describe en la sección 4.1 de la [especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749). La aplicación nativa obtiene un token de acceso para el usuario mediante el protocolo OAuth 2.0. Este token de acceso se envía después en la respuesta a la API web, que autoriza al usuario y devuelve el recurso deseado.

## <a name="diagram"></a>Diagrama

![Diagrama de aplicación nativa a API web](./media/authentication-scenarios/native_app_to_web_api.png)

## <a name="protocol-flow"></a>Flujo de protocolos

Si usa las bibliotecas de autenticación de AD, la mayoría de los detalles del protocolo que se describen a continuación se administran automáticamente, por ejemplo, la ventana emergente del explorador, el almacenamiento en caché de los tokens y la administración de los tokens de actualización.

1. Mediante una ventana emergente del explorador, la aplicación nativa realiza una solicitud al extremo de autorización de Azure AD. Esta solicitud incluye el identificador de la aplicación y el identificador URI de redirección de la aplicación nativa, tal como se muestra en Azure Portal, así como el identificador URI del identificador de la aplicación para la API web. Si el usuario todavía no ha iniciado sesión, se le pide que lo haga.
1. Azure AD autentica al usuario. Si se trata de una aplicación de varios inquilinos y se requiere consentimiento para usarla, se pedirá el consentimiento del usuario si todavía no lo ha dado. Después de conceder consentimiento y tras la correcta autenticación, Azure AD emite una respuesta de código de autorización al URI de redirección de la aplicación cliente.
1. Cuando Azure AD devuelve una respuesta de código de autorización al URI de redirección, la aplicación cliente detiene la interacción con el explorador y extrae el código de autorización de la respuesta. Con este código de autorización, la aplicación cliente envía una solicitud al punto de conexión de token de Azure AD que incluye el código de autorización, detalles sobre la aplicación cliente (URI de redirección e identificador de la aplicación), además del recurso deseado (URI del identificador de la aplicación para la API web).
1. Azure AD valida el código de autorización e información sobre la aplicación cliente y la API web. Tras la correcta validación, Azure AD devuelve dos tokens: un token de acceso de JWT y un token de actualización de JWT. Además, Azure AD devuelve información básica sobre el usuario como, por ejemplo, el nombre para mostrar y el identificador de inquilino.
1. A través de HTTPS, la aplicación cliente usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación “Bearer” en el encabezado Authorization de la solicitud a la API web. A continuación, la API web valida el token de JWT y, si la validación es correcta, devuelve el recurso deseado.
1. Cuando el token de acceso expire, la aplicación cliente recibirá un mensaje de error que indica que el usuario debe volver a autenticarse. Si la aplicación tiene un token de actualización válido, puede usarse para adquirir un nuevo token de acceso sin pedir al usuario que vuelva a iniciar sesión. Si el token de actualización expira, será necesario que la aplicación vuelva a autenticar interactivamente al usuario.

> [!NOTE]
> El token de actualización que emite Azure AD se puede usar para obtener acceso a varios recursos. Por ejemplo, si tiene una aplicación cliente con permiso para llamar a dos API web, el token de actualización puede usarse para obtener también un token de acceso a la otra API web.

## <a name="code-samples"></a>Ejemplos de código

Vea los ejemplos de código en escenarios de aplicación nativa a API web. Agregamos nuevos ejemplos continuamente, así que no olvide consultarlos con frecuencia. [Aplicación nativa a API web](sample-v1-code.md#desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api).

## <a name="app-registration"></a>Registro de aplicación

Para registrar una aplicación con el extremo de la versión 1.0 de Azure AD, consulte [registrar una aplicación](quickstart-register-app.md).

* Un solo inquilino: tanto la aplicación nativa como la API web deben estar registradas en el mismo directorio de Azure AD. La API web se puede configurar para que exponga un conjunto de permisos, que se usan para limitar el acceso de la aplicación nativa a sus recursos. A continuación, la aplicación cliente selecciona los permisos deseados en el menú desplegable "Permisos para otras aplicaciones" de Azure Portal.
* Multiinquilino: en primer lugar, la aplicación nativa siempre se registra en el directorio del desarrollador o del editor. En segundo lugar, la aplicación nativa está configurada para indicar los permisos que requiere para ser funcional. La lista de permisos necesarios se muestra en un cuadro de diálogo cuando un usuario o administrador del directorio de destino da su consentimiento a la aplicación, lo que la pone a disposición de la organización. Algunas aplicaciones solo requieren permisos de nivel de usuario, para los que cualquier usuario de la organización puede dar el consentimiento. Otras aplicaciones requieren permisos de nivel de administrador, para los que un usuario de la organización no puede dar su consentimiento. Solo un administrador de directorio puede dar su consentimiento a las aplicaciones que requieran este nivel de permisos. Cuando el usuario o el administrador dan su consentimiento, solo la API web queda registrada en el directorio. 

## <a name="token-expiration"></a>Expiración del token

Cuando la aplicación nativa usa su código de autorización para obtener un token de acceso de JWT, también recibe un token de actualización de JWT. Cuando el token de acceso expira, se puede usar el token de actualización para volver a autenticar al usuario sin solicitarle que inicie sesión de nuevo. Este token de actualización se usa a continuación para autenticar al usuario, lo que se traduce en un token de acceso y un token de actualización nuevos.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre otros [tipos y escenarios de aplicaciones](app-types.md).
- Más información sobre los [conceptos básicos de autenticación](authentication-scenarios.md) de Azure AD.
