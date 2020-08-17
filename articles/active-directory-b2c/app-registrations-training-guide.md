---
title: Nueva experiencia Registros de aplicaciones en Azure AD B2C
description: Una introducción a la nueva experiencia de registro de aplicaciones en Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/25/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: f9b493e862ed70bbeab9ac509c79003a535366d0
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116536"
---
# <a name="the-new-app-registrations-experience-for-azure-active-directory-b2c"></a>La nueva experiencia Registros de aplicaciones para Azure Active Directory B2C

La nueva experiencia **[Registros de aplicaciones](https://aka.ms/b2cappregistrations)** para Azure Active Directory B2C (Azure AD B2C) ya está disponible con carácter general. Si está más familiarizado con la experiencia de **Aplicaciones** para el registro de aplicaciones para Azure AD B2C, lo que aquí se denomina "experiencia heredada", esta guía le permitirá comenzar a usar la nueva experiencia.

## <a name="overview"></a>Información general
Anteriormente, tenía que administrar las aplicaciones de Azure AD B2C orientadas al consumidor por separado del resto de las aplicaciones con la experiencia heredada. Eso implicaba distintas experiencias de creación de aplicaciones en distintos lugares de Azure.

La nueva experiencia muestra todos los registros de aplicaciones de Azure AD B2C y Azure AD en un solo lugar y proporciona una manera coherente de administrarlos. Desde la creación de una aplicación orientada al cliente hasta la administración de una aplicación con permisos de Microsoft Graph para la administración de recursos, solo tiene que aprender una forma de hacer las cosas.

Puede acceder a la experiencia nueva si navega a **Registros de aplicaciones** en un inquilino de Azure AD B2C desde los servicios **Azure AD B2C** o **Azure Active Directory** en Azure Portal.

La experiencia Registros de aplicaciones de Azure AD B2C se basa en la [experiencia de registro de aplicaciones](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) general para cualquier inquilino de Azure AD, pero se adapta a los inquilinos de Azure AD B2C.

## <a name="whats-not-changing"></a>¿Qué no va a cambiar?
- Las aplicaciones y las configuraciones relacionadas se pueden encontrar tal y como están en la nueva experiencia. No es necesario volver a registrar las aplicaciones y los usuarios de las aplicaciones no tendrán que iniciar sesión de nuevo. 

> [!NOTE]
> Para ver todas las aplicaciones creadas anteriormente, vaya a la hoja **Registros de aplicaciones** y seleccione la pestaña **Todas las aplicaciones**. Se mostrarán las aplicaciones creadas en la experiencia heredada, la experiencia nueva y las creadas en el servicio Azure AD.

## <a name="key-new-features"></a>Nuevas características clave

-   En una **lista de aplicaciones unificadas** se muestran todas las aplicaciones que se autentican con Azure AD B2C y Azure AD en un solo lugar para su comodidad. Además, puede aprovechar las características que ya están disponibles para las aplicaciones de Azure AD, incluidas la fecha **Creada el**, el estado **Certificados y secretos**, la barra de búsqueda y muchas más.

-   El **registro de aplicaciones combinado** permite registrar rápidamente una aplicación, con independencia de que esté orientada al cliente o sirva para acceder a Microsoft Graph.

- El panel **Puntos de conexión** permite identificar rápidamente los puntos de conexión pertinentes para el escenario, como la configuración de OpenID Connect, los metadatos de SAML, Microsoft Graph API y los [puntos de conexión de flujo de usuario de OAuth 2.0](tokens-overview.md#endpoints). 

- En **Permisos de API** y **Exponer una API** se proporciona una administración más extensa de ámbitos, permisos y consentimientos. Ahora también puede asignar permisos de MS Graph y Azure AD Graph a una aplicación.

-   Ahora **Propietarios** y **Manifiesto** están disponibles para las aplicaciones que se autentican con Azure AD B2C. Puede agregar propietarios para los registros y editar directamente las propiedades de la aplicación [mediante el editor de manifiestos](../active-directory/develop/reference-app-manifest.md).


## <a name="new-supported-account-types"></a>Nuevos tipos de cuenta admitidos

En la experiencia nueva, puede seleccionar un tipo de cuenta admitido entre las opciones siguientes:
- Solo las cuentas de este directorio organizativo.
- Cuentas en cualquier directorio organizativo (cualquier directorio de Azure AD: multiinquilino).
- Cuentas de cualquier directorio de la organización o cualquier proveedor de identidades. Para autenticar a los usuarios con Azure AD B2C.

Para comprender los distintos tipos de cuenta, seleccione **Ayúdame a elegir** en la experiencia de creación. 

En la experiencia heredada, las aplicaciones siempre se crean como aplicaciones orientadas al cliente. Para esas aplicaciones, el tipo de cuenta se establece en **Cuentas de cualquier directorio de la organización o cualquier proveedor de identidades. Para autenticar a los usuarios con Azure AD B2C**.
> [!NOTE]
> Esta opción es necesaria para poder ejecutar flujos de usuario de Azure AD B2C para autenticar a los usuarios de esta aplicación. Aprenda [cómo registrar una aplicación para usarla con flujos de usuario.](tutorial-register-applications.md)

También puede utilizar esta opción para usar Azure AD B2C como proveedor de servicios SAML. [Más información](identity-provider-adfs2016-custom.md).

## <a name="applications-for-devops-scenarios"></a>Aplicaciones para escenarios de DevOps
Puede usar los otros tipos de cuenta para crear una aplicación a fin de administrar los escenarios de DevOps, como el uso de Microsoft Graph para cargar directivas de Identity Experience Framework o aprovisionar usuarios. Aprenda [cómo registrar una aplicación de Microsoft Graph para administrar recursos de Azure AD B2C](microsoft-graph-get-started.md).

Es posible que no vea todos los permisos de Microsoft Graph, ya que muchos de estos permisos no se aplican a los usuarios del consumidor de Azure B2C. [Obtenga más información sobre la administración de usuarios mediante Microsoft Graph](manage-user-accounts-graph-api.md).  

## <a name="admin-consent-and-offline_accessopenid-scopes"></a>Consentimiento del administrador y los ámbitos offline_access y openid  
<!-- Azure AD B2C doesn't support user consent. That is, when a user signs into an application, the user doesn't see a screen requesting consent for the application permissions. All permissions have to be granted through admin consent.  -->

El ámbito **openid** es necesario para que Azure AD B2C permita el inicio de sesión de los usuarios en una aplicación. El ámbito **offline_access** es necesario para emitir tokens de actualización para un usuario. Estos ámbitos se han agregado previamente y se les ha asignado el consentimiento de administrador de forma predeterminada. Ahora, puede agregar fácilmente permisos para estos ámbitos durante el proceso de creación si se asegura de que la opción **Grant admin consent to openid and offline_access permissions** (Conceder consentimiento de administrador a los permisos openid y offline_access) está seleccionada. De lo contrario, los permisos de Microsoft Graph se pueden agregar con el consentimiento de administrador en la opción **Permisos de API** de una aplicación existente.

Obtenga más información sobre [los permisos y el consentimiento](../active-directory/develop/v2-permissions-and-consent.md).

## <a name="platformsauthentication-reply-urlsredirect-uris"></a>Plataformas/Autenticación: direcciones URL de respuesta e identificadores URI de redirección
En la experiencia heredada, los distintos tipos de plataforma se administraban en **Propiedades** como direcciones URL de respuesta para aplicaciones web o API, y URI de redirección para clientes nativos. Los "clientes nativos" también se conocen como "clientes públicos" e incluyen aplicaciones para iOS, macOS, Android y otros tipos de aplicaciones de escritorio y para dispositivos móviles. 

En la experiencia nueva, las direcciones URL de respuesta y los URI de redirección se conocen como URI de redirección y se pueden encontrar en la sección **Autenticación** de una aplicación. Los registros de aplicaciones no se limitan a una aplicación web o una aplicación nativa. Puede usar el mismo registro de aplicación para todos estos tipos de plataforma si registra los URI de redirección correspondientes. 

Es obligatorio que los URI de redirección estén asociados con un tipo de aplicación, ya sea web o pública (para dispositivos móviles y de escritorio). [Más información sobre los URI de redireccionamiento](../active-directory/develop/quickstart-configure-app-access-web-apis.md#add-redirect-uris-to-your-application)

<!-- Whether an application should be treated as a public client is inferred at run-time from the Redirect URI platform type, if possible. The **Treat application as a public client** setting should be set to **Yes** for flows that might not use a redirect URI, such as ROPC flows. -->

Las plataformas **iOS/macOS** y **Android** son un tipo de cliente público. Proporcionan una manera fácil de configurar aplicaciones iOS/macOS o Android con los URI de redirección correspondientes para su uso con MSAL. Obtenga más información sobre las [opciones de configuración de la aplicación](../active-directory/develop/msal-client-applications.md).


## <a name="application-certificates--secrets"></a>Certificados y secretos de aplicación

En la experiencia nueva, en lugar de **Claves**, se usa la hoja **Certificados y secretos** para administrar los certificados y los secretos. Los certificados y secretos permiten a las aplicaciones identificarse en el servicio de autenticación al recibir tokens en una ubicación de dirección web (mediante un esquema HTTPS). Al autenticarse en Azure AD, se recomienda usar un certificado en lugar de un secreto de cliente para los escenarios de credenciales de cliente. No se pueden usar certificados para autenticarse en Azure AD B2C.


## <a name="features-not-applicable-in-azure-ad-b2c-tenants"></a>Características no aplicables en inquilinos de Azure AD B2C
Las funcionalidades de registros de aplicaciones de Azure AD siguientes no son aplicables o no están disponibles en los inquilinos de Azure AD B2C:
- **Roles y administradores**: requiere una licencia Azure AD Premium P1 o P2 que no está disponible actualmente para Azure AD B2C.
- **Personalización de marca**: la personalización de la interfaz de usuario o la experiencia del usuario se configura en la experiencia **Personalización de marca de empresa** o como parte de un flujo de usuario. Aprenda a [personalizar la interfaz de usuario en Azure Active Directory B2C](customize-ui-overview.md).
- **Comprobación del dominio del publicador**: la aplicación está registrada en *.onmicrosoft.com*, que no es un dominio comprobado. Además, el dominio del publicador se usa principalmente para conceder el consentimiento del usuario, que no se aplica a las aplicaciones de Azure AD B2C para la autenticación de usuarios. [Más información sobre el dominio de publicador](https://docs.microsoft.com/azure/active-directory/develop/howto-configure-publisher-domain).
- **Configuración de token**: el token se configura como parte de un flujo de usuario en lugar de una aplicación.
- Actualmente, la experiencia **Inicios rápidos** no está disponible para inquilinos de Azure AD B2C.
- Actualmente, la hoja **Asistente de integración** no está disponible para inquilinos de Azure AD B2C.


## <a name="limitations"></a>Limitaciones
La nueva experiencia tiene las siguientes limitaciones:
- En la actualidad, Azure AD B2C no distingue entre poder emitir tokens de acceso o de identificador para flujos implícitos; los dos tipos de tokens están disponibles para el flujo de concesión implícita si la opción **Tokens de id.** está seleccionada en la hoja **Autenticación**.
<!-- - Azure AD B2C doesn't currently support the single-page application "SPA" app type.  -->
- En la interfaz de usuario no se admite el cambio del valor para las cuentas admitidas. Tendrá que usar el manifiesto de aplicación, a menos que cambie entre Azure AD de inquilino único y multiinquilino.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con la nueva experiencia de registro:
* Aprenda a [registrar una aplicación web](tutorial-register-applications.md).
* Aprenda a [registrar una API web](add-web-api-application.md).
* Aprenda a [registrar una aplicación cliente nativa](add-native-application.md).
* Aprenda a [registrar una aplicación de Microsoft Graph para administrar recursos de Azure AD B2C](microsoft-graph-get-started.md).
* Aprenda a [usar Azure AD B2C como proveedor de servicios SAML.](identity-provider-adfs2016-custom.md)
* Más información sobre los [tipos de aplicación](application-types.md).
