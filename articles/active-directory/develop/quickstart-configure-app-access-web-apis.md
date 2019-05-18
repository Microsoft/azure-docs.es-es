---
title: 'Configuración de una aplicación para acceder a las API web: plataforma de identidad de Microsoft'
description: Obtenga información acerca de cómo configurar una aplicación registrada con la plataforma de identidad de Microsoft para incluir los identificadores URI de redireccionamiento, las credenciales o los permisos para acceder a las API web.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7b2f5331b22f6292b00c367c6abb8cd4f439a70c
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413975"
---
# <a name="quickstart-configure-a-client-application-to-access-web-apis"></a>Inicio rápido: Configuración de una aplicación cliente para tener acceso a las API web

Para que una aplicación cliente web o confidencial pueda participar en un flujo de concesión de autorización que requiera autenticación (y obtener un token de acceso), debe establecer credenciales seguras. El método de autenticación predeterminado compatible con Azure Portal es un identificador de cliente y una clave secreta.

Además, para que un cliente pueda tener acceso a una API web expuesta por una aplicación de recursos (como Microsoft Graph API), el marco de consentimiento garantiza que el cliente obtenga la concesión de los permisos necesarios en función de los solicitados. De forma predeterminada, todas las aplicaciones pueden elegir permisos de Microsoft Graph API. El [permiso "Iniciar sesión y leer el perfil de usuario" de Graph API](https://developer.microsoft.com/graph/docs/concepts/permissions_reference#user-permissions) se selecciona de forma predeterminada. Puede seleccionar entre [dos tipos de permisos](developer-glossary.md#permissions) para cada API web que desee:

* **Permisos de la aplicación**: la aplicación cliente necesita acceso directo a la API web como sí misma (sin contexto de usuario). Este tipo de permiso requiere el consentimiento del administrador y no está disponible para aplicaciones cliente públicas (para dispositivos de escritorio y móviles).
* **Permisos delegados**: la aplicación cliente necesita acceso a la API web como el usuario que inició sesión, pero con acceso limitado por el permiso seleccionado. Este tipo de permiso lo puede conceder un usuario a menos que el permiso requiera el consentimiento del administrador.

  > [!NOTE]
  > La adición de un permiso delegado a una aplicación no concede consentimiento automáticamente a los usuarios del inquilino. Los usuarios tienen todavía que conceder manualmente el consentimiento para los permisos delegados agregados en tiempo de ejecución, a menos que lo haga el administrador en nombre de todos los usuarios.

En esta guía de inicio rápido, le mostraremos cómo configurar la aplicación mediante:

* [Adición de identificadores URI de redireccionamiento a la aplicación](#add-redirect-uris-to-your-application)
* [Adición de credenciales a la aplicación web](#add-credentials-to-your-web-application)
* [Adición de permisos para acceder a las API web](#add-permissions-to-access-web-apis)

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, asegúrese de que completa estos requisitos previos:

* Aprenda sobre los [permisos y consentimiento](v2-permissions-and-consent.md) admitidos, que son importantes comprender al crear aplicaciones que necesitan ser utilizadas por otros usuarios o aplicaciones.
* Tiene un inquilino que tenga las aplicaciones registradas en él.
  * Si no tiene aplicaciones registradas, [aprenda a registrar aplicaciones con la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>Inicio de sesión en Azure Portal y selección de la aplicación

Antes de configurar la aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones**.
1. Busque y seleccione la aplicación que desea configurar. Cuando haya seleccionado la aplicación, verá la página **Introducción** de la aplicación o la de registro principal.
1. Siga los pasos para configurar la aplicación para acceder a las API web: 
    * [Adición de identificadores URI de redireccionamiento a la aplicación](#add-redirect-uris-to-your-application)
    * [Adición de credenciales a la aplicación web](#add-credentials-to-your-web-application)
    * [Adición de permisos para acceder a las API web](#add-permissions-to-access-web-apis)

## <a name="add-redirect-uris-to-your-application"></a>Adición de identificadores URI de redireccionamiento a la aplicación

[![Adición de URI de redireccionamiento personalizados para aplicaciones cliente web y públicas](./media/quickstart-update-azure-ad-app-preview/authentication-redirect-uris-expanded.png)](./media/quickstart-update-azure-ad-app-preview/authentication-redirect-uris-expanded.png#lightbox)

Para agregar un identificador URI de redireccionamiento a la aplicación:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Autenticación**.

1. Para agregar un URI de redireccionamiento personalizado para aplicaciones cliente web y públicas, siga estos pasos:

   1. Busque la sección **URI de redireccionamiento**.
   1. Seleccione el tipo de aplicación que se va a crear, **Web** o **Cliente público (móvil y escritorio)**.
   1. Escriba el identificador URI de redireccionamiento para la aplicación.
      * Para aplicaciones web, proporcione la dirección URL base de la aplicación. Por ejemplo, `http://localhost:31544` podría ser la dirección URL de una aplicación web que se ejecuta en la máquina local. Los usuarios utilizan esta dirección URL para iniciar sesión en una aplicación cliente web.
      * Para aplicaciones públicas, proporcione el identificador URI que utiliza Azure AD para devolver las respuestas de los tokens. Escriba un valor específico para la aplicación, por ejemplo, https://MyFirstApp.

1. Para elegir entre los identificadores URI de redireccionamiento sugeridos para clientes públicos (para dispositivos móviles o de escritorio), siga estos pasos:

    1. Localice la sección **URI de redireccionamiento sugeridos para clientes públicos (para dispositivos móviles o de escritorio)**.
    1. Seleccione los identificadores URI de redireccionamiento adecuados para su aplicación mediante las casillas.

## <a name="add-credentials-to-your-web-application"></a>Adición de credenciales a la aplicación web

[![Adición de certificados y secretos de cliente](./media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets-expanded.png)](./media/quickstart-update-azure-ad-app-preview/credentials-certificates-secrets-expanded.png#lightbox)

Para agregar una credencial a la aplicación web:

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Certificados y secretos**.

1. Para agregar un certificado, siga estos pasos:

    1. Seleccione **Cargar certificado**.
    1. Seleccione el archivo que desea cargar. Debe ser uno de los siguientes tipos de archivo: .cer, .pem, .crt.
    1. Seleccione **Agregar**.

1. Para agregar un secreto de cliente, siga estos pasos:

    1. Seleccione **Nuevo secreto de cliente**.
    1. Agregue una descripción para el secreto de cliente.
    1. Seleccione una duración.
    1. Seleccione **Agregar**.

> [!NOTE]
> Después de guardar los cambios de configuración, la columna en el extremo derecho contendrá el valor del secreto de cliente. **Asegúrese de copiar el valor** para su uso en el código de la aplicación cliente, ya que no es accesible una vez se sale de esta página.

## <a name="add-permissions-to-access-web-apis"></a>Adición de permisos para acceder a las API web

[![Adición de permisos de API](./media/quickstart-update-azure-ad-app-preview/api-permissions-expanded.png)](./media/quickstart-update-azure-ad-app-preview/api-permissions-expanded.png#lightbox)

Para agregar permisos para tener acceso a las API de recursos desde el cliente:

1. Desde la página **Introducción** de la aplicación, seleccione **Permisos de API**.
1. Seleccione el botón **Agregar un permiso**.
1. De forma predeterminada, la vista le permite seleccionar en las **API de Microsoft**. Seleccione la sección de las API que le interesa:
    * **API de Microsoft**: le permite seleccionar los permisos para las API de Microsoft, como Microsoft Graph.
    * **API usadas en mi organización**: le permite seleccionar los permisos para las API que se han expuesto por la organización o para las API con las que se ha integrado la organización.
    * **Mis API**: le permite seleccionar los permisos para las API que ha expuesto.
1. Una vez que haya seleccionado las API, verá la página **Solicitud de permisos de API**. Si la API expone permisos delegados y de aplicación, seleccione el tipo de permiso que necesita la aplicación.
1. Cuando termine, seleccione **Agregar permisos**. Volverá a la página **Permisos de API**, donde los permisos se han guardado y agregado a la tabla.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de estas otras guías de inicio rápido de administración de aplicaciones relacionadas para aplicaciones:

* [Registro de una aplicación con la plataforma de identidad de Microsoft](quickstart-register-app.md)
* [Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md)
* [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)
* [Eliminación de una aplicación registrada con la plataforma de identidad de Microsoft](quickstart-remove-app.md)

Para obtener más información acerca de los dos objetos de Azure AD que representan una aplicación registrada y la relación entre ellos, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).

Para obtener más información acerca de las directrices de personalización de marca que hay que usar al desarrollar aplicaciones con Azure Active Directory, consulte [Directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).
