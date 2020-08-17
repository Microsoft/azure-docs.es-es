---
title: Procedimientos recomendados para la plataforma de identidad de Microsoft | Azure
description: Conozca los procedimientos recomendados, las recomendaciones y los descuidos más comunes al realizar la integración con la plataforma de identidad de Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/08/2020
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, jesakowi
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 036c40395e5da5ebc09a87e420893d7dbd2ec668
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116808"
---
# <a name="microsoft-identity-platform-best-practices-and-recommendations"></a>Procedimientos recomendados y recomendaciones de la plataforma de identidad de Microsoft

En este artículo se resaltan los procedimientos recomendados, las recomendaciones y los descuidos más comunes en la integración con la plataforma de identidad de Microsoft.  Esta lista de comprobación le guía hacia una integración segura y de alta calidad. Revise esta lista periódicamente para asegurarse de que mantiene la calidad y la seguridad de la integración de la aplicación con la plataforma de identidad. La lista de comprobación no está pensada para revisar toda la aplicación. El contenido de la lista de comprobación está sujeto a cambios a medida que introducimos mejoras en la plataforma.

Si acaba de empezar, consulte la [documentación de la plataforma de identidad de Microsoft](index.yml) para información sobre aspectos básicos de la autenticación, escenarios de aplicación en la plataforma de identidad de Microsoft y mucho más.

Utilice la siguiente lista de comprobación para asegurarse de que su aplicación está efectivamente integrada con la [plataforma de identidad de Microsoft](./index.yml).

> [!TIP]
> El *Asistente de integración* en Azure Portal puede ayudarle a aplicar muchos de estos procedimientos recomendados y recomendaciones. Seleccione cualquiera de los [registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) en Azure Portal y, luego, seleccione el elemento de menú **Asistente de integración (versión preliminar)** para empezar a usar el asistente.

## <a name="basics"></a>Aspectos básicos

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Lea y comprenda las [directivas de la plataforma de Microsoft](https://go.microsoft.com/fwlink/?linkid=2090497&clcid=0x409). Asegúrese de que la aplicación se adhiere a los términos descritos, ya que están pensados para proteger a los usuarios y a la plataforma.

## <a name="ownership"></a>Propiedad

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Asegúrese de que la información asociada con la cuenta que ha usado para registrar y administrar las aplicaciones está actualizada.

## <a name="branding"></a>Personalización de marca

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Siga las [directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Proporcione un nombre significativo y un logotipo para la aplicación. Esta información aparece en la [petición de consentimiento de la aplicación](application-consent-experience.md). Asegúrese de que el nombre y el logotipo son representativos de su producto y empresa para que los usuarios puedan tomar decisiones informadas. Asegúrese de que no infringen ninguna marca comercial.

## <a name="privacy"></a>Privacidad

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Proporciona los vínculos a los términos del servicio y la declaración de privacidad de la aplicación.

## <a name="security"></a>Seguridad

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Administrar los URI de redireccionamiento: <ul><li>Mantenga la propiedad de todos los URI de redireccionamiento y mantenga actualizados los registros DNS de los mismos.</li><li>No utilice caracteres comodín (*) en los URI.</li><li>Para las aplicaciones web, asegúrese de que todos los URI son seguros y están cifrados (por ejemplo, mediante esquemas de https).</li><li>Para los clientes públicos, utilice los URI de redireccionamiento específicos de la plataforma si es aplicable (sobre todo para iOS y Android). En caso contrario, use el URI de redireccionamiento con una gran cantidad de aleatoriedad para evitar conflictos cuando llame de nuevo a la aplicación.</li><li>Si se utiliza la aplicación desde un agente web aislado, puede usar `https://login.microsoftonline.com/common/oauth2/nativeclient`.</li><li>Revise y recorte todos los identificadores URI de redirección que no se usen o que sean innecesarios de manera regular.</li></ul>

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Si la aplicación se registra en un directorio, minimice y supervise manualmente la lista de propietarios del registro de aplicaciones.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) No habilite la compatibilidad con el [flujo de concesión implícita de OAuth2](v2-oauth2-implicit-grant-flow.md) a menos que se requiera de forma explícita. Más información sobre el escenario válido [aquí](v2-oauth2-implicit-grant-flow.md#suitable-scenarios-for-the-oauth2-implicit-grant).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Prescinda del nombre de usuario/contraseña. No use el [flujo de credenciales de la contraseña de propietario del recurso (ROPC)](v2-oauth-ropc.md), que controla directamente las contraseñas de usuario. Este flujo requiere un alto grado de confianza y exposición del usuario, por lo que solo se debería usar cuando no se puedan usar otros flujos más seguros. Este flujo todavía es necesario en algunos escenarios (como DevOps), pero tenga en cuenta que su uso impondrá restricciones a la aplicación.  Para conocer enfoques más modernos, lea [Flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Proteja y administre sus credenciales de aplicación confidenciales para aplicaciones web, API web y aplicaciones de demonio. Utilice las [credenciales del certificado](active-directory-certificate-credentials.md), no las credenciales de la contraseña (secretos de cliente). Si debe utilizar una credencial de contraseña, no la configure manualmente. No almacene las credenciales en el código o en la configuración y nunca permita que las manipulen los seres humanos. Si es posible, utilice [identidades administradas para los recursos de Azure](../managed-identities-azure-resources/overview.md) o [Azure Key Vault](../../key-vault/general/basic-concepts.md) para almacenar y cambiar periódicamente las credenciales.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Asegúrese de que la aplicación solicite permisos con privilegios mínimos. Solo pida los permisos que la aplicación necesita absolutamente y únicamente cuando los necesite. Comprenda los diferentes [tipos de permisos](v2-permissions-and-consent.md#permission-types). Use únicamente los permisos de aplicación si es necesario; utilice permisos delegados siempre que sea posible. Para obtener una lista completa de los permisos de Microsoft Graph, consulte esta [referencia sobre los permisos](/graph/permissions-reference).

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Si protege una API mediante la plataforma de identidad de Microsoft, piense detenidamente en los permisos que debería exponer. Considere cuál es la granularidad correcta para la solución y qué permisos requieren el consentimiento del administrador. Compruebe los permisos esperados en los tokens entrantes antes de tomar cualquier decisión de autorización.

## <a name="implementation"></a>Implementación

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Utilice soluciones de autenticación modernas (OAuth 2.0, [OpenID Connect](v2-protocols-oidc.md)) para iniciar sesión de forma segura.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) No programa directamente con protocolos como OAuth 2.0 y Open ID. En su lugar, aproveche la [biblioteca de autenticación de Microsoft (MSAL)](msal-overview.md). Las bibliotecas MSAL encapsulan los protocolos de seguridad de forma segura en una biblioteca fácil de usar, y se obtiene compatibilidad integrada con escenarios de [acceso condicional](../conditional-access/overview.md) e [inicio de sesión único (SSO)](../manage-apps/what-is-single-sign-on.md) en todo el dispositivo, así como compatibilidad integrada con el almacenamiento en caché de tokens. Para más información, consulte la lista de [bibliotecas cliente](reference-v2-libraries.md#microsoft-supported-client-libraries) y [bibliotecas de middleware](reference-v2-libraries.md#microsoft-supported-server-middleware-libraries) admitidas por Microsoft y la lista de [bibliotecas cliente de terceros compatibles](reference-v2-libraries.md#compatible-client-libraries).<br/><br/>Si debe codificar manualmente los protocolos de autenticación, debe seguir una metodología como la de [SDL de Microsoft](https://www.microsoft.com/sdl/default.aspx). Preste mucha atención a las consideraciones de seguridad en las especificaciones de estándares para cada protocolo.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Migrar las aplicaciones existentes de la [biblioteca de autenticación de Azure Active Directory (ADAL)](../azuread-dev/active-directory-authentication-libraries.md) a la [biblioteca de autenticación de Microsoft](msal-overview.md). MSAL es la solución más reciente de plataforma de identidad de Microsoft y es preferible a ADAL. Está disponible en .NET, JavaScript, Android, iOS, macOS y también en versión preliminar pública para Python y Java. Conozca mejor cómo migrar [ADAL.NET](msal-net-migration.md) y [ADAL.js](msal-compare-msal-js-and-adal-js.md), y las aplicaciones de [ADAL.NET y de agente iOS](msal-net-migration-ios-broker.md).

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) En el caso de las aplicaciones móviles, configure cada plataforma con la experiencia de registro de aplicaciones. Para que la aplicación pueda aprovechar las ventajas de Microsoft Authenticator o del Portal de empresa de Microsoft para el inicio de sesión único, debe tener configurado un "URI de redirección del agente". De esta manera, Microsoft puede devolver el control a la aplicación después de la autenticación. Al configurar cada plataforma, la experiencia de registro de aplicaciones le guía por el proceso. Use el inicio rápido para descargar un ejemplo práctico. En iOS, use los agentes y la vista web siempre que sea posible.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) En aplicaciones web o API web, mantenga una caché de tokens por cada cuenta.  En el caso de las aplicaciones web, la caché de tokens debe estar protegida con clave mediante el identificador de cuenta.  En el caso de las API web, la cuenta debe estar protegida con clave mediante el hash del token usado para llamar a la API. MSAL.NET proporciona serialización de caché de tokens personalizada en las subplataformas .NET Framework y .NET Core. Por motivos de seguridad y rendimiento, se recomienda serializar una caché por cada usuario. Para más información, lea sobre la [serialización de la caché de tokens](msal-net-token-cache-serialization.md#token-cache-for-a-web-app-confidential-client-application).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Si los datos que necesita la aplicación están disponibles con [Microsoft Graph](https://developer.microsoft.com/graph), solicite permisos para estos datos mediante el punto de conexión de Microsoft Graph en lugar de la API individual.

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) No mire el valor del token de acceso ni intente analizarlo como cliente.  Pueden cambiar de valor o formatos o incluso cifrarse sin previo aviso: use siempre el valor de id_token si el cliente necesita saber algo sobre el usuario o llame a Microsoft Graph.  Solo las API web deben analizar los tokens de acceso (ya que son las que definen el formato y la configuración de las claves de cifrado).

## <a name="end-user-experience"></a>Experiencia del usuario final

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) [Comprenda la experiencia de consentimiento](application-consent-experience.md) y configure las partes de la petición de consentimiento de la aplicación para que los usuarios finales y los administradores tengan suficiente información para determinar si confían en la aplicación.

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Minimice el número de veces que un usuario necesita especificar las credenciales de inicio de sesión mientras utiliza la aplicación al intentar la autenticación silenciosa (adquisición de tokens silenciosos) antes de los flujos interactivos.

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) No use "prompt=consent" para cada inicio de sesión. Solo utilice prompt=consent si ha determinado que necesita pedir consentimiento para obtener permisos adicionales (por ejemplo, si ha cambiado los permisos necesarios de la aplicación).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Si procede, enriquezca su aplicación con datos de usuario. Usar [Microsoft Graph API](https://developer.microsoft.com/graph) es una forma fácil de hacerlo. La herramienta [Probador de Graph](https://developer.microsoft.com/graph/graph-explorer) que le ayudará a empezar.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Registre el conjunto completo de permisos que la aplicación requiere para que los administradores puedan otorgar el consentimiento fácilmente al inquilino. Utilice el [consentimiento incremental](../azuread-dev/azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent) en tiempo de ejecución para ayudar a los usuarios a comprender por qué la aplicación está solicitando permisos que pueden afectar o confundir a los usuarios cuando se solicitan al iniciar la aplicación por primera vez.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Implemente una [experiencia de cierre de sesión único limpia](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut). Es un requisito de privacidad y seguridad, y contribuye a una buena experiencia de usuario.

## <a name="testing"></a>Prueba

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Compruebe si hay [directivas de acceso condicional](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut) que puedan afectar a la capacidad de los usuarios para utilizar la aplicación.

![casilla](./media/active-directory-integration-checklist/checkbox-two.svg) Pruebe la aplicación con todas las cuentas posibles que piensa admitir (por ejemplo, cuentas profesionales o educativas, cuentas personales de Microsoft, cuentas secundarias y cuentas soberanas).

## <a name="additional-resources"></a>Recursos adicionales

Explore la información detallada acerca de la versión 2.0:

* [Introducción a la Plataforma de identidad de Microsoft (versión 2.0)](v2-overview.md)
* [Referencia a los protocolos de la plataforma de identidad de Microsoft](active-directory-v2-protocols.md)
* [Referencia de los tokens de acceso](access-tokens.md)
* [Referencia de los tokens de identificador](id-tokens.md)
* [Referencia de las bibliotecas de autenticación](reference-v2-libraries.md)
* [Permisos y consentimiento en la plataforma de identidad de Microsoft](v2-permissions-and-consent.md)
* [Microsoft Graph API](https://developer.microsoft.com/graph)