---
title: Descripción del flujo de concesión implícita de OAuth2 en Azure AD | Microsoft Docs
description: Sepa más de la implementación del flujo de concesión implícita de OAuth2 de Azure Active Directory y vea si es adecuado para su aplicación.
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 90e42ff9-43b0-4b4f-a222-51df847b2a8d
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8fe0ee8021ae7e70654a161e37d072195bbc035f
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545259"
---
# <a name="understanding-the-oauth2-implicit-grant-flow-in-azure-active-directory-ad"></a>Descripción del flujo de concesión implícita de OAuth2 de Azure Active Directory (AD)

[!INCLUDE [active-directory-develop-applies-v1](../../../includes/active-directory-develop-applies-v1.md)]

La concesión implícita de OAuth2 es conocida por tratarse de la lista más amplia de problemas de seguridad de la especificación de OAuth2. Además, es el enfoque que ha implementado ADAL JS y el que recomendamos a la hora de crear aplicaciones SPA. ¿Qué ventajas aporta? Se trata de analizar los pros y los contras: cuando se adopta, la concesión implícita es el mejor enfoque que puede aplicarse a las aplicaciones que usan una API web a través de JavaScript en un explorador.

## <a name="what-is-the-oauth2-implicit-grant"></a>¿Qué es la concesión implícita de OAuth2?

La [concesión de código de autorización de OAuth2](https://tools.ietf.org/html/rfc6749#section-1.3.1) por antonomasia es la concesión de autorización que utiliza dos puntos de conexión independientes. El punto de conexión de autorización se utiliza para la fase de interacción del usuario, lo que resulta en un código de autorización. El cliente usa entonces el punto de conexión de token para intercambiar el código para un token de acceso y, a menudo, también para un token de actualización. Las aplicaciones web son necesarias para presentar sus propias credenciales de aplicación al punto de conexión de token para que el servidor de autorización pueda autenticar al cliente.

La [concesión implícita de OAuth2](https://tools.ietf.org/html/rfc6749#section-1.3.2) es una variante de otras concesiones de autorización. Permite a un cliente obtener un token de acceso (y el id_token, cuando se usa [OpenId Connect](https://openid.net/specs/openid-connect-core-1_0.html)) directamente desde el punto de conexión de autorización, sin establecer contacto con el punto de conexión del token ni autenticar al cliente. Esta variante se diseñó para las aplicaciones basadas en JavaScript que se ejecutan en un explorador web: en la especificación de OAuth2 original, los tokens se devuelven en un fragmento de URI. De este modo, los bits de token estarán disponibles en el código JavaScript del cliente, pero garantiza que no se incluirán en las redirecciones hacia el servidor. La devolución de tokens a través del explorador se redirigen directamente desde el punto de conexión de autorización. También tiene la ventaja de eliminar los requisitos para llamadas entre orígenes, que son necesarios si se requiere la aplicación JavaScript para ponerse en contacto con el punto de conexión de token.

Una característica importante de la concesión implícita de OAuth2 es el hecho de que los flujos nunca devuelven tokens de actualización al cliente. En la sección siguiente se muestra como esto no es realmente necesario y, de hecho, supondría un problema de seguridad.

## <a name="suitable-scenarios-for-the-oauth2-implicit-grant"></a>Escenarios adecuados para la concesión implícita de OAuth2

La especificación de OAuth2 declara que la concesión implícita se ha diseñado para habilitar aplicaciones de agente de usuario, es decir, aplicaciones JavaScript que se ejecutan en un explorador. La característica definitoria de esas aplicaciones es que el código JavaScript se utiliza para acceder a los recursos del servidor (normalmente, una API web) y actualizar en consecuencia la experiencia de usuario de la aplicación. Piense en aplicaciones como Gmail o Outlook Web Access: al seleccionar un mensaje de la bandeja de entrada, solo cambia el panel de visualización de mensajes para mostrar la nueva selección, mientras que el resto de la página permanece sin modificar. A diferencia de las aplicaciones web basadas en redireccionamientos tradicionales, donde cada interacción del usuario produce un postback de página completa y una representación de página completa de la nueva respuesta del servidor.

Las aplicaciones que llevan el enfoque basado en JavaScript hasta su extremo se denominan aplicaciones de página única o SPA. La idea es que estas aplicaciones solo publican una página HTML inicial y el código JavaScript asociado, y todas las interacciones subsiguientes se controlan mediante llamadas a la API Web realizadas a través de JavaScript. Sin embargo, los enfoques híbridos, donde la aplicación se basa fundamentalmente en eventos postback, pero realiza llamadas de JavaScript ocasionales, no son habituales. El debate sobre el uso del flujo implícito es pertinente para este tipo de enfoques.

Por lo general, las aplicaciones basadas en redirecciones protegen sus solicitudes a través de las cookies; sin embargo, este enfoque no funciona tan bien para aplicaciones JavaScript. Las cookies solo funcionan con el dominio para el que se han generado, aunque las llamadas de JavaScript pueden ser dirigidas a otros dominios. De hecho, esto pasará con frecuencia. Piense en las aplicaciones que invocan Microsoft Graph API, la API de Office o la API de Azure: todas residen fuera del dominio desde donde se publica la aplicación. Una tendencia que se está volviendo más popular en las aplicaciones JavaScript es no tener ningún back-end y depender exclusivamente de API web de terceros para implementar su función empresarial.

Actualmente, el método preferido para proteger las llamadas a una API web es usar el enfoque de token de portador de OAuth2, donde cada llamada viene acompañada de un token de acceso OAuth2. La API web examina el token de acceso entrante y, si se encuentran en él los ámbitos necesarios, concede acceso a la operación solicitada. El flujo implícito proporciona un cómodo mecanismo para que las aplicaciones JavaScript obtengan tokens de acceso para una API web, con lo que ofrece numerosas ventajas con respecto a las cookies:

* Pueden obtenerse tokens de forma confiable sin necesidad de realizar llamadas de origen cruzado: gracias al registro obligatorio del URI de redirección en el que se devuelven los tokens, se garantiza que estos no van a sustituirse.
* Las aplicaciones JavaScript pueden obtener tantos tokens de acceso como necesiten para la cantidad de API web que tengan como destino; sin restricciones de dominios.
* Las características de HTML5 como el almacenamiento local o de sesión conceden un control total sobre el almacenamiento en caché de tokens y administración de la duración, mientras que la administración de cookies es opaca para la aplicación.
* Los tokens de acceso no son susceptibles a los ataques de falsificación de solicitud entre sitios (CSRF).

El flujo de concesión implícita no emite tokens de actualización, principalmente por motivos de seguridad. Los tokens de actualización no tienen tantas limitaciones como los de acceso, que conceden muchos más permisos y, por tanto, pueden producir más daños en caso de que se filtren. En el flujo implícito, los tokens se entregan en la dirección URL, por lo tanto, el riesgo de interceptación es mayor que en la concesión de código de autorización.

Sin embargo, una aplicación JavaScript posee otro mecanismo para renovar los tokens de acceso sin preguntar las credenciales al usuario repetidamente. La aplicación puede usar un elemento iframe oculto para realizar nuevas solicitudes de token en el punto de conexión de autorización de Azure AD: siempre y cuando el explorador siga teniendo una sesión activa (es decir, una cookie de sesión) en el dominio de Azure AD, la solicitud de autenticación podrá realizarse correctamente sin necesidad de que tenga que interactuar el usuario.

Este modelo concede a la aplicación JavaScript la capacidad de renovar los tokens de acceso de forma independiente e incluso de adquirir unos nuevos para una nueva API (siempre que el usuario haya dado previamente su consentimiento). Esto evita la carga agregada de adquisición, mantenimiento y protección de un artefacto de valor elevado, como un token de actualización. El artefacto que posibilita la renovación silenciosa, la cookie de sesión de Azure AD, se administra fuera de la aplicación. Otra ventaja de este enfoque es que un usuario puede cerrar la sesión desde Azure AD, mediante cualquiera de las aplicaciones que han iniciado sesión en Azure AD y que se ejecutan en cualquiera de las pestañas del explorador. Esto dará como resultado la eliminación de la cookie de sesión de Azure AD y la aplicación JavaScript perderá automáticamente la capacidad de renovar tokens para el usuario que ha cerrado la sesión.

## <a name="is-the-implicit-grant-suitable-for-my-app"></a>¿Es adecuada la concesión implícita para mi aplicación?

La concesión implícita presenta más riesgos que otras concesiones, y las áreas a las que debe prestar atención están bien documentadas (por ejemplo, [Uso incorrecto del token de acceso para hacerse pasar por el propietario del recurso en el flujo implícito][OAuth2-Spec-Implicit-Misuse] y [Modelo de amenazas de OAuth 2.0 y consideraciones de seguridad][OAuth2-Threat-Model-And-Security-Implications]). Sin embargo, el perfil de mayor riesgo se debe, en gran medida, al hecho de que este tipo de concesión está diseñada para permitir aplicaciones que ejecutan código activo servido por un recurso remoto en un explorador. Si planea una arquitectura de SPA, no dispone de ningún componente de back-end o tiene planeado invocar una API web a través de JavaScript, se recomienda utilizar el flujo implícito para obtener tokens.

Si su aplicación es un cliente nativo, el flujo implícito no es una buena elección. La ausencia de la cookie de sesión de Azure AD en el contexto de un cliente nativo priva a su aplicación de los medios de mantener una sesión de larga duración. Lo que significa que la aplicación pedirá confirmación repetidamente al usuario al obtener los tokens de acceso para los nuevos recursos.

Si está desarrollando una aplicación web que incluye un back-end y que usa una API desde el código de back-end, el flujo implícito tampoco es una buena elección. Otras concesiones ofrecen muchas más posibilidades. Por ejemplo, la concesión de credenciales de cliente de OAuth2 proporciona la capacidad de obtener tokens que reflejan los permisos asignados a la misma aplicación, a diferencia de las delegaciones de usuario. Esto significa que el cliente tiene la capacidad de mantener el acceso mediante programación a los recursos, incluso cuando el usuario no está activamente implicado en una sesión. No solo eso, sino que dichas concesiones ofrecen mayores garantías de seguridad. Por ejemplo, los tokens de acceso nunca pasan por el explorador del usuario, porque no se arriesgan a ser guardados en el historial del explorador. La aplicación cliente también puede realizar una autenticación segura cuando se solicita un token.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una lista completa de recursos para desarrolladores, incluida la información de referencia de los protocolos y flujos de concesión de autorización de OAuth2 compatibles con Azure AD, consulte la [Guía del desarrollador de Azure AD][AAD-Developers-Guide].
* Consulte [Integración con Azure Active Directory][ACOM-How-To-Integrate] para obtener información más detallada sobre el proceso de integración de aplicaciones.

<!--Image references-->

<!--Reference style links in use-->
[AAD-Developers-Guide]:azure-ad-developers-guide.md
[ACOM-How-And-Why-Apps-Added-To-AAD]: active-directory-how-applications-are-added.md
[ACOM-How-To-Integrate]: active-directory-how-to-integrate.md
[OAuth2-Spec-Implicit-Misuse]: https://tools.ietf.org/html/rfc6749#section-10.16
[OAuth2-Threat-Model-And-Security-Implications]: https://tools.ietf.org/html/rfc6819
