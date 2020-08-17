---
title: Creación de la página de aterrizaje de su oferta de SaaS gratuita o de evaluación en el marketplace comercial
description: Aprenda cómo crear una página de aterrizaje para su oferta de SaaS gratuita o de evaluación.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 08/06/2020
ms.openlocfilehash: 96e23c22568229ec5f5ba2365747e261b7e471ad
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87921391"
---
# <a name="build-the-landing-page-for-your-free-or-trial-saas-offer-in-the-commercial-marketplace"></a>Creación de la página de aterrizaje de su oferta de SaaS gratuita o de evaluación en el marketplace comercial

Este artículo le guía por el proceso de creación de una página de aterrizaje para una aplicación de SaaS gratuita o de evaluación que se venderá en el marketplace comercial de Microsoft.

## <a name="overview"></a>Información general

Puede pensar en la página de aterrizaje como el "recibidor" de la oferta de software como servicio (SaaS). Después de que el cliente elija obtener la aplicación, el marketplace comercial le dirigirá a la página de aterrizaje para activar y configurar su suscripción a la aplicación de SaaS. Al crear una oferta de software como servicio (SaaS), en el Centro de partners, puede elegir entre [vender a través de Microsoft](partner-center-portal/create-new-saas-offer.md) o no hacerlo. Si solo quiere que se muestre su oferta en el marketplace comercial de Microsoft y no vender a través de Microsoft, puede especificar la forma en la que los clientes potenciales pueden interactuar con la oferta. Al habilitar la opción de anuncio **Obténgalo ahora (de forma gratuita)** o **Evaluación gratuita**, debe especificar una URL de página de aterrizaje a la que el usuario pueda dirigirse para acceder a la suscripción o evaluación gratuita.

El propósito de la página de aterrizaje es simplemente recibir al usuario para que pueda activar la evaluación gratuita o la suscripción gratuita. Con Azure Active Directory (Azure AD) y Microsoft Graph, habilitará el inicio de sesión único (SSO) para el usuario y obtendrá detalles importantes sobre el usuario que puede usar para activar su evaluación gratuita o suscripción gratuita, incluido su nombre, dirección de correo electrónico y organización.

Dado que la información necesaria para activar la suscripción es limitada y la proporcionan Azure AD y Microsoft Graph, no debería haber ninguna necesidad de solicitar información que requiera más que el consentimiento básico. Si necesita detalles de usuario que requieran un consentimiento adicional para la aplicación, debe solicitar esta información una vez completada la activación de la suscripción. Esto permite la activación de la suscripción sin problemas para el usuario y reduce el riesgo de abandono.

La página de aterrizaje incluye normalmente la siguiente información y llamadas a la acción:

- Presente el nombre y los detalles de la evaluación gratuita o la suscripción gratuita. Por ejemplo, especifique los límites de uso o la duración de una evaluación.
- Presente los detalles de la cuenta del usuario, incluidos el nombre y los apellidos, la organización y el correo electrónico.
- Pida al usuario que confirme o sustituya los distintos detalles de la cuenta.
- Guíe al usuario en los pasos siguientes después de la activación. Por ejemplo, recibir un correo electrónico de bienvenida, administrar la suscripción, obtener soporte técnico o leer la documentación.

Las siguientes secciones de este artículo le guiarán en el proceso de creación de una página de aterrizaje:

1. [Cree un registro de aplicaciones de Azure AD](#create-an-azure-ad-app-registration) para la página de aterrizaje.
2. [Use un ejemplo de código como punto de partida](#use-a-code-sample-as-a-starting-point) para su aplicación.
3. [Lea información de las notificaciones codificadas en el token de identificación](#read-information-from-claims-encoded-in-the-id-token), recibida de Azure AD después del inicio de sesión, que se envió con la solicitud.
4. [Use la API de Microsoft Graph](#use-the-microsoft-graph-api) para recopilar información adicional, según sea necesario.

## <a name="create-an-azure-ad-app-registration"></a>Creación de un registro de aplicación de Azure AD

El marketplace comercial está totalmente integrado con Azure AD. Los usuarios llegan al marketplace autenticados con una [cuenta de Azure AD o una cuenta de Microsoft (MSA)](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis#terminology). Después de adquirir una evaluación gratuita o una subscripción gratuita a través de la oferta únicamente de anuncio, el usuario va del marketplace comercial a la dirección URL de la página de aterrizaje para activar y administrar su suscripción a la aplicación de SaaS. Debe permitir que el usuario inicie sesión en la aplicación con el SSO de Azure AD. (La dirección URL de la página de aterrizaje se especifica en la [página de configuración técnica](partner-center-portal/offer-creation-checklist.md#technical-configuration-page) de la oferta).

El primer paso para usar la identidad es asegurarse de que la página de aterrizaje esté registrada como aplicación de Azure AD. Registrar la aplicación le permite usar Azure AD para autenticar usuarios y solicitar acceso a recursos de usuarios. Se puede considerar la definición de la aplicación, lo que permite que el servicio sepa cómo emitir tokens para la aplicación en función de la configuración de la aplicación.

### <a name="register-a-new-application-using-the-azure-portal"></a>Registro de una aplicación nueva mediante Azure Portal

Para empezar, siga las instrucciones para [registrar una nueva aplicación](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app). Para permitir que los usuarios de otras empresas visiten la aplicación, debe elegir **Accounts in any organizational directory (any Azure AD directory—multitenant) and personal Microsoft accounts (like Skype or Xbox)** (Cuentas en cualquier directorio de la organización (en cualquier directorio de Azure AD: multiinquilino) y cuentas personales de Microsoft (como Skype o Xbox)) cuando se le pregunte quién puede usar la aplicación.

Si tiene previsto consultar la API de Microsoft Graph, [configure la nueva aplicación para acceder a las API web](https://docs.microsoft.com/azure/active-directory/develop/quickstart-configure-app-access-web-apis). Al seleccionar los permisos de API para esta aplicación, el valor predeterminado **User.Read** es suficiente para recopilar información básica sobre el usuario para que el proceso de incorporación sea automático y sin problemas. No solicite ningún permiso de la API con la etiqueta **necesita el consentimiento del administrador**, ya que esto impedirá que todos los usuarios que no sean administradores visiten la página de aterrizaje.

Si requiere permisos elevados como parte del proceso de incorporación o aprovisionamiento, considere la posibilidad de usar la funcionalidad de [consentimiento incremental](https://aka.ms/incremental-consent) de Azure AD para que todos los usuarios enviados desde el marketplace puedan interactuar inicialmente con la página de aterrizaje.

## <a name="use-a-code-sample-as-a-starting-point"></a>Uso de un ejemplo de código como punto de partida

Microsoft ha proporcionado varias aplicaciones de ejemplo que implementan un sitio web sencillo con el inicio de sesión de Azure AD habilitado. Una vez registrada la aplicación en Azure AD, la hoja **Inicio rápido** ofrece una lista de tipos de aplicaciones y pilas de desarrollo habituales (Figura 1). Elija el que coincida con su entorno y siga las instrucciones para descargarlo y configurarlo.

***Figura 1: Hoja de inicio rápido en Azure Portal***

:::image type="content" source="./media/azure-ad-saas/azure-ad-quickstart-blade.png" alt-text="Muestra la hoja de inicio rápido en Azure Portal.":::

Después de descargar el código y configurar el entorno de desarrollo, cambie los valores de configuración de la aplicación para que reflejen el identificador de la aplicación, el identificador del inquilino y el secreto de cliente que anotó en el procedimiento anterior. Tenga en cuenta que los pasos exactos variarán en función del ejemplo que use.

## <a name="read-information-from-claims-encoded-in-the-id-token"></a>Lectura de la información de las notificaciones codificadas en el token de identificador

Como parte del flujo de [OpenID Connect](https://docs.microsoft.com/azure/active-directory/develop/v2-protocols-oidc), Azure AD agrega un [token de identificador](https://docs.microsoft.com/azure/active-directory/develop/id-tokens) a la solicitud cuando se envía al usuario a la página de aterrizaje. Este token contiene varios elementos de información básica que podrían ser útiles en el proceso de activación, incluida la información que se muestra en esta tabla.

| Value | Descripción |
| ------------ | ------------- |
| aud | Audiencia objetivo para este token. En este caso, debe coincidir con el identificador de la aplicación y debe validadarse. |
| preferred_username | Nombre de usuario principal del usuario visitante. Puede ser una dirección de correo electrónico, un número de teléfono u otro identificador. |
| email | Dirección de correo electrónico del usuario. Tenga en cuenta que este campo puede estar vacío. |
| name | Valor en lenguaje natural que identifica al firmante del token. En este caso, será el nombre del usuario. |
| oid | Identificador en el sistema de identidades de Microsoft que identifica de forma única al usuario entre aplicaciones. Microsoft Graph devuelve este valor como la propiedad Id. de una cuenta de usuario determinada. |
| tid | Identificador que representa el inquilino de Azure AD de donde proviene el usuario. En el caso de una identidad de MSA, siempre será `9188040d-6c67-4c5b-b112-36a304b66dad`. Para obtener más información, consulte la nota de la siguiente sección: Uso de la API de Microsoft Graph. |
| sub | Identificador que identifica de forma única al usuario en esta aplicación específica. |
|||

## <a name="use-the-microsoft-graph-api"></a>Uso de Microsoft Graph API

El token de identificador contiene información básica para identificar al usuario, pero el proceso de activación puede requerir detalles adicionales, como la empresa del usuario, para completar el proceso de incorporación. Use la [API de Microsoft Graph](https://docs.microsoft.com/graph/use-the-api) para solicitar esta información para evitar que el usuario vuelva a escribir estos detalles. Los permisos **User.Read** estándar incluyen la siguiente información, de forma predeterminada:

| Value | Descripción |
| ------------ | ------------- |
| DisplayName | Nombre que se muestra en la libreta de direcciones para el usuario. |
| givenName | Nombre del usuario. |
| jobTitle | Cargo del usuario. |
| mail | Dirección SMTP del usuario. |
| mobilePhone | Número de teléfono móvil principal del usuario. |
| preferredLanguage | Código ISO 639-1 del idioma preferido del usuario. |
| surname | Apellidos del usuario. |
|||

Se pueden seleccionar propiedades adicionales, como el nombre de la empresa del usuario o la ubicación (país) del usuario, para su inclusión en la solicitud. Para obtener más información, consulte [Propiedades para el tipo de recurso de usuario](https://docs.microsoft.com/graph/api/resources/user?view=graph-rest-1.0#properties).

La mayoría de las aplicaciones que se registran con Azure AD conceden permisos delegados para leer la información del usuario del inquilino de Azure AD de su empresa. Cualquier solicitud a Microsoft Graph de esta información debe ir acompañada de un token de acceso como autenticación. Los pasos específicos para generar el token de acceso dependerán de la pila de tecnología que esté usando, pero el código de ejemplo contendrá un ejemplo. Para más información, consulte [Obtener acceso en nombre de un usuario](https://docs.microsoft.com/graph/auth-v2-user).

> [!NOTE]
> Las cuentas del inquilino de MSA (con el identificador de inquilino `9188040d-6c67-4c5b-b112-36a304b66dad`) no devolverán más información de la que ya se ha recopilado con el token de identificador. Por lo tanto, puede omitir esta llamada a Graph API para estas cuentas.

## <a name="next-steps"></a>Pasos siguientes
- [Creación de una oferta de SaaS en el marketplace comercial](./partner-center-portal/create-new-saas-offer.md)
