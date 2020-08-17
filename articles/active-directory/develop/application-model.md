---
title: Modelo de aplicación | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre el proceso de registro de la aplicación para que pueda integrarse con la Plataforma de identidad de Microsoft (v2.0).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/28/2020
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda, sureshja, hirsin
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 5aca96a9c3bc4e8f1061f677e316565b10014ac9
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88117488"
---
# <a name="application-model"></a>Modelo de aplicación

Las aplicaciones pueden iniciar por sí mismas la sesión de los usuarios o delegar el inicio de sesión en un proveedor de identidades. En este tema se describen los pasos necesarios para registrar una aplicación en la Plataforma de identidad de Microsoft.

## <a name="registering-an-application"></a>Registro de una aplicación

Para que un proveedor de identidades sepa que un usuario tiene acceso a una determinada aplicación, tanto el usuario como la aplicación deben estar registrados con el proveedor de identidades. Cuando registra la aplicación con Azure AD, proporciona una configuración de identidad para la aplicación, lo que permite integrarla con la Plataforma de identidad de Microsoft. El registro de la aplicación también le permite:

* Personalizar la marca de la aplicación en el cuadro de diálogo de inicio de sesión. Esto es importante, ya que es el primer contacto que tendrá un usuario con la aplicación.
* Decidir si desea permitir que los usuarios únicamente puedan iniciar sesión si pertenecen a su organización. (aplicación de un solo inquilino) o si los usuarios podrán iniciar sesión con una cuenta profesional o educativa (aplicación de varios inquilinos). También puede permitir cuentas Microsoft personales o una cuenta de las redes sociales: LinkedIn, Google, etc.
* Solicitar permisos de ámbito. Por ejemplo, puede solicitar el ámbito "user.read", que concede permiso para leer el perfil del usuario conectado.
* Definir ámbitos que definan el acceso a la API web. Normalmente, cuando una aplicación desea acceder a la API, tiene que solicitar permisos para los ámbitos que se definen.
* Compartir un secreto con la Plataforma de identidad de Microsoft que demuestra la identidad de la aplicación.  Esto es aplicable cuando la aplicación es una aplicación cliente confidencial. Una aplicación cliente confidencial es una aplicación que puede almacenar las credenciales de forma segura. Se necesita un servidor back-end de confianza para almacenar las credenciales.

Una vez registrada la aplicación, se le proporcionará un identificador único que compartirá con la Plataforma de identidad de Microsoft cuando solicite tokens. Si la aplicación es una [aplicación cliente confidencial](developer-glossary.md#client-application), también compartirá el secreto o la clave pública, en función de si se utilizaron certificados o secretos.

La Plataforma de identidad de Microsoft representa a las aplicaciones que usan un modelo que cumple dos funciones principales:

* Identificar la aplicación mediante los protocolos de autenticación que admite.
* Proporcionar todos los identificadores, las direcciones URL, los secretos y la información relacionada que se necesitan para la autenticación.

Plataforma de identidad de Microsoft:

* Contiene todos los datos necesarios para admitir la autenticación en tiempo de ejecución.
* Contiene todos los datos para decidir qué recursos podría necesitar una aplicación para obtener acceso y en qué circunstancias debería cumplimentarse una solicitud.
* Proporciona la infraestructura necesaria para implementar el aprovisionamiento de la aplicación dentro del inquilino del desarrollador de la aplicación y en cualquier otro inquilino de Azure AD.
* Controla el consentimiento del usuario en el momento de solicitud del token y facilitar el aprovisionamiento dinámico de aplicaciones en varios inquilinos.

El **consentimiento** es el proceso de un propietario de recursos para conceder autorización a una aplicación cliente para acceder a recursos protegidos, bajo permisos específicos, en nombre del propietario del recurso. Plataforma de identidad de Microsoft:

* Permite a los usuarios y administradores conceder o denegar el consentimiento dinámicamente para que la aplicación acceda a recursos en su nombre.
* Permite a los administradores decidir qué pueden hacer las aplicaciones en última instancia, qué usuarios pueden utilizar aplicaciones específicas y cómo se accede a los recursos de directorio.

## <a name="multi-tenant-apps"></a>Aplicaciones multiinquilino

En la Plataforma de identidad de Microsoft, un [objeto de aplicación](developer-glossary.md#application-object) describe una aplicación. Durante la implementación, la Plataforma de identidad de Microsoft usa el objeto de aplicación como plano técnico para crear una [entidad de servicio](developer-glossary.md#service-principal-object), que representa una instancia específica de una aplicación de un directorio o inquilino. La entidad de servicio define qué es lo que puede hacer realmente la aplicación en un directorio de destino específico, quién puede usarla, a qué recursos tiene acceso, etc. La plataforma de identidad de Microsoft crea una entidad de servicio desde un objeto de aplicación a través del [consentimiento](developer-glossary.md#consent).

En el siguiente diagrama se muestra un flujo de aprovisionamiento de la plataforma de identidad de Microsoft basado en el consentimiento. Aparecen dos inquilinos: *A* y *B*.

* El *inquilino A* es el propietario de la aplicación.
* El *inquilino B* crea instancias de la aplicación mediante una entidad de servicio.

![Flujo de aprovisionamiento simplificado basado en el consentimiento](./media/authentication-scenarios/simplified-provisioning-flow-consent-driven.svg)

En este flujo de aprovisionamiento:

1. Un usuario del inquilino B intenta iniciar sesión con la aplicación y el punto de conexión de autorización solicita un token para la aplicación.
1. Se adquieren y verifican las credenciales de usuario para la autenticación.
1. El usuario debe dar su consentimiento para que la aplicación tenga acceso al inquilino B.
1. La Plataforma de identidad de Microsoft usa el objeto de aplicación del inquilino A como plano técnico para crear una entidad de servicio en el inquilino B.
1. El usuario recibe el token solicitado.

Puede repetir este proceso con otros inquilinos. El inquilino A conserva el plano técnico de la aplicación (objeto de aplicación). Los usuarios y administradores de todos los demás inquilinos en los que se concede consentimiento a la aplicación conservan el control sobre lo que la aplicación puede hacer con el objeto de la entidad de servicio correspondiente de cada inquilino. Para más información, consulte [Objetos de aplicación y de entidad de servicio en la plataforma de identidad de Microsoft](app-objects-and-service-principals.md).

## <a name="next-steps"></a>Pasos siguientes

Para ver otros temas en los que se traten aspectos básicos de la autenticación y la autorización:

* Consulte [Autenticación frente a autorización](authentication-vs-authorization.md) para obtener información sobre los aspectos básicos de la autenticación y la autorización en la Plataforma de identidad de Microsoft.
* Consulte [Tokens de seguridad](security-tokens.md) para obtener información sobre el acceso a los tokens, la actualización de los tokens y el uso de tokens de identificador en la autenticación y la autorización.
* Consulte [Flujo de inicio de sesión de aplicaciones](app-sign-in-flow.md) para obtener información sobre el flujo de inicio de sesión de las aplicaciones web, de escritorio y móviles en la Plataforma de identidad de Microsoft.

Para más información sobre el modelo de aplicación:

* Consulte [Cómo y por qué se agregan aplicaciones a Azure AD](active-directory-how-applications-are-added.md) para más información sobre los objetos de aplicación y las entidades de servicio en la Plataforma de identidad de Microsoft.
* Consulte [Inquilinos en Azure Active Directory](single-and-multi-tenant-apps.md) para más información sobre las aplicaciones de un solo inquilino y las aplicaciones de varios inquilinos.
* Consulte la [documentación de Azure Active Directory B2C](../../active-directory-b2c/index.yml) para más información sobre cómo Azure AD también proporciona Azure Active Directory B2C para que las organizaciones puedan iniciar sesión de los usuarios, normalmente clientes, con identidades sociales como una cuenta de Google.