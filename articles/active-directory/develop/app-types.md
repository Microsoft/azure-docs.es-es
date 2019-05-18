---
title: Tipos de aplicaciones en v1.0 | Azure
description: Describe los tipos de aplicaciones y escenarios admitidos por el punto de conexión v2.0 de Azure Active Directory.
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
ms.openlocfilehash: efed9e35aed729c9efa39b0772b681d8c53ba7b8
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540661"
---
# <a name="application-types-in-v10"></a>Tipos de aplicaciones en v1.0

Azure Active Directory (Azure AD) admite la autenticación de una serie de arquitecturas de aplicaciones modernas, todas ellas basadas en los protocolos OAuth 2.0 u OpenID Connect estándar del sector.

En el diagrama siguiente, se ilustran los escenarios y los tipos de aplicación, además de cómo pueden agregarse distintos componentes:

![Tipos de aplicaciones y escenarios](./media/authentication-scenarios/application_types_and_scenarios.png)

Estos son los cinco escenarios principales de aplicación que admite Azure AD:

- **[Aplicación de página única (SPA)](single-page-application.md)**: un usuario tiene que iniciar sesión en una aplicación de página única protegida por Azure AD.
- **[Explorador web a aplicación web](web-app.md)**: un usuario tiene que iniciar sesión en una aplicación web protegida por Azure AD.
- **[Aplicación nativa a API web](native-app.md)**: una aplicación nativa que se ejecuta en teléfonos, tabletas o equipos tiene que autenticar a un usuario para obtener recursos de una API web protegida por Azure AD.
- **[Aplicación web a API web](web-api.md)**: una aplicación web tiene que obtener recursos de una API web protegida por Azure AD.
- **[Aplicación de servidor o de demonio a API web](service-to-service.md)**: una aplicación de demonio o de servidor sin interfaz de usuario web tiene que obtener recursos de una API web protegida por Azure AD.

Siga los vínculos para obtener más información acerca de cada tipo de aplicación y entender los escenarios generales antes de empezar a trabajar con el código. También puede obtener información sobre las diferencias que necesita saber al escribir una aplicación específica que funcione con el punto de conexión v1.0 o v2.0.

> [!NOTE]
> No todas las características y escenarios de Azure AD son compatibles con la versión 2.0 del punto de conexión. Para determinar si debe usar la versión 2.0 del punto de conexión, obtenga información sobre las [limitaciones de esta versión](active-directory-v2-limitations.md).

Puede desarrollar cualquiera de los escenarios o aplicaciones que se describen en este documento utilizando diferentes lenguajes y plataformas. Todos cuentan con completos ejemplos de código que están disponibles en la guías de [ejemplos de código de v1.0 por escenario](sample-v1-code.md) y [ejemplos de código de v2.0 por escenario](sample-v2-code.md). También puede descargar los ejemplos de código directamente desde el [repositorio de ejemplos de GitHub](https://github.com/Azure-Samples?q=active-directory) correspondiente.

Además, si la aplicación necesita un determinado fragmento o segmento de un escenario de un extremo a otro, dicha funcionalidad se podrá agregar de manera independiente en la mayoría de los casos. Por ejemplo, si tiene una aplicación nativa que llama a una API web, puede agregar fácilmente una aplicación web que también llame a la API web.

## <a name="app-registration"></a>Registro de aplicación

### <a name="registering-an-app-that-uses-the-azure-ad-v10-endpoint"></a>Registro de una aplicación que utiliza el punto de conexión v1.0 de Azure AD

Cualquier aplicación que externalice la autenticación a Azure AD debe registrarse en un directorio. Este paso implica informar a Azure AD de la aplicación, incluidos, entre otros, la dirección URL donde se encuentra, la dirección URL a la que se envían las respuestas tras la autenticación y el URI que identifica la aplicación. Esta información es necesaria por algunos motivos importantes:

* Azure AD necesita comunicarse con la aplicación cuando administra el inicio de sesión o intercambia tokens. La información que se pasa entre Azure AD y la aplicación incluye lo siguiente:
  
  * **URI del identificador de la aplicación**: identificador de una aplicación. Este valor se envía a Azure AD durante la autenticación para indicar para qué aplicación el llamador quiere un token. Además, este valor se incluye en el token de manera que la aplicación sepa que se trata del objetivo.
  * **URL de respuesta** y **URI de redirección**: en el caso de una API web o aplicación web, la dirección URL de respuesta es la ubicación a la que Azure AD enviará la respuesta de autenticación, incluido un token si la autenticación fue correcta. En el caso de una aplicación nativa, el URI de redirección es un identificador único al que Azure AD redirigirá el agente de usuario de una solicitud de OAuth 2.0.
  * **Id. de la aplicación**: identificador de una aplicación que Azure AD genera cuando se registra la aplicación. Al solicitar un token o código de autorización, se envían la clave y el identificador de la aplicación a Azure AD durante la autenticación.
  * **Clave**: clave que se envía junto con un identificador de la aplicación al autenticarse en Azure AD para llamar a una API web.
* Azure AD debe asegurarse de que la aplicación tiene los permisos necesarios para obtener acceso a los datos del directorio, a otras aplicaciones de la organización, etc.

Para más información, aprenda a [registrar una aplicación](quickstart-register-app.md).

## <a name="single-tenant-and-multi-tenant-apps"></a>Aplicaciones de un solo inquilino y aplicaciones multiinquilino

El aprovisionamiento se entiende mejor cuando se comprende que existen dos categorías de aplicaciones que se pueden desarrollar e integrarse con Azure AD:

* **Aplicación de un solo inquilino**: el uso de una aplicación de un solo inquilino se destina a una sola organización. Suele tratarse de aplicaciones de línea de negocio (LOB) escritas por un desarrollador de la empresa. A una aplicación de un solo inquilino solo obtienen acceso usuarios de un solo directorio y, por tanto, solo es necesario aprovisionarla en un directorio. Suele tratarse de aplicaciones registradas por un desarrollador de la organización.
* **Aplicación multiempresa**: el uso de una aplicación multiempresa se destina a varias organizaciones, no solo a una. Suele tratarse de aplicaciones de software como servicio (SaaS) escritas por un proveedor de software independiente (ISV). Las aplicaciones multiempresa se deben aprovisionar en cada uno de los directorios en los que se usarán, lo que requiere el consentimiento del usuario o del administrador para registrarlas. El proceso de consentimiento se inicia cuando una aplicación se ha registrado en el directorio y se le proporciona acceso a Graph API o quizás a otra API web. Cuando un usuario o un administrador de otra organización inicia sesión para usar la aplicación, se muestra un cuadro de diálogo que indica los permisos requeridos por la aplicación. A continuación, el usuario o el administrador pueden dar su consentimiento a la aplicación, lo que proporciona a la aplicación acceso a los datos establecidos y, por último, registra la aplicación en el directorio. Para obtener más información, consulte [Información general del marco de consentimiento](consent-framework.md).

### <a name="additional-considerations-when-developing-single-tenant-or-multi-tenant-apps"></a>Consideraciones adicionales al desarrollar aplicaciones de un solo inquilino o multiempresa

Cuando se desarrolla una aplicación multiempresa en lugar de una aplicación de un solo inquilino hay que tener en cuenta otros aspectos. Por ejemplo, si pone la aplicación a disposición de usuarios de varios directorios, necesitará un mecanismo para determinar en qué inquilino se encuentran. Una aplicación de un solo inquilino únicamente necesita buscar un usuario en su propio directorio, mientras que una aplicación multiempresa debe identificar un usuario específico en todos los directorios de Azure AD. Para realizar esta tarea, Azure AD proporciona un extremo de autenticación común donde cualquier aplicación multiempresa puede dirigir solicitudes de inicio de sesión, en lugar de un extremo específico del inquilino. Este punto de conexión es https://login.microsoftonline.com/common para todos los directorios en Azure AD, mientras que un punto de conexión específico del inquilino podría ser https://login.microsoftonline.com/contoso.onmicrosoft.com. Es especialmente importante tener en cuenta el extremo común al desarrollar la aplicación porque necesitará toda la lógica necesaria para administrar varios inquilinos durante el inicio y el cierre de sesión y la validación de tokens.

Si actualmente desarrolla una aplicación de un solo inquilino pero quiere ponerla a disposición de varias organizaciones, puede realizar cambios fácilmente en la aplicación y en su configuración en Azure AD para que pueda aceptar varios inquilinos. Además, Azure AD usa la misma clave de firma para todos los tokens en todos los directorios, tanto si se proporciona autenticación para una aplicación de un solo inquilino como para una aplicación multiempresa de varios inquilinos.

Cada uno de los escenarios incluidos en este documento incluye una subsección en donde se describen sus requisitos de aprovisionamiento. Si quiere obtener información más detallada sobre el aprovisionamiento de aplicaciones en Azure AD y las diferencias entre aplicaciones de un solo inquilino y multiempresa, vea [Integración de aplicaciones con Azure Active Directory](quickstart-v1-integrate-apps-with-azure-ad.md). Siga leyendo para comprender los escenarios de aplicación comunes de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre otros [recursos básicos de autenticación](authentication-scenarios.md) de Azure AD.
