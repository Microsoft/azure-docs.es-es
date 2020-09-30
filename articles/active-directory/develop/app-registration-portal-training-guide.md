---
title: Nueva experiencia de los registros de aplicaciones de Azure Portal
titleSuffix: Microsoft identity platform
description: Introducción a la nueva experiencia de Registros de aplicaciones de Azure Portal
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 11/8/2019
ms.author: marsma
ms.reviewer: lenalepa, alamaral
ms.custom: aaddev
ms.openlocfilehash: 13242a41e1d10b0df031bf10fd646d9ec3cf47c3
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89437775"
---
# <a name="the-new-azure-portal-app-registration-experience"></a>Nueva experiencia de los registros de aplicaciones de Azure Portal

La nueva experiencia de [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de Azure Portal presenta numerosas mejoras. Si está familiarizado con el portal de registro de aplicaciones (apps.dev.microsoft.com) para registrar o administrar aplicaciones, lo que se menciona aquí como la "experiencia anterior", esta guía lo ayudará a comenzar a usar la nueva experiencia.

## <a name="whats-not-changing"></a>¿Qué no va a cambiar?

- Las aplicaciones y las configuraciones relacionadas se pueden encontrar tal y como están en la nueva experiencia. No es necesario volver a registrar las aplicaciones y los usuarios de las aplicaciones no tendrán que iniciar sesión de nuevo.

    > [!NOTE]
    > Debe iniciar sesión con la cuenta que usó para registrar las aplicaciones con el fin de encontrarlas en Azure Portal. Se recomienda que compruebe si el usuario que ha iniciado sesión en Azure Portal coincide con el que ha iniciado sesión en el Portal de registro de aplicaciones comparando la dirección de correo electrónico de su perfil.
    >
    > En algunos casos, especialmente cuando se inicia sesión con cuentas personales de Microsoft (por ejemplo, Outlook, Live, Xbox, etc.) con una dirección de correo electrónico de Azure AD, descubrimos que, cuando se visita Azure Portal desde la experiencia anterior, se inicia sesión en una cuenta diferente con el mismo correo electrónico en su inquilino de Azure AD. Si sigue pensando que faltan las aplicaciones, cierre la sesión e iníciela con la cuenta correcta.

- Todavía no se admiten las aplicaciones del SDK de Live creadas con cuentas de Microsoft personales en Azure Portal y seguirán estando en la experiencia anterior en un futuro próximo.

## <a name="key-changes"></a>Cambios clave

-   En la experiencia anterior, las aplicaciones se registraban de forma predeterminada como aplicaciones *convergentes* que admiten todas las cuentas de organización (multiinquilino), así como las cuentas personales de Microsoft. Esto no se pudo modificar a través de la experiencia anterior, lo que dificultaba la creación de aplicaciones que solo admitían las cuentas de organización (ya sean multiinquilino o de un solo inquilino).
    La nueva experiencia permite registrar aplicaciones que admiten todas esas opciones. [Más información sobre los tipos de aplicaciones](active-directory-v2-registration-portal.md)

-   En la nueva experiencia, si su cuenta de Microsoft personal también está en un inquilino de Azure AD, verá tres pestañas, una con todas las aplicaciones del inquilino, otra con las aplicaciones propias del inquilino y una última con las aplicaciones de su cuenta personal. Por lo tanto, si cree que faltan aplicaciones registradas con su cuenta de Microsoft personal, consulte la pestaña de **aplicaciones de su cuenta personal**.

-   En la nueva experiencia, puede cambiar fácilmente entre los inquilinos. Para ello, vaya a su perfil y elija cambiar el directorio.

## <a name="list-of-applications"></a>Lista de aplicaciones

-   En la lista de nuevas aplicaciones se muestran las registradas a través de la experiencia de Registros de aplicaciones heredada en Azure Portal (aplicaciones que inician sesión mediante cuentas de Azure AD), así como las registradas a través del [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) (aplicaciones que inician sesión mediante Azure AD y cuentas de Microsoft personales).

-   La lista de nuevas aplicaciones tiene dos columnas adicionales: La columna **Fecha de creación** y la columna **Certificados y secretos** que muestra el estado (Actual, Expira próximamente o Expirado) de las credenciales que se han registrado en la aplicación.

## <a name="new-app-registration"></a>Nuevo registro de aplicaciones

En la experiencia anterior, para registrar una aplicación solo era necesario proporcionar un nombre. Las aplicaciones que se crearon se registraron como aplicaciones *convergentes* que admiten todos los directorios organizativos (multiinquilino), así como las cuentas personales de Microsoft.  Esto no se pudo modificar a través de la experiencia anterior, lo que dificultaba la creación de aplicaciones que solo admitían cuentas de la organización (ya sean de un solo inquilino o multiinquilino). [Más información sobre los tipos de cuenta admitidos](v2-supported-account-types.md)

En la nueva experiencia, debe proporcionar un nombre para la aplicación y elegir los tipos de cuenta admitidos. También puede proporcionar un identificador URI de redirección.
Si proporciona un URI de redirección, tendrá que especificar si es de tipo web o público (nativo/móvil y escritorio). Para más información sobre cómo registrar una aplicación mediante la nueva experiencia de registros de aplicaciones, consulte esta [guía de inicio rápido](quickstart-register-app.md).

## <a name="app-management-page"></a>Página Administración de la aplicación

La experiencia antigua tenía una sola página de administración de aplicaciones con las siguientes secciones: Propiedades, Secretos de aplicación, Plataformas, Propietarios, Permisos de Microsoft Graph, Perfil y Opciones avanzadas.

La nueva experiencia de Azure Portal presenta estas características en páginas independientes. Aquí es donde puede encontrar la funcionalidad equivalente:

- Propiedades: el nombre y el identificador de la aplicación se encuentra en la página de información general.
- Secretos de aplicación se encuentra en la página Certificados y secretos.
- La configuración de plataformas está en la página Autenticación.
- Los permisos de Microsoft Graph están en la página Permisos de API junto con otros permisos.
- La funcionalidad Perfil está en la página Personalización de marca.
- Opciones avanzadas: la compatibilidad del SDK de Live está en la página Autenticación.

## <a name="application-secretscertificates--secrets"></a>Secretos de aplicación/Certificados y secretos

En la nueva experiencia, se ha cambiado el nombre **Secretos de aplicación** a **Certificados y secretos**. Además, las **claves públicas** se conocen como **certificados** y las **contraseñas** se conocen como **secretos de cliente**. Decidimos no incluir esta funcionalidad en la nueva experiencia por motivos de seguridad, por lo tanto, ya no se puede generar un nuevo par de claves.

## <a name="platformsauthentication-reply-urlsredirect-uris"></a>Plataformas/Autenticación: direcciones URL de respuesta e identificadores URI de redirección
En la experiencia anterior, una aplicación tenía una sección Plataformas para tipos web, nativos y de API web con el objetivo de configurar direcciones URL de redireccionamiento, direcciones URL de cierre de sesión y el flujo implícito.

En la nueva experiencia, las direcciones URL de respuesta se encuentran en la sección Autenticación de una aplicación. Además, se hace referencia a ellos como identificadores URI de redirección y el formato de los identificadores URI de redirección ha cambiado. Estos deben estar asociados con un tipo de aplicación (web o cliente público: móvil y escritorio). [Más información](quickstart-register-app.md#add-a-redirect-uri)

Las API web se configuran en la página Exponer una API.

> [!NOTE]
> Pruebe la nueva experiencia de configuración de autenticación, donde puede configurar su aplicación según la plataforma o dispositivo que tenga como destino. [Más información](quickstart-register-app.md#configure-platform-settings)

## <a name="microsoft-graph-permissionsapi-permissions"></a>Permisos de Microsoft Graph/Permisos de API

-   Al seleccionar una API en la experiencia anterior, solo podía elegir entre API de Microsoft Graph. En la nueva experiencia, puede elegir entre muchas API de Microsoft, como la de Microsoft Graph, las API de su organización y sus API. Esto se presenta en tres pestañas: API de Microsoft, API usadas en mi organización o Mis API. La barra de búsqueda de la pestaña API usadas en mi organización busca por entidades de servicio en el inquilino.

    > [!NOTE]
    > Si la aplicación no está asociada a un inquilino, no verá esta pestaña. Para más información sobre cómo solicitar permisos mediante la nueva experiencia, consulte esta [guía de inicio rápido](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/develop/quickstart-configure-app-access-web-apis.md).

-   La experiencia anterior no tenía un botón **Conceder permisos**. En la nueva experiencia, hay una sección Otorgar consentimiento con un botón **Conceder consentimiento de administrador** en la sección Permisos de API de una aplicación. Solo un administrador puede conceder consentimiento y este botón solo está habilitado para administradores. Cuando un administrador selecciona el botón **Conceder consentimiento de administrador**, el consentimiento del administrador se concede a todos los permisos solicitados.

## <a name="profile"></a>Perfil
En la experiencia anterior, la funcionalidad Perfil tenía los ajustes de configuración Logotipo, Dirección URL de la página principal, Dirección URL de términos del servicio y URL de la declaración de privacidad. En la nueva experiencia, se pueden encontrar en la página Personalización de marca.

## <a name="application-manifest"></a>Manifiesto de aplicación
En la nueva experiencia, la página Manifiesto permite editar y actualizar los atributos de la aplicación. Para más información, consulte [Manifiesto de aplicación](reference-app-manifest.md).

## <a name="new-ui"></a>Nueva interfaz de usuario
Hay una nueva interfaz de usuario para las propiedades que anteriormente solo se podían establecer mediante el editor de manifiestos o la API, o que no existían.

-   La opción Implicit grant flow (Flujo de concesión implícita) (oauth2AllowImplicitFlow) se encuentra en la página Autenticación. A diferencia de la experiencia antigua, puede habilitar tokens de acceso o tokens de identificador, o bien ambos.

-   Las opciones Ámbitos definidos con esta API (oauth2Permissions) y Aplicaciones cliente autorizadas (preAuthorizedApplications) se pueden configurar a través de la página Exponer una API. Para más información sobre cómo configurar una aplicación para que sea una API web y exponer permisos o ámbitos, consulte esta [guía de inicio rápido](quickstart-configure-app-expose-web-apis.md).

-   La opción Dominio del publicador (que se muestra a los usuarios en la [petición de consentimiento de la \'aplicación](application-consent-experience.md)) se encuentra en la página Personalización de marca. Para más información sobre cómo configurar un dominio de publicador, consulte este [artículo sobre procedimientos](howto-configure-publisher-domain.md).

## <a name="limitations"></a>Limitaciones

La nueva experiencia tiene las siguientes limitaciones:

-   La nueva experiencia todavía no admite la característica Registros de aplicaciones para los inquilinos de Azure AD B2C.

-   La nueva experiencia todavía no es compatible con las aplicaciones del SDK de Live creadas con cuentas personales de Microsoft.

-   En la interfaz de usuario no se admite el cambio del valor para las cuentas admitidas. Debe usar el manifiesto de aplicación, a menos que vaya a cambiar entre Azure AD de inquilino único y multiempresa.

   > [!NOTE]
   > Si es usuario de una cuenta personal de Microsoft en el inquilino de Azure AD y el administrador de inquilinos tiene acceso restringido a Azure Portal, puede obtener acceso denegado. Sin embargo, si entró a través del acceso directo escribiendo Registros de aplicaciones en la barra de búsqueda o anclándolo, podrá acceder a la nueva experiencia.

## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con la nueva experiencia de registro de aplicaciones, consulte el artículo [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](quickstart-register-app.md).
